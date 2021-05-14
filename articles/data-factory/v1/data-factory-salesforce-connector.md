---
title: Movimiento de datos de Salesforce mediante el uso de Azure Data Factory
description: Aprenda a mover datos de Salesforce usando Azure Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 61ba29c656859f21c135b0466e5d48440f7a8d17
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741584"
---
# <a name="move-data-from-salesforce-by-using-azure-data-factory"></a>Movimiento de datos de Salesforce mediante el uso de Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-salesforce-connector.md)
> * [Versión 2 (versión actual)](../connector-salesforce.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el artículo sobre el [conector de Salesforce en V2](../connector-salesforce.md).

En este artículo se describe cómo puede usar la actividad de copia en Data Factory de Azure para copiar datos de Salesforce en cualquier almacén de datos que aparezca en la columna Receptores en la tabla de [orígenes y receptores admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Este artículo se basa en el artículo sobre [movimiento de datos y actividad de copia](data-factory-data-movement-activities.md) que presenta una introducción general del movimiento de datos con la actividad de copia y las combinaciones de almacén de datos admitidas.

Actualmente, Azure Data Factory solo admite mover datos de Salesforce a [almacenes de datos receptores compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats), pero no de otros almacenes de datos a Salesforce.

## <a name="supported-versions"></a>Versiones compatibles
Este conector admite las siguientes ediciones de Salesforce: Developer Edition, Professional Edition, Enterprise Edition o Unlimited Edition. Y admite la copia del dominio personalizado, producción y espacio aislado de Salesforce.

## <a name="prerequisites"></a>Prerrequisitos
* Debe estar habilitado el permiso API. Consulte [How do I enable API access in Salesforce by permission set?](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)
* Para copiar datos de Salesforce en almacenes de datos locales, tiene que tener Data Management Gateway versión 2.0 o posterior instalado en su entorno local.

## <a name="salesforce-request-limits"></a>Límites de solicitudes de Salesforce
Salesforce tiene límites para el número total de solicitudes de API y el de solicitudes de API simultáneas. Tenga en cuenta los siguientes puntos:

- Si el número de solicitudes simultáneas supera el límite, se produce la limitación y verá errores aleatorios.
- Si el número total de solicitudes supera el límite, la cuenta de Salesforce se bloqueará durante 24 horas.

También podría recibir el error "REQUEST_LIMIT_EXCEEDED" en ambos escenarios. Consulte la sección API Request Limits (Límites de solicitudes de API) del artículo [Salesforce Developer Limits](https://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf) (Límites de desarrollador de Salesforce) para más información.

## <a name="getting-started"></a>Introducción
Puede crear una canalización con actividad de copia que mueva datos desde Salesforce mediante el uso de diferentes herramientas o API.

La manera más fácil de crear una canalización es usar el **Asistente para copiar**. Consulte [Tutorial: Creación de una canalización mediante el Asistente para copia](data-factory-copy-data-wizard-tutorial.md) para ver un tutorial rápido sobre la creación de una canalización utilizando el Asistente para copia de datos.

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos.
2. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia.
3. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para ver un ejemplo con definiciones de JSON para entidades de Data Factory que se emplean para copiar datos de Salesforce, consulte la sección [Ejemplo JSON: Copia de datos de Salesforce a un blob de Azure](#json-example-copy-data-from-salesforce-to-azure-blob) de este artículo.

Las secciones siguientes proporcionan detalles sobre las propiedades JSON que se usan para definir entidades de Data Factory específicas de Salesforce:

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
En la tabla siguiente se proporciona la descripción de los elementos JSON específicos al servicio vinculado de Salesforce.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type debe establecerse en: **Salesforce**. |Sí |
| environmentUrl | Especifique la URL de la instancia de Salesforce. <br><br> - La dirección predeterminada es "https:\//login.salesforce.com". <br> - Para copiar datos desde el espacio aislado, especifique "https://test.salesforce.com". <br> - Para copiar datos del dominio personalizado, especifique, por ejemplo, "https://[dominio].my.salesforce.com". |No |
| username |Especifique el nombre de usuario de la cuenta de usuario. |Sí |
| password |Especifique la contraseña para la cuenta de usuario. |Sí |
| securityToken |Especifique el token de seguridad para la cuenta de usuario. Consulte [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtención de un token de seguridad) para ver instrucciones sobre cómo restablecer u obtener un token de seguridad. Para más información acerca de los tokens de seguridad en general, consulte [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)(Seguridad y la API). |Sí |

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo sobre [creación de conjuntos de datos](data-factory-create-datasets.md) . Las secciones como structure, availability y policy del código JSON del conjunto de datos son similares para todos los tipos de conjunto de datos (SQL Azure, blob de Azure, tabla de Azure, etc.).

La sección **typeProperties** es diferente en cada tipo de conjunto de datos y proporciona información acerca de la ubicación de los datos en el almacén de datos. La sección typeProperties del conjunto de datos del tipo **RelationalTable** tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla de Salesforce. |No (si se especifica una **consulta** de **RelationalSource**) |

> [!IMPORTANT]
> La parte "__c" del nombre de la API es necesaria para cualquier objeto personalizado.

![Captura de pantalla que muestra los detalles de la definición del objeto personalizado, donde aparecen los nombres de API de los objetos personalizados.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para obtener una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [creación de canalizaciones](data-factory-create-pipelines.md) . Propiedades como name, description, tablas input y output y varias directivas están disponibles para todos los tipos de actividades.

Las propiedades disponibles en la sección typeProperties de la actividad varían con cada tipo de actividad. Para la actividad de copia, varían en función de los tipos de orígenes y receptores.

En la actividad de copia cuando el origen es del tipo **RelationalSource** (lo que incluye Salesforce), en la sección typeProperties, están disponibles las propiedades siguientes:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Consulta de SQL-92 o de [Salesforce Object Query Language (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm). Por ejemplo: `select * from MyTable__c`. |No (si se especifica **tableName** de **dataset**) |

> [!IMPORTANT]
> La parte "__c" del nombre de la API es necesaria para cualquier objeto personalizado.

![Captura de pantalla muestra los campos personalizados y las relaciones, donde aparecen los nombres de API de los objetos personalizados.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## <a name="query-tips"></a>Sugerencias de consulta
### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>Recuperación de datos mediante la cláusula WHERE en la columna DateTime
Cuando se especifica la consulta SQL o SOQL, preste atención a la diferencia del formato de fecha y hora. Por ejemplo:

* **Ejemplo SOQL**:`$$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', WindowStart, WindowEnd)`
* **Ejemplo de SQL**:
    * **Uso del Asistente para copia para especificar la consulta:** `$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)`
    * **Uso de la edición JSON para especificar la consulta (carácter de escape adecuado):** `$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\\'{0:yyyy-MM-dd HH:mm:ss}\\'}} AND LastModifiedDate < {{ts\\'{1:yyyy-MM-dd HH:mm:ss}\\'}}', WindowStart, WindowEnd)`

### <a name="retrieving-data-from-salesforce-report"></a>Recuperación de datos de informes de Salesforce
Puede recuperar datos de informes de Salesforce especificando las consultas como `{call "<report name>"}`; por ejemplo,. `"query": "{call \"TestReport\"}"`.

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>Recuperación de los registros eliminados desde la papelera de reciclaje de Salesforce
Para consultar los registros eliminados temporalmente de papelera de reciclaje de Salesforce, puede especificar **"IsDeleted = 1"** en la consulta. Por ejemplo,

* Para consultar solo los registros eliminados, especifique "select * from MyTable__c **where IsDeleted= 1**"
* Para consultar todos los registros, tanto existentes como eliminados, especifique "select * from MyTable__c **where IsDeleted = 0 or IsDeleted = 1**"

## <a name="json-example-copy-data-from-salesforce-to-azure-blob"></a>Ejemplo JSON: Copia de datos de Salesforce a un blob de Azure
En el siguiente ejemplo, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Se muestra cómo copiar datos desde la base de datos Salesforce a Azure Blob Storage. Sin embargo, los datos se pueden copiar en cualquiera de los receptores indicados [aquí](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

Aquí tiene los artefactos de Data Factory que necesita crear para implementar el escenario. Las secciones que siguen a la lista proporcionan detalles acerca de estos pasos.

* Un servicio vinculado del tipo [Salesforce](#linked-service-properties)
* Un servicio vinculado del tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de entrada del tipo [RelationalTable](#dataset-properties)
* Un [conjunto de datos](data-factory-create-datasets.md) de salida del tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
* Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [RelationalSource](#copy-activity-properties) y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

**Servicio vinculado de Salesforce**

Este ejemplo usa el servicio vinculado de **Salesforce** . Consulte la sección sobre el [servicio vinculado de Salesforce](#linked-service-properties) para ver las propiedades que admite este servicio vinculado. Consulte [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtención de un token de seguridad) para ver instrucciones sobre cómo restablecer u obtener un token de seguridad.

```json
{
    "name": "SalesforceLinkedService",
    "properties":
    {
        "type": "Salesforce",
        "typeProperties":
        {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
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
**Conjunto de datos de entrada de Salesforce**

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"
        },
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

Si se establece **external** en **true**, se informa al servicio Data Factory que el conjunto de datos es externo a Data Factory y que no lo genera ninguna actividad de la factoría de datos.

> [!IMPORTANT]
> La parte "__c" del nombre de la API es necesaria para cualquier objeto personalizado.

![Captura de pantalla que muestra los detalles de la definición del objeto personalizado, donde aparecen la etiqueta en singular, la etiqueta en plural, el nombre de objeto y el nombre de API.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

**Conjunto de datos de salida de blob de Azure**

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1).

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/alltypes_c"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Canalización con actividad de copia:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de la canalización JSON, el tipo **source** se establece en **RelationalSource** y el tipo **sink** se establece en **BlobSink**.

Consulte [Propiedades del tipo RelationalSource](#copy-activity-properties) para obtener la lista de propiedades admitidas por RelationalSource.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2016-06-01T18:00:00",
        "end":"2016-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[
        {
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "SalesforceInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"
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
> [!IMPORTANT]
> La parte "__c" del nombre de la API es necesaria para cualquier objeto personalizado.

![Captura de pantalla que muestra los campos personalizados y relaciones con los nombres de API seleccionados.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)


### <a name="type-mapping-for-salesforce"></a>Asignación de tipos para Salesforce

| Tipo de Salesforce | Tipo basado en .NET |
| --- | --- |
| Numeración automática |String |
| Casilla de verificación |Boolean |
| Moneda |Decimal |
| Date |DateTime |
| Fecha y hora |DateTime |
| Email |String |
| Identificador |String |
| Relación de búsqueda |String |
| Lista desplegable de selección múltiple |String |
| Number |Decimal |
| Percent |Decimal |
| Teléfono |String |
| Lista desplegable |String |
| Texto |String |
| Área de texto |String |
| Área de texto (largo) |String |
| Área de texto (enriquecido) |String |
| Texto (cifrado) |String |
| URL |String |

> [!NOTE]
> Para asignar columnas del conjunto de datos de origen a columnas del conjunto de datos del receptor, consulte [Mapping dataset columns in Azure Data Factory](data-factory-map-columns.md) (Asignación de columnas de conjunto de datos en Azure Data Factory).

[!INCLUDE [data-factory-structure-for-rectangular-datasets](includes/data-factory-structure-for-rectangular-datasets.md)]

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.
