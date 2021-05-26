---
title: Creación y administración de ubicaciones personalizadas en Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 05/25/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
ms.custom: references_regions, devx-track-azurecli
description: Uso de ubicaciones personalizadas para implementar servicios de PaaS de Azure en clústeres de Kubernetes habilitado para Azure Arc
ms.openlocfilehash: 15309599b12b10344b59d46c47c11dfa243726db
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367192"
---
# <a name="create-and-manage-custom-locations-on-azure-arc-enabled-kubernetes"></a>Creación y administración de ubicaciones personalizadas en Kubernetes habilitado para Azure Arc

Como extensión de ubicación de Azure, *Ubicaciones personalizadas* proporciona un modo para que los administradores de inquilinos usen sus clústeres de Kubernetes habilitado para Azure Arc como ubicaciones de destino para implementar instancias de servicios de Azure. Entre los ejemplos de recursos de Azure se incluyen SQL Managed Instance habilitado para Azure Arc e Hiperescala de PostgreSQL habilitada para Azure Arc.

De forma similar a las ubicaciones de Azure, los usuarios finales dentro del inquilino con acceso a Ubicaciones personalizadas pueden implementar recursos allí mediante el proceso privado de su empresa.

En este artículo aprenderá a:
> [!div class="checklist"]
> * Habilitar ubicaciones personalizadas en el clúster de Kubernetes habilitado para Azure Arc.
> * Implemente la extensión de clúster de servicio de Azure de la instancia de servicio de Azure en el clúster.
> * Cree una ubicación personalizada en el clúster de Kubernetes habilitado para Azure Arc.

Puede encontrar una información general conceptual sobre esta característica en el artículo sobre [ubicaciones de clúster: Kubernetes habilitado para Azure Arc](conceptual-custom-locations.md).

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Instale o actualice la CLI de Azure](/cli/azure/install-azure-cli) a la versión 2.16.0 o posteriores.

- Instale las extensiones de la CLI de Azure siguientes:
    - `connectedk8s` (versión 1.1.0 o posterior)
    - `k8s-extension` (versión 0.2.0 o posterior)
    - `customlocation` (versión 0.1.0 o posterior) 
  
    ```azurecli
    az extension add --name connectedk8s
    az extension add --name k8s-extension
    az extension add --name customlocation
    ```
    
    Si ya instaló previamente las extensiones `connectedk8s`, `k8s-extension` y `customlocation`, actualícelas a la versión más reciente con el siguiente comando:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-extension
    az extension update --name customlocation
    ```

- Compruebe que se completó el registro de proveedor de `Microsoft.ExtendedLocation`.
    1. Escriba los siguientes comandos:
    
    ```azurecli
    az provider register --namespace Microsoft.ExtendedLocation
    ```

    2. Supervise el proceso de registro. El registro puede tardar un máximo de 10 minutos.
    
    ```azurecli
    az provider show -n Microsoft.ExtendedLocation -o table
    ```

- Compruebe que tiene un [clúster conectado a Kubernetes habilitado para Azure Arc](quickstart-connect-cluster.md) existente.
    - [Actualice los agentes](agent-upgrade.md#manually-upgrade-agents) a la versión 1.1.0 o posterior.

>[!NOTE]
>**Regiones admitidas para ubicaciones personalizadas:**
>* Este de EE. UU.
>* Oeste de Europa

## <a name="enable-custom-locations-on-cluster"></a>Habilitación de ubicaciones personalizadas en el clúster

Para habilitar esta característica en el clúster, ejecute el siguiente comando:

```console
az connectedk8s enable-features -n <clusterName> -g <resourceGroupName> --features cluster-connect custom-locations
```

> [!NOTE]
> 1. La característica Ubicaciones personalizadas depende de la característica Conexión de clúster. Por lo tanto, es necesario habilitar ambas características para que funcionen las ubicaciones personalizadas.
> 2. `az connectedk8s enable-features` debe ejecutarse en una máquina en el que el archivo `kubeconfig` apunte al clúster en el que se van a habilitar las características.
> 3. Si ha iniciado sesión en la CLI de Azure con una entidad de servicio, se deben conceder [permisos adicionales](troubleshooting.md#enable-custom-locations-using-service-principal) a la entidad de servicio antes de habilitar la característica de ubicación personalizada.

## <a name="create-custom-location"></a>Creación de una ubicación personalizada

1. Implemente la extensión de clúster de servicio de Azure de la instancia de servicio de Azure que finalmente quiera en el clúster:

    * Azure Arc: Data Services habilitado

        ```azurecli
        az k8s-extension create --name <extensionInstanceName> --extension-type microsoft.arcdataservices --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName> --scope cluster --release-namespace arc --config Microsoft.CustomLocation.ServiceAccount=sa-bootstrapper
        ```
        > [!NOTE]
        > El proxy de salida sin autenticación y el proxy de salida con autenticación básica son compatibles con la extensión de clúster de Data Services habilitada para Arc. El proxy de salida que espera certificados de confianza no está admitido actualmente.


    * [Azure App Service en Azure Arc](../../app-service/overview-arc-integration.md)

        ```azurecli
        az k8s-extension create --name <extensionInstanceName> --extension-type 'Microsoft.Web.Appservice' --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName> --scope cluster --release-namespace appservice-ns --configuration-settings "Microsoft.CustomLocation.ServiceAccount=default" --configuration-settings "appsNamespace=appservice-ns" 
        ```

    * [Event Grid en Kubernetes](/azure/event-grid/kubernetes/overview)

        ```azurecli
          az k8s-extension create --name <extensionInstanceName> --extension-type Microsoft.EventGrid --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName> --scope cluster --release-namespace eventgrid-ext --configuration-protected-settings-file protected-settings-extension.json --configuration-settings-file settings-extension.json
        ```

1. Obtenga el identificador de Azure Resource Manager del clúster de Kubernetes habilitado para Azure Arc, al que se hace referencia en pasos posteriores como `connectedClusterId`:

    ```azurecli
    az connectedk8s show -n <clusterName> -g <resourceGroupName>  --query id -o tsv
    ```

1. Obtenga el identificador de Azure Resource Manager de la extensión de clúster implementada a partir del clúster de Kubernetes habilitado para Azure Arc, al que se hace referencia en pasos posteriores como `extensionId`:

    ```azurecli
    az k8s-extension show --name <extensionInstanceName> --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName>  --query id -o tsv
    ```

1. Cree una ubicación personalizada haciendo referencia al clúster de Kubernetes habilitado para Azure Arc y la extensión:

    ```azurecli
    az customlocation create -n <customLocationName> -g <resourceGroupName> --namespace arc --host-resource-id <connectedClusterId> --cluster-extension-ids <extensionId>
    ```

## <a name="next-steps"></a>Pasos siguientes

- Conéctese de forma segura al clúster mediante [Conexión de clúster](cluster-connect.md).
- Continúe con [Azure App Service en Azure Arc](../../app-service/overview-arc-integration.md) para obtener instrucciones de un extremo a otro sobre cómo instalar extensiones, crear ubicaciones personalizadas y crear el entorno de Kubernetes de App Service. 
- Cree un tema de Event Grid y una suscripción a evento para [Event Grid en Kubernetes](/azure/event-grid/kubernetes/overview).
- Obtenga más información sobre las [extensiones de Kubernetes habilitadas para Azure Arc](extensions.md#currently-available-extensions) disponibles actualmente.

