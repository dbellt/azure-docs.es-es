---
title: Ubicaciones personalizadas en Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
ms.custom: references_regions
description: Uso de ubicaciones personalizadas para implementar servicios de PaaS de Azure en clústeres de Kubernetes habilitado para Azure Arc
ms.openlocfilehash: b3a0d89f0c352b8344aea68a613653eae43a41e4
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108147776"
---
# <a name="custom-locations-on-azure-arc-enabled-kubernetes"></a>Ubicaciones personalizadas en Kubernetes habilitado para Azure Arc

Como extensión de ubicación de Azure, *Ubicaciones personalizadas* proporciona un modo para que los administradores de inquilinos usen sus clústeres de Kubernetes habilitado para Azure Arc como ubicaciones de destino para implementar instancias de servicios de Azure. Entre los ejemplos de recursos de Azure se incluyen SQL Managed Instance habilitado para Azure Arc e Hiperescala de PostgreSQL habilitada para Azure Arc.

De forma similar a las ubicaciones de Azure, los usuarios finales dentro del inquilino con acceso a Ubicaciones personalizadas pueden implementar recursos allí mediante el proceso privado de su empresa.

Puede encontrar una información general conceptual sobre esta característica en el artículo sobre [ubicaciones de clúster: Kubernetes habilitado para Azure Arc](conceptual-custom-locations.md).

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Instalar o actualizar la CLI de Azure](/cli/azure/install-azure-cli) a la versión >= 2.16.0.

- Extensiones de la CLI de Azure `connectedk8s` (versión >= 1.1.0), `k8s-extension` (versión >= 0.2.0) y `customlocation` (versión >= 0.1.0). Instale estas extensiones de la CLI de Azure mediante la ejecución de los siguientes comandos:
  
    ```azurecli
    az extension add --name connectedk8s
    az extension add --name k8s-extension
    az extension add --name customlocation
    ```
    
    Si las extensiones `connectedk8s`, `k8s-extension` y `customlocation` ya están instaladas, puede actualizarlas a la versión más reciente con el siguiente comando:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-extension
    az extension update --name customlocation
    ```

- Se ha completado el registro del proveedor para `Microsoft.ExtendedLocation`.
    1. Escriba los siguientes comandos:
    
    ```azurecli
    az provider register --namespace Microsoft.ExtendedLocation
    ```

    2. Supervise el proceso de registro. El registro puede tardar un máximo de 10 minutos.
    
    ```azurecli
    az provider show -n Microsoft.ExtendedLocation -o table
    ```

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

## <a name="create-custom-location"></a>Creación de una ubicación personalizada

1. Crear un clúster de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, utilice nuestro [inicio rápido](quickstart-connect-cluster.md).
    - [Actualice los agentes a la](agent-upgrade.md#manually-upgrade-agents) versión >= 1.1.0.

1. Implemente la extensión de clúster del servicio de Azure cuya instancia quiera a partir de la ubicación personalizada:

    ```azurecli
    az k8s-extension create --name <extensionInstanceName> --extension-type microsoft.arcdataservices --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName> --scope cluster --release-namespace arc --config Microsoft.CustomLocation.ServiceAccount=sa-bootstrapper
    ```

    > [!NOTE]
    > El proxy de salida sin autenticación y el proxy de salida con autenticación básica son compatibles con la extensión de clúster de Data Services habilitada para Arc. El proxy de salida que espera certificados de confianza no está admitido actualmente.

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

> [!div class="nextstepaction"]
> Conéctese de forma segura al clúster mediante [Conexión de clúster](cluster-connect.md)