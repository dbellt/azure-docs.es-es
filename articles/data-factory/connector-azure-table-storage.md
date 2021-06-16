---
title: Copia de datos hacia y desde Azure Table Storage
description: Aprenda a copiar datos desde almacenes de origen compatibles a Azure Table Storage o desde Table Storage a almacenes de receptor compatibles mediante Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 03/17/2021
ms.openlocfilehash: 557d8755131bcc60dfe07e3d068ac37d0a5d6f44
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675130"
---
# <a name="copy-data-to-and-from-azure-table-storage-by-using-azure-data-factory"></a>Copia de datos con Azure Table Storage como origen o destino mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-azure-table-connector.md)
> * [Versión actual](connector-azure-table-storage.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos con Azure Table Storage como origen o destino. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de esta actividad.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Table Storage es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde cualquier almacén de datos de origen admitido a Table Storage. También puede copiar datos de Table Storage en cualquier almacén de datos receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector de Azure Table Storage permite copiar datos utilizando las autenticaciones de clave de cuenta y firma de acceso compartido de servicio.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporciona información acerca de las propiedades que se usan para definir entidades de Data Factory específicas de Table Storage.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

### <a name="use-an-account-key"></a>Uso de una clave de cuenta

Puede crear un servicio vinculado de Azure Storage mediante la clave de cuenta. Esto proporciona a la factoría de datos acceso global a Storage. Se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en **AzureTableStorage**. |Sí |
| connectionString | Especifique la información necesaria para conectarse a Storage para la propiedad connectionString. <br/>También puede colocar la clave de cuenta en Azure Key Vault y extraer la configuración `accountKey` de la cadena de conexión. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. |Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar los entornos Integration Runtime (autohospedado) (si el almacén de datos se encuentra en una red privada) o Azure Integration Runtime. Si no se especifica, se usará Azure Integration Runtime. |No |

>[!NOTE]
>Si estaba usando el servicio vinculado de tipo "AzureStorage", todavía se admite tal cual, pero es preferible que use este nuevo tipo "AzureTableStorage" a partir de ahora.

**Ejemplo**:

```json
{
    "name": "AzureTableStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la clave de cuenta en Azure Key Vault**

```json
{
    "name": "AzureTableStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;",
            "accountKey": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-shared-access-signature-authentication"></a>Uso de la autenticación con firma de acceso compartido

También puede crear un servicio vinculado de Storage mediante una firma de acceso compartido. Esto proporciona a la factoría de datos acceso restringido o limitado por el tiempo a todos los recursos o a algunos específicos del almacenamiento.

Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Puede utilizarla para conceder a un cliente permisos limitados para objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos. No tiene que compartir las claves de acceso de su cuenta. La firma de acceso compartido es un URI que incluye en sus parámetros de consulta toda la información necesaria para el acceso autenticado a un recurso de almacenamiento. Para obtener acceso a los recursos de almacenamiento con la firma de acceso compartido, el cliente solo tiene que pasar la firma de acceso compartido al método o constructor adecuados. Para obtener más información sobre las firmas de acceso compartido, consulte [Uso de firmas de acceso compartido (SAS): Comprender el modelo de firma de acceso compartido](../storage/common/storage-sas-overview.md).

> [!NOTE]
> Data Factory ahora admite **firmas de acceso compartido de servicio** y **firmas de acceso compartido de cuenta**. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../storage/common/storage-sas-overview.md). 

> [!TIP]
> Puede ejecutar los siguientes comandos de PowerShell para generar una firma de acceso compartido de servicio para la cuenta de almacenamiento. Reemplace los marcadores de posición y conceda el permiso necesario.
> `$context = New-AzStorageContext -StorageAccountName <accountName> -StorageAccountKey <accountKey>`
> `New-AzStorageContainerSASToken -Name <containerName> -Context $context -Permission rwdl -StartTime <startTime> -ExpiryTime <endTime> -FullUri`

Para usar la autenticación con firma de acceso compartido, se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en **AzureTableStorage**. |Sí |
| sasUri | Especifique el URI de SAS del URI de firma de acceso compartido a la tabla. <br/>Marque este campo como SecureString para almacenarlo de forma segura en Data Factory. También puede colocar el token de SAS en Azure Key Vault para aprovechar la rotación automática y quitar la parte del token. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar los entornos Azure Integration Runtime autohospedado (si el almacén de datos se encuentra en una red privada) o Azure Integration Runtime. Si no se especifica, se usará Azure Integration Runtime. |No |

>[!NOTE]
>Si estaba usando el servicio vinculado de tipo "AzureStorage", todavía se admite tal cual, pero es preferible que use este nuevo tipo "AzureTableStorage" a partir de ahora.

**Ejemplo**:

```json
{
    "name": "AzureTableStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource e.g. https://<account>.table.core.windows.net/<table>?sv=<storage version>&amp;st=<start time>&amp;se=<expire time>&amp;sr=<resource>&amp;sp=<permissions>&amp;sip=<ip range>&amp;spr=<protocol>&amp;sig=<signature>>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la clave de cuenta en Azure Key Vault**

```json
{
    "name": "AzureTableStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource without token e.g. https://<account>.table.core.windows.net/<table>>"
            },
            "sasToken": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Cuando cree un URI de firma de acceso compartido, tenga en cuenta lo siguiente:

- Establezca los permisos de lectura y escritura adecuados en los objetos, en función de cómo se utilizará el servicio vinculado (lectura, escritura, lectura y escritura) en la factoría de datos.
- Establezca la **hora de expiración** adecuadamente. Asegúrese de que el acceso a los objetos de Storage no expirará durante el período activo de la canalización.
- El URI debe crearse en el nivel correcto de la tabla en función de la necesidad.

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Azure Table.

Para copiar datos con Azure Table como origen o destino, establezca la propiedad type del conjunto de datos en **AzureTable**. Se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en **AzureTable**. |Sí |
| tableName |Nombre de la tabla en la instancia de base de datos de Table Storage a la que hace referencia el servicio vinculado. |Sí |

**Ejemplo**:

```json
{
    "name": "AzureTableDataset",
    "properties":
    {
        "type": "AzureTable",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Azure Table storage linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

### <a name="schema-by-data-factory"></a>Esquema de Data Factory

En los almacenes de datos sin esquemas como Azure Table, Data Factory deduce el esquema de una de las maneras siguientes:

* Si especifica la asignación de columnas en la actividad de copia, Data Factory usará la lista de columnas de origen para recuperar los datos. En este caso, si una fila no contiene un valor para una columna, se proporciona un valor nulo para ella.
* Si no especifica la asignación de columnas en la actividad de copia, Data Factory deduce el esquema usando la primera fila de los datos. En este caso, si la primera fila no contiene el esquema completo (por ejemplo, algunas columnas tienen un valor nulo), algunas columnas se pierden en el resultado de la operación de copia.

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen de Azure Table.

### <a name="azure-table-as-a-source-type"></a>Azure Table como tipo de origen

Si va a copiar datos desde Azure Table Storage, establezca el tipo de origen de la actividad de copia en **AzureTableSource**. En la sección **source** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia tiene que establecerse en **AzureTableSource**. |Sí |
| AzureTableSourceQuery |Utilice la consulta personalizada de Table Storage para leer los datos.<br/>La consulta de origen consiste en una asignación directa desde la opción de consulta `$filter`, compatible con Azure Table Storage. Obtenga más información sobre la sintaxis en [este documento](/rest/api/storageservices/querying-tables-and-entities#supported-query-options) y consulte los ejemplos de la siguiente [sección de azureTableSourceQuery](#azuretablesourcequery-examples). |No |
| azureTableSourceIgnoreTableNotFound |Indica si se debe permitir la excepción de la tabla no existir.<br/>Los valores válidos son **True** y **False** (valor predeterminado). |No |

### <a name="azuretablesourcequery-examples"></a>ejemplos de azureTableSourceQuery

>[!NOTE]
>Se agota el tiempo de espera de la operación de consulta de Azure Table en 30 segundos según [impone Azure Table service](/rest/api/storageservices/setting-timeouts-for-table-service-operations). Aprenda a optimizar la consulta en el artículo [Diseño de consulta](../storage/tables/table-storage-design-for-query.md).

En Azure Data Factory, si quiere filtrar los datos por una columna de tipo DateTime, consulte este ejemplo:

```json
"azureTableSourceQuery": "LastModifiedTime gt datetime'2017-10-01T00:00:00' and LastModifiedTime le datetime'2017-10-02T00:00:00'"
```

Si quiere filtrar los datos por una columna de tipo cadena, consulte este ejemplo:

```json
"azureTableSourceQuery": "LastModifiedTime ge '201710010000_0000' and LastModifiedTime le '201710010000_9999'"
```

Si usa el parámetro de canalización, convierta el valor de fecha y hora al formato apropiado de acuerdo con los ejemplos anteriores.

### <a name="azure-table-as-a-sink-type"></a>Azure Table como tipo de receptor

Para copiar datos a la Tabla de Azure, establezca el tipo de receptor de la actividad de copia en **AzureTableSink**. En la sección **sink** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en **AzureTableSink**. |Sí |
| azureTableDefaultPartitionKeyValue |Valor predeterminado de la clave de la partición que puede usar el receptor. |No |
| azureTablePartitionKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de partición. Si no se especifica, se utiliza "AzureTableDefaultPartitionKeyValue" como clave de la partición. |No |
| azureTableRowKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de fila. Si no se especifica, use un GUID para cada fila. |No |
| azureTableInsertType |Modo de insertar datos en la tabla de Azure. Esta propiedad controla si los valores de las filas existentes en la tabla de salida con claves de partición y de fila coincidentes se van a reemplazar o a combinar. <br/><br/>Los valores permitidos son: **merge** (valor predeterminado) y **replace**. <br/><br> Esta configuración aplica en el nivel de fila, no en el nivel de tabla. Ninguna opción elimina filas de la tabla de salida que no existen en la entrada. Consulte los temas [Insert or Merge Entity](/rest/api/storageservices/Insert-Or-Merge-Entity) (Insertar o combinar entidad) e [Insert or Replace Entity](/rest/api/storageservices/Insert-Or-Replace-Entity) (Insertar o remplazar entidad) para más información acerca de cómo funcionan estas opciones (combinación y reemplazo). |No |
| writeBatchSize |Inserta datos en Azure Table cuando se alcanza el valor de writeBatchSize o writeBatchTimeout.<br/>Los valores permitidos son: enteros (número de filas). |No (el valor predeterminado es 10 000) |
| writeBatchTimeout |Inserta datos en Azure Table cuando se alcanza el valor de writeBatchSize o writeBatchTimeout.<br/>Los valores permitidos son intervalos de tiempo. Un ejemplo es "00:20:00" (20 minutos). |No (el valor predeterminado es 90 segundos; el tiempo de expiración predeterminado del cliente de almacenamiento) |
| maxConcurrentConnections |Número máximo de conexiones simultáneas establecidas en el almacén de datos durante la ejecución de la actividad. Especifique un valor solo cuando quiera limitar las conexiones simultáneas.| No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToAzureTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Table output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureTableSink",
                "azureTablePartitionKeyName": "<column name>",
                "azureTableRowKeyName": "<column name>"
            }
        }
    }
]
```

### <a name="azuretablepartitionkeyname"></a>azureTablePartitionKeyName

Asigne una columna de origen a una de destino con la propiedad **"translator"** para poder usar la columna de destino como azureTablePartitionKeyName.

En el ejemplo siguiente, la columna de origen DivisionID se asigna a la columna de destino DivisionID:

```json
"translator": {
    "type": "TabularTranslator",
    "columnMappings": "DivisionID: DivisionID, FirstName: FirstName, LastName: LastName"
}
```

El valor "DivisionID" se especifica como clave de partición.

```json
"sink": {
    "type": "AzureTableSink",
    "azureTablePartitionKeyName": "DivisionID"
}
```

## <a name="data-type-mapping-for-azure-table"></a>Asignación de tipos de datos de Azure Table Storage

Al copiar datos con Azure Table Storage como origen o destino, se utilizan las siguientes asignaciones de tipos de datos de Azure Table en los tipos de datos provisionales de Data Factory. Para más información acerca de la forma en que la actividad de copia asigna el tipo de datos y el esquema de origen al receptor, consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md).

Al mover datos a y desde Azure Table, se usan las siguientes [asignaciones definidas por Azure Table](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) desde tipos OData de Azure Table a tipos .NET y viceversa.

| Tipo de datos de Azure Table Storage | Tipo de datos provisionales de Data Factory | Detalles |
|:--- |:--- |:--- |
| Edm.Binary |byte[] |Matriz de bytes de hasta 64 KB. |
| Edm.Boolean |bool |Valor booleano. |
| Edm.DateTime |DateTime |Valor de 64 bits expresado como hora universal coordinada (UTC). El intervalo admitido de DateTime comienza a la medianoche del 1 de enero de 1601 D.C. (E.C.), UTC. El intervalo finaliza el 31 de diciembre de 9999. |
| Edm.Double |double |Valor de punto flotante de 64 bits. |
| Edm.Guid |Guid |Identificador único global de 128 bits. |
| Edm.Int32 |Int32 |Entero de 32 bits. |
| Edm.Int64 |Int64 |Entero de 64 bits. |
| Edm.String |String |Valor codificado mediante UTF-16. Los valores de cadena pueden tener hasta 64 KB. |

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
