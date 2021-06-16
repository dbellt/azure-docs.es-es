---
title: Uso de plantillas de Resource Manager en Data Factory
description: Aprenda a crear y usar plantillas de Azure Resource Manager para crear entidades de Data Factory.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: ec2e7d2f80e19d2e001fec0a3949f469b33149d2
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957140"
---
# <a name="use-templates-to-create-azure-data-factory-entities"></a>Uso de plantillas para crear entidades de Azure Data Factory
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. 

## <a name="overview"></a>Información general
Cuando utiliza Azure Data Factory para sus necesidades de integración de datos, es posible que tenga que reutilizar el mismo patrón en distintos entornos o implementar la misma tarea repetidamente dentro de la misma solución. Las plantillas le permiten implementar y administrar estos escenarios de una forma sencilla. Las plantillas de Azure Data Factory son ideales para los escenarios que implican la reutilización y la repetición.

Imagine una situación en la que una organización tiene 10 plantas de fabricación distribuidas por todo el mundo. Los registros de cada planta se almacenan en una base de datos de SQL Server independiente. La empresa desea crear un único almacén de datos en la nube para análisis ad hoc. También quiere tener la misma lógica pero diferentes configuraciones para los entornos de desarrollo, prueba y producción.

En este caso, se deberá repetir una tarea dentro del mismo entorno pero con valores diferentes en 10 factorías de datos para cada planta de fabricación. En efecto, la **repetición** está presente. La creación de plantillas permite la abstracción de este flujo genérico (es decir, de las canalizaciones que tengan las mismas actividades en cada factoría de datos), pero usa un archivo de parámetros distinto para cada planta de fabricación.

Además, como la organización desea implementar estas 10 factorías de datos varias veces en distintos entornos, las plantillas pueden utilizar esta característica de **reusabilidad** mediante los archivos de parámetros independientes para entornos de desarrollo, prueba y producción.

## <a name="templating-with-azure-resource-manager"></a>Creación de plantillas con Azure Resource Manager
Las [plantillas de Azure Resource Manager](../../azure-resource-manager/templates/overview.md) son una excelente manera de lograr la creación de plantillas en Azure Data Factory. Las plantillas de Resource Manager definen la infraestructura y la configuración de la solución de Azure mediante un archivo JSON. Como las plantillas de Azure Resource Manager funcionan con todos los servicios de Azure (o la mayoría de ellos), se pueden usar ampliamente para administrar fácilmente todos los recursos de Azure. Consulte [Creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md) para conocer más información acerca de estas plantillas en general.

## <a name="tutorials"></a>Tutoriales
Consulte los siguientes tutoriales para obtener instrucciones detalladas para crear entidades de Data Factory mediante plantillas de Resource Manager:

* [Tutorial: Creación de una canalización para copiar datos mediante el uso de plantillas de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
* [Tutorial: Creación de una canalización para procesar datos mediante el uso de plantillas de Azure Resource Manager](data-factory-build-your-first-pipeline.md)

## <a name="data-factory-templates-on-github"></a>Plantillas de Data Factory en GitHub
Consulte las siguientes plantillas de inicio rápido de Azure en GitHub:

* [Creación de una factoría de datos para copiar datos de Azure Blob Storage a Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-blob-to-sql-copy)
* [Create a Data factory with Hive activity on Azure HDInsight cluster](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-hive-transformation) (Creación de una instancia de Data Factory con actividad de Hive en un clúster de Azure HDInsight)
* [Create a Data factory to copy data from Salesforce to Azure Blobs](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-salesforce-to-blob-copy) (Creación de una instancia de Data Factory para copiar datos desde Salesforce en Azure Blobs)
* [Creación de una factoría de datos que encadena actividades: copiar datos desde un servidor FTP en blobs de Azure, invocar un script de Hive en un clúster de HDInsight a petición para transformar los datos y copiar el resultado en Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-data-factory-ftp-hive-blob)

No dude en compartir sus plantillas de Azure Data Factory en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/). Consulte la [Guía de contribución](https://github.com/Azure/azure-quickstart-templates/tree/master/1-CONTRIBUTION-GUIDE) cuando desarrolle plantillas que se puedan compartir a través de este repositorio.

Las secciones siguientes proporcionan detalles acerca de cómo definir los recursos de Data Factory en una plantilla de Resource Manager.

## <a name="defining-data-factory-resources-in-templates"></a>Definición de recursos de Data Factory en plantillas
La plantilla de nivel superior para definir una factoría de datos es:

```JSON
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": { ...
},
"variables": { ...
},
"resources": [
{
    "name": "[parameters('dataFactoryName')]",
    "apiVersion": "[variables('apiVersion')]",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "westus",
    "resources": [
    { "type": "linkedservices",
        ...
    },
    {"type": "datasets",
        ...
    },
    {"type": "dataPipelines",
        ...
    }
}
```

### <a name="define-data-factory"></a>Definición de factoría de datos
Puede definir una factoría de datos en la plantilla de Resource Manager como se muestra en el ejemplo siguiente:

```JSON
"resources": [
{
    "name": "[variables('<mydataFactoryName>')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "East US"
}
```
El dataFactoryName se define en "variables" como:

```JSON
"dataFactoryName": "[concat('<myDataFactoryName>', uniqueString(resourceGroup().id))]",
```

### <a name="define-linked-services"></a>Definición de servicios vinculados

```JSON
"type": "linkedservices",
"name": "[variables('<LinkedServiceName>')]",
"apiVersion": "2015-10-01",
"dependsOn": [ "[variables('<dataFactoryName>')]" ],
"properties": {
    ...
}
```

Consulte [Servicio vinculado de almacenamiento](data-factory-azure-blob-connector.md#azure-storage-linked-service) o [Servicios vinculados de procesos](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) para más información acerca de las propiedades JSON para el servicio vinculado específico que desea implementar. El parámetro "dependsOn" especifica el nombre de la factoría de datos correspondiente. En la siguiente definición de JSON, se muestra un ejemplo de definición de un servicio vinculado para Azure Storage:

### <a name="define-datasets"></a>Definición de conjuntos de datos

```JSON
"type": "datasets",
"name": "[variables('<myDatasetName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<myDatasetLinkedServiceName>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    ...
}
```
Consulte [Almacenes de datos que se admiten](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para más información acerca de las propiedades JSON para el tipo de conjunto de datos específico que desea implementar. Observe que el parámetro "dependsOn" especifica el nombre de la factoría de datos correspondiente y el servicio vinculado de almacenamiento. En la siguiente definición de JSON, se muestra un ejemplo de definición de un tipo de Azure Blob Storage:

```JSON
"type": "datasets",
"name": "[variables('storageDataset')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('storageLinkedServiceName')]"
],
"apiVersion": "2015-10-01",
"properties": {
"type": "AzureBlob",
"linkedServiceName": "[variables('storageLinkedServiceName')]",
"typeProperties": {
    "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
    "fileName": "[parameters('sourceBlobName')]",
    "format": {
        "type": "TextFormat"
    }
},
"availability": {
    "frequency": "Hour",
    "interval": 1
}
```

### <a name="define-pipelines"></a>Definición de canalizaciones

```JSON
"type": "dataPipelines",
"name": "[variables('<mypipelineName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<inputDatasetLinkedServiceName>')]",
    "[variables('<outputDatasetLinkedServiceName>')]",
    "[variables('<inputDataset>')]",
    "[variables('<outputDataset>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    activities: {
        ...
    }
}
```

Consulte [Definición de canalizaciones](data-factory-create-pipelines.md#pipeline-json) para más información acerca de las propiedades JSON para definir la canalización específica y las actividades que desea implementar. Tenga en cuenta que el parámetro "dependsOn" especifica el nombre de la factoría de datos y todos los servicios vinculados o conjuntos de datos correspondientes. En el siguiente fragmento de código JSON se muestra un ejemplo de una canalización que copia datos de Azure Blob Storage en Azure SQL Database:

```JSON
"type": "datapipelines",
"name": "[variables('pipelineName')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]",
    "[variables('azureSqlLinkedServiceName')]",
    "[variables('blobInputDatasetName')]",
    "[variables('sqlOutputDatasetName')]"
],
"apiVersion": "2015-10-01",
"properties": {
    "activities": [
    {
        "name": "CopyFromAzureBlobToAzureSQL",
        "description": "Copy data frm Azure blob to Azure SQL",
        "type": "Copy",
        "inputs": [
            {
                "name": "[variables('blobInputDatasetName')]"
            }
        ],
        "outputs": [
            {
                "name": "[variables('sqlOutputDatasetName')]"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "BlobSource"
            },
            "sink": {
                "type": "SqlSink",
                "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
            },
            "translator": {
                "type": "TabularTranslator",
                "columnMappings": "Column0:FirstName,Column1:LastName"
            }
        },
        "Policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 3,
            "timeout": "01:00:00"
        }
    }
    ],
    "start": "2016-10-03T00:00:00Z",
    "end": "2016-10-04T00:00:00Z"
}
```
## <a name="parameterizing-data-factory-template"></a>Parametrización de la plantilla de Data Factory
Para obtener los procedimientos recomendados sobre parametrización, consulte [Procedimientos recomendados para crear plantillas de Azure Resource Manager](../../azure-resource-manager/templates/best-practices.md). Por lo general, se debe minimizar el uso de parámetros, especialmente si se pueden utilizar variables en su lugar. Solo proporcione parámetros en los escenarios siguientes:

* La configuración varía según el entorno (por ejemplo: desarrollo, prueba y producción)
* Secretos (por ejemplo, las contraseñas)

Si necesita extraer los secretos de [Azure Key Vault](../../key-vault/general/overview.md) al implementar las entidades de Azure Data Factory mediante plantillas, especifique el **almacén de claves** y el **nombre secreto** tal como se muestra en el ejemplo siguiente:

```JSON
"parameters": {
    "storageAccountKey": {
        "reference": {
            "keyVault": {
                "id":"/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.KeyVault/vaults/<keyVaultName>",
             },
            "secretName": "<secretName>"
           },
       },
       ...
}
```

> [!NOTE]
> Aunque todavía no se admite la exportación de plantillas para factorías de datos ya existentes, esto está en proyecto.
>
>