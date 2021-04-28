---
title: Uso de Conexión de clúster para conectarse a clústeres de Kubernetes habilitados para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
description: Uso de Conexión de clúster para conectarse de manera segura a clústeres de Kubernetes habilitados para Azure Arc
ms.openlocfilehash: 54a462164e4b992451cc66f8a0ec229aff27f2e1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145598"
---
# <a name="use-cluster-connect-to-connect-to-azure-arc-enabled-kubernetes-clusters"></a>Uso de Conexión de clúster para conectarse a clústeres de Kubernetes habilitados para Azure Arc

Con Conexión de clústeres, puede conectarse de manera segura a los clústeres de Kubernetes habilitados para Azure Arc sin necesidad de habilitar ningún puerto de entrada en el firewall. El acceso al `apiserver` del clúster de Kubernetes habilitado para Arc habilita los siguientes escenarios:
* Habilita la depuración interactiva y la solución de problemas.
* Proporciona acceso de clúster a los servicios de Azure para [ubicaciones personalizadas](custom-locations.md) y otros recursos creados en él.

Puede encontrar información general y algunos conceptos sobre esta característica en el artículo [Conexión de clúster: Kubernetes habilitado para Azure Arc](conceptual-cluster-connect.md).

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Requisitos previos   

- [Instale o actualice la CLI de Azure](/cli/azure/install-azure-cli) a la versión 2.16.0 o posteriores.

- Instale la extensión `connectedk8s` de la CLI de Azure, versión 1.1.0 o posteriores.

    ```azurecli
    az extension add --name connectedk8s
    ```
  
    Si ya ha instalado la extensión `connectedk8s`, actualícela a la versión más reciente:
    
    ```azurecli
    az extension update --name connectedk8s
    ```

- Un clúster conectado de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, use nuestra [guía de inicio rápido](quickstart-connect-cluster.md).
    - [Actualice los agentes](agent-upgrade.md#manually-upgrade-agents) a la versión 1.1.0 o posteriores.

- Habilite la Conexión de clúster en cualquier clúster de Kubernetes habilitado para Azure Arc; para ello, ejecute el siguiente comando en una máquina en la que el archivo `kubeconfig` apunte al clúster correspondiente:

    ```azurecli
    az connectedk8s enable-features --features cluster-connect -n <clusterName> -g <resourceGroupName>
    ```

- Habilite los siguientes puntos de conexión, además de los que se mencionan en [Conexión de un clúster de Kubernetes a Azure Arc](quickstart-connect-cluster.md#meet-network-requirements):

    | Punto de conexión | Port |
    |----------------|-------|
    |`*.servicebus.windows.net` | 443 |
    |`*.guestnotificationservice.azure.com` | 443 |

## <a name="usage"></a>Uso

Se admiten dos opciones de autenticación con la característica Conexión de clúster: 
* Azure Active Directory (Azure AD) 
* Token de cuenta de servicio

### <a name="option-1-azure-active-directory"></a>Opción 1: Uso de Azure Active Directory

1. Con el archivo `kubeconfig` que apunta al `apiserver` de su clúster de Kubernetes, cree un ClusterRoleBinding o un RoleBinding para la entidad de Azure AD (entidad de servicio o usuario) que requiere acceso:

    **Para el usuario:**
    
    ```console
    kubectl create clusterrolebinding admin-user-binding --clusterrole cluster-admin --user=<testuser>@<mytenant.onmicrosoft.com>
    ```

    **Para la aplicación de Azure AD:**

    1. Obtenga el `objectId` asociado a la aplicación de Azure AD:

        ```azurecli
        az ad sp show --id <id> --query objectId -o tsv
        ```

    1. Cree un ClusterRoleBinding o RoleBinding para la entidad de Azure AD (entidad de servicio o usuario) que necesita acceder a este clúster:
       
        ```console
        kubectl create clusterrolebinding admin-user-binding --clusterrole cluster-admin --user=<objectId>
        ```

1. Después de iniciar sesión en la CLI de Azure con la entidad de Azure AD correspondiente, obtenga el archivo `kubeconfig` de Conexión de clúster necesario para comunicarse con el clúster desde cualquier lugar (incluso desde fuera del firewall que protege al clúster):

    ```azurecli
    az connectedk8s proxy -n <cluster-name> -g <resource-group-name>
    ```

1. Use `kubectl` para enviar solicitudes al clúster:

    ```console
    kubectl get pods
    ```
    
    Ahora debería mostrarse una respuesta del clúster que contiene la lista de todos los pods en el espacio de nombres `default`.

### <a name="option-2-service-account-bearer-token"></a>Opción 2: Token de portador de la cuenta de servicio

1. Con el archivo `kubeconfig` que apunta al `apiserver` del clúster de Kubernetes, cree una cuenta de servicio en cualquier espacio de nombres (el siguiente comando la crea en el espacio de nombres predeterminado):

    ```console
    kubectl create serviceaccount admin-user
    ```

1. Cree un ClusterRoleBinding o un RoleBinding para conceder a esta [cuenta de servicio los permisos adecuados en el clúster](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-rolebinding):

    ```console
    kubectl create clusterrolebinding admin-user-binding --clusterrole cluster-admin --serviceaccount default:admin-user
    ```

1. Obtenga el token de la cuenta de servicio con los siguientes comandos.

    ```console
    SECRET_NAME=$(kubectl get serviceaccount admin-user -o jsonpath='{$.secrets[0].name}')
    ```

    ```console
    TOKEN=$(kubectl get secret ${SECRET_NAME} -o jsonpath='{$.data.token}' | base64 -d | sed $'s/$/\\\n/g')
    ```

1. Obtenga el archivo `kubeconfig` de Conexión de clúster necesaria para comunicarse con el clúster desde cualquier lugar (incluso desde fuera del firewall que protege al clúster):

    ```azurecli
    az connectedk8s proxy -n <cluster-name> -g <resource-group-name> --token $TOKEN
    ```

1. Use `kubectl` para enviar solicitudes al clúster:

    ```console
    kubectl get pods
    ```

    Ahora debería mostrarse una respuesta del clúster que contiene la lista de todos los pods en el espacio de nombres `default`.

## <a name="known-limitations"></a>Restricciones conocidas

Cuando envíe solicitudes al clúster de Kubernetes, si la entidad Azure AD utilizada forma parte de más de 200 grupos, se presentará el siguiente error, ya que se trata de una limitación conocida:

```console
You must be logged in to the server (Error:Error while retrieving group info. Error:Overage claim (users with more than 200 group membership) is currently not supported. 
```

Para eludir este error:
1. Cree una [entidad de servicio](/cli/azure/create-an-azure-service-principal-azure-cli). Es menos probable que una de estas sea miembro de más de 200 grupos.
1. [Inicie sesión](/cli/azure/create-an-azure-service-principal-azure-cli#sign-in-using-a-service-principal) en la CLI de Azure con la entidad de servicio antes de ejecutar el comando `az connectedk8s proxy`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> Configuración de [Azure AD RBAC](azure-rbac.md) en los clústeres