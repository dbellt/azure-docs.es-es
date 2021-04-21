---
title: Uso de Azure AD en Azure Kubernetes Service
description: Aprenda a usar Azure AD en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 02/1/2021
ms.author: miwithro
ms.openlocfilehash: 3db9f8d895b4c13b5f969859f422e7b566722ffc
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783078"
---
# <a name="aks-managed-azure-active-directory-integration"></a>Integración de Azure Active Directory administrado por AKS

La integración de Azure AD administrado por AKS está diseñada para simplificar la experiencia de integración de Azure AD, donde los usuarios debían crear una aplicación cliente y otra de servidor, y donde el inquilino de Azure AD debía conceder permisos de lectura para Active Directory. En la nueva versión, el proveedor de recursos de AKS administra las aplicaciones de cliente y servidor.

## <a name="azure-ad-authentication-overview"></a>Introducción a la autenticación de Azure AD

Los administradores de clústeres pueden configurar el control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) en función de la identidad de un usuario o la pertenencia a grupos de directorios. La autenticación de Azure AD se proporciona a los clústeres de AKS con OpenID Connect. OpenID Connect es una capa de identidad creada basándose en el protocolo OAuth 2.0. Puede encontrar más información sobre OpenID Connect en la [documentación de OpenID Connect][open-id-connect].

Obtenga más información sobre el flujo de integración de Azure AD en la [documentación de los conceptos de la integración de Azure Active Directory](concepts-identity.md#azure-ad-integration).

## <a name="limitations"></a>Limitaciones 

* No se puede deshabilitar la integración de Azure AD administrados por AKS.
* No se admite cambiar un clúster integrado de Azure AD administrado por AKS por AAD heredado
* Los clústeres habilitados para RBAC que no son de Kubernetes no se admiten para la integración de Azure AD administrado por AKS.
* No se admite el cambio del inquilino de Azure AD asociado a la integración de Azure AD administrado por AKS.

## <a name="prerequisites"></a>Requisitos previos

* La CLI de Azure, versión 2.11.0 o posterior
* Kubectl con la versión [1.18.1](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1181), como mínimo, o [kubelogin](https://github.com/Azure/kubelogin)
* Si usa [helm](https://github.com/helm/helm), versión mínima de helm 3.3.

> [!Important]
> Debe usar Kubectl con la versión 1.18.1, como mínimo, o kubelogin. La diferencia entre las versiones secundarias de Kubernetes y kubectl no debe ser superior a una versión. Si no usa la versión correcta, observará problemas de autenticación.

Para instalar kubectl y kubelogin, use los siguientes comandos:

```azurecli-interactive
sudo az aks install-cli
kubectl version --client
kubelogin --version
```

Siga [estas instrucciones](https://kubernetes.io/docs/tasks/tools/install-kubectl/) para otros sistemas operativos.

## <a name="before-you-begin"></a>Antes de empezar

Para el clúster, necesita un grupo de Azure AD. Este grupo es necesario como grupo de administradores para que el clúster conceda permisos de administrador de clústeres. Puede usar un grupo de Azure AD existente o crear uno nuevo. Anote el identificador de objeto del grupo de Azure AD.

```azurecli-interactive
# List existing groups in the directory
az ad group list --filter "displayname eq '<group-name>'" -o table
```

Para crear un grupo de Azure AD para los administradores de clúster, use el comando siguiente:

```azurecli-interactive
# Create an Azure AD group
az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup
```

## <a name="create-an-aks-cluster-with-azure-ad-enabled"></a>Creación de un clúster de AKS con Azure AD habilitado

Cree un clúster de AKS con los comandos de la CLI siguientes.

Crear un grupo de recursos de Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```

Cree un clúster de AKS y habilite el acceso de administración para el grupo de Azure AD

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Un clúster de Azure AD administrado por AKS creado correctamente contiene la sección siguiente en el cuerpo de la respuesta
```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Una vez creado el clúster, puede empezar a acceder a él.

## <a name="access-an-azure-ad-enabled-cluster"></a>Acceso a un clúster habilitado para Azure AD

Necesitará el rol integrado del [usuario del clúster de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) para realizar los pasos siguientes.

Obtenga las credenciales de usuario para acceder al clúster:
 
```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
Siga las instrucciones para iniciar sesión.

Use el comando "kubectl get nodes" para ver los nodos del clúster:

```azurecli-interactive
kubectl get nodes

NAME                       STATUS   ROLES   AGE    VERSION
aks-nodepool1-15306047-0   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-1   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-2   Ready    agent   102m   v1.15.10
```
Configure el [control de acceso basado en rol de Azure (RBAC)](./azure-ad-rbac.md) para configurar grupos de seguridad adicionales para los clústeres.

## <a name="troubleshooting-access-issues-with-azure-ad"></a>Solución de problemas de acceso con Azure AD

> [!Important]
> En los pasos que se describen a continuación se omite la autenticación de grupo de Azure AD normal. Úselos únicamente en caso de emergencia.

Si está bloqueado permanentemente al no tener acceso a un grupo de Azure AD válido con acceso al clúster, de todos modos puede obtener las credenciales de administrador para acceder directamente al clúster.

Para llevar a cabo estos pasos, necesitará tener acceso al rol integrado [Administrador de clúster de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role).

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
```

## <a name="enable-aks-managed-azure-ad-integration-on-your-existing-cluster"></a>Habilitación de la integración de Azure AD administrados por AKS en el clúster existente

Puede habilitar la integración de Azure AD administrado por AKS en el clúster habilitado para RBAC de Kubernetes existente. Asegúrese de establecer el grupo de administradores para mantener el acceso al clúster.

```azurecli-interactive
az aks update -g MyResourceGroup -n MyManagedCluster --enable-aad --aad-admin-group-object-ids <id-1> [--aad-tenant-id <id>]
```

Una activación correcta de un clúster de Azure AD administrado por AKS contiene la sección siguiente en el cuerpo de la respuesta:

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Vuelva a descargar las credenciales de usuario para acceder al clúster; para ello, siga los pasos que se indican [aquí][access-cluster].

## <a name="upgrading-to-aks-managed-azure-ad-integration"></a>Actualización a la integración de Azure AD administrado por AKS

Si el clúster usa la integración de Azure AD heredada, puede actualizar a la integración de Azure AD administrado por AKS.

```azurecli-interactive
az aks update -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Una migración correcta de un clúster de Azure AD administrado por AKS contiene la sección siguiente en el cuerpo de la respuesta:

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Si quiere acceder al clúster, siga los pasos que se indican [aquí][access-cluster].

## <a name="non-interactive-sign-in-with-kubelogin"></a>Inicio de sesión no interactivo con kubelogin

Hay algunos escenarios no interactivos, como las canalizaciones de integración continua, que no están disponibles actualmente con kubectl. Puede usar [`kubelogin`](https://github.com/Azure/kubelogin) para acceder al clúster con el inicio de sesión no interactivo de la entidad de servicio.

## <a name="use-conditional-access-with-azure-ad-and-aks"></a>Uso de acceso condicional con Azure AD y AKS

Al integrar Azure AD con el clúster de AKS, también puede usar el [acceso condicional][aad-conditional-access] para controlar el acceso al clúster.

> [!NOTE]
> El acceso condicional de Azure AD es una capacidad de Azure AD Premium.

Para crear una directiva de acceso condicional de ejemplo para usarla con AKS, complete los siguientes pasos:

1. En la parte superior de Azure Portal, busque y seleccione Azure Active Directory.
1. En el menú de Azure Active Directory del lado izquierdo, seleccione *Aplicaciones empresariales*.
1. En el menú de aplicaciones empresariales del lado izquierdo, seleccione *Acceso condicional*.
1. En el menú de acceso condicional del lado izquierdo, seleccione *Directivas* y después *Nueva directiva*.
    :::image type="content" source="./media/managed-aad/conditional-access-new-policy.png" alt-text="Incorporación de una directiva de acceso condicional":::
1. Escriba un nombre para la directiva, como *aks-policy*.
1. Haga clic en *Usuarios y grupos* y, luego, debajo de *Incluir* seleccione *Seleccionar usuarios y grupos*. Elija los usuarios y grupos en los que quiere aplicar la directiva. En este ejemplo, elija el mismo grupo de Azure AD que tiene acceso de administración al clúster.
    :::image type="content" source="./media/managed-aad/conditional-access-users-groups.png" alt-text="Selección de usuarios o grupos para aplicar la directiva de acceso condicional":::
1. Seleccione *Aplicaciones en la nube o acciones* y, después, debajo de *Incluir*, escoja *Seleccionar aplicaciones*. Busque *Azure Kubernetes Service* y seleccione *Azure Kubernetes Service AAD Server*.
    :::image type="content" source="./media/managed-aad/conditional-access-apps.png" alt-text="Selección de Azure Kubernetes Service AD Server para aplicar la directiva de acceso condicional":::
1. En *Controles de acceso*, seleccione *Conceder*. Seleccione *Conceder acceso* y después *Requerir que el dispositivo esté marcado como compatible*.
    :::image type="content" source="./media/managed-aad/conditional-access-grant-compliant.png" alt-text="Selección para permitir solo los dispositivos compatibles con la directiva de acceso condicional":::
1. En *Habilitar directiva*, seleccione *Activar* y, después, *Crear*.
    :::image type="content" source="./media/managed-aad/conditional-access-enable-policy.png" alt-text="Habilitación de la directiva de acceso condicional":::

Obtenga las credenciales de usuario para acceder al clúster, por ejemplo:

```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

Siga las instrucciones para iniciar sesión.

Use el comando `kubectl get nodes` para ver los nodos del clúster:

```azurecli-interactive
kubectl get nodes
```

Siga las instrucciones para iniciar sesión de nuevo. Observe que hay un mensaje de error que indica que inició sesión correctamente, pero el administrador requiere que el dispositivo que solicita acceso esté administrado por su instancia de Azure AD para acceder al recurso.

En Azure Portal, vaya a Azure Active Directory, seleccione *Aplicaciones empresariales* y, después, en *Actividad* seleccione *Inicios de sesión*. Observe que hay una entrada en la parte superior con un valor de *Estado* de *Error* y un *Acceso condicional* *Correcto*. Seleccione la entrada y, a continuación, seleccione *Acceso condicional* en *Detalles*. Observe que se muestra su directiva de acceso condicional.

:::image type="content" source="./media/managed-aad/conditional-access-sign-in-activity.png" alt-text="Error en la entrada de inicio de sesión debido a la directiva de acceso condicional":::

## <a name="configure-just-in-time-cluster-access-with-azure-ad-and-aks"></a>Configuración del acceso Just-In-Time al clúster con Azure AD y AKS

Otra opción para el control de acceso al clúster es usar Privileged Identity Management (PIM) con las solicitudes Just-in-Time.

>[!NOTE]
> PIM es una funcionalidad de Azure AD Premium que requiere una SKU P2 prémium. Para más información sobre las SKU de Azure AD, consulte la [guía de precios][aad-pricing].

Para integrar las solicitudes de acceso Just-in-Time con un clúster de AKS mediante la integración de Azure AD administrado por AKS, complete los pasos siguientes:

1. En la parte superior de Azure Portal, busque y seleccione Azure Active Directory.
1. Tome nota del identificador de inquilino, al que se hace referencia en el resto de estas instrucciones como `<tenant-id>` :::image type="content" source="./media/managed-aad/jit-get-tenant-id.png" alt-text="En un explorador web, se muestra la pantalla de Azure Portal para Azure Active Directory con el identificador del inquilino resaltado.":::
1. En *Administrar*, en el menú de Azure Active Directory de la parte izquierda, seleccione *Grupos* y haga clic en *Nuevo grupo*.
    :::image type="content" source="./media/managed-aad/jit-create-new-group.png" alt-text="Muestra la pantalla de grupos de Active Directory en Azure Portal con la opción &quot;Nuevo grupo&quot; resaltada.":::
1. Asegúrese de que está seleccionado un tipo de grupo de *Seguridad* y especifique un nombre de grupo, como *myJITGroup*. En *Azure AD Roles can be assigned to this group (Preview)* (Se pueden asignar roles de Azure AD a este grupo [versión preliminar]), seleccione *Sí*. Por último, seleccione *Crear*.
    :::image type="content" source="./media/managed-aad/jit-new-group-created.png" alt-text="Muestra la pantalla de creación de nuevo grupo de Azure Portal.":::
1. Se le redirigirá a la página *Grupos*. Seleccione el grupo recién creado y tome nota del identificador de objeto, al que se hace referencia en el resto de estas instrucciones como `<object-id>`.
    :::image type="content" source="./media/managed-aad/jit-get-object-id.png" alt-text="Muestra la pantalla de Azure Portal con el grupo recién creado, donde se resalta el identificador de objeto.":::
1. Implemente un clúster de AKS con la integración de Azure AD administrado por AKS mediante los valores `<tenant-id>` y `<object-id>` de antes:
    ```azurecli-interactive
    az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <object-id> --aad-tenant-id <tenant-id>
    ```
1. De nuevo en Azure Portal, en el menú de *Actividad* situado a la izquierda seleccione *Privileged Access (Preview)* (Acceso con privilegios [versión preliminar]) y elija *Enable Privileged Access* (Habilitar acceso con privilegios).
    :::image type="content" source="./media/managed-aad/jit-enabling-priv-access.png" alt-text="Se muestra la página Privileged Access (Preview) (Acceso con privilegios [versión preliminar]) de Azure Portal, con &quot;Enable privileged access&quot; (Habilitar acceso con privilegios) resaltado.":::
1. Seleccione *Agregar asignaciones* para empezar a conceder acceso.
    :::image type="content" source="./media/managed-aad/jit-add-active-assignment.png" alt-text="Se muestra la pantalla Privileged access (Preview) (Acceso con privilegios [versión preliminar]) de Azure Portal una vez habilitada la opción. Se resalta la opción &quot;Agregar asignaciones&quot;.":::
1. Seleccione un rol de *miembro* y elija los usuarios y grupos a los que desea conceder el acceso al clúster. Los administradores de grupos pueden modificar estas asignaciones en cualquier momento. Elija *Siguiente* cuando esté listo para continuar.
    :::image type="content" source="./media/managed-aad/jit-adding-assignment.png" alt-text="Se muestra la pantalla Pertenencia de Agregar asignaciones de Azure Portal, con un usuario de ejemplo seleccionado para agregarse como miembro. Se resalta la opción &quot;Siguiente&quot;.":::
1. Elija un tipo de asignación *Activa*, la duración deseada y especifique una justificación. Cuando esté listo para continuar, seleccione *Asignar*. Para más información sobre los tipos de asignación, consulte [Asignación de la elegibilidad para un grupo de acceso con privilegios (versión preliminar) en Privileged Identity Management][aad-assignments].
    :::image type="content" source="./media/managed-aad/jit-set-active-assignment-details.png" alt-text="Se muestra la pantalla Configuración de Agregar asignaciones de Azure Portal. Se ha seleccionado un tipo de asignación &quot;Activa&quot; y se ha especificado una justificación de ejemplo. Se resalta la opción &quot;Asignar&quot;.":::

Una vez que se han realizado las asignaciones, compruebe que el acceso Just-in-Time funciona mediante el acceso al clúster. Por ejemplo:

```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

Siga los pasos para iniciar sesión.

Use el comando `kubectl get nodes` para ver los nodos del clúster:

```azurecli-interactive
kubectl get nodes
```

Tenga en cuenta el requisito de autenticación y siga los pasos para autenticarse. Si todo se ha hecho correctamente, debería ver un resultado similar al siguiente:

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-61156405-vmss000000   Ready    agent   6m36s   v1.18.14
aks-nodepool1-61156405-vmss000001   Ready    agent   6m42s   v1.18.14
aks-nodepool1-61156405-vmss000002   Ready    agent   6m33s   v1.18.14
```

### <a name="troubleshooting"></a>Solución de problemas

Si `kubectl get nodes` devuelve un error parecido al siguiente:

```output
Error from server (Forbidden): nodes is forbidden: User "aaaa11111-11aa-aa11-a1a1-111111aaaaa" cannot list resource "nodes" in API group "" at the cluster scope
```

Asegúrese de que el administrador del grupo de seguridad haya dado a su cuenta una asignación *Activa*.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [integración de Azure RBAC para la autorización de Kubernetes][azure-rbac-integration].
* Aprenda sobre la [integración de Azure AD con RBAC de Kubernetes][azure-ad-rbac].
* Use [kubelogin](https://github.com/Azure/kubelogin) para obtener acceso a las características de autenticación de Azure que no están disponibles en kubectl.
* Obtenga más información sobre los [conceptos de identidad de Kubernetes y AKS][aks-concepts-identity].
* Use las [plantillas de Azure Resource Manager (ARM)][aks-arm-template] para crear clústeres habilitados para Azure AD administrado por AKS.

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters
[aad-pricing]: https://azure.microsoft.com/pricing/details/active-directory/

<!-- LINKS - Internal -->
[aad-conditional-access]: ../active-directory/conditional-access/overview.md
[azure-rbac-integration]: manage-azure-rbac.md
[aks-concepts-identity]: concepts-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-group-create]: /cli/azure/group#az_group_create
[open-id-connect]:../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az_ad_user_show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
[access-cluster]: #access-an-azure-ad-enabled-cluster
[aad-migrate]: #upgrading-to-aks-managed-azure-ad-integration
[aad-assignments]: ../active-directory/privileged-identity-management/groups-assign-member-owner.md#assign-an-owner-or-member-of-a-group
