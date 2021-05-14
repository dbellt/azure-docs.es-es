---
title: 'Tutorial: Creación de una canalización para mover datos con Azure PowerShell '
description: En este tutorial, creará una canalización de Azure Data Factory con una actividad de copia mediante Azure PowerShell.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 1e97a8bc7e30aa67e8029dc5499eeddfcf0e006e
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753342"
---
# <a name="tutorial-create-a-data-factory-pipeline-that-moves-data-by-using-azure-powershell"></a>Tutorial: Creación de una canalización de Data Factory que mueve datos mediante Azure PowerShell
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Asistente para copia](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Plantilla de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-powershell.md). 

En este artículo, aprenderá a usar PowerShell para crear una factoría de datos con una canalización que copia datos desde Azure Blob Storage hasta Azure SQL Database. Si no está familiarizado con Azure Data Factory, lea el artículo [Introducción a Azure Data Factory](data-factory-introduction.md) antes de realizar este tutorial.   

En este tutorial, creará una canalización con una actividad en ella: la actividad de copia. La actividad de copia realiza la copia de los datos de un almacén de datos admitido en un almacén de datos receptor. Para obtener una lista de almacenes de datos que se admiten como orígenes y receptores, consulte los [almacenes de datos admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats). La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md).

pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Varias actividades en una canalización](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> Este artículo no abarca todos los cmdlets de Factoría de datos. Vea [Referencia de cmdlets de Data Factory](/powershell/module/az.datafactory) para obtener la documentación completa sobre estos cmdlets.
> 
> La canalización de datos de este tutorial copia datos de un almacén de datos de origen a un almacén de datos de destino. Para ver un tutorial acerca de cómo transformar datos mediante Azure Data Factory, consulte [Tutorial: Compilación de una canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Complete los [requisitos previos del tutorial](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
- Instale **Azure PowerShell**. Siga las instrucciones de [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="steps"></a>Pasos
Estos son los pasos que se realizan en este tutorial:

1. Cree una **factoría de datos** de Azure. En este paso, creará una factoría de datos llamada ADFTutorialDataFactoryPSH. 
1. Cree **servicios vinculados** en la factoría de datos. En este paso, creará dos tipos de servicios vinculados: Microsoft Azure Storage y Azure SQL Database. 

    AzureStorageLinkedService vincula una cuenta de Azure Storage a la factoría de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó un contenedor y se cargaron datos en esta cuenta de almacenamiento.   

    AzureSqlLinkedService vincula Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó una tabla SQL en esta base de datos.   
1. Cree **conjuntos de datos** de entrada y salida en la factoría de datos.  

    El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. Además, el conjunto de datos de blobs de entrada especifica el contenedor y la carpeta que contiene los datos de entrada.  

    Igualmente, el servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su base de datos. Además, el conjunto de datos de la tabla SQL de salida especifica la tabla de la base de datos en la que se copian los datos de Blob Storage.
1. Cree una **canalización** en la factoría de datos. En este paso, se crea una canalización con una actividad de copia.   

    Esta actividad copia los datos de un blob de Azure Blob Storage en una tabla de Azure SQL Database. Puede utilizar una actividad de copia en una canalización para copiar datos desde cualquier origen admitido a cualquier destino admitido. Para ver una lista de los almacenes de datos admitidos, consulte el artículo [Actividades de movimiento de datos](data-factory-data-movement-activities.md#supported-data-stores-and-formats). 
1. Supervise la canalización. En este paso, **supervisará** los segmentos de los conjuntos de datos de entrada y salida con PowerShell.

## <a name="create-a-data-factory"></a>Crear una factoría de datos
> [!IMPORTANT]
> Complete los [requisitos previos del tutorial](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) si aún no lo ha hecho.   

Una factoría de datos puede tener una o más canalizaciones. Una canalización puede tener una o más actividades. Por ejemplo, una actividad de copia para copiar datos desde un origen a un almacén de datos de destino o una actividad de Hive de HDInsight para ejecutar un script de Hive que transforme los datos de entrada para generar datos de salida. Comencemos con la creación de la factoría de datos en este paso.

1. Inicie **PowerShell**. Mantenga Azure PowerShell abierto hasta el final de este tutorial. Si lo cierra y vuelve a abrirlo, deberá ejecutar los comandos de nuevo.

    Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en Azure Portal:

    ```powershell
    Connect-AzAccount
    ```   

    Ejecute el siguiente comando para ver todas las suscripciones de esta cuenta:

    ```powershell
    Get-AzSubscription
    ```

    Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Reemplace **&lt;NameOfAzureSubscription**&gt; por el nombre de su suscripción de Azure:

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```
1. Cree un grupo de recursos de Azure con el nombre: **ADFTutorialResourceGroup** mediante la ejecución del siguiente comando:

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    En algunos de los pasos de este tutorial se supone que se usa el grupo de recursos denominado **ADFTutorialResourceGroup**. Si usa un otro grupo de recursos, deberá usarlo en lugar de ADFTutorialResourceGroup en este tutorial.
1. Ejecute el cmdlet **New-AzDataFactory** para crear una instancia de Data Factory denominada **ADFTutorialDataFactoryPSH**:  

    ```powershell
    $df=New-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"
    ```
    Puede que ya se esté usando este nombre. Por lo tanto, para que el nombre de la factoría de datos sea único, agregue un prefijo o un sufijo (por ejemplo,  ADFTutorialDataFactoryPSH05152017) y vuelva a ejecutar el comando.  

Tenga en cuenta los siguientes puntos:

* El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el siguiente error, cambie el nombre (por ejemplo, yournameADFTutorialDataFactoryPSH). Use este nombre en lugar de ADFTutorialFactoryPSH mientras lleva a cabo los pasos de este tutorial. Consulte [Data Factory: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Data Factory.

    ```
    Data factory name "ADFTutorialDataFactoryPSH" is not available
    ```
* Para crear instancias de Data Factory, debe ser administrador o colaborador en la suscripción de Azure.
* El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.
* Puede que aparezca el siguiente mensaje de error: "**La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory**". Realice una de las siguientes acciones e intente publicar de nuevo:

  * En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

    Ejecute el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory:

    ```powershell
    Get-AzResourceProvider
    ```
  * Inicie sesión en [Azure Portal](https://portal.azure.com) mediante la suscripción de Azure. Vaya a una hoja de Data Factory o cree una instancia de Data Factory en Azure Portal. Esta acción registra automáticamente el proveedor.

## <a name="create-linked-services"></a>Crear servicios vinculados
Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En este tutorial, no se usa ningún servicio de proceso, como Azure HDInsight o Azure Data Lake Analytics. Se usan dos almacenes de datos del tipo Azure Storage (origen) y Azure SQL Database (destino). 

Por lo tanto, se crean dos servicios vinculados llamados AzureStorageLinkedService y AzureSqlLinkedService de los tipos: AzureStorage y AzureSqlDatabase.  

AzureStorageLinkedService vincula una cuenta de Azure Storage a la factoría de datos. Esta cuenta de almacenamiento es la que se usó para crear un contenedor y con la que se cargaron los datos como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

AzureSqlLinkedService vincula Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó la tabla emp en esta base de datos. 

### <a name="create-a-linked-service-for-an-azure-storage-account"></a>Creación de un servicio vinculado para una cuenta de almacenamiento de Azure
En este paso, vinculará su cuenta de Azure Storage con su factoría de datos.

1. Cree un archivo JSON con el nombre **AzureStorageLinkedService.json** en la carpeta **C:\ADFGetStartedPSH** con el siguiente contenido: (si todavía no existe, cree la carpeta ADFGetStartedPSH).

    > [!IMPORTANT]
    > Reemplace &lt;accountname&gt; y &lt;accountkey&gt; por el nombre y la clave de su cuenta de Azure Storage antes de guardar el archivo. 

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
1. En **Azure PowerShell**, cambie a la carpeta **ADFGetStartedPSH**.
1. Ejecute el cmdlet **New-AzDataFactoryLinkedService** para crear el servicio vinculado: **AzureStorageLinkedService**. Tanto el cmdlet como otros cmdlets de Data Factory que se usan en este tutorial requieren que se pasen los valores de los parámetros **ResourceGroupName** y **DataFactoryName**. Como alternativa, puede pasar el objeto DataFactory devuelto por el cmdlet New-AzDataFactory sin escribir ResourceGroupName y DataFactoryName cada vez que ejecute un cmdlet. 

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureStorageLinkedService.json
    ```
    Este es la salida de ejemplo:

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ``` 

    Otra manera de crear este servicio vinculado es especificar el nombre del grupo de recursos y el nombre de la factoría de datos en lugar de especificar el objeto DataFactory.  

    ```powershell
    New-AzDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName <Name of your data factory> -File .\AzureStorageLinkedService.json
    ```

### <a name="create-a-linked-service-for-azure-sql-database"></a>Creación de un servicio vinculado para Azure SQL Database
En este paso, vinculará Azure SQL Database con su factoría de datos.

1. Cree un archivo JSON con el nombre AzureSqlLinkedService.json en la carpeta C:\ADFGetStartedPSH con el siguiente contenido:

    > [!IMPORTANT]
    > Reemplace &lt;servername&gt;, &lt;databasename&gt;, &lt;username@servername&gt; y &lt;password&gt; por los nombres del servidor, de la base de datos, de la cuenta de usuario y de la contraseña.

    ```json
    {
        "name": "AzureSqlLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
     }
    ```
1. Ejecute el siguiente comando para crear un servicio vinculado:

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json
    ```

    Este es la salida de ejemplo:

    ```
    LinkedServiceName : AzureSqlLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ```

   Confirme que la opción **Permitir el acceso a servicios de Azure** esté activada para el servidor. Para comprobarla y activarla, siga estos pasos:

    1. Inicie sesión en el [Azure Portal](https://portal.azure.com)
    1. Haga clic en **Más servicios >** a la izquierda, y haga clic en **Servidores SQL** en la categoría **BASES DE DATOS**.
    1. Seleccione el servidor en la lista de servidores SQL Server.
    1. En la hoja SQL Server, haga clic en el vínculo **Mostrar configuración del firewall**.
    1. En la hoja **Configuración de firewall**, haga clic en **ACTIVADA** para **Permitir el acceso a los servicios de Azure**.
    1. Haga clic en **Guardar** en la barra de herramientas. 

## <a name="create-datasets"></a>Creación de conjuntos de datos
En el paso anterior, creó servicios vinculados para vincular una cuenta de Azure Storage y Azure SQL Database con la factoría de datos. En este paso, defina dos conjuntos de datos llamados InputDataset y OutputDataset, que representan los datos de entrada y salida que se almacenan en los almacenes de datos a los que hacen referencia AzureStorageLinkedService y AzureSqlLinkedService, respectivamente.

El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. Además, el conjunto de datos de blobs de entrada (InputDataset) especifica el contenedor y la carpeta que contiene los datos de entrada.  

Igualmente, el servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su base de datos. Además, el conjunto de datos de la tabla SQL de salida (OutputDataset) especifica la tabla de la base de datos en la que se copian los datos de Blob Storage. 

### <a name="create-an-input-dataset"></a>Creación de un conjunto de datos de entrada
En este paso, se crea un conjunto de datos llamado InputDataset, que apunta a un archivo de blobs (emp.txt) en la carpeta raíz de un contenedor de blobs (adftutorial), en la instancia de Azure Storage representada por el servicio vinculado AzureStorageLinkedService. Si no especifica un valor para fileName (o puede omitirlo), los datos de todos los blobs en la carpeta de entrada se copian en el destino. En este tutorial, especifique un valor para fileName.  

1. Cree un archivo JSON llamado **InputDataset.json** en la carpeta **C:\ADFGetStartedPSH** con el siguiente contenido:

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "emp.txt",
                "folderPath": "adftutorial/",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "external": true,
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
     }
    ```

    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

    | Propiedad | Descripción |
    |:--- |:--- |
    | type | La propiedad type se establece en **AzureBlob** porque los datos residen en una instancia de Azure Blob Storage. |
    | linkedServiceName | Hace referencia al servicio **AzureStorageLinkedService** que creó anteriormente. |
    | folderPath | Especifica el **contenedor** de blobs y la **carpeta** que contiene los blobs de entrada. En este tutorial, adftutorial es el contenedor de blobs y folder es la carpeta raíz. | 
    | fileName | Esta propiedad es opcional. Si omite esta propiedad, se seleccionan todos los archivos de folderPath. En este tutorial, se especifica **emp.txt** en fileName, por lo que solo se selecciona ese archivo para su procesamiento. |
    | format -> type |El archivo de entrada tiene formato de texto, por lo que usamos **TextFormat**. |
    | columnDelimiter | Las columnas del archivo de entrada están delimitadas por **coma (`,`)** . |
    | frequency/interval | La frecuencia está establecida en **Hour** y el intervalo es **1**, lo que significa que los segmentos de entrada estarán disponibles **cada hora**. En otras palabras, el servicio Data Factory busca los datos de entrada cada hora en la carpeta raíz del contenedor de blobs (**adftutorial**) que se ha especificado. Busca los datos entre las horas de inicio y finalización de la canalización, no antes ni después de esas horas.  |
    | external | Esta propiedad se establece en **true** si esta canalización no ha generado los datos. Los datos de entrada de este tutorial están en el archivo emp.txt, que no lo generó esta canalización, por lo que establecemos esta propiedad en true. |

    Para más información acerca de estas propiedades JSON, consulte el [artículo sobre el conector de blob de Azure](data-factory-azure-blob-connector.md#dataset-properties).
1. Ejecute el comando siguiente para crear el conjunto de datos de Factoría de datos.

    ```powershell  
    New-AzDataFactoryDataset $df -File .\InputDataset.json
    ```
    Este es la salida de ejemplo:

    ```
    DatasetName       : InputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureBlobDataset
    Policy            : Microsoft.Azure.Management.DataFactories.Common.Models.Policy
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

### <a name="create-an-output-dataset"></a>Crear un conjunto de datos de salida
En esta parte del paso se crea un conjunto de datos de salida denominado **OutputDataset**. Este conjunto de datos apunta a una tabla SQL de Azure SQL Database representada por **AzureSqlLinkedService1**. 

1. Cree un archivo JSON llamado **OutputDataset.json** en la carpeta **C:\ADFGetStartedPSH** con el siguiente contenido:

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```

    En la siguiente tabla se ofrecen descripciones de las propiedades JSON que se usan en el fragmento de código:

    | Propiedad | Descripción |
    |:--- |:--- |
    | type | La propiedad type se establece en **AzureSqlTable** porque los datos se copian en una tabla de una base de Azure SQL Database. |
    | linkedServiceName | Hace referencia al servicio **AzureSqlLinkedService** que creó anteriormente. |
    | tableName | Especifica la **tabla** en la que se copian los datos. | 
    | frequency/interval | La frecuencia se establece en **Hour** y el intervalo es **1**, lo que significa que los sectores de salida se producen **cada hora** entre las horas de inicio y finalización de la canalización, no antes ni después de esas horas.  |

    En la tabla emp de la base de datos hay tres columnas: **ID**, **FirstName** y **LastName**. ID es una columna de identidad, por lo que deberá especificar solo **FirstName** y **LastName** aquí.

    Para más información acerca de estas propiedades JSON, consulte el artículo sobre el [conector de Azure SQL Database](data-factory-azure-sql-connector.md#dataset-properties).
1. Ejecute el comando siguiente para crear el conjunto de datos de factoría de datos.

    ```powershell   
    New-AzDataFactoryDataset $df -File .\OutputDataset.json
    ```

    Este es la salida de ejemplo:

    ```
    DatasetName       : OutputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureSqlTableDataset
    Policy            :
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

## <a name="create-a-pipeline"></a>Crear una canalización
En este paso, creará una canalización con una **actividad de copia** que utiliza **InputDataset** como entrada y **OutputDataset** como salida.

Actualmente, el conjunto de datos de salida es lo que impulsa la programación. En este tutorial, el conjunto de datos de salida está configurado para generar un segmento una vez cada hora. La canalización tiene una hora de inicio y una hora de finalización con un día de diferencia, es decir, 24 horas. Por lo tanto, la canalización produce 24 segmentos del conjunto de datos de salida. 

1. Cree un archivo JSON llamado **ADFTutorialPipeline.json** en la carpeta **C:\ADFGetStartedPSH** con el siguiente contenido:

    ```json   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "SqlSink",
                "writeBatchSize": 10000,
                "writeBatchTimeout": "60:00:00"
              }
            },
            "Policy": {
              "concurrency": 1,
              "executionPriorityOrder": "NewestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
        ],
        "start": "2017-05-11T00:00:00Z",
        "end": "2017-05-12T00:00:00Z"
      }
    } 
    ```
    Tenga en cuenta los siguientes puntos:

   - En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). En las soluciones de Data Factory, también puede usar [actividades de transformación de datos](data-factory-data-transformation-activities.md).
   - La entrada de la actividad está establecida en **InputDataset**, mientras que la salida está establecida en **OutputDataset**. 
   - En la sección **typeProperties**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor. Consulte la lista de [almacenes de datos que se admiten](data-factory-data-movement-activities.md#supported-data-stores-and-formats) como orígenes y receptores de la actividad de copia. Para más información sobre cómo usar un almacén de datos admitido específico como receptor de origen, haga clic en el vínculo en la tabla.  

     Reemplace el valor de la propiedad **start** por el día actual y el valor **end** por el próximo día. Puede especificar solo la parte de fecha y omitir la parte de hora de la fecha y hora. Por ejemplo, "03-02-2016", que es equivalente a "03-02-2016T00:00:00Z"

     Las fechas y horas de inicio y de finalización deben estar en [formato ISO](https://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2016-10-14T16:32:41Z. La hora de finalización ( **end** ) es opcional, pero se utilizará en este tutorial. 

     Si no especifica ningún valor para la propiedad **end**, se calcula como "**start + 48 horas**". Para ejecutar la canalización indefinidamente, especifique **9999-09-09** como valor de propiedad **end**.

     En el ejemplo anterior hay 24 segmentos de datos, ya que cada segmento de datos se produce cada hora.

     Para obtener descripciones de las propiedades JSON en una definición de canalización, consulte cómo [crear canalizaciones](data-factory-create-pipelines.md). Para obtener descripciones de las propiedades JSON en una definición de actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). Para ver las descripciones de las propiedades JSON admitidas por BlobSource, consulte el artículo sobre el [conector de Azure Blob](data-factory-azure-blob-connector.md). Para ver las descripciones de las propiedades JSON admitidas por SqlSink, consulte el artículo sobre el [conector de Azure SQL Database](data-factory-azure-sql-connector.md).
1. Ejecute el comando siguiente para crear la tabla de factoría de datos.

    ```powershell   
    New-AzDataFactoryPipeline $df -File .\ADFTutorialPipeline.json
    ```

    Este es la salida de ejemplo: 

    ```
    PipelineName      : ADFTutorialPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.PipelinePropertie
    ProvisioningState : Succeeded
    ```

**¡Enhorabuena!** Ha creado correctamente una factoría de datos de Azure con una canalización para copiar datos desde una instancia de Azure Blob Storage hasta Azure SQL Database. 

## <a name="monitor-the-pipeline"></a>Supervisar la canalización
En este paso, se usa Azure PowerShell para supervisar lo que ocurre en una instancia de Azure Data Factory.

1. Reemplace &lt;DataFactoryName&gt; por el nombre de la factoría de datos, ejecute **Get-AzDataFactory**, y asigne el resultado a una variable $df.

    ```powershell  
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name <DataFactoryName>
    ```

    Por ejemplo:
    ```powershell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH0516
    ```

    Después, imprima el contenido de $df para ver el siguiente resultado: 

    ```
    PS C:\ADFGetStartedPSH> $df

    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DataFactoryId     : 6f194b34-03b3-49ab-8f03-9f8a7b9d3e30
    ResourceGroupName : ADFTutorialResourceGroup
    Location          : West US
    Tags              : {}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DataFactoryProperties
    ProvisioningState : Succeeded
    ```
1. Ejecute **Get-AzDataFactorySlice** para obtener la información acerca de todos los segmentos de **OutputDataset**, que es el conjunto de datos de salida de la canalización.  

    ```powershell   
    Get-AzDataFactorySlice $df -DatasetName OutputDataset -StartDateTime 2017-05-11T00:00:00Z
    ```

   Esta configuración debe coincidir con el valor de **Inicio** del JSON de canalización. Debe ver 24 segmentos, uno para cada hora de 12 A.M. del día actual hasta las 12 A.M. del día siguiente.

   Estos son tres segmentos de ejemplo de la salida: 

    ``` 
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 11:00:00 PM
    End               : 5/12/2017 12:00:00 AM
    RetryCount        : 0
    State             : Ready
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 9:00:00 PM
    End               : 5/11/2017 10:00:00 PM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 8:00:00 PM
    End               : 5/11/2017 9:00:00 PM
    RetryCount        : 0
    State             : Waiting
    SubState          : ConcurrencyLimit
    LatencyStatus     :
    LongRetryCount    : 0
    ```
1. Ejecute **Get-AzDataFactoryRun** para más información de la actividad que se ejecuta para un segmento **específico**. Copie el valor de fecha y hora de la salida del comando anterior para especificar el valor del parámetro StartDateTime. 

    ```powershell  
    Get-AzDataFactoryRun $df -DatasetName OutputDataset -StartDateTime "5/11/2017 09:00:00 PM"
    ```

   Este es la salida de ejemplo: 

    ```
    Id                  : c0ddbd75-d0c7-4816-a775-704bbd7c7eab_636301332000000000_636301368000000000_OutputDataset
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : ADFTutorialDataFactoryPSH0516
    DatasetName         : OutputDataset
    ProcessingStartTime : 5/16/2017 8:00:33 PM
    ProcessingEndTime   : 5/16/2017 8:01:36 PM
    PercentComplete     : 100
    DataSliceStart      : 5/11/2017 9:00:00 PM
    DataSliceEnd        : 5/11/2017 10:00:00 PM
    Status              : Succeeded
    Timestamp           : 5/16/2017 8:00:33 PM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : CopyFromBlobToSQL
    PipelineName        : ADFTutorialPipeline
    Type                : Copy  
    ```

Consulte [Referencia de cmdlets de Data Factory](/powershell/module/az.datafactory) para obtener la documentación completa sobre los cmdlets de Data Factory.

## <a name="summary"></a>Resumen
En este tutorial, ha creado una factoría de datos de Azure para copiar datos desde un blob de Azure hasta Azure SQL Database. Ha usado PowerShell para crear la factoría de datos, los servicios vinculados, los conjuntos de datos y una canalización. Estos son los pasos de alto nivel que realizó en este tutorial:  

1. Ha creado una **factoría de datos** de Azure.
1. Ha creado **servicios vinculados**.

   a. Un servicio vinculado **Azure Storage** para vincular la cuenta de almacenamiento de Azure que contiene datos de entrada.     
   b. Un servicio vinculado **Azure SQL** para vincular la base de datos SQL que contiene los datos de salida.
1. Ha creado **conjuntos de datos** que describen los datos de entrada y salida de las canalizaciones.
1. Ha creado una **canalización** con una **actividad de copia** con un origen **BlobSource** y un receptor **SqlSink**.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha usado Azure Blob Storage como almacén de datos de origen y Azure SQL Database como almacén de datos de destino en una operación de copia. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes y destinos de la actividad de copia: 

[!INCLUDE [data-factory-supported-data-stores](includes/data-factory-supported-data-stores.md)]

Para aprender a copiar datos hacia y desde un almacén de datos, haga clic en el vínculo del almacén de datos en la tabla. 

