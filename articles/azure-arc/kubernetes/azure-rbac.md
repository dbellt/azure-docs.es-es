---
title: Azure RBAC para clústeres Kubernetes habilitados para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
description: Uso de RBAC de Azure para comprobaciones de autorización en los clústeres de Kubernetes habilitados para Azure Arc
ms.openlocfilehash: 63621391da3dec966e9d0375a8671b7413a0b222
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804732"
---
# <a name="integrate-azure-active-directory-with-azure-arc-enabled-kubernetes-clusters"></a>Integración de Azure Active Directory con clústeres de Kubernetes habilitado para Azure Arc

Los tipos de objeto [ClusterRoleBinding y RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) de Kubernetes ayudan a definir la autorización en Kubernetes de forma nativa. Con esta característica, puede usar Azure Active Directory y las asignaciones de roles de Azure para controlar las comprobaciones de autorización en el clúster. Esto implica que ahora puede usar las asignaciones de roles de Azure para controlar de forma pormenorizada quién puede leer, escribir y eliminar los objetos de Kubernetes, como Deployment, Pod y Service.

Puede encontrar información general y algunos conceptos sobre esta característica en el artículo [Azure RBAC: Kubernetes habilitado para Azure Arc](conceptual-azure-rbac.md).

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Instale o actualice la CLI de Azure](/cli/azure/install-azure-cli) a la versión 2.16.0 o posteriores.

- Instale la extensión `connectedk8s` de la CLI de Azure, versión 1.1.0 o posteriores.

    ```azurecli
    az extension add --name connectedk8s
    ```
    
    Si la extensión `connectedk8s` ya está instalada, puede actualizarla a su versión más reciente con el siguiente comando: 

    ```azurecli
    az extension update --name connectedk8s
    ```

- Un clúster conectado de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, use nuestra [guía de inicio rápido](quickstart-connect-cluster.md).
    - [Actualice los agentes](agent-upgrade.md#manually-upgrade-agents) a la versión 1.1.0 o posteriores.

> [!NOTE]
> Esta característica no se puede configurar para las ofertas de Kubernetes administrado de proveedores en la nube, como Elastic Kubernetes Service o Google Kubernetes Engine, en las que el usuario no tiene acceso al clúster `apiserver`. En el caso de los clústeres de Azure Kubernetes Service (AKS), esta [característica está disponible de forma nativa](../../aks/manage-azure-rbac.md) y no requiere que el clúster de AKS esté conectado a Azure Arc.

## <a name="set-up-azure-ad-applications"></a>Configuración de aplicaciones de Azure AD

### <a name="create-server-application"></a>Creación de la aplicación de servidor

1. Cree una nueva aplicación de Azure AD y obtenga su valor de `appId`, que se usa en pasos posteriores como `serverApplicationId`:

    ```azurecli
    az ad app create --display-name "<clusterName>Server" --identifier-uris "https://<clusterName>Server" --query appId -o tsv
    ```

1. Actualice las notificaciones de pertenencia a grupos de aplicaciones:

    ```azurecli
    az ad app update --id <serverApplicationId> --set groupMembershipClaims=All
    ```

1. Cree una entidad de servicio y obtenga el valor del campo `password`, que es necesario más adelante como `serverApplicationSecret` al habilitar esta característica en el clúster:

    ```azurecli
    az ad sp create --id <serverApplicationId>
    az ad sp credential reset --name <serverApplicationId> --credential-description "ArcSecret" --query password -o tsv
    ```

1. Conceda permisos a la API de la aplicación:

    ```azurecli
    az ad app permission add --id <serverApplicationId> --api 00000003-0000-0000-c000-000000000000 --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope
    az ad app permission grant --id <serverApplicationId> --api 00000003-0000-0000-c000-000000000000
    ```

    > [!NOTE]
    > * Un administrador de inquilinos de Azure debe ejecutar este paso.
    > * Para usar esta característica en producción, se recomienda crear una aplicación de servidor diferente para cada clúster.

### <a name="create-client-application"></a>Creación de la aplicación cliente

1. Cree una nueva aplicación de Azure AD y obtenga su valor de "appId", que se usa en pasos posteriores como `clientApplicationId`:

    ```azurecli
    az ad app create --display-name "<clusterName>Client" --native-app --reply-urls "https://<clusterName>Client" --query appId -o tsv
    ```

2. Cree una entidad de servicio para esta aplicación cliente:

    ```azurecli
    az ad sp create --id <clientApplicationId>
    ```

3. Obtenga el valor de `oAuthPermissionId` para la aplicación de servidor:

    ```azurecli
    az ad app show --id <serverApplicationId> --query "oauth2Permissions[0].id" -o tsv
    ```

4. Conceda los permisos necesarios para la aplicación cliente:

    ```azurecli
    az ad app permission add --id <clientApplicationId> --api <serverApplicationId> --api-permissions <oAuthPermissionId>=Scope
    az ad app permission grant --id <clientApplicationId> --api <serverApplicationId>
    ```

## <a name="create-a-role-assignment-for-the-server-application"></a>Creación de una asignación de roles para la aplicación de servidor

La aplicación de servidor necesita los permisos `Microsoft.Authorization/*/read` para comprobar si el usuario que realiza la solicitud está autorizado en los objetos de Kubernetes que forman parte de la solicitud.

1. Cree un archivo llamado accessCheck.json con el siguiente contenido:

    ```json
    {
      "Name": "Read authorization",
      "IsCustom": true,
      "Description": "Read authorization",
      "Actions": ["Microsoft.Authorization/*/read"],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/<subscription-id>"
      ]
    }
    ```

    Reemplace `<subscription-id>` por el identificador de suscripción real.

2. Ejecute el siguiente comando para crear un nuevo rol personalizado:

    ```azurecli
    az role definition create --role-definition ./accessCheck.json
    ```

3. Del resultado del comando anterior, almacene el valor del campo `id`, que se usa en pasos posteriores como `roleId`.

4. Cree una asignación de roles en la aplicación de servidor como si fuera el usuario asignado mediante el rol creado anteriormente:

    ```azurecli
    az role assignment create --role <roleId> --assignee <serverApplicationId> --scope /subscriptions/<subscription-id>
    ```

## <a name="enable-azure-rbac-on-cluster"></a>Habilitación de Azure RBAC en el clúster

1. Ejecute el siguiente comando para habilitar Azure RBAC en el clúster de Kubernetes habilitado para Arc:

    ```console
    az connectedk8s enable-features -n <clusterName> -g <resourceGroupName> --features azure-rbac --app-id <serverApplicationId> --app-secret <serverApplicationSecret>
    ```
    
    > [!NOTE]
    > 1. Antes de ejecutar el comando anterior, asegúrese de que el archivo `kubeconfig` de la máquina apunta al clúster en el que se va a habilitar la característica Azure RBAC.
    > 2. Use `--skip-azure-rbac-list` con el comando anterior para obtener una lista separada por comas de los nombres de usuario/correos electrónicos/identificadores de OAuth en proceso de comprobación de autorización mediante los objetos ClusterRoleBinding y RoleBinding nativos de Kubernetes, en lugar de Azure RBAC.

### <a name="for-a-generic-cluster-where-there-is-no-reconciler-running-on-apiserver-specification"></a>Para un clúster genérico en el que no hay ningún reconciliador en ejecución en la especificación de apiserver:

1. Conéctese mediante SSH a cada nodo principal del clúster y ejecute los pasos siguientes:

    1. Abra el manifiesto `apiserver` en modo de edición:
        
        ```console
        sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
        ```

    1. Agregue la siguiente especificación después de `volumes`:
        
        ```yml
        - name: azure-rbac
          secret:
            secretName: azure-arc-guard-manifests
        ```

    1. Agregue la siguiente especificación después de `volumeMounts`:

        ```yml
        - mountPath: /etc/guard
          name: azure-rbac
          readOnly: true
        ```

    1. Agregue los siguientes argumentos de `apiserver`:

        ```yml
        - --authentication-token-webhook-config-file=/etc/guard/guard-authn-webhook.yaml
        - --authentication-token-webhook-cache-ttl=5m0s
        - --authorization-webhook-cache-authorized-ttl=5m0s
        - --authorization-webhook-config-file=/etc/guard/guard-authz-webhook.yaml
        - --authorization-webhook-version=v1
        - --authorization-mode=Node,Webhook,RBAC
        ```
    
        Si el clúster de Kubernetes es de la versión 1.19.0 o posteriores, también se deben configurar los siguientes valores de `apiserver argument`:

        ```yml
        - --authentication-token-webhook-version=v1
        ```

    1. Guarde y salga del editor para actualizar el pod `apiserver`.


### <a name="for-a-cluster-created-using-cluster-api"></a>Para un clúster creado mediante la API de clústeres

1. Copie el secreto de Guard que contiene los archivos de configuración de webhook de autenticación y autorización `from the workload cluster` en la máquina:

    ```console
    kubectl get secret azure-arc-guard-manifests -n kube-system -o yaml > azure-arc-guard-manifests.yaml
    ```

1. Cambie el valor del campo `namespace` del archivo `azure-arc-guard-manifests.yaml` por el espacio de nombres en el clúster de administración en el que se aplicarán los recursos personalizados para la creación de clústeres de carga de trabajo.

1. Aplique el siguiente manifiesto:

    ```console
    kubectl apply -f azure-arc-guard-manifests.yaml
    ```

1. Ejecute `kubectl edit kcp <clustername>-control-plane` para editar el objeto `KubeadmControlPlane`:
    
    1. Agregue el siguiente fragmento de código después de `files:`:
    
        ```console
        - contentFrom:
            secret:
              key: guard-authn-webhook.yaml
              name: azure-arc-guard-manifests
          owner: root:root
          path: /etc/kubernetes/guard-authn-webhook.yaml
          permissions: "0644"
        - contentFrom:
            secret:
              key: guard-authz-webhook.yaml
              name: azure-arc-guard-manifests
          owner: root:root
          path: /etc/kubernetes/guard-authz-webhook.yaml
          permissions: "0644"
        ```

    1. Agregue el siguiente fragmento de código después de `apiServer:` -> `extraVolumes:`:
    
        ```console
        - hostPath: /etc/kubernetes/guard-authn-webhook.yaml
            mountPath: /etc/guard/guard-authn-webhook.yaml
            name: guard-authn
            readOnly: true
        - hostPath: /etc/kubernetes/guard-authz-webhook.yaml
            mountPath: /etc/guard/guard-authz-webhook.yaml
            name: guard-authz
            readOnly: true
        ```

    1. Agregue el siguiente fragmento de código después de `apiServer:` -> `extraArgs:`:
    
        ```console
        authentication-token-webhook-cache-ttl: 5m0s
        authentication-token-webhook-config-file: /etc/guard/guard-authn-webhook.yaml
        authentication-token-webhook-version: v1
        authorization-mode: Node,Webhook,RBAC
        authorization-webhook-cache-authorized-ttl: 5m0s
        authorization-webhook-config-file: /etc/guard/guard-authz-webhook.yaml
        authorization-webhook-version: v1
        ```

    1. Guarde y salga para actualizar el objeto `KubeadmControlPlane`. Espere a que estos cambios se realicen en el clúster de carga de trabajo.


## <a name="create-role-assignments-for-users-to-access-the-cluster"></a>Creación de asignaciones de roles para que los usuarios accedan al clúster

Los propietarios del recurso de Kubernetes habilitado para Azure Arc pueden usar roles integrados o roles personalizados para conceder a otros usuarios acceso al clúster de Kubernetes.

### <a name="built-in-roles"></a>Roles integrados

| Role | Descripción |
|---|---|
| [Visor de Azure Arc Kubernetes](../../role-based-access-control/built-in-roles.md#azure-arc-kubernetes-viewer) | Permite el acceso de solo lectura para ver la mayoría de los objetos en un espacio de nombres. Este rol no permite ver secretos. Esto se debe a que el permiso `read` en los secretos permitiría el acceso a las credenciales `ServiceAccount` del espacio de nombres, que a su vez permitiría el acceso a la API mediante dicha cuenta `ServiceAccount` (una forma de elevación de privilegios). |
| [Escritor de Azure Arc Kubernetes](../../role-based-access-control/built-in-roles.md#azure-arc-kubernetes-writer) | Permite el acceso de lectura y escritura para ver la mayoría de los objetos en un espacio de nombres. Este rol no permite la visualización o modificación de roles o enlaces de roles. No obstante, este rol permite acceder a secretos y ejecutar pods como cualquier `ServiceAccount` en el espacio de nombres, por lo que se puede usar para obtener los niveles de acceso de la API de cualquier `ServiceAccount` en el espacio de nombres. |
| [Administrador de Azure Arc Kubernetes](../../role-based-access-control/built-in-roles.md#azure-arc-kubernetes-admin) | Permite el acceso de administrador. Está diseñado para concederse dentro de un espacio de nombres mediante RoleBinding. Si se usa un elemento RoleBinding, permite el acceso de lectura y escritura a la mayoría de los recursos de un espacio de nombres, incluida la capacidad de crear roles y enlaces de roles dentro del espacio de nombres. Este rol no permite el acceso de escritura a la cuota de recursos o al espacio de nombres en sí. |
| [Administrador de clústeres de Azure Arc Kubernetes](../../role-based-access-control/built-in-roles.md#azure-arc-kubernetes-cluster-admin) | Permite el acceso de superusuario para ejecutar cualquier acción en cualquier recurso. Cuando se usa en un elemento ClusterRoleBinding, ofrece control total sobre todos los recursos del clúster y todos los espacios de nombres. Cuando se usa en un RoleBinding, proporciona control total sobre cada recurso del espacio de nombres del enlace de roles, incluido el propio espacio de nombres.|

Puede crear asignaciones de roles en el ámbito del clúster de Kubernetes habilitado para Arc desde la hoja `Access Control (IAM)` del recurso de clúster en Azure Portal. También puede usar comandos de la CLI de Azure, como se muestra a continuación:

```azurecli
az role assignment create --role "Azure Arc Kubernetes Cluster Admin" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID
```

donde `AZURE-AD-ENTITY-ID` podría ser un nombre de usuario (por ejemplo, testuser@mytenant.onmicrosoft.com) o incluso el `appId` de una entidad de servicio.

El siguiente es otro ejemplo de creación de una asignación de roles en el ámbito de un espacio de nombres específico dentro del clúster:

```azurecli
az role assignment create --role "Azure Arc Kubernetes Viewer" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID/namespaces/<namespace-name>
```

> [!NOTE]
> Aunque se puede usar Azure Portal o la CLI para crear asignaciones de roles que tienen como ámbito el clúster, las asignaciones de roles que tienen como ámbito los espacios de nombres solo se pueden crear mediante la CLI.

### <a name="custom-roles"></a>Roles personalizados

Puede crear su propia definición de roles para usarla en las asignaciones de roles.

Examine el siguiente ejemplo una definición de roles que solo permite a un usuario leer las implementaciones. Para obtener más información, consulte la [lista completa de acciones de datos que puede usar para crear una definición de roles](../../role-based-access-control/resource-provider-operations.md#microsoftkubernetes).

Copie el siguiente objeto JSON en un archivo denominado custom-role.jsen. Reemplace el marcador de posición `<subscription-id>` por el identificador de la suscripción real. El siguiente rol personalizado usa una de las acciones de datos y le permite ver todas las implementaciones del ámbito (clúster/espacio de nombres) en el que se creó la asignación de roles.

```json
{
    "Name": "Arc Deployment Viewer",
    "Description": "Lets you view all deployments in cluster/namespace.",
    "Actions": [],
    "NotActions": [],
    "DataActions": [
        "Microsoft.Kubernetes/connectedClusters/apps/deployments/read"
    ],
    "NotDataActions": [],
    "assignableScopes": [
        "/subscriptions/<subscription-id>"
    ]
}
```

1. Ejecute el siguiente comando desde la carpeta donde se guardó `custom-role.json` para crear la definición de roles:

    ```bash
    az role definition create --role-definition @custom-role.json
    ```

1. Cree una asignación de roles con esta definición de roles personalizada:

    ```bash
    az role assignment create --role "Arc Deployment Viewer" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID/namespaces/<namespace-name>
    ```

## <a name="configure-kubectl-with-user-credentials"></a>Configuración de kubectl con credenciales de usuario

Hay dos maneras de obtener el archivo `kubeconfig` necesario para acceder al clúster:
1. Use la característica [Conexión de clúster](cluster-connect.md) ( `az connectedk8s proxy`) del clúster de Kubernetes habilitado para Azure Arc.
1. El administrador de clústeres comparte el archivo `kubeconfig` con todos los demás usuarios.

### <a name="if-you-are-accessing-cluster-using-cluster-connect-feature"></a>Si accede al clúster mediante la característica Conexión de clústeres

Ejecute el siguiente comando para iniciar el proceso de proxy:

```console
az connectedk8s proxy -n <clusterName> -g <resourceGroupName>
```

Una vez que se está ejecutando el proceso de proxy, puede abrir otra pestaña en la consola para [empezar a enviar solicitudes al clúster](#sending-requests-to-cluster).

### <a name="if-cluster-admin-shared-the-kubeconfig-file-with-you"></a>Si el administrador del clúster compartió el archivo `kubeconfig` con el usuario 

1. Ejecute el siguiente comando para establecer las credenciales del usuario:

    ```console
    kubectl config set-credentials <testuser>@<mytenant.onmicrosoft.com> \
    --auth-provider=azure \
    --auth-provider-arg=environment=AzurePublicCloud \
    --auth-provider-arg=client-id=<clientApplicationId> \
    --auth-provider-arg=tenant-id=<tenantId> \
    --auth-provider-arg=apiserver-id=<serverApplicationId>
    ```

1. Abra el archivo `kubeconfig` que ha creado antes. En `contexts`, compruebe el contexto asociado a los puntos del clúster con las credenciales de usuario que creó en el paso anterior.

1. Agregue el valor **config-mode** debajo de la configuración de usuario:
  
    ```console
    name: testuser@mytenant.onmicrosoft.com
    user:
        auth-provider:
        config:
            apiserver-id: $SERVER_APP_ID
            client-id: $CLIENT_APP_ID
            environment: AzurePublicCloud
            tenant-id: $TENANT_ID
            config-mode: "1"
        name: azure
    ```

## <a name="sending-requests-to-cluster"></a>Envío de solicitudes al clúster

1. Ejecutar cualquier comando `kubectl`. Por ejemplo:
  * `kubectl get nodes` 
  * `kubectl get pods`

1. Una vez que se le solicite la autenticación basada en explorador, copie la dirección URL de inicio de sesión del dispositivo `https://microsoft.com/devicelogin` y ábrala en el explorador web.

1. Escriba el código que apareció en la consola; después, copie y pegue el código del terminal en la solicitud de autenticación del dispositivo.

1. Escriba el nombre de usuario (testuser@mytenant.onmicrosoft.com) y la contraseña asociadas.

1. Si ve un mensaje de error similar al siguiente, significa que no está autorizado para acceder al recurso solicitado:

    ```console
    Error from server (Forbidden): nodes is forbidden: User "testuser@mytenant.onmicrosoft.com" cannot list resource "nodes" in API group "" at the cluster scope: User doesn't have access to the resource in Azure. Update role assignment to allow access.
    ```

    Un administrador debe crear una nueva asignación de roles para autorizar a este usuario el acceso al recurso.

## <a name="use-conditional-access-with-azure-ad"></a>Uso del acceso condicional con Azure AD

Al integrar Azure AD con el clúster de Kubernetes habilitado para Arc, también puede usar el [acceso condicional](../../active-directory/conditional-access/overview.md) para controlar el acceso al clúster.

> [!NOTE]
> El acceso condicional de Azure AD es una capacidad de Azure AD Premium.

Para crear una directiva de acceso condicional de ejemplo para usarla con el clúster, complete los siguientes pasos:

1. En la parte superior de Azure Portal, busque y seleccione Azure Active Directory.
1. En el menú de Azure Active Directory del lado izquierdo, seleccione *Aplicaciones empresariales*.
1. En el menú de aplicaciones empresariales del lado izquierdo, seleccione *Acceso condicional*.
1. En el menú de acceso condicional del lado izquierdo, seleccione *Directivas* y después *Nueva directiva*.
1. En el menú de acceso condicional del lado izquierdo, seleccione *Directivas* y después *Nueva directiva*.
    
    [ ![Adición de una directiva de acceso condicional](./media/azure-rbac/conditional-access-new-policy.png) ](./media/azure-rbac/conditional-access-new-policy.png#lightbox)

1. Escriba un nombre para la directiva, como *arc-k8s-policy*.
1. Haga clic en *Usuarios y grupos* y, luego, debajo de *Incluir* seleccione *Seleccionar usuarios y grupos*. Elija los usuarios y grupos en los que quiere aplicar la directiva. En este ejemplo, elija el mismo grupo de Azure AD que tiene acceso de administración al clúster.

    [ ![Selección de usuarios o grupos para aplicar la directiva de acceso condicional](./media/azure-rbac/conditional-access-users-groups.png) ](./media/azure-rbac/conditional-access-users-groups.png#lightbox)

1. Seleccione *Aplicaciones en la nube o acciones* y, después, debajo de *Incluir*, escoja *Seleccionar aplicaciones*. Busque y seleccione la aplicación de servidor que creó anteriormente.

    [ ![Selección de la aplicación de servidor para aplicar la directiva de acceso condicional ](./media/azure-rbac/conditional-access-apps.png) ](./media/azure-rbac/conditional-access-apps.png#lightbox)

1. En *Controles de acceso*, seleccione *Conceder*. Seleccione *Conceder acceso* y después *Requerir que el dispositivo esté marcado como compatible*.

    [ ![Selección para permitir solo los dispositivos compatibles con la directiva de acceso condicional](./media/azure-rbac/conditional-access-grant-compliant.png) ](./media/azure-rbac/conditional-access-grant-compliant.png#lightbox)
    
1. En *Habilitar directiva*, seleccione *Activar* y, después, *Crear*.

    [ ![Habilitación de la directiva de acceso condicional](./media/azure-rbac/conditional-access-enable-policies.png) ](./media/azure-rbac/conditional-access-enable-policies.png#lightbox)

Vuelva a acceder al clúster. Por ejemplo, mediante la ejecución del comando `kubectl get nodes` para ver los nodos del clúster:

```console
kubectl get nodes
```

Siga las instrucciones para iniciar sesión de nuevo. Observe que hay un mensaje de error que indica que inició sesión correctamente, pero el administrador requiere que el dispositivo que solicita acceso esté administrado por su instancia de Azure AD para acceder al recurso.

En Azure Portal, vaya a Azure Active Directory, seleccione *Aplicaciones empresariales* y, después, en *Actividad* seleccione *Inicios de sesión*. Observe que hay una entrada en la parte superior con un valor de *Estado* de *Error* y un *Acceso condicional* *Correcto*. Seleccione la entrada y, a continuación, seleccione *Acceso condicional* en *Detalles*. Observe que se muestra su directiva de acceso condicional.

[ ![Error en la entrada de inicio de sesión debido a la directiva de acceso condicional](./media/azure-rbac/conditional-access-sign-in-activity.png) ](./media/azure-rbac/conditional-access-sign-in-activity.png#lightbox)

## <a name="configure-just-in-time-cluster-access-with-azure-ad"></a>Configuración del acceso Just-In-Time al clúster con Azure AD

Otra opción para el control de acceso al clúster es usar Privileged Identity Management (PIM) con las solicitudes Just-in-Time.

>[!NOTE]
> PIM es una funcionalidad de Azure AD Premium que requiere una SKU P2 prémium. Para más información sobre las SKU de Azure AD, consulte la [guía de precios](https://azure.microsoft.com/pricing/details/active-directory/).

Para configurar solicitudes de acceso Just-In-Time para el clúster, siga estos pasos:

1. En la parte superior de Azure Portal, busque y seleccione Azure Active Directory.
1. Tome nota del identificador de inquilino, al que se hace referencia como <tenant-id> en el resto de estas instrucciones.

    [ ![Detalles del inquilino de AAD](./media/azure-rbac/jit-get-tenant-id.png) ](./media/azure-rbac/jit-get-tenant-id.png#lightbox)

1. En *Administrar*, en el menú de Azure Active Directory de la parte izquierda, seleccione *Grupos* y haga clic en *Nuevo grupo*.

    [ ![Selección de un nuevo grupo](./media/azure-rbac/jit-create-new-group.png) ](./media/azure-rbac/jit-create-new-group.png#lightbox)

1. Asegúrese de que está seleccionado un tipo de grupo de *Seguridad* y especifique un nombre de grupo, como *myJITGroup*. En *Azure AD Roles can be assigned to this group (Preview)* (Se pueden asignar roles de Azure AD a este grupo [versión preliminar]), seleccione *Sí*. Por último, seleccione *Crear*.

    [ ![Creación de un nuevo grupo](./media/azure-rbac/jit-new-group-created.png) ](./media/azure-rbac/jit-new-group-created.png#lightbox)

1. Se le redirigirá a la página *Grupos*. Seleccione el grupo recién creado y tome nota del identificador de objeto, al que se hace referencia en el resto de estas instrucciones como `<object-id>`.

    [ ![Grupo creado](./media/azure-rbac/jit-get-object-id.png) ](./media/azure-rbac/jit-get-object-id.png#lightbox)

1. De nuevo en Azure Portal, en el menú de *Actividad* situado a la izquierda seleccione *Privileged Access (Preview)* (Acceso con privilegios [versión preliminar]) y elija *Enable Privileged Access* (Habilitar acceso con privilegios).

    [ ![Habilitación del acceso con privilegios](./media/azure-rbac/jit-enabling-priv-access.png) ](./media/azure-rbac/jit-enabling-priv-access.png#lightbox)

1. Seleccione *Agregar asignaciones* para empezar a conceder acceso.

    [ ![Adición de una asignación activa](./media/azure-rbac/jit-add-active-assignment.png) ](./media/azure-rbac/jit-add-active-assignment.png#lightbox)

1. Seleccione un rol de *miembro* y elija los usuarios y grupos a los que desea conceder el acceso al clúster. Los administradores de grupos pueden modificar estas asignaciones en cualquier momento. Elija *Siguiente* cuando esté listo para continuar.

    [ ![Adición de una asignación](./media/azure-rbac/jit-adding-assignment.png) ](./media/azure-rbac/jit-adding-assignment.png#lightbox)

1. Elija un tipo de asignación *Activa*, la duración deseada y especifique una justificación. Cuando esté listo para continuar, seleccione *Asignar*. Para más información sobre los tipos de asignación, consulte [Asignación de la elegibilidad para un grupo de acceso con privilegios (versión preliminar) en Privileged Identity Management](../../active-directory/privileged-identity-management/groups-assign-member-owner.md#assign-an-owner-or-member-of-a-group).

    [ ![Elección de propiedades para la asignación](./media/azure-rbac/jit-set-active-assignment.png) ](./media/azure-rbac/jit-set-active-assignment.png#lightbox)

Una vez que se han realizado las asignaciones, compruebe que el acceso Just-in-Time funciona mediante el acceso al clúster. Por ejemplo:

Use el comando `kubectl get nodes` para ver los nodos del clúster:

```console
kubectl get nodes
```

Tenga en cuenta el requisito de autenticación y siga los pasos para autenticarse. Si todo se ha hecho correctamente, debería ver un resultado similar al siguiente:

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME      STATUS   ROLES    AGE      VERSION
node-1    Ready    agent    6m36s    v1.18.14
node-2    Ready    agent    6m42s    v1.18.14
node-3    Ready    agent    6m33s    v1.18.14
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> Conéctese de forma segura al clúster mediante [Conexión de clúster](cluster-connect.md)