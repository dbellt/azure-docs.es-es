---
title: Copia de datos con Oracle como origen o destino mediante Data Factory
description: Aprenda a copiar datos desde o a una base de datos de Oracle local mediante Azure Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 02fc142a08176aa577250417c0e394218e832f34
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100387349"
---
# <a name="copy-data-to-or-from-oracle-on-premises-by-using-azure-data-factory"></a>Copia de datos con una instancia local de Oracle como origen o destino mediante Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-onprem-oracle-connector.md)
> * [Versión 2 (versión actual)](../connector-oracle.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Azure Data Factory. Si usa la versión actual del servicio Azure Data Factory, consulte [Oracle connector in V2](../connector-oracle.md) (Conector Oracle en V2).


En este artículo se explica el uso de la actividad de copia en Azure Data Factory para mover datos con una base de datos de Oracle local como origen o destino. Este artículo se basa en [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md), que presenta información general del movimiento de datos con la actividad de copia.

## <a name="supported-scenarios"></a>Escenarios admitidos

Puede copiar datos *de una base de datos de Oracle* a los siguientes almacenes de datos:

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Puede copiar datos de los siguientes almacenes de datos *a una base de datos de Oracle*:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="prerequisites"></a>Prerrequisitos

Data Factory admite la conexión a orígenes de Oracle locales mediante la puerta de enlace de administración de datos. Consulte [Data Management Gateway](data-factory-data-management-gateway.md) para obtener más información acerca de Data Management Gateway. Para obtener instrucciones paso a paso sobre cómo configurar la puerta de enlace en una canalización de datos para mover datos, consulte [Traslado de datos desde local a la nube](data-factory-move-data-between-onprem-and-cloud.md).

La puerta de enlace es necesaria incluso si Oracle está hospedado en una máquina virtual de infraestructura como servicio (IaaS) de Azure. Puede instalar la puerta de enlace en la misma máquina virtual de IaaS como almacén de datos o en una máquina virtual diferente, siempre y cuando la puerta de enlace se pueda conectar a la base de datos.

> [!NOTE]
> Para obtener sugerencias para solucionar problemas de conexión o puerta de enlace, consulte [Solución de problemas de la puerta de enlace](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).

## <a name="supported-versions-and-installation"></a>Versiones compatibles e instalación

Este conector de Oracle admite dos versiones de controladores:

- **Controlador de Microsoft para Oracle (recomendado)**: a partir de Data Management Gateway versión 2.7, se instala automáticamente un controlador de Microsoft para Oracle con la puerta de enlace. No es necesario instalar ni actualizar el controlador para establecer la conectividad con Oracle. También puede experimentar un mejor rendimiento de la copia mediante el uso de este controlador. Se admiten las siguientes versiones de bases de datos de Oracle:
  - Oracle 12c R1 (12.1)
  - Oracle 11g R1, R2 (11.1, 11.2)
  - Oracle 10g R1, R2 (10.1, 10.2)
  - Oracle 9i R1, R2 (9.0.1, 9.2)
  - Oracle 8i R3 (8.1.7)

    > [!NOTE]
    > El servidor proxy de Oracle no se admite.

    > [!IMPORTANT]
    > Actualmente, el controlador de Microsoft para Oracle solo permite la copia de datos desde Oracle. El controlador no admite la escritura en Oracle. La funcionalidad de conexión de prueba de la pestaña **Diagnósticos de Data Management Gateway** no admite este controlador. Como alternativa, puede usar al Asistente para copiar para validar la conectividad.
    >

- **Proveedor de datos de Oracle para .NET:** es posible usar el proveedor de datos de Oracle para copiar datos desde Oracle o en Oracle. Este componente se incluye en [Oracle Data Access Components for Windows](https://www.oracle.com/technetwork/topics/dotnet/downloads/) (Componentes de acceso a datos Oracle para Windows). Instale la versión adecuada (32 o 64 bits) en la máquina en la que está instalada la puerta de enlace. [Proveedor de datos de Oracle para NET 12.1](https://docs.oracle.com/database/121/ODPNT/InstallSystemRequirements.htm#ODPNT149) puede tener acceso a bases de datos Oracle 10g Release 2 o versiones posteriores.

    Si selecciona **Instalación de XCopy**, complete los pasos que se describen en el archivo readme.htm. Se recomienda seleccionar el instalador que tiene interfaz de usuario (no el instalador de XCopy).

    Después de instalar el proveedor, reinicie el servicio de host de la puerta de enlace de administración de datos en la máquina con el applet Servicios o el Administrador de configuración de la puerta de enlace de administración de datos.

Si utiliza el asistente para copia para crear la canalización de copia, el tipo de controlador se determina automáticamente. El controlador de Microsoft se usa de forma predeterminada, a menos que la versión de la puerta de enlace sea anterior a la versión 2.7 o seleccione Oracle como receptor.

## <a name="get-started"></a>Introducción

Puede crear una canalización con una actividad de copia. La canalización mueve los datos a o desde una base de datos de Oracle local mediante el uso de diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el Asistente para copiar. Consulte [Tutorial: Creación de una canalización con la actividad de copia mediante el Asistente para copia de Data Factory](data-factory-copy-data-wizard-tutorial.md) para ver un tutorial rápido sobre la creación de una canalización utilizando el Asistente para copia de datos.

También puede usar una de las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** o la **API REST**. Consulte el [tutorial de la actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso sobre cómo crear una canalización que tenga una actividad de copia.

Tanto si usa las herramientas como las API, complete los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Crear una **factoría de datos**. Una factoría de datos puede contener una o más canalizaciones.
2. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos. Por ejemplo, si copia datos de una base de datos de Oracle a Azure Blob Storage, crea dos servicios vinculados para vincular la base de datos de Oracle y la cuenta de Azure Storage a su factoría de datos. Para conocer las propiedades de los servicios vinculados que son específicas de Oracle, consulte [Propiedades del servicio vinculado](#linked-service-properties).
3. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia. En el ejemplo mencionado en el paso anterior, cree un conjunto de datos para especificar la tabla de la base de datos de Oracle que contiene los datos de entrada. Cree también otro conjunto de datos para especificar el contenedor de blobs y la carpeta que contiene los datos copiados de la base de datos de Oracle. Para conocer las propiedades del conjunto de datos que son específicas de Oracle, consulte [Propiedades del conjunto de datos](#dataset-properties).
4. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida. En el ejemplo anterior, se usa **OracleSource** como origen y **BlobSink** como receptor para la actividad de copia. De igual forma, si realiza la copia de Azure Blob Storage a una base de datos de Oracle, usa **BlobSource** y **OracleSink** en la actividad de copia. Para conocer las propiedades de la actividad de copia que son específicas de la base de datos de Oracle, consulte [Propiedades de la actividad de copia](#copy-activity-properties). Para obtener más información sobre cómo usar un almacén de datos como origen o receptor, seleccione el vínculo para el almacén de datos en la sección anterior.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory: servicios vinculados, conjuntos de datos y la canalización. Al usar herramientas o API (excepto para la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para ver ejemplos con definiciones de JSON de entidades de Data Factory que se usan para copiar datos a una base de datos de Oracle local o desde ella, consulte Ejemplos de JSON.

Las secciones siguientes proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

En la tabla siguiente se describen los elementos JSON específicos del servicio vinculado de Oracle:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad **type** debe establecerse en: **OnPremisesOracle**. |Sí |
| driverType | Especifique qué controlador usar para copiar datos en bases de datos de Oracle o desde ellas. Los valores permitidos son **Microsoft** y **ODP** (valor predeterminado). Consulte [Versiones compatibles e instalación](#supported-versions-and-installation) para obtener información detallada sobre los controladores. | No |
| connectionString | Especifique la información necesaria para conectarse a la instancia de la base de datos de Oracle para la propiedad **connectionString**. | Sí |
| gatewayName | Nombre de la puerta de enlace que se usa para conectarse al servidor local de Oracle. |Sí |

**Ejemplo: Uso del controlador de Microsoft**

> [!TIP]
> Si aparece un error que dice "ORA-01025: parámetro UPI fuera del intervalo" y tiene la versión 8i de Oracle, agregue `WireProtocolMode=1` a la cadena de conexión y vuelva a intentarlo:

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Ejemplo: uso del controlador de ODP**

Para obtener información acerca de los formatos permitidos, consulte [Proveedor de datos de Oracle para .NET](https://www.connectionstrings.com/oracle-data-provider-for-net-odp-net/).

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>))); User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Creación de conjuntos de datos](data-factory-create-datasets.md).

Las secciones del archivo JSON de un conjunto de datos, como structure, availability y policy, son similares para todos los tipos de datos (por ejemplo, para Oracle, Azure Blob Storage y almacenamiento de tablas de Azure).

La sección **typeProperties** es diferente en cada tipo de conjunto de datos y proporciona información acerca de la ubicación de los datos en el almacén de datos. La sección **typeProperties** del conjunto de datos de tipo **OracleTable** tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |El nombre de la tabla de la base de datos Oracle a la que hace referencia el servicio vinculado. |No (si se especifica **oracleReaderQuery** o **OracleSource**) |

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Creación de canalizaciones](data-factory-create-pipelines.md).

Las propiedades como nombre, descripción, tablas de entrada y salida, y directivas están disponibles para todos los tipos de actividades.

> [!NOTE]
> La actividad de copia toma solo una entrada y genera una única salida.

Las propiedades que están disponibles en la sección **typeProperties** de la actividad varían según cada tipo de actividad. Las propiedades de la actividad de copia varían en función de los tipos de orígenes y receptores.

### <a name="oraclesource"></a>OracleSource

En la actividad de copia, si el origen es de tipo **OracleSource**, están disponibles las propiedades siguientes en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| oracleReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: "select \* from **MyTable**". <br/><br/>Si no se especifica, esta instrucción SQL se ejecuta: "select \* from **MyTable**" |No<br />(si se especifica **tableName** de **dataset**) |

### <a name="oraclesink"></a>OracleSink

**OracleSink** admite las siguientes propiedades:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |**timespan**<br/><br/> Ejemplo: 00:30:00 (30 minutos) |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcanza el valor de **writeBatchSize**. |Entero (número de filas) |No (valor predeterminado: 100) |
| sqlWriterCleanupScript |Especifica una consulta para que se ejecute la actividad de copia para que se limpien los datos de un segmento específico. |Una instrucción de consulta. |No |
| sliceIdentifierColumnName |Especifica el nombre de columna para que la actividad de copia rellene un identificador de segmento generado automáticamente. El valor de **sliceIdentifierColumnName** se usa para borrar datos de un determinado segmento al volver a ejecutar. |Nombre de una columna con el tipo de datos **binary(32)**. |No |

## <a name="json-examples-for-copying-data-to-and-from-the-oracle-database"></a>Ejemplos de JSON para copiar datos a la base de datos de Oracle y desde esta

En los siguientes ejemplos, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Los ejemplos muestran cómo copiar datos desde una base de datos de Oracle a Azure Blob Storage y viceversa. Sin embargo, los datos se pueden copiar en cualquiera de los receptores indicados en [Formatos y almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

**Ejemplo: Copia de datos de Oracle a Azure Blob Storage**

El ejemplo consta de las siguientes entidades de Data Factory:

* Un servicio vinculado de tipo [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [OracleSource](data-factory-onprem-oracle-connector.md#copy-activity-properties) como origen y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) como receptor.

En el ejemplo de copian datos de una tabla de una base de datos de Oracle local en un blob cada hora. Para obtener más información acerca de las diferentes propiedades que se usan en el ejemplo, vea las secciones posteriores a los ejemplos.

**Servicio vinculado de Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Servicio vinculado de almacenamiento de blobs de Azure**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Conjunto de datos de entrada de Oracle**

En el ejemplo se supone que ha creado una tabla denominada **MyTable** en Oracle. Contiene una columna denominada **timestampcolumn** para datos de series temporales.

Si se establece **external**: **true**, se informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y de que no lo genera ninguna actividad de la factoría de datos.

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2014-02-27T12:00:00",
            "frequency": "Hour"
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

**Conjunto de datos de salida de blob de Azure**

Los datos se escriben en un nuevo blob cada hora (**frecuencia**: **hora**, **intervalo**: **1**). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```json
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

**Canalización con actividad de copia**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **OracleSource** y el tipo **sink**, en **BlobSink**. La consulta SQL que especifica con la propiedad **oracleReaderQuery** selecciona los datos de la última hora que se van a copiar.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for a copy activity",
        "activities":[
            {
                "name": "OracletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": " OracleInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "OracleSource",
                        "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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

**Ejemplo: Copia de datos desde una instancia de Azure Blob Storage a Oracle**

En este ejemplo se muestra cómo se copian datos de una cuenta de Azure Blob Storage a una base de datos Oracle local. Sin embargo, puede copiar datos *directamente* desde cualquiera de los orígenes enumerados en [Almacenes de datos y formatos que se admiten](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia de Azure Data Factory.

El ejemplo consta de las siguientes entidades de Data Factory:

* Un servicio vinculado de tipo [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* Una [canalización](data-factory-create-pipelines.md) con una actividad de copia que usa [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) como origen y [OracleSink](data-factory-onprem-oracle-connector.md#copy-activity-properties) como receptor.

El ejemplo copia datos de un blob a una tabla de una base de datos de Oracle local cada hora. Para obtener más información acerca de las diferentes propiedades que se usan en el ejemplo, vea las secciones posteriores a los ejemplos.

**Servicio vinculado de Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>)));
            User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Servicio vinculado de almacenamiento de blobs de Azure**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Conjunto de datos de entrada de blob de Azure:**

Los datos se seleccionan de un nuevo blob cada hora (**frecuencia**: **hora**, **intervalo**: **1**). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso a la carpeta usa la parte de año, mes y día de la hora de inicio. El nombre de archivo usa la parte de hora de la hora de inicio. Si se establece **external**: **true**, se informa al servicio Data Factory de que la tabla es externa a la factoría de datos y no la produce ninguna actividad de dicha factoría.

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
            "frequency": "Day",
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

**Conjunto de datos de salida de Oracle**

En el ejemplo se supone que ha creado una tabla denominada **MyTable** en Oracle. Cree la tabla en Oracle con el mismo número de columnas que espera que contenga el archivo CSV de Blob. Se agregan nuevas filas a la tabla cada hora.

```json
{
    "name": "OracleOutput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Day",
            "interval": "1"
        }
    }
}
```

**Canalización con actividad de copia**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **BlobSource** y el tipo **sink**, en **OracleSink**.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-05T19:00:00",
        "description":"pipeline with a copy activity",
        "activities":[
            {
                "name": "AzureBlobtoOracle",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "OracleOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "OracleSink"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
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


## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

### <a name="problem-1-net-framework-data-provider"></a>Problema 1: proveedor de datos .NET Framework

**Mensaje de error**

```text
Copy activity met invalid parameters: 'UnknownParameterName', Detailed message: Unable to find the requested .NET Framework Data Provider. It may not be installed.
```

**Causas posibles:**

* No se instaló el proveedor de datos de .NET Framework para Oracle.
* El proveedor de datos de .NET Framework para Oracle se instaló en .NET Framework 2.0 y no se encuentra en las carpetas de .NET Framework 4.0.

**Resolución**

* Si no ha instalado el proveedor de .NET para Oracle, [instálelo](https://www.oracle.com/technetwork/topics/dotnet/downloads/) y, después, intente de nuevo este escenario.
* Si ve el mensaje de error incluso después de instalar el proveedor, lleve a cabo los siguientes pasos:
    1. Abra el archivo machine.config de .NET 2.0 desde la carpeta: <system disk\>:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG\machine.config.
    2. Busque **Proveedor de datos de Oracle para .NET**. Debería encontrar una entrada, tal como se muestra en el siguiente ejemplo, en **system.data** > **DbProviderFactories**: `<add name="Oracle Data Provider for .NET" invariant="Oracle.DataAccess.Client" description="Oracle Data Provider for .NET" type="Oracle.DataAccess.Client.OracleClientFactory, Oracle.DataAccess, Version=2.112.3.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />`
* Copie esta entrada en el archivo machine.config en la siguiente carpeta .NET 4.0: \><disco del sistema\>:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config. Después, cambie la versión a 4.xxx.x.x.
* Instale <Ruta de instalación de ODP.NET\>\11.2.0\client_1\odp.net\bin\4\Oracle.DataAccess.dll en la caché global de ensamblados (GAC) ejecutando **gacutil /i [ruta de acceso del proveedor]**.

### <a name="problem-2-datetime-formatting"></a>Problema 2: formato de fecha y hora

**Mensaje de error**

```text
Message=Operation failed in Oracle Database with the following error: 'ORA-01861: literal does not match format string'.,Source=,''Type=Oracle.DataAccess.Client.OracleException,Message=ORA-01861: literal does not match format string,Source=Oracle Data Provider for .NET,'.
```

**Resolución**

Es posible que deba ajustar la cadena de consulta en la actividad de copia en función de cómo estén configuradas las fechas en la base de datos de Oracle. A continuación se facilita un ejemplo (mediante el uso de la función **to_date**):

```console   
"oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= to_date(\\'{0:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') AND timestampcolumn < to_date(\\'{1:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') ', WindowStart, WindowEnd)"
```


## <a name="type-mapping-for-oracle"></a>Asignación de tipos para Oracle

Como se mencionó en [Actividades del movimiento de datos](data-factory-data-movement-activities.md), la actividad de copia realiza conversiones automáticas de los tipos de origen a los tipos de receptor con el siguiente enfoque de dos pasos:

1. Conversión de tipos de origen nativos al tipo .NET.
2. Conversión del tipo .NET al tipo de receptor nativo.

Al mover datos de Oracle, se usan las siguientes asignaciones del tipo de datos de Oracle al tipo de .NET, y viceversa:

| Tipo de datos de Oracle | Tipo de datos de .NET Framework |
| --- | --- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(solo compatible con Oracle 10g y versiones posteriores cuando se usa un controlador de Microsoft) |
| CHAR |String |
| CLOB |String |
| DATE |DateTime |
| FLOAT |Decimal, String (si la precisión > 28) |
| INTEGER |Decimal, String (si la precisión > 28) |
| INTERVAL YEAR TO MONTH |Int32 |
| INTERVAL DAY TO SECOND |TimeSpan |
| LONG |String |
| LONG RAW |Byte[] |
| NCHAR |String |
| NCLOB |String |
| NUMBER |Decimal, String (si la precisión > 28) |
| NVARCHAR2 |String |
| RAW |Byte[] |
| ROWID |String |
| timestamp |DateTime |
| TIMESTAMP WITH LOCAL TIME ZONE |DateTime |
| TIMESTAMP WITH TIME ZONE |DateTime |
| UNSIGNED INTEGER |Number |
| VARCHAR2 |String |
| XML |String |

> [!NOTE]
> Los tipos de datos **INTERVAL YEAR TO MONTH** e **INTERVAL DAY TO SECOND** no se admiten cuando se utiliza un controlador de Microsoft.

## <a name="map-source-to-sink-columns"></a>Asignación de columnas de origen a columnas de receptor

Para información sobre cómo asignar columnas en el conjunto de datos de origen a columnas en el conjunto de datos de receptor, consulte [Asignación de columnas de conjunto de datos en Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lectura repetible de orígenes relacionales

Cuando se copian datos desde un almacén de datos relacionales, se debe tener presente la repetibilidad para evitar resultados imprevistos. En Azure Data Factory, puede volver a ejecutar un segmento manualmente. También puede configurar una directiva de reintentos para un conjunto de datos con el fin de que un segmento se vuelva a ejecutar cuando se produce un error. Cuando se vuelve a ejecutar un segmento, manualmente o a través de una directiva de reintentos, debe asegurarse de que los mismos datos se lean sin importar el número de ejecuciones. Para más información, consulte [Lecturas repetibles desde orígenes relacionales](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Rendimiento y optimización

Vea el artículo [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para obtener información acerca de los factores claves que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory. También puede obtener información acerca de diversas formas para optimizarlo.
