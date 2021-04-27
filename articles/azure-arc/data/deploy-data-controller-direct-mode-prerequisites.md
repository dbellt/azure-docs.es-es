---
title: Requisitos previos | Modo de conexión directa
description: Requisitos previos para implementar el controlador de datos en modo de conexión directa.
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 03/31/2021
ms.topic: overview
ms.openlocfilehash: 940c08dc56e8b0f2217e556da1af31b44ceeda0a
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107716302"
---
# <a name="deploy-data-controller---direct-connect-mode-prerequisites"></a>Implementación del controlador de datos: modo de conexión directa (requisitos previos)

En este artículo se describe cómo prepararse para implementar un controlador de servicios de datos habilitados para Azure Arc en modo de conexión directa.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

A modo de resumen general, los requisitos previos incluyen lo siguiente:

1. Instalación de herramientas
1. Incorporación de extensiones
1. Creación de la entidad de servicio y configuración de roles para métricas
1. Conexión del clúster de Kubernetes a Azure mediante Kubernetes habilitado para Azure Arc

Una vez completados estos requisitos previos, puede consultar [Implementación del controlador de datos de Azure Arc | Modo de conexión directa](deploy-data-controller-direct-mode.md).

En las demás secciones de este artículo se identifican los requisitos previos.

## <a name="install-tools"></a>Instalación de herramientas

- Helm 3.3+[(instalar](https://helm.sh/docs/intro/install/))
- CLI de Azure ([instalar](/sql/azdata/install/deploy-install-azdata))

## <a name="add-extensions-for-azure-cli"></a>Agregar extensiones para la CLI de Azure

También se necesitan las siguientes extensiones:
- Extensión `k8s-extension` de la CLI de Azure (0.2.0)
- CLI de Azure `customlocation` (0.1.0)

El comando `az` de ejemplo y sus extensiones de CLI sería:

```console
$ az version
{
  "azure-cli": "2.19.1",
  "azure-cli-core": "2.19.1",
  "azure-cli-telemetry": "1.0.6",
  "extensions": {
    "connectedk8s": "1.1.0",
    "customlocation": "0.1.0",
    "k8s-configuration": "1.0.0",
    "k8s-extension": "0.2.0"
  }
}
```

## <a name="create-service-principal-and-configure-roles-for-metrics"></a>Creación de una entidad de servicio y configuración de roles para métricas

Siga los pasos detallados en el artículo sobre la [carga de métricas](upload-metrics-and-logs-to-azure-monitor.md), cree una entidad de servicio y conceda los roles tal como se describe en el artículo. 

La información de ClientID, TenantID y secreto de cliente de SPN será necesaria para [implementar el controlador de datos de Azure Arc](deploy-data-controller-direct-mode.md). 

## <a name="connect-kubernetes-cluster-to-azure-using-azure-arc-enabled-kubernetes"></a>Conexión del clúster de Kubernetes a Azure mediante Kubernetes habilitado para Azure Arc

Para completar esta tarea, siga los pasos descritos en [Conexión de un clúster de Kubernetes existente a Azure Arc](../kubernetes/quickstart-connect-cluster.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez completados estos requisitos previos, puede consultar [Implementación del controlador de datos de Azure Arc | Modo de conexión directa](deploy-data-controller-direct-mode.md).
