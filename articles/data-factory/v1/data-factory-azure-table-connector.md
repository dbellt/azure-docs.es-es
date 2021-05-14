---
title: Movimiento de datos desde o hacia tablas de Azure
description: Obtenga información acerca de cómo mover los datos hacia y desde Azure Table Storage mediante Azure Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 282ee54515998658fdc4eee07a176126ac5f5bb2
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108767030"
---
# <a name="move-data-to-and-from-azure-table-using-azure-data-factory"></a>Movimiento de datos hacia y desde Tabla de Azure mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-azure-table-connector.md)
> * [Versión 2 (versión actual)](../connector-azure-table-storage.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Azure Table Storage connector in V2](../connector-azure-table-storage.md) (Conector de Azure SQL Database en V2).

En este artículo se explica el uso de la actividad de copia en Azure Data Factory para mover datos hacia Azure Table Storage y desde este servicio. Se basa en la información general que ofrece el artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md). 

Puede copiar datos de cualquier almacén de datos de origen compatible a Azure Table Storage o de Azure Table Storage a cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="getting-started"></a>Introducción
Puede crear una canalización con una actividad de copia que mueva datos con Azure Table Storage como origen o destino mediante el uso de diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el **Asistente para copiar**. Consulte [Tutorial: Creación de una canalización mediante el Asistente para copia](data-factory-copy-data-wizard-tutorial.md) para ver un tutorial rápido sobre la creación de una canalización utilizando el Asistente para copia de datos.

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia. 

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor: 

1. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos.
2. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia. 
3. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida. 

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para obtener ejemplos con definiciones de JSON para entidades de Data Factory que se utilizan para copiar datos con Azure Table Storage como origen o destino, consulte la sección [Ejemplos de JSON](#json-examples) de este artículo.

Las secciones siguientes proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory específicas de Azure Table Storage: 

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
Hay dos tipos de servicios vinculados que puede usar para vincular un almacenamiento de blobs de Azure a una Factoría de datos de Azure. Son las siguientes: Servicio vinculado **AzureStorage** y servicio vinculado **AzureStorageSas**. El servicio vinculado de Azure Storage proporciona a la factoría de datos acceso global a Azure Storage. Mientras que el servicio vinculado de SAS (firma de acceso compartido) de Azure Storage proporciona a la factoría de datos acceso restringido/controlado por tiempo a Azure Storage. No existen otras diferencias entre estos dos servicios vinculados. Elija el servicio vinculado que se adapte a sus necesidades. En las siguientes secciones se ofrecen más detalles sobre estos dos servicios vinculados.

[!INCLUDE [data-factory-azure-storage-linked-services](includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones como structure, availability y policy del código JSON del conjunto de datos son similares para todos los tipos de conjunto de datos (SQL Azure, blob de Azure, tabla de Azure, etc.).

La sección typeProperties es diferente en cada tipo de conjunto de datos y proporciona información acerca de la ubicación de los datos en el almacén de datos. La sección **typeProperties** del conjunto de datos de tipo **AzureTable** tiene las propiedades siguientes.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de Base de datos de tablas de Azure a la que hace referencia el servicio vinculado. |Sí. Cuando se especifica un elemento tableName sin azureTableSourceQuery, se copian todos los registros de la tabla en el destino. Si también se especifica azureTableSourceQuery, los registros de la tabla que satisfacen los requisitos de la consulta se copian en el destino. |

### <a name="schema-by-data-factory"></a>Esquema de Data Factory
En los almacenes de datos sin esquemas como Tabla de Azure, el servicio Data Factory deduce el esquema de una de las maneras siguientes:

1. Si especifica la estructura de los datos mediante la propiedad **structure** en la definición del conjunto de datos, el servicio Data Factory respeta esta como la estructura del esquema. En este caso, si una fila no contiene un valor para una columna, se proporciona un valor nulo para ella.
2. Si no especifica la estructura de los datos mediante la propiedad **structure** en la definición del conjunto de datos, Data Factory deduce el esquema usando la primera fila de los datos. En este caso, si la primera fila no contiene el esquema completo, algunas columnas se pierden en el resultado de la operación de copia.

Por lo tanto, para los orígenes de datos sin esquemas, lo mejor es especificar la estructura de los datos mediante la propiedad **structure** .

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, conjuntos de datos de entrada y salida, y directivas) están disponibles para todos los tipos de actividades.

Por otra parte, las propiedades disponibles en la sección typeProperties de la actividad varían con cada tipo de actividad. Para la actividad de copia, varían en función de los tipos de orígenes y receptores.

**AzureTableSource** admite las siguientes propiedades en la sección typeProperties:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| AzureTableSourceQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta de tabla de Azure. Consulte los ejemplos en la sección siguiente. |No. Cuando se especifica un elemento tableName sin azureTableSourceQuery, se copian todos los registros de la tabla en el destino. Si también se especifica azureTableSourceQuery, los registros de la tabla que satisfacen los requisitos de la consulta se copian en el destino. |
| azureTableSourceIgnoreTableNotFound |Indica si se omite la excepción de la tabla inexistente. |TRUE<br/>FALSE |No |

### <a name="azuretablesourcequery-examples"></a>ejemplos de azureTableSourceQuery
Si la columna de la Tabla de Azure es de tipo cadena:

```JSON
azureTableSourceQuery": "$$Text.Format('PartitionKey ge \\'{0:yyyyMMddHH00_0000}\\' and PartitionKey le \\'{0:yyyyMMddHH00_9999}\\'', SliceStart)"
```

Si la columna de la Tabla de Azure es de tipo datetime:

```JSON
"azureTableSourceQuery": "$$Text.Format('DeploymentEndTime gt datetime\\'{0:yyyy-MM-ddTHH:mm:ssZ}\\' and DeploymentEndTime le datetime\\'{1:yyyy-MM-ddTHH:mm:ssZ}\\'', SliceStart, SliceEnd)"
```

**AzureTableSink** admite las siguientes propiedades en la sección typeProperties:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |Valor predeterminado de la clave de la partición que puede usar el receptor. |Valor de cadena. |No |
| azureTablePartitionKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de partición. Si no se especifica, se utiliza AzureTableDefaultPartitionKeyValue como clave de la partición. |Un nombre de columna. |No |
| azureTableRowKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de fila. Si no se especifica, use un GUID para cada fila. |Un nombre de columna. |No |
| azureTableInsertType |Modo de insertar datos en la tabla de Azure.<br/><br/>Esta propiedad controla si los valores de las filas existentes en la tabla de salida con claves de partición y de fila coincidentes se van a reemplazar o a combinar. <br/><br/>Consulte los temas [Insert or Merge Entity](/rest/api/storageservices/Insert-Or-Merge-Entity) (Insertar o combinar entidad) e [Insert or Replace Entity](/rest/api/storageservices/Insert-Or-Replace-Entity) (Insertar o remplazar entidad) para más información sobre cómo funcionan estas opciones (combinación y reemplazo). <br/><br> Esta configuración se aplica en el nivel de fila, no en el nivel de tabla, y ninguna opción elimina filas de la tabla de salida que no existan en la entrada. |merge (predeterminado)<br/>replace |No |
| writeBatchSize |Inserta datos en la tabla de Azure cuando se alcanza el valor de writeBatchSize o writeBatchTimeout. |Entero (número de filas) |No (valor predeterminado: 10000) |
| writeBatchTimeout |Inserta datos en la tabla de Azure cuando se alcanza el valor de writeBatchSize o writeBatchTimeout. |timespan<br/><br/>Ejemplo: "00:20:00" (20 minutos). |No (el valor predeterminado de intervalo de tiempo del cliente de almacenamiento es 90 segundos) |

### <a name="azuretablepartitionkeyname"></a>azureTablePartitionKeyName
Asigne una columna de origen a una columna de destino con la propiedad JSON de traductor para poder usar la columna de destino como azureTablePartitionKeyName.

En el ejemplo siguiente, la columna de origen DivisionID se asigna a la columna de destino: DivisionID.  

```JSON
"translator": {
    "type": "TabularTranslator",
    "columnMappings": "DivisionID: DivisionID, FirstName: FirstName, LastName: LastName"
}
```
El valor DivisionID se especifica como clave de partición.

```JSON
"sink": {
    "type": "AzureTableSink",
    "azureTablePartitionKeyName": "DivisionID",
    "writeBatchSize": 100,
    "writeBatchTimeout": "01:00:00"
}
```
## <a name="json-examples"></a>Ejemplos de JSON
En los siguientes ejemplos, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). En ellos se muestra cómo copiar datos entre Azure Table Storage y Azure Blob Storage. En cambio, los datos pueden copiarse **directamente** desde cualquiera de los orígenes a cualquiera de los receptores admitidos. Para más información, vea la sección "Almacenes de datos y formatos que se admiten" del artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

## <a name="example-copy-data-from-azure-table-to-azure-blob"></a>Ejemplo: Copia de datos de una tabla de Azure a un blob de Azure
El ejemplo siguiente muestra:

1. Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) (usado para tabla y blob).
2. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureTable](#dataset-properties).
3. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. La [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa AzureTableSource y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

El ejemplo copia los datos que pertenecen a la partición predeterminada de una tabla de Azure a un blob cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado de Almacenamiento de Azure:**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory admite dos tipos de servicios vinculados de Azure Storage: **AzureStorage** y **AzureStorageSas**. En el primer caso, especifique la cadena de conexión que incluye la clave de cuenta. En el segundo, especifique el Uri de firma de acceso compartido (SAS). Para más información, consulte la sección [Servicios vinculados](#linked-service-properties).  

**Conjunto de datos de entrada de tabla de Azure:**

El ejemplo se supone que ha creado una tabla "MyTable" en la tabla de Azure.

Si se establece "external": "true", se informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y que no lo genera ninguna actividad de la factoría de datos.

```JSON
{
  "name": "AzureTableInput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Conjunto de datos de salida de blob de Azure:**

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Actividad de copia en una canalización con AzureTableSource y BlobSink:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **AzureTableSource** y el tipo **sink**, en **BlobSink**. La consulta SQL especificada con la propiedad **AzureTableSourceQuery** selecciona los datos de la partición predeterminada cada hora, que se van a copiar.

```JSON
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureTabletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureTableInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureTableSource",
                        "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

## <a name="example-copy-data-from-azure-blob-to-azure-table"></a>Ejemplo: Copia de datos de un blob de Azure a una tabla de Azure
El ejemplo siguiente muestra:

1. Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) (usado para tabla y blob)
2. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
3. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureTable](#dataset-properties).
4. Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) y [AzureTableSink](#copy-activity-properties).

En el ejemplo se copian datos de series temporales de un blob de Azure a una tabla de Azure cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado de Azure Storage (para tabla y blob de Azure):**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

Azure Data Factory admite dos tipos de servicios vinculados de Azure Storage: **AzureStorage** y **AzureStorageSas**. En el primer caso, especifique la cadena de conexión que incluye la clave de cuenta. En el segundo, especifique el Uri de firma de acceso compartido (SAS). Para más información, consulte la sección [Servicios vinculados](#linked-service-properties).

**Conjunto de datos de entrada de blob de Azure:**

Los datos se seleccionan de un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa la parte year, month y day de la hora de inicio y el nombre de archivo, la parte hour. El valor "external": "true" informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Conjunto de datos de salida de tabla de Azure:**

El ejemplo copia los datos a una tabla denominada "MyTable" en la tabla de Azure. Cree una tabla de Azure con el mismo número de columnas que espera que contenga el archivo CSV de blob. Se agregan nuevas filas a la tabla cada hora.

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Actividad de copia en una canalización con BlobSource y AzureTableSink:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **BlobSource** y el tipo **sink**, en **AzureTableSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoTable",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureTableOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "AzureTableSink",
            "writeBatchSize": 100,
            "writeBatchTimeout": "01:00:00"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
## <a name="type-mapping-for-azure-table"></a>Asignación de tipos para tabla de Azure
Como se mencionó en el artículo sobre [actividades del movimiento de datos](data-factory-data-movement-activities.md) , la actividad de copia realiza conversiones automáticas de los tipos de origen a los tipos de receptor con el siguiente enfoque de dos pasos.

1. Conversión de tipos de origen nativos al tipo .NET
2. Conversión de tipo .NET al tipo del receptor nativo

Al mover datos a y desde Azure Table, se usan las siguientes [asignaciones definidas por Azure Table service](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) desde tipos OData de Azure Table a tipos .NET y viceversa.

| Tipo de datos OData | Tipo .NET | Detalles |
| --- | --- | --- |
| Edm.Binary |byte[] |Matriz de bytes de hasta 64 KB. |
| Edm.Boolean |bool |Valor booleano. |
| Edm.DateTime |DateTime |Valor de 64 bits expresado como hora universal coordinada (UTC). El intervalo admitido de DateTime comienza a las 12:00 de la noche del 1 de enero de 1601 D.C. (E.C.), UTC. El intervalo finaliza el 31 de diciembre de 9999. |
| Edm.Double |double |Valor de punto flotante de 64 bits. |
| Edm.Guid |Guid |Identificador único global de 128 bits. |
| Edm.Int32 |Int32 |Entero de 32 bits. |
| Edm.Int64 |Int64 |Entero de 64 bits. |
| Edm.String |String |Valor codificado mediante UTF-16. Los valores de cadena pueden tener hasta 64 KB. |

### <a name="type-conversion-sample"></a>Ejemplo de conversión de tipo
El siguiente es un ejemplo de la copia de datos desde un blob de Azure a una tabla de Azure con conversiones de tipo.

Supongamos que el conjunto de datos de Blob está en formato CSV y contiene tres columnas. Una de ellas es una columna de fecha y hora con un formato de fecha y hora personalizado mediante los nombres abreviados de los días de la semana en francés.

Defina el conjunto de datos de origen de Blob como se indica a continuación, junto con las definiciones de tipo para las columnas.

```JSON
{
    "name": " AzureBlobInput",
    "properties":
    {
        "structure":
        [
            { "name": "userid", "type": "Int64"},
            { "name": "name", "type": "String"},
            { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
        ],
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
            "fileName":"myfile.csv",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "external": true,
        "availability":
        {
            "frequency": "Hour",
            "interval": 1,
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
Teniendo en cuenta la asignación del tipo OData de Azure Table al tipo .NET anterior, definiría la tabla en Azure Table con el siguiente esquema.

**Esquema de tabla de Azure:**

| Nombre de la columna | Tipo |
| --- | --- |
| userid |Edm.Int64 |
| name |Edm.String |
| lastlogindate |Edm.DateTime |

A continuación, defina el conjunto de datos de Azure Table de la manera siguiente. No es necesario especificar la sección "structure" con la información de tipo porque ya se especificó en el almacén de datos subyacente.

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

En este caso, Data Factory realiza automáticamente conversiones de tipos, incluido el campo Datetime con el formato personalizado de fecha y hora, siguiendo la referencia cultural fr-fr al mover datos de Blob a Azure Table.

> [!NOTE]
> Para asignar columnas del conjunto de datos de origen a columnas del conjunto de datos del receptor, consulte [Mapping dataset columns in Azure Data Factory](data-factory-map-columns.md) (Asignación de columnas de conjunto de datos en Azure Data Factory).

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.