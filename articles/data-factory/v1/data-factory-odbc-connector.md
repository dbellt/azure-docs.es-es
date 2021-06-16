---
title: Movimiento de datos desde almacenes de datos de ODBC
description: Obtenga información sobre cómo mover datos desde almacenes de datos ODBC mediante Azure Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/19/2018
ms.author: jingwang
ms.custom: devx-track-azurepowershell
robots: noindex
ms.openlocfilehash: 43987b333279675204179a90f01f2aa668abb2c1
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665680"
---
# <a name="move-data-from-odbc-data-stores-using-azure-data-factory"></a>Movimiento de datos desde almacenes de datos ODBC mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-odbc-connector.md)
> * [Versión 2 (versión actual)](../connector-odbc.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Conector ODBC en V2](../connector-odbc.md).


En este artículo se explica el uso de la actividad de copia en Azure Data Factory para mover datos desde un almacén de datos ODBC local. Se basa en la información general que ofrece el artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

Puede copiar datos de un almacén de datos ODBC a cualquier almacén de datos receptor admitido. Consulte la tabla de [almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como receptores. Data Factory solo admite actualmente el movimiento de datos de un almacén de datos ODBC a otros almacenes de datos, pero no viceversa.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="enabling-connectivity"></a>Habilitación de la conectividad
El servicio Factoría de datos admite la conexión a orígenes ODBC locales mediante Data Management Gateway. Consulte el artículo sobre cómo [mover datos entre ubicaciones locales y la nube](data-factory-move-data-between-onprem-and-cloud.md) para obtener información acerca de Data Management Gateway, así como instrucciones paso a paso sobre cómo configurar la puerta de enlace. Use la puerta de enlace para conectar con un almacén de datos ODBC, incluso si está hospedado en una máquina virtual de IaaS de Azure.

Puede instalar la puerta de enlace en el mismo equipo local o en la máquina virtual de Azure como almacén de datos ODBC. Sin embargo, se recomienda que instale la puerta de enlace en un equipo o máquina virtual IaaS de Azure independiente para evitar la contención de recursos y mejorar el rendimiento. Al instalar la puerta de enlace en una máquina independiente, la máquina debería poder acceder a la máquina con el almacén de datos ODBC.

Aparte de Data Management Gateway, también debe instalar el controlador ODBC para el almacén de datos en la máquina de puerta de enlace.

> [!NOTE]
> Consulte [Solución de problemas de la puerta de enlace](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) para obtener sugerencias para solucionar problemas de conexión o puerta de enlace.

## <a name="getting-started"></a>Introducción
Puede crear una canalización con una actividad de copia que mueva datos desde un almacén de datos ODBC mediante diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el **Asistente para copiar**. Consulte [Tutorial: Creación de una canalización mediante el Asistente para copia](data-factory-copy-data-wizard-tutorial.md) para ver un tutorial rápido sobre la creación de una canalización utilizando el Asistente para copia de datos.

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos.
2. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia.
3. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para ver un ejemplo con definiciones de JSON para entidades de Data Factory que se emplean para copiar datos de un almacén de datos ODBC, consulte la sección [Ejemplo JSON: Copia de datos de un almacén de datos ODBC a un blob de Azure](#json-example-copy-data-from-odbc-data-store-to-azure-blob) de este artículo.

Las secciones siguientes proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory específicas de un almacén de datos ODBC:

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio vinculado de ODBC.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type debe establecerse en: **OnPremisesOdbc** |Sí |
| connectionString |La parte de la credencial de no acceso de la cadena de conexión, así como una credencial cifrada opcional. Vea ejemplos en las secciones siguientes. <br/><br/>Puede especificar la cadena de conexión con un patrón como `"Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;"` o utilizar el DSN (nombre de origen de datos) de sistema que se ha configurado en la máquina de puerta de enlace con `"DSN=<name of the DSN>;"` (se necesita especificar la parte de la credencial en el servicio vinculado según corresponda). |Sí |
| credencial |La parte de la credencial de acceso de la cadena de conexión especificada en formato de valor de propiedad específico del controlador. Ejemplo: `"Uid=<user ID>;Pwd=<password>;RefreshToken=<secret refresh token>;"`. |No |
| authenticationType |Tipo de autenticación que se usa para conectarse al almacén de datos ODBC. Los valores posibles son: Anónima y básica. |Sí |
| userName |Especifique el nombre de usuario si usa la autenticación básica. |No |
| password |Especifique la contraseña de la cuenta de usuario que se especificó para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que el servicio Factoría de datos debe usar para conectarse al almacén de datos ODBC. |Sí |

### <a name="using-basic-authentication"></a>Uso de la autenticación básica

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```
### <a name="using-basic-authentication-with-encrypted-credentials"></a>Uso de la autenticación básica con credenciales cifradas
Las credenciales se pueden cifrar mediante el cmdlet [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) (versión 1.0 de Azure PowerShell) o [New-AzureDataFactoryEncryptValue](/previous-versions/azure/dn834940(v=azure.100)) (versión 0.9 o una versión anterior de Azure PowerShell).

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=myserver.database.windows.net; Database=TestDatabase;;EncryptedCredential=eyJDb25uZWN0...........................",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-anonymous-authentication"></a>Uso de autenticación anónima

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "connectionString": "Driver={SQL Server};Server={servername}.database.windows.net; Database=TestDatabase;",
            "credential": "UID={uid};PWD={pwd}",
            "gatewayName": "mygateway"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones como structure, availability y policy del código JSON del conjunto de datos son similares para todos los tipos de conjunto de datos (SQL Azure, blob de Azure, tabla de Azure, etc.).

La sección **typeProperties** es diferente en cada tipo de conjunto de datos y proporciona información acerca de la ubicación de los datos en el almacén de datos. La sección typeProperties del conjunto de datos del tipo **RelationalTable** (que incluye el conjunto de datos ODBC) tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en el almacén de datos ODBC. |Sí |

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, tablas de entrada y salida, y directivas) están disponibles para todos los tipos de actividades.

Por otra parte, las propiedades disponibles en la sección **typeProperties** de la actividad varían con cada tipo de actividad. Para la actividad de copia, varían en función de los tipos de orígenes y receptores.

En la actividad de copia, si el origen es del tipo **RelationalSource** (que incluye ODBC), las propiedades siguientes están disponibles en la sección typeProperties:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: select * from MyTable. |Sí |


## <a name="json-example-copy-data-from-odbc-data-store-to-azure-blob"></a>Ejemplo JSON: Copia de datos de un almacén de datos ODBC a un blob de Azure
En este ejemplo se proporcionan definiciones de JSON que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). En ellos, se muestra cómo copiar datos de un origen ODBC a Azure Blob Storage. Sin embargo, los datos se pueden copiar en cualquiera de los receptores indicados [aquí](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

El ejemplo consta de las siguientes entidades de factoría de datos:

1. Un servicio vinculado del tipo [OnPremisesOdbc](#linked-service-properties).
2. Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
3. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [RelationalTable](#dataset-properties).
4. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [RelationalSource](#copy-activity-properties) y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

El ejemplo copia los datos del resultado de una consulta en un almacén de datos ODBC en un blob cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

En primer lugar, configure la puerta de enlace de administración de datos. Las instrucciones se encuentran en el artículo sobre cómo [mover datos entre ubicaciones locales y en la nube](data-factory-move-data-between-onprem-and-cloud.md) .

**Servicio vinculado de ODBC** En este ejemplo se usa la autenticación básica. Consulte la sección [Propiedades del servicio vinculado de ODBC](#linked-service-properties) para conocer los diferentes tipos de autenticación que se pueden usar.

```json
{
    "name": "OnPremOdbcLinkedService",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```

**Servicio vinculado de Azure Storage**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

**Conjunto de datos de entrada de ODBC**

El ejemplo asume que ha creado una tabla, "MyTable", en una base de datos ODBC y que contiene una columna denominada "timestampcolumn" para los datos de series temporales.

Si se establece "external": "true", se informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y que no lo genera ninguna actividad de la factoría de datos.

```json
{
    "name": "ODBCDataSet",
    "properties": {
        "published": false,
        "type": "RelationalTable",
        "linkedServiceName": "OnPremOdbcLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```json
{
    "name": "AzureBlobOdbcDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/odbc/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
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
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Actividad de copia en una canalización con origen ODBC (RelationalSource) y receptor blob (BlobSink)**

La canalización contiene una actividad de copia que está configurada para usar estos conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de la canalización JSON, el tipo **source** se establece en **RelationalSource** y el tipo **sink** se establece en **BlobSink**. La consulta SQL especificada para la propiedad **query** selecciona los datos de la última hora que se van a copiar.

```json
{
    "name": "CopyODBCToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "OdbcDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOdbcDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "OdbcToBlob"
            }
        ],
        "start": "2016-06-01T18:00:00Z",
        "end": "2016-06-01T19:00:00Z"
    }
}
```
### <a name="type-mapping-for-odbc"></a>Asignación de tipos para ODBC
Como se mencionó en el artículo sobre [actividades del movimiento de datos](data-factory-data-movement-activities.md) , la actividad de copia realiza conversiones automáticas de los tipos de origen a los tipos de receptor con el siguiente enfoque de dos pasos:

1. Conversión de tipos de origen nativos al tipo .NET
2. Conversión de tipo .NET al tipo del receptor nativo

Al mover datos desde almacenes de datos ODBC, los tipos de datos ODBC se asignan a tipos de .NET, tal y como se mencionó en el tema [Asignar tipos de datos ODBC](/dotnet/framework/data/adonet/odbc-data-type-mappings) .

## <a name="map-source-to-sink-columns"></a>Asignación de columnas de origen a columnas de receptor
Para obtener más información sobre la asignación de columnas del conjunto de datos de origen a las del conjunto de datos receptor, consulte [Asignación de columnas de conjunto de datos de Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lectura repetible de orígenes relacionales
Cuando se copian datos desde almacenes de datos relacionales, hay que tener presente la repetibilidad para evitar resultados imprevistos. En Azure Data Factory, puede volver a ejecutar un segmento manualmente. También puede configurar la directiva de reintentos para un conjunto de datos con el fin de que un segmento se vuelva a ejecutar cuando se produce un error. Cuando se vuelve a ejecutar un segmento, debe asegurarse de que los mismos datos se lean sin importar el número de ejecuciones. Consulte [Lectura repetible de orígenes relacionales](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="troubleshoot-connectivity-issues"></a>Solución de problemas de conectividad
Use la pestaña **Diagnósticos** del **Administrador de configuración de Data Management Gateway** para solucionar problemas de conexión.

1. Inicie el **Administrador de configuración de Data Management Gateway**. Puede ejecutar C:\Archivos de programa\Microsoft Data Management Gateway\1.0\Shared\ConfigManager.exe directamente, o bien buscar **Gateway** para encontrar un vínculo a la aplicación **Microsoft Data Management Gateway**, tal y como se muestra en la imagen siguiente.

    ![Buscar puerta de enlace](./media/data-factory-odbc-connector/search-gateway.png)
2. Cambie a la pestaña **Diagnósticos** .

    ![Diagnóstico de puerta de enlace](./media/data-factory-odbc-connector/data-factory-gateway-diagnostics.png)
3. Seleccione el **tipo** de almacén de datos (el servicio vinculado).
4. Especifique la **autenticación** y escriba las **credenciales**, o bien escriba la **cadena de conexión** que se usa para conectarse al almacén de datos.
5. Haga clic en **Probar conexión** para probar la conexión con el almacén de datos.

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.
