---
title: Referencia de datos de supervisión de Azure Blob Storage | Microsoft Docs
description: Referencia de registro y métricas para la supervisión de datos de Azure Blob Storage.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 10/02/2020
ms.author: normesta
ms.subservice: logs
ms.custom: subject-monitoring
ms.openlocfilehash: 8f3cf8303a0667d3479500d6c3956be3b21959c0
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111592257"
---
# <a name="azure-blob-storage-monitoring-data-reference"></a>Referencia de datos de supervisión de Azure Blob Storage

Consulte [Supervisión de Azure Storage](monitor-blob-storage.md) para más información sobre la recopilación y el análisis de datos de supervisión de Azure Storage.

## <a name="metrics"></a>Métricas

En las tablas siguientes se indican las métricas de plataforma recopiladas de Azure Storage. 

### <a name="capacity-metrics"></a>Métricas de capacidad

Los valores de las métricas de capacidad se actualizan diariamente (hasta 24 horas). El intervalo de agregación define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de agregación que se admite en las métricas de capacidad es una hora (PT1H).

Azure Storage proporciona las siguientes métricas de capacidad en Azure Monitor.

#### <a name="account-level"></a>Nivel de cuenta

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="blob-storage"></a>Blob Storage

En esta tabla se muestran [métricas de Blob Storage](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsblobservices).

| Métrica | Descripción |
| ------------------- | ----------------- |
| BlobCapacity | El total de almacenamiento de blobs que se utiliza en la cuenta de almacenamiento. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 <br/> Dimensiones: **BlobType** y **BlobTier** ([Definition](#metrics-dimensions)) |
| BlobCount    | El número de objetos BLOB almacenados en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 <br/> Dimensiones: **BlobType** y **BlobTier** ([Definition](#metrics-dimensions)) |
| BlobProvisionedSize | Cantidad de almacenamiento aprovisionado en la cuenta de Storage. Esta métrica solo se puede aplicar a las cuentas de Storage Premium. <br/><br/> Unidad: bytes <br/> Tipo de agregación: Average |
| ContainerCount    | El número de contenedores que hay en la cuenta de almacenamiento. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |
| IndexCapacity     | Cantidad de almacenamiento que usa el índice jerárquico de ADLS Gen2. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Average <br/> Ejemplo de valor: 1024 |

### <a name="transaction-metrics"></a>Métricas de transacciones

Se emiten en todas las solicitudes enviadas a una cuenta de almacenamiento de Azure Storage a Azure Monitor. En caso de que no haya actividad en la cuenta de almacenamiento, no habrá ningún dato en las métricas de transacciones del período. Todas las métricas de transacción están disponibles en el nivel de cuenta y de servicio de Blob Storage. El intervalo de agregación define el intervalo de tiempo en el que se presentan los valores de las métricas. Los intervalos de agregación compatible para todas las métricas de transacciones son PT1H y PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Storage admite las siguientes dimensiones para las métricas en Azure Monitor.

### <a name="dimensions-available-to-all-storage-services"></a>Dimensiones disponibles para todos los servicios de almacenamiento

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

### <a name="dimensions-specific-to-blob-storage"></a>Dimensiones específicas para Blob Storage

| Nombre de dimensión | Descripción |
| ------------------- | ----------------- |
| **BlobType** | Solo el tipo de blob de las métricas de Blob. Los valores admitidos son **BlockBlob**, **PageBlob** y **Azure Data Lake Storage**. Los blobs en anexos se incluyen en **BlockBlob**. |
| **BlobTier** | Azure Storage ofrece distintos niveles de acceso que le permiten almacenar datos de objeto de blob de la manera más rentable. Obtenga más información en [Nivel de blob de Azure Storage](../blobs/storage-blob-storage-tiers.md). Entre los valores admitidos se incluyen: <br/> <li>**Acceso frecuente**: nivel de acceso frecuente</li> <li>**Acceso esporádico**: nivel de acceso esporádico</li> <li>**Archivo**: nivel de archivo</li> <li>**Premium**: nivel Premium para blob en bloques</li> <li>**P4/P6/P10/P15/P20/P30/P40/P50/P60**: tipos de nivel para blob en páginas premium</li> <li>**Estándar**: tipo de nivel para blob en páginas estándar</li> <li>**Sin nivel**: Tipo de nivel para cuentas de almacenamiento de uso general v1</li> |

En el caso de las dimensiones compatibles con métricas, es preciso especificar el valor de la dimensión para ver los valores correspondientes de las métricas. Por ejemplo, si observa el valor **Transactions** en las respuestas correctas, es preciso filtrar la dimensión **ResponseType** por **Success**. Si examina el valor **BlobCount** de Blob en bloques, es preciso filtrar la dimensión **BlobType** por **BlockBlob**.

## <a name="resource-logs-preview"></a>Registros de recursos (versión preliminar)

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de la nube pública. Esta versión preliminar habilita los registros de blobs (incluido Azure Data Lake Storage Gen2), archivos, colas, tablas, cuentas de almacenamiento Premium en cuentas de almacenamiento de uso general v1 y v2. Las cuentas de almacenamiento clásico no son compatibles.

En la tabla siguiente se indican las propiedades de los registros de recursos de Azure Storage cuando se recopilan en registros de Azure Monitor o Azure Storage. Las propiedades describen la operación, el servicio y el tipo de autorización que se ha usado para realizar la operación.

### <a name="fields-that-describe-the-operation"></a>Campos que describen la operación

```json
{
    "time": "2019-02-28T19:10:21.2123117Z",
    "resourceId": "/subscriptions/12345678-2222-3333-4444-555555555555/resourceGroups/mytestrp/providers/Microsoft.Storage/storageAccounts/testaccount1/blobServices/default",
    "category": "StorageWrite",
    "operationName": "PutBlob",
    "operationVersion": "2017-04-17",
    "schemaVersion": "1.0",
    "statusCode": 201,
    "statusText": "Success",
    "durationMs": 5,
    "callerIpAddress": "192.168.0.1:11111",
    "correlationId": "ad881411-201e-004e-1c99-cfd67d000000",
    "location": "uswestcentral",
    "uri": "http://mystorageaccount.blob.core.windows.net/cont1/blobname?timeout=10"
}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Campos que describen cómo se ha autenticado la operación

```json
{
    "identity": {
        "authorization": [
            {
                "action": "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read",
                "principals": [
                    {
                        "id": "fde5ba15-4355-4223-b811-cccccccccccc",
                        "type": "User"
                    }
                ],
                "roleAssignmentId": "ecf75cb8-491c-4a25-ad6e-aaaaaaaaaaaa",
                "roleDefinitionId": "b7e6dc6d-f1e8-4753-8033-ffffffffffff"
            }
        ],
        "requester": {
            "appId": "691458b9-1327-4635-9f55-bbbbbbbbbbbb",
            "audience": "https://storage.azure.com/",
            "objectId": "fde5ba15-4355-4223-b811-cccccccccccc",
            "tenantId": "72f988bf-86f1-41af-91ab-dddddddddddd",
            "tokenIssuer": "https://sts.windows.net/72f988bf-86f1-41af-91ab-eeeeeeeeeeee/"
           },
        "type": "OAuth"
    },
}

```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Campos que describen el servicio

```json
{
    "properties": {
        "accountName": "testaccount1",
        "requestUrl": "https://testaccount1.blob.core.windows.net:443/upload?restype=container&comp=list&prefix=&delimiter=%2F&marker=&maxresults=30&include=metadata&_=1551405598426",
        "userAgentHeader": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/17.17134",
        "referrerHeader": "blob:https://ms.portal.azure.com/6f50025f-3b88-488d-b29e-3c592a31ddc9",
        "clientRequestId": "",
        "etag": "",
        "serverLatencyMs": 63,
        "serviceType": "blob",
        "operationCount": 0,
        "requestHeaderSize": 2658,
        "requestBodySize": 0,
        "responseHeaderSize": 295,
        "responseBodySize": 2018,
        "contentLengthHeader": 0,
        "requestMd5": "",
        "serverMd5": "",
        "lastModifiedTime": "",
        "conditionsUsed": "",
        "smbTreeConnectID" : "0x3",
        "smbPersistentHandleID" : "0x6003f",
        "smbVolatileHandleID" : "0xFFFFFFFF00000065",
        "smbMessageID" : "0x3b165",
        "smbCreditsConsumed" : "0x3",
        "smbCommandDetail" : "0x2000 bytes at offset 0xf2000",
        "smbFileId" : " 0x9223442405598953",
        "smbSessionID" : "0x8530280128000049",
        "smbCommandMajor" : "0x6",
        "smbCommandMinor" : "DirectoryCloseAndDelete"
    }

}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>Consulte también

- También puede consultar [Supervisión de Azure Storage](monitor-blob-storage.md) para ver una descripción de la supervisión de Azure Storage.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
