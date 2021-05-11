---
title: 'Inicio rápido: conexión de un clúster de Kubernetes existente a Azure Arc'
description: En este inicio rápido aprenderá a conectarse a un clúster de Kubernetes habilitado para Azure Arc.
author: mgoedtel
ms.author: magoedte
ms.service: azure-arc
ms.topic: quickstart
ms.date: 03/03/2021
ms.custom: template-quickstart, references_regions, devx-track-azurecli
keywords: Kubernetes, Arc, Azure, cluster
ms.openlocfilehash: de701e79677f7e862d953eb5f03534b4eaefb367
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290783"
---
# <a name="quickstart-connect-an-existing-kubernetes-cluster-to-azure-arc"></a>Inicio rápido: conexión de un clúster de Kubernetes existente a Azure Arc 

En este inicio rápido, se descubrirán las ventajas de Kubernetes habilitado para Azure Arc y se conectará un clúster de Kubernetes existente a Azure Arc. Para obtener una introducción conceptual sobre la conexión de clústeres a Azure Arc, consulte el [artículo Arquitectura de agente de Kubernetes habilitado para Azure Arc](./conceptual-agent-architecture.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

* Un clúster de Kubernetes en funcionamiento. Si no tiene ninguno, puede crear un clúster mediante una de estas opciones:
    * [Kubernetes en Docker (KIND)](https://kind.sigs.k8s.io/)
    * Creación de un clúster de Kubernetes con Docker para [Mac](https://docs.docker.com/docker-for-mac/#kubernetes) o [Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
    * Clúster de Kubernetes autoadministrado mediante la [API de clúster](https://cluster-api.sigs.k8s.io/user/quick-start.html)
    * Si desea conectar un clúster de OpenShift a Azure Arc, es preciso que ejecute el siguiente comando una sola vez en el clúster antes de ejecutar `az connectedk8s connect`:
        
        ```console
        oc adm policy add-scc-to-user privileged system:serviceaccount:azure-arc:azure-arc-kube-aad-proxy-sa
        ```

    >[!NOTE]
    > El clúster debe tener al menos un nodo de sistema operativo y el tipo de arquitectura `linux/amd64`. Los clústeres que solo tienen nodos `linux/arm64` aún no se admiten.
    
* Un archivo `kubeconfig` y un contexto que apunte al clúster.
* Permisos de lectura y escritura en el tipo de recurso de Kubernetes habilitado para Azure Arc (`Microsoft.Kubernetes/connectedClusters`).

* Instale la [versión más reciente de Helm 3](https://helm.sh/docs/intro/install).

* [Instale o actualice la CLI de Azure](/cli/azure/install-azure-cli) a la versión 2.16.0 o posteriores.
* Instale la extensión `connectedk8s` de la CLI de Azure, versión 1.0.0 o posteriores:
  
  ```azurecli
  az extension add --name connectedk8s
  ```

>[!TIP]
> Si la extensión `connectedk8s` ya está instalada, actualícela a su versión más reciente con el siguiente comando: `az extension update --name connectedk8s`

>[!NOTE]
>La lista de regiones admitidas por Kubernetes habilitado para Azure Arc se puede consultar [aquí](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc).

>[!NOTE]
> Si desea usar ubicaciones personalizadas en el clúster, use las regiones Este de EE  UU. u Oeste de Europa para conectar el clúster, ya que las ubicaciones personalizadas solo están disponibles en estas regiones por ahora. Todas las demás características de Kubernetes habilitado para Azure Arc están disponibles en todas las regiones indicadas anteriormente.

## <a name="meet-network-requirements"></a>Cumplimiento de los requisitos de red

>[!IMPORTANT]
>Los agentes de Azure Arc requieren que funcionen los siguientes protocolos, puertos y direcciones URL de salida:
>* TCP en el puerto 443: `https://:443`
>* TCP en el puerto 9418: `git://:9418`
  
| Punto de conexión (DNS) | Descripción |  
| ----------------- | ------------- |  
| `https://management.azure.com`                                                                                 | Necesario para que el agente se conecte a Azure y registre el clúster.                                                        |  
| `https://<region>.dp.kubernetesconfiguration.azure.com` | Punto de conexión de plano de datos para que el agente inserte información de la configuración de estado y recuperación de cambios.                                      |  
| `https://login.microsoftonline.com`                                                                            | Necesario para capturar y actualizar tokens de Azure Resource Manager.                                                                                    |  
| `https://mcr.microsoft.com`                                                                            | Necesario para extraer imágenes de contenedor para agentes de Azure Arc.                                                                  |  
| `https://eus.his.arc.azure.com`, `https://weu.his.arc.azure.com`, `https://wcus.his.arc.azure.com`, `https://scus.his.arc.azure.com`, `https://sea.his.arc.azure.com`, `https://uks.his.arc.azure.com`, `https://wus2.his.arc.azure.com`, `https://ae.his.arc.azure.com`, `https://eus2.his.arc.azure.com`, `https://ne.his.arc.azure.com` |  Se requiere para extraer certificados de Managed Service Identity (MSI) asignados por el sistema.                                                                  |

## <a name="register-providers-for-azure-arc-enabled-kubernetes"></a>Registro de proveedores en Kubernetes habilitado para Azure Arc

1. Escriba los siguientes comandos:
    ```azurecli
    az provider register --namespace Microsoft.Kubernetes
    az provider register --namespace Microsoft.KubernetesConfiguration
    az provider register --namespace Microsoft.ExtendedLocation
    ```
2. Supervise el proceso de registro. El registro puede tardar un máximo de 10 minutos.
    ```azurecli
    az provider show -n Microsoft.Kubernetes -o table
    az provider show -n Microsoft.KubernetesConfiguration -o table
    az provider show -n Microsoft.ExtendedLocation -o table
    ```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos:  

```console
az group create --name AzureArcTest -l EastUS -o table
```

```output
Location    Name
----------  ------------
eastus      AzureArcTest
```

## <a name="connect-an-existing-kubernetes-cluster"></a>Conexión de un clúster de Kubernetes existente

1. Para conectar un clúster de Kubernetes a Azure Arc, utilice el comando siguiente:
    ```console
    az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
    ```

    ```output
    Helm release deployment succeeded

    {
      "aadProfile": {
        "clientAppId": "",
        "serverAppId": "",
        "tenantId": ""
      },
      "agentPublicKeyCertificate": "xxxxxxxxxxxxxxxxxxx",
      "agentVersion": null,
      "connectivityStatus": "Connecting",
      "distribution": "gke",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
      "identity": {
        "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "type": "SystemAssigned"
      },
      "infrastructure": "gcp",
      "kubernetesVersion": null,
      "lastConnectivityTime": null,
      "location": "eastus",
      "managedIdentityCertificateExpirationTime": null,
      "name": "AzureArcTest1",
      "offering": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "AzureArcTest",
      "tags": {},
      "totalCoreCount": null,
      "totalNodeCount": null,
      "type": "Microsoft.Kubernetes/connectedClusters"
    }
    ```

> [!TIP]
> El comando anterior sin el parámetro location especificado crea el recurso de Kubernetes habilitado para Azure Arc en la misma ubicación que el grupo de recursos. Para crear el recurso de Kubernetes habilitado para Azure Arc en una ubicación diferente, especifique `--location <region>` o `-l <region>` al ejecutar el comando `az connectedk8s connect`.

> [!NOTE]
> Si ha iniciado sesión en la CLI de Azure mediante una entidad de servicio, se requieren [permisos adicionales](troubleshooting.md#enable-custom-locations-using-service-principal) en la entidad de servicio para habilitar la característica de ubicación personalizada al conectar el clúster a Azure Arc.

## <a name="verify-cluster-connection"></a>Comprobación de la conexión del clúster

Para ver una lista de los clústeres conectados, use el siguiente comando:  

```console
az connectedk8s list -g AzureArcTest -o table
```

```output
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

> [!NOTE]
> Después de la incorporación del clúster, los metadatos del mismo (versión del clúster, versión del agente, número de nodos, etc.) tardan entre 5 y 10 minutos en aparecer en la página de información general del recurso de Kubernetes compatible con Azure Arc en Azure Portal.

## <a name="connect-using-an-outbound-proxy-server"></a>Conexión mediante un servidor proxy de salida

Si el clúster se encuentra detrás de un servidor proxy de salida, tanto la CLI de Azure como los agentes de Kubernetes habilitado para Azure Arc deben enrutar sus solicitudes a través del servidor proxy de salida. 


1. Establezca las variables de entorno necesarias para que CLI de Azure use el servidor proxy de salida:

    * Si usa bash, ejecute el siguiente comando con los valores adecuados:

        ```bash
        export HTTP_PROXY=<proxy-server-ip-address>:<port>
        export HTTPS_PROXY=<proxy-server-ip-address>:<port>
        export NO_PROXY=<cluster-apiserver-ip-address>:<port>
        ```

    * Si usa PowerShell, ejecute el siguiente comando con los valores adecuados:

        ```powershell
        $Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"
        ```

2. Ejecute el comando connect con los parámetros de proxy especificados:

    ```console
    az connectedk8s connect -n <cluster-name> -g <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --proxy-cert <path-to-cert-file>
    ```

> [!NOTE]
> * Especifique `excludedCIDR` en `--proxy-skip-range` para asegurarse de que la comunicación interna del clúster no se interrumpe para los agentes.
> * `--proxy-http` `--proxy-https` y `--proxy-skip-range` se esperan para la mayoría de los entornos de proxy de salida. `--proxy-cert` *solo* se necesita si hay que insertar en el almacén de certificados de confianza de pods del agente certificados de confianza que espera el proxy.

## <a name="view-azure-arc-agents-for-kubernetes"></a>Visualización de agentes de Azure Arc para Kubernetes

Kubernetes habilitado para Azure Arc implementa algunos operadores en el espacio de nombres `azure-arc`. 

1. Para ver estas implementaciones y pods, utilice:

    ```console
    kubectl -n azure-arc get deployments,pods
    ```

1. Compruebe que todos los pods están en el estado `Running`.

    ```output
    NAME                                        READY      UP-TO-DATE  AVAILABLE  AGE
    deployment.apps/cluster-metadata-operator     1/1             1        1      16h
    deployment.apps/clusteridentityoperator       1/1             1        1      16h
    deployment.apps/config-agent                  1/1             1        1      16h
    deployment.apps/controller-manager            1/1             1        1      16h
    deployment.apps/flux-logs-agent               1/1             1        1      16h
    deployment.apps/metrics-agent                 1/1             1        1      16h
    deployment.apps/resource-sync-agent           1/1             1        1      16h

    NAME                                           READY    STATUS   RESTART AGE
    pod/cluster-metadata-operator-7fb54d9986-g785b  2/2     Running  0       16h
    pod/clusteridentityoperator-6d6678ffd4-tx8hr    3/3     Running  0       16h
    pod/config-agent-544c4669f9-4th92               3/3     Running  0       16h
    pod/controller-manager-fddf5c766-ftd96          3/3     Running  0       16h
    pod/flux-logs-agent-7c489f57f4-mwqqv            2/2     Running  0       16h
    pod/metrics-agent-58b765c8db-n5l7k              2/2     Running  0       16h
    pod/resource-sync-agent-5cf85976c7-522p5        3/3     Running  0       16h
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el recurso Kubernetes habilitado para Azure Arc, todos los recursos de configuración asociados *y* todos los agentes que se ejecuten en el clúster mediante la CLI de Azure, utilice el siguiente comando:

```azurecli
az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
```

>[!NOTE]
>Si se usa Azure Portal para eliminar el recurso Kubernetes habilitado para Azure Arc, se eliminan todos los recursos de configuración asociados, pero *no* se elimina ningún agente que se ejecute en el clúster. El procedimiento recomendado es eliminar el recurso Kubernetes habilitado para Azure Arc mediante `az connectedk8s delete`, en lugar de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

En el siguiente artículo aprenderá a implementar configuraciones en un clúster de Kubernetes conectado mediante GitOps.
> [!div class="nextstepaction"]
> [Implementación de configuraciones mediante GitOps](tutorial-use-gitops-connected-cluster.md)