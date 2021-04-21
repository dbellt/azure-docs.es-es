---
title: Referencia de datos de supervisión de Azure Queue Storage
description: Referencia de registros y métricas para la supervisión de datos de Azure Queue Storage.
author: normesta
services: azure-monitor
ms.author: normesta
ms.date: 04/20/2021
ms.topic: reference
ms.service: azure-monitor
ms.subservice: logs
ms.custom: subject-monitoring
ms.openlocfilehash: 506f5a46688f597b8ac5db341c5bbe5eb5fb67c8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763152"
---
# <a name="azure-queue-storage-monitoring-data-reference"></a>Referencia de datos de supervisión de Azure Queue Storage

Consulte [Supervisión de Azure Storage](monitor-queue-storage.md) para más información sobre la recopilación y el análisis de datos de supervisión de Azure Storage.

## <a name="metrics"></a>Métricas

En las tablas siguientes se indican las métricas de plataforma recopiladas de Azure Storage.

### <a name="capacity-metrics"></a>Métricas de capacidad

Los valores de las métricas de capacidad se actualizan diariamente (hasta 24 horas). El intervalo de agregación define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de agregación que se admite en las métricas de capacidad es una hora (PT1H).

Azure Storage proporciona las siguientes métricas de capacidad en Azure Monitor.

#### <a name="account-level-capacity-metrics"></a>Métricas de capacidad de nivel de cuenta

[!INCLUDE [Account-level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="queue-storage-metrics"></a>Métricas de Queue Storage

En esta tabla se muestran [métricas de Queue Storage](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsqueueservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| **QueueCapacity** | La cantidad de Queue Storage que utiliza la cuenta de almacenamiento. <br><br> Unidad: `Bytes` <br> Tipo de agregación: `Average` <br> Valor o ejemplo: `1024` |
| **QueueCount** | El número de colas que hay en la cuenta de almacenamiento. <br><br> Unidad: `Count` <br> Tipo de agregación: `Average` <br> Valor o ejemplo: `1024` |
| **QueueMessageCount** | El número de mensajes de la colas no expirados que hay en la cuenta de almacenamiento. <br><br> Unidad: `Count` <br> Tipo de agregación: `Average` <br> Valor o ejemplo: `1024` |

### <a name="transaction-metrics"></a>Métricas de transacciones

Se emiten en todas las solicitudes enviadas a una cuenta de almacenamiento de Azure Storage a Azure Monitor. En caso de que no haya actividad en la cuenta de almacenamiento, no habrá ningún dato en las métricas de transacciones del período. Todas las métricas de transacción están disponibles en el nivel de cuenta y de servicio de Queue Storage. El intervalo de agregación define el intervalo de tiempo en el que se presentan los valores de las métricas. Los intervalos de agregación compatible para todas las métricas de transacciones son PT1H y PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Storage admite las siguientes dimensiones para las métricas en Azure Monitor.

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

## <a name="resource-logs-preview"></a>Registros de recursos (versión preliminar)

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de la nube pública. Esta versión preliminar habilita los registros de blobs (incluido Azure Data Lake Storage Gen2), archivos, colas, tablas, cuentas de almacenamiento Premium en cuentas de almacenamiento de uso general v1 y v2. Las cuentas de almacenamiento clásico no son compatibles.

En la tabla siguiente se indican las propiedades de los registros de recursos de Azure Storage cuando se recopilan en registros de Azure Monitor o Azure Storage. Las propiedades describen la operación, el servicio y el tipo de autorización que se ha usado para realizar la operación.

### <a name="fields-that-describe-the-operation"></a>Campos que describen la operación

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Campos que describen cómo se ha autenticado la operación

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Campos que describen el servicio

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>Consulte también

- Consulte [Supervisión de Azure Queue Storage](monitor-queue-storage.md) para ver una descripción de la supervisión de Azure Storage.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
