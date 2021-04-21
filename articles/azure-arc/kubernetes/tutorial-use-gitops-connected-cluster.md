---
title: 'Tutorial: Implementación de configuraciones mediante GitOps en un clúster de Kubernetes habilitado para Azure Arc'
description: 'En este tutorial se muestra cómo aplicar configuraciones en un clúster de Kubernetes habilitado para Azure Arc. Para ver una introducción conceptual sobre este proceso, consulte el artículo Configuraciones y GitOps: Kubernetes habilitado para Azure Arc.'
author: shashankbarsin
ms.author: shasb
ms.service: azure-arc
ms.topic: tutorial
ms.date: 03/02/2021
ms.custom: template-tutorial , devx-track-azurecli
ms.openlocfilehash: 0f1172ffa0d29734e7ec005bf2812eeca215aa0d
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484173"
---
# <a name="tutorial-deploy-configurations-using-gitops-on-an-azure-arc-enabled-kubernetes-cluster"></a>Tutorial: Implementación de configuraciones mediante GitOps en un clúster de Kubernetes habilitado para Azure Arc 

En este tutorial, aplicará configuraciones mediante GitOps en un clúster de Kubernetes habilitado para Azure Arc. Aprenderá a:

> [!div class="checklist"]
> * Crear una configuración en un clúster de Kubernetes habilitado para Azure Arc con un repositorio de Git de ejemplo.
> * Comprobar que la configuración se haya creado correctamente.
> * Aplicar la configuración desde un repositorio de Git privado.
> * Asegurarse de la configuración de Kubernetes.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un clúster conectado de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, siga las indicaciones de [Inicio rápido: Conexión a un clúster de Kubernetes habilitado para Azure Arc](quickstart-connect-cluster.md).
- Entender las ventajas y la arquitectura de esta característica. Más información en el artículo [Configuraciones y GitOps: Kubernetes habilitado para Azure Arc](conceptual-configurations.md).
- Instale la extensión `k8s-configuration` de la CLI de Azure, versión 1.0.0 o posteriores:
  
  ```azurecli
  az extension add --name k8s-configuration
  ```

    >[!TIP]
    > Si la extensión `k8s-configuration` ya está instalada, puede actualizarla a su versión más reciente con el siguiente comando: `az extension update --name k8s-configuration`

## <a name="create-a-configuration"></a>Creación de una configuración

El [repositorio de ejemplo](https://github.com/Azure/arc-k8s-demo) que se utiliza en este artículo está estructurado en torno al rol de un operador de clústeres. Los manifiestos de este repositorio aprovisionan algunos espacios de nombres, implementan cargas de trabajo y proporcionan una configuración específica del equipo. Si se utiliza este repositorio con GitOps, se crean los siguientes recursos en el clúster:

* Espacios de nombres: `cluster-config`, `team-a`, `team-b`
* Implementación: `cluster-config/azure-vote`
* ConfigMap: `team-a/endpoints`

`config-agent` sondea Azure para buscar configuraciones nuevas o actualizadas. Esta tarea tardará hasta 30 segundos.

Si va a asociar un repositorio privado a la configuración, complete los pasos siguientes de [Aplicación de la configuración desde un repositorio de Git privado](#apply-configuration-from-a-private-git-repository).

## <a name="use-azure-cli"></a>Uso de CLI de Azure
Use la extensión de la CLI de Azure para `k8s-configuration`, a fin de vincular el clúster conectado al [repositorio de Git de ejemplo](https://github.com/Azure/arc-k8s-demo). 
1. Asigne a esta configuración el nombre `cluster-config`.
1. Indique al agente que implemente el operador en el espacio de nombres `cluster-config`.
1. Otorgue al operador permisos `cluster-admin`.

    ```azurecli
    az k8s-configuration create --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name cluster-config --operator-namespace cluster-config --repository-url https://github.com/Azure/arc-k8s-demo --scope cluster --cluster-type connectedClusters
    ```

    ```output
    {
      "complianceStatus": {
      "complianceState": "Pending",
      "lastConfigApplied": "0001-01-01T00:00:00",
      "message": "{\"OperatorMessage\":null,\"ClusterState\":null}",
      "messageLevel": "3"
      },
      "configurationProtectedSettings": {},
      "enableHelmOperator": false,
      "helmOperatorProperties": null,
      "id": "/subscriptions/<sub id>/resourceGroups/<group name>/providers/Microsoft.Kubernetes/connectedClusters/<cluster name>/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
      "name": "cluster-config",
      "operatorInstanceName": "cluster-config",
      "operatorNamespace": "cluster-config",
      "operatorParams": "--git-readonly",
      "operatorScope": "cluster",
      "operatorType": "Flux",
      "provisioningState": "Succeeded",
      "repositoryPublicKey": "",
      "repositoryUrl": "https://github.com/Azure/arc-k8s-demo",
      "resourceGroup": "MyRG",
      "sshKnownHostsContents": "",
      "systemData": {
        "createdAt": "2020-11-24T21:22:01.542801+00:00",
        "createdBy": null,
        "createdByType": null,
        "lastModifiedAt": "2020-11-24T21:22:01.542801+00:00",
        "lastModifiedBy": null,
        "lastModifiedByType": null
      },
      "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
      ```

### <a name="use-a-public-git-repository"></a>Uso de un repositorio de Git público

| Parámetro | Formato |
| ------------- | ------------- |
| `--repository-url` | http[s]://servidor/repositorio[.git] o git://servidor/repositorio[.git]

### <a name="use-a-private-git-repository-with-ssh-and-flux-created-keys"></a>Uso de un repositorio de Git privado con claves SSH y creadas por Flux

Agregue la clave pública generada por Flux a la cuenta de usuario del proveedor de servicios de Git. Si la clave se agrega al repositorio en lugar de a la cuenta de usuario, use `git@` en lugar de `user@` en la dirección URL. 

Vaya a la sección [Aplicación de la configuración desde un repositorio de Git privado](#apply-configuration-from-a-private-git-repository) para obtener más detalles.


| Parámetro | Formato | Notas
| ------------- | ------------- | ------------- |
| `--repository-url` | ssh://user@server/repo[.git] o user@server:repo[.git] | `git@` puede reemplazar a `user@`

### <a name="use-a-private-git-repository-with-ssh-and-user-provided-keys"></a>Uso de un repositorio de Git privado con claves SSH y proporcionadas por el usuario

Proporcione una clave privada propia de forma directa o en un archivo. La clave debe estar en [formato PEM](https://aka.ms/PEMformat) y terminar con el carácter de nueva línea (\n). 

Agregue la clave pública asociada a la cuenta de usuario del proveedor de servicios de Git. Si la clave se agrega al repositorio en lugar de a la cuenta de usuario, use `git@` en lugar de `user@`. 

Vaya a la sección [Aplicación de la configuración desde un repositorio de Git privado](#apply-configuration-from-a-private-git-repository) para obtener más detalles.  

| Parámetro | Formato | Notas |
| ------------- | ------------- | ------------- |
| `--repository-url`  | ssh://user@server/repo[.git] o user@server:repo[.git] | `git@` puede reemplazar a `user@` |
| `--ssh-private-key` | Clave codificada en Base64 en [formato PEM](https://aka.ms/PEMformat) | La clave se proporciona directamente |
| `--ssh-private-key-file` | Ruta de acceso completa al archivo local | Se proporciona la ruta de acceso completa al archivo local que contiene la clave en formato PEM

### <a name="use-a-private-git-host-with-ssh-and-user-provided-known-hosts"></a>Uso de un repositorio de Git privado con SSH y hosts conocidos proporcionados por el usuario

El operador Flux mantiene una lista de hosts de Git comunes en su archivo de hosts conocidos para autenticar el repositorio de Git antes de establecer la conexión SSH. Si usa un repositorio de Git *no común* o un host de Git propio, puede proporcionar la clave de host para que Flux pueda identificar el repositorio. 

Al igual que con las claves privadas, puede proporcionar el contenido de known_hosts directamente o en un archivo. Al proporcionar su propio contenido, utilice las [especificaciones de formato del contenido de known_hosts](https://aka.ms/KnownHostsFormat), junto con cualquiera de los escenarios de claves SSH anteriores.

| Parámetro | Formato | Notas |
| ------------- | ------------- | ------------- |
| `--repository-url`  | ssh://user@server/repo[.git] o user@server:repo[.git] | `git@` puede reemplazar a `user@` |
| `--ssh-known-hosts` | Con codificación base64 | Proporcione el contenido de hosts conocidos directamente |
| `--ssh-known-hosts-file` | Ruta de acceso completa al archivo local | Proporcione el contenido de hosts conocidos en un archivo local |

### <a name="use-a-private-git-repository-with-https"></a>Uso de un repositorio de Git privado con HTTPS

| Parámetro | Formato | Notas |
| ------------- | ------------- | ------------- |
| `--repository-url` | https://server/repo [.git] | HTTPS con autenticación básica |
| `--https-user` | Cadena sin codificar o codificada en Base64 | Nombre de usuario HTTPS |
| `--https-key` | Cadena sin codificar o codificada en Base64 | Token de acceso personal o contraseña de HTTPS

>[!NOTE]
>* La versión 1.2.0+ del gráfico del operador Helm es compatible con la autenticación privada de la versión de Helm de HTTPS.
>* La versión de Helm de HTTPS no está admitida en los clústeres administrados por AKS.
>* Si necesita Flux para acceder al repositorio de Git mediante el proxy, tendrá que actualizar los agentes de Azure Arc con la configuración de proxy. Para obtener más información, consulte [Conexión mediante un servidor proxy de salida](./quickstart-connect-cluster.md#connect-using-an-outbound-proxy-server).


## <a name="additional-parameters"></a>Parámetros adicionales

Personalice la configuración con los siguientes parámetros opcionales:

| Parámetro | Descripción |
| ------------- | ------------- |
| `--enable-helm-operator`| Modificador para habilitar la compatibilidad con las implementaciones de gráficos de Helm. |
| `--helm-operator-params` | Valores de gráfico para el operador de Helm (si está habilitado). Por ejemplo, `--set helm.versions=v3`. |
| `--helm-operator-chart-version` | Versión del gráfico para el operador de Helm (si está habilitado). Use la versión 1.2.0 o superior. Valor predeterminado: "1.2.0". |
| `--operator-namespace` | Nombre para el espacio de nombres del operador. Valor predeterminado: "default". Máx.: 23 caracteres. |
| `--operator-params` | Parámetros para el operador. Se debe proporcionar entre comillas simples. Por ejemplo: ```--operator-params='--git-readonly --sync-garbage-collection --git-branch=main'``` 

### <a name="options-supported-in----operator-params"></a>Opciones admitidas en `--operator-params`:

| Opción | Descripción |
| ------------- | ------------- |
| `--git-branch`  | Rama del repositorio de Git que se va a usar para los manifiestos de Kubernetes. El valor predeterminado es "master". Los repositorios más recientes tienen una rama raíz denominada `main`, en cuyo caso debe establecer `--git-branch=main`. |
| `--git-path`  | Ruta de acceso relativa dentro del repositorio de Git para que Flux localice los manifiestos de Kubernetes. |
| `--git-readonly` | El repositorio de Git se considerará de solo lectura. Flux no intentará escribir en él. |
| `--manifest-generation`  | Si está habilitado, Flux buscará .flux.yaml y ejecutará Kustomize u otros generadores de manifiestos. |
| `--git-poll-interval`  | Período en el que se sondea el repositorio de Git en busca de confirmaciones nuevas. El valor predeterminado es `5m` (5 minutos). |
| `--sync-garbage-collection`  | Si está habilitado, Flux eliminará los recursos que ha creado, pero que ya no están presentes en Git. |
| `--git-label`  | Etiqueta para mantener un seguimiento del progreso de la sincronización. Se usa para etiquetar la rama de Git.  El valor predeterminado es `flux-sync`. |
| `--git-user`  | Nombre de usuario para la confirmación de Git. |
| `--git-email`  | Correo electrónico que se va a usar para la confirmación de Git. 

Si no quiere que Flux escriba en el repositorio, y no se han establecido `--git-user` ni `--git-email`, se establecerá `--git-readonly` automáticamente.

Para obtener más información, consulte la [documentación de Flux](https://aka.ms/FluxcdReadme).

> [!TIP]
> Puede crear una configuración en Azure Portal en la pestaña **GitOps** del recurso de Kubernetes habilitado para Azure Arc.

## <a name="validate-the-configuration"></a>Validación de la configuración

Utilice la CLI de Azure para asegurarse de que la configuración se haya creado correctamente.

```azurecli
az k8s-configuration show --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

El recurso de configuración se actualizará con el estado de cumplimiento, mensajes y la información de depuración.

```output
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2020-12-10T18:26:52.801000+00:00",
    "message": "...",
    "messageLevel": "Information"
  },
  "configurationProtectedSettings": {},
  "enableHelmOperator": false,
  "helmOperatorProperties": {
    "chartValues": "",
    "chartVersion": ""
  },
  "id": "/subscriptions/<sub id>/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
  "name": "cluster-config",
  "operatorInstanceName": "cluster-config",
  "operatorNamespace": "cluster-config",
  "operatorParams": "--git-readonly",
  "operatorScope": "cluster",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "...",
  "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git",
  "resourceGroup": "AzureArcTest",
  "sshKnownHostsContents": null,
  "systemData": {
    "createdAt": "2020-12-01T03:58:56.175674+00:00",
    "createdBy": null,
    "createdByType": null,
    "lastModifiedAt": "2020-12-10T18:30:56.881219+00:00",
    "lastModifiedBy": null,
    "lastModifiedByType": null
},
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

Cuando se crea o actualiza una configuración, se realizan varias operaciones:

1. El `config-agent` de Azure Arc supervisa Azure Resource Manager para detectar configuraciones nuevas o actualizadas (`Microsoft.KubernetesConfiguration/sourceControlConfigurations`) y observa la nueva configuración con estado `Pending`.
1. El `config-agent` lee las propiedades de configuración y crea el espacio de nombres de destino.
1. `controller-manager` de Azure Arc crea una cuenta de servicio de Kubernetes y la asigna a [ClusterRoleBinding o RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) para los permisos adecuados (ámbito `cluster` o `namespace`). Implementa luego una instancia de `flux`.
1. Si está usando la opción de SSH con claves generadas por Flux, `flux` genera una clave SSH y registra la clave pública.
1. `config-agent` notifica de nuevo el estado al recurso de configuración en Azure.

Mientras se produce el proceso de aprovisionamiento, el recurso de configuración pasará por algunos cambios de estado. Supervise el progreso con el comando `az k8s-configuration show ...` anterior:

| Cambio de fase | Descripción |
| ------------- | ------------- |
| `complianceStatus`-> `Pending` | Representa los estados inicial y en curso. |
| `complianceStatus` -> `Installed`  | `config-agent` ha configurado el clúster correctamente e implementado `flux` sin errores. |
| `complianceStatus` -> `Failed` | `config-agent` ha producido un error en la implementación de `flux`. Los detalles se proporcionan en el cuerpo de la respuesta de `complianceStatus.message`. |

## <a name="apply-configuration-from-a-private-git-repository"></a>Aplicación de la configuración desde un repositorio de Git privado

Si usa un repositorio de Git privado, tiene que configurar la clave pública SSH en el repositorio. O bien proporciona la clave pública SSH, o bien la genera Flux. Puede configurar la clave pública en el repositorio de Git específico o en el usuario de Git que tiene acceso al repositorio. 

### <a name="get-your-own-public-key"></a>Obtención de una clave pública propia

Si ha generado claves SSH propias, ya dispone de las claves públicas y privadas.

#### <a name="get-the-public-key-using-azure-cli"></a>Obtención de la clave pública mediante la CLI de Azure 

Utilice lo siguiente en la CLI de Azure si Flux genera las claves.

```console
$ az k8s-configuration show --resource-group <resource group name> --cluster-name <connected cluster name> --name <configuration name> --cluster-type connectedClusters --query 'repositoryPublicKey' 
"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAREDACTED"
```

#### <a name="get-the-public-key-from-the-azure-portal"></a>Obtención de la clave pública desde Azure Portal

Siga estos pasos en Azure Portal si Flux genera las claves.

1. En Azure Portal, vaya al recurso de clúster conectado.
2. En la página del recurso, seleccione "GitOps" y vea la lista de configuraciones de este clúster.
3. Seleccione la configuración que usa el repositorio de Git privado.
4. En la parte inferior de la ventana contextual que se abre, copie la **clave pública del repositorio**.

#### <a name="add-public-key-using-github"></a>Adición de clave pública mediante GitHub

Use una de las siguientes opciones:

* Opción 1: Agregar la clave pública a la cuenta de usuario (se aplica a todos los repositorios de la cuenta):  
    1. Abra GitHub y haga clic en el icono del perfil en la esquina superior derecha de la página.
    2. Haga clic en **Configuración**.
    3. Haga clic en **SSH and GPG keys** (Claves SSH y GPG).
    4. Haga clic en **New SSH key** (Nueva clave SSH).
    5. Proporcione un título.
    6. Pegue la clave pública sin las comillas.
    7. Haga clic en **Add SSH key** (Agregar clave SSH).

* Opción 2: Agregar la clave pública como clave de implementación al repositorio de Git (solo se aplica a este repositorio):  
    1. Abra GitHub y vaya al repositorio.
    1. Haga clic en **Configuración**.
    1. Haga clic en **Deploy keys** (Implementar claves).
    1. Haga clic en **Add deploy key** (Agregar clave de implementación).
    1. Proporcione un título.
    1. Active **Allow write access** (Permitir acceso de escritura).
    1. Pegue la clave pública sin las comillas.
    1. Haga clic en **Agregar clave**.

#### <a name="add-public-key-using-an-azure-devops-repository"></a>Adición de una clave pública mediante un repositorio de Azure DevOps

Siga estos pasos para agregar la clave a las claves SSH:

1. En **Configuración de usuario** en la parte superior derecha (junto a la imagen de perfil), haga clic en **Claves públicas de SSH**.
1. Seleccione **+ Nueva clave**.
1. Proporcione un nombre.
1. Pegue la clave pública sin las comillas.
1. Haga clic en **Agregar**.

## <a name="validate-the-kubernetes-configuration"></a>Validación de la configuración de Kubernetes

Una vez que `config-agent` ha instalado la instancia de `flux`, los recursos que se mantienen en el repositorio de Git deben empezar a fluir al clúster. Use el comando a continuación para comprobar que se hayan creado los espacios de nombres, las implementaciones y los recursos:

```console
kubectl get ns --show-labels
```

```output
NAME              STATUS   AGE    LABELS
azure-arc         Active   24h    <none>
cluster-config    Active   177m   <none>
default           Active   29h    <none>
itops             Active   177m   fluxcd.io/sync-gc-mark=sha256.9oYk8yEsRwWkR09n8eJCRNafckASgghAsUWgXWEQ9es,name=itops
kube-node-lease   Active   29h    <none>
kube-public       Active   29h    <none>
kube-system       Active   29h    <none>
team-a            Active   177m   fluxcd.io/sync-gc-mark=sha256.CS5boSi8kg_vyxfAeu7Das5harSy1i0gc2fodD7YDqA,name=team-a
team-b            Active   177m   fluxcd.io/sync-gc-mark=sha256.vF36thDIFnDDI2VEttBp5jgdxvEuaLmm7yT_cuA2UEw,name=team-b
```

Se puede ver que se han creado los espacios de nombres `team-a`, `team-b`, `itops` y `cluster-config`.

El operador `flux` se ha implementado en el espacio de nombres `cluster-config`, tal como se indica en el recurso de configuración:

```console
kubectl -n cluster-config get deploy  -o wide
```

```output
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
cluster-config   1/1     1            1           3h    flux         docker.io/fluxcd/flux:1.16.0   instanceName=cluster-config,name=flux
memcached        1/1     1            1           3h    memcached    memcached:1.5.15               name=memcached
```

## <a name="further-exploration"></a>Exploración adicional

Puede explorar los demás recursos implementados como parte del repositorio de configuración mediante:

```console
kubectl -n team-a get cm -o yaml
kubectl -n itops get all
```
## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine una configuración mediante la CLI de Azure o Azure Portal. Después de ejecutar el comando de eliminación, el recurso de configuración se eliminará de inmediato en Azure. En el plazo de 10 minutos debería darse la eliminación completa de los objetos asociados del clúster. Si la configuración está en un estado de error cuando se quita, la eliminación completa de los objetos asociados puede tardar hasta una hora.

Cuando se elimina una configuración con el ámbito `namespace`, Azure Arc no elimina el espacio de nombres para evitar interrumpir las cargas de trabajo existentes. Si es necesario, puede eliminar este espacio de nombres manualmente mediante `kubectl`.

```azurecli
az k8s-configuration delete --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

> [!NOTE]
> Los cambios en el clúster que sean el resultado de implementaciones desde el repositorio de Git sometido a seguimiento no se eliminan cuando se elimina la configuración.

## <a name="next-steps"></a>Pasos siguientes

En el siguiente tutorial aprenderá a implementar CI/CD con GitOps.
> [!div class="nextstepaction"]
> [Implementación de CI/CD con GitOps](./tutorial-gitops-ci-cd.md)
