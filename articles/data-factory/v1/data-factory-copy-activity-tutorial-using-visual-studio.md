---
title: 'Tutorial: Creación de una canalización con la actividad de copia mediante Visual Studio '
description: En este tutorial, creará una canalización de Azure Data Factory con una actividad de copia mediante Visual Studio.
author: linda33wj
ms.service: data-factory
ms.custom: vs-azure, devx-track-azurepowershell
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 82025728f354b5522517e210071aeb4c7528d7cf
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677725"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-visual-studio"></a>Tutorial: Creación de una canalización con la actividad de copia mediante Visual Studio
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Asistente para copia](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Plantilla de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-dot-net.md). 

En este artículo, aprenderá a usar Microsoft Visual Studio para crear una factoría de datos con una canalización que copia datos desde una instancia de Azure Blob Storage hasta Azure SQL Database. Si no está familiarizado con Azure Data Factory, lea el artículo [Introducción a Azure Data Factory](data-factory-introduction.md) antes de realizar este tutorial.   

En este tutorial, creará una canalización con una actividad en ella: la actividad de copia. La actividad de copia realiza la copia de los datos de un almacén de datos admitido en un almacén de datos receptor. Para obtener una lista de almacenes de datos que se admiten como orígenes y receptores, consulte los [almacenes de datos admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats). La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md).

pero cualquier canalización puede tener más de una actividad. También puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Varias actividades en una canalización](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE] 
> La canalización de datos de este tutorial copia datos de un almacén de datos de origen a un almacén de datos de destino. Para ver un tutorial acerca de cómo transformar datos mediante Azure Data Factory, consulte [Tutorial: Compilación de una canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. Lea el artículo [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) y complete los pasos de los **requisitos previos** .       
2. Para crear instancias de Data Factory, es preciso ser miembro del rol [Colaborador de Data Factory](../../role-based-access-control/built-in-roles.md#data-factory-contributor) en el nivel de grupo de recursos o suscripción.
3. Debe tener lo siguiente instalado en el equipo: 
   * Visual Studio 2013 o Visual Studio 2015.
   * Descargue el SDK de Azure para Visual Studio 2013 o Visual Studio 2015. Vaya a la [página Descargas de Azure](https://azure.microsoft.com/downloads/) y haga clic en **VS 2013** o **VS 2015** en la sección **.NET**.
   * Descargue el complemento más reciente de Azure Data Factory para Visual Studio: [VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) o [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005). También puede actualizar el complemento con el procedimiento siguiente: En el menú, haga clic en **Herramientas** -> **Extensiones y actualizaciones** -> **En línea** -> **Galería de Visual Studio** -> **Microsoft Azure Data Factory Tools for Visual Studio (Herramientas de Microsoft Azure Data Factory para Visual Studio)**  -> **Actualizar**.

## <a name="steps"></a>Pasos

Estos son los pasos que se realizan en este tutorial:

1. Cree **servicios vinculados** en la factoría de datos. En este paso, creará dos tipos de servicios vinculados: Microsoft Azure Storage y Azure SQL Database. 

    AzureStorageLinkedService vincula una cuenta de Azure Storage a la factoría de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó un contenedor y se cargaron datos en esta cuenta de almacenamiento.   

    AzureSqlLinkedService vincula Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó una tabla SQL en esta base de datos.     

2. Cree **conjuntos de datos** de entrada y salida en la factoría de datos.  

    El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. Además, el conjunto de datos de blobs de entrada especifica el contenedor y la carpeta que contiene los datos de entrada.  

    De forma similar, el servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a Azure SQL Database. Además, el conjunto de datos de la tabla SQL de salida especifica la tabla de la base de datos en la que se copian los datos de Blob Storage.

3. Cree una **canalización** en la factoría de datos. En este paso, se crea una canalización con una actividad de copia.   

    Esta actividad copia los datos de un blob de Azure Blob Storage en una tabla de Azure SQL Database. Puede utilizar una actividad de copia en una canalización para copiar datos desde cualquier origen admitido a cualquier destino admitido. Para ver una lista de los almacenes de datos admitidos, consulte el artículo [Actividades de movimiento de datos](data-factory-data-movement-activities.md#supported-data-stores-and-formats).

4. Cree una **factoría de datos** de Azure al implementar entidades de Data Factory (servicios vinculados, conjuntos de datos o tablas y canalizaciones). 

## <a name="create-visual-studio-project"></a>Creación de un proyecto de Visual Studio

1. Inicie **Visual Studio 2015**. Haga clic en **Archivo**, seleccione **Nuevo** y, luego, haga clic en **Proyecto**. Debería ver el cuadro de diálogo **Nuevo proyecto** .  

2. En el cuadro de diálogo **Nuevo proyecto**, seleccione la plantilla **DataFactory** y haga clic en **Proyecto vacío de factoría de datos**.  

    ![Cuadro de diálogo Nuevo proyecto](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-project-dialog.png)

3. Especifique el nombre del proyecto, la ubicación y el nombre de la solución y, a continuación, haga clic en **Aceptar**.

    ![Explorador de soluciones](./media/data-factory-copy-activity-tutorial-using-visual-studio/solution-explorer.png)    

## <a name="create-linked-services"></a>Crear servicios vinculados

Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En este tutorial, no se usa ningún servicio de proceso, como Azure HDInsight o Azure Data Lake Analytics. Se usan dos almacenes de datos del tipo Azure Storage (origen) y Azure SQL Database (destino). 

Por tanto, crea dos tipos de servicios vinculados: AzureStorage y AzureSqlDatabase.  

El servicio vinculado Azure Storage vincula una cuenta de Azure Storage a la factoría de datos. Esta cuenta de almacenamiento es la que se usó para crear un contenedor y con la que se cargaron los datos como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

Un servicio vinculado Azure SQL vincula Azure SQL Database con la factoría de datos. Los datos que se copian desde Blob Storage se almacenan en esta base de datos. Como parte de los [requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md), se creó la tabla emp en esta base de datos.

Los servicios vinculados vinculan almacenes de datos o servicios de proceso con una factoría de datos de Azure. Consulte los [almacenes de datos compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats) para ver todos orígenes y receptores que admite la actividad de copia. Consulte los [servicios vinculados de procesos](data-factory-compute-linked-services.md) para ver la lista de servicios de proceso compatibles con Data Factory. En este tutorial no se usan servicios de proceso. 

### <a name="create-the-azure-storage-linked-service"></a>Creación del servicio vinculado de Azure Storage

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **Servicios vinculados**, seleccione **Agregar** y haga clic en **Nuevo elemento**.      

2. En el cuadro de diálogo **Agregar nuevo elemento**, seleccione **Servicios vinculados de Azure Storage** en la lista y haga clic en **Agregar**. 

    ![Nuevo servicio vinculado](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-linked-service-dialog.png)

3. Reemplace `<accountname>` y `<accountkey>`* por el nombre de la cuenta de Azure Storage y su clave. 

    ![Servicio vinculado de Azure Storage](./media/data-factory-copy-activity-tutorial-using-visual-studio/azure-storage-linked-service.png)

4. Guarde el archivo **AzureStorageLinkedService1.json** .

    Para más información sobre las propiedades JSON en la definición de servicio vinculado, vea el artículo [Conector de Azure Blob Storage](data-factory-azure-blob-connector.md#linked-service-properties).

### <a name="create-the-azure-sql-linked-service"></a>Cree el servicio vinculado SQL de Azure.

1. Haga doble clic con el botón derecho en el nodo **Servicios vinculados** en el **Explorador de soluciones** de nuevo, apunte a **Agregar** y haga clic en **Nuevo elemento**. 
2. Esta vez, seleccione **Servicios vinculados de SQL Azure** y haga clic en **Agregar**. 
3. En el **archivo AzureSqlLinkedService1.json**, reemplace `<servername>`, `<databasename>`, `<username@servername>` y `<password>` por los nombres del servidor, la base de datos, la cuenta de usuario y la contraseña.    
4. Guarde el archivo **AzureSqlLinkedService1.json** . 

    Para más información acerca de estas propiedades JSON, consulte [Conector de Azure SQL Database](data-factory-azure-sql-connector.md#linked-service-properties).

## <a name="create-datasets"></a>Creación de conjuntos de datos

En el paso anterior, creó servicios vinculados para vincular una cuenta de Azure Storage y Azure SQL Database con la factoría de datos. En este paso, defina dos conjuntos de datos llamados InputDataset y OutputDataset, que representan los datos de entrada y salida que se almacenan en los almacenes de datos a los que hacen referencia AzureStorageLinkedService1 y AzureSqlLinkedService1, respectivamente.

El servicio vinculado Azure Storage especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a su cuenta de Azure Storage. Además, el conjunto de datos de blobs de entrada (InputDataset) especifica el contenedor y la carpeta que contiene los datos de entrada.  

De forma similar, el servicio vinculado de Azure SQL Database especifica la cadena de conexión que el servicio Data Factory utiliza en tiempo de ejecución para conectarse a Azure SQL Database. Además, el conjunto de datos de la tabla SQL de salida (OutputDataset) especifica la tabla de la base de datos en la que se copian los datos de Blob Storage. 

### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada

En este paso, se crea un conjunto de datos llamado InputDataset, que apunta a un archivo de blobs (emp.txt) en la carpeta raíz de un contenedor de blobs (adftutorial), en la instancia de Azure Storage representada por el servicio vinculado AzureStorageLinkedService1. Si no especifica un valor para fileName (o puede omitirlo), los datos de todos los blobs en la carpeta de entrada se copian en el destino. En este tutorial, especifique un valor para fileName. 

Aquí, se usa el término "tablas" en lugar de "conjuntos de datos". Una tabla es un conjunto de datos rectangular y es el único tipo de conjunto de datos que se admite en este momento. 

1. Haga clic con el botón derecho en **Tablas** en el **Explorador de soluciones**, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. En el cuadro de diálogo **Agregar nuevo elemento**, seleccione **Blob de Azure** y haga clic en **Agregar**.   
3. Reemplace el texto JSON con el texto siguiente y guarde el archivo **AzureBlobLocation1.json** . 

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
      "linkedServiceName": "AzureStorageLinkedService1",
      "typeProperties": {
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
    | type | La propiedad type se establece en **AzureBlob** porque los datos residen en Azure Blob Storage. |
    | linkedServiceName | Hace referencia al elemento **AzureStorageLinkedService** que creó anteriormente. |
    | folderPath | Especifica el **contenedor** de blobs y la **carpeta** que contiene los blobs de entrada. En este tutorial, adftutorial es el contenedor de blobs y folder es la carpeta raíz. | 
    | fileName | Esta propiedad es opcional. Si omite esta propiedad, se seleccionan todos los archivos de folderPath. En este tutorial, se especifica **emp.txt** en fileName, por lo que solo se selecciona ese archivo para su procesamiento. |
    | format -> type |El archivo de entrada tiene formato de texto, por lo que usamos **TextFormat**. |
    | columnDelimiter | Las columnas del archivo de entrada están delimitadas por un **carácter de coma (`,`)** . |
    | frequency/interval | La frecuencia está establecida en **Hour** (Hora) y el intervalo es **1**, lo que significa que los segmentos de entrada estarán disponibles **cada hora**. En otras palabras, el servicio Data Factory busca los datos de entrada cada hora en la carpeta raíz del contenedor de blobs (**adftutorial**) que se ha especificado. Busca los datos entre las horas de inicio y finalización de la canalización, no antes ni después de esas horas.  |
    | external | Esta propiedad se establece en **true** si esta canalización no ha generado los datos. Los datos de entrada de este tutorial están en el archivo emp.txt, que no se generó con esta canalización, por lo que establecemos esta propiedad en true. |

    Para más información acerca de estas propiedades JSON, consulte el [artículo sobre el conector de blob de Azure](data-factory-azure-blob-connector.md#dataset-properties).   

### <a name="create-output-dataset"></a>Creación del conjunto de datos de salida

En este paso se crea un conjunto de datos de salida denominado **OutputDataset**. Este conjunto de datos apunta a una tabla SQL de Azure SQL Database representada por **AzureSqlLinkedService1**. 

1. Haga clic con el botón derecho en **Tablas** en el **Explorador de soluciones** de nuevo, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. En el cuadro de diálogo **Agregar nuevo elemento**, seleccione **SQL de Azure** y haga clic en **Agregar**. 
3. Reemplace el texto JSON con el JSON siguiente y guarde el archivo **AzureSqlTableLocation1.json** .

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
            "linkedServiceName": "AzureSqlLinkedService1",
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

## <a name="create-pipeline"></a>Creación de una canalización

En este paso, creará una canalización con una **actividad de copia** que utiliza **InputDataset** como entrada y **OutputDataset** como salida.

Actualmente, el conjunto de datos de salida es lo que impulsa la programación. En este tutorial, el conjunto de datos de salida está configurado para generar un segmento una vez cada hora. La canalización tiene una hora de inicio y una hora de finalización con un día de diferencia, es decir, 24 horas. Por lo tanto, la canalización produce 24 segmentos del conjunto de datos de salida. 

1. Haga clic con el botón derecho en **Canalizaciones** en el **Explorador de soluciones**, seleccione **Agregar** y haga clic en **Nuevo elemento**.  
2. Seleccione **Copiar canalización de datos** en el cuadro de diálogo **Agregar nuevo elemento** y haga clic en **Agregar**. 
3. Reemplace el JSON por el siguiente JSON y guarde el archivo **CopyActivity1.json** .

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
           "style": "StartOfInterval",
           "retry": 0,
           "timeout": "01:00:00"
         }
       }
     ],
     "start": "2017-05-11T00:00:00Z",
     "end": "2017-05-12T00:00:00Z",
     "isPaused": false
   }
  }
    ```   
   - En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). En las soluciones de Data Factory, también puede usar [actividades de transformación de datos](data-factory-data-transformation-activities.md).
   - La entrada de la actividad está establecida en **InputDataset**, mientras que la salida está establecida en **OutputDataset**. 
   - En la sección **typeProperties**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor. Consulte la lista de [almacenes de datos que se admiten](data-factory-data-movement-activities.md#supported-data-stores-and-formats) como orígenes y receptores de la actividad de copia. Para más información sobre cómo usar un almacén de datos admitido específico como receptor de origen, haga clic en el vínculo en la tabla.  
     
     Reemplace el valor de la propiedad **start** por el día actual y el valor **end** por el próximo día. Puede especificar solo la parte de fecha y omitir la parte de hora de la fecha y hora. Por ejemplo, "03-02-2016", que es equivalente a "03-02-2016T00:00:00Z"
     
     Las fechas y horas de inicio y de finalización deben estar en [formato ISO](https://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2016-10-14T16:32:41Z. La hora de finalización ( **end** ) es opcional, pero se utilizará en este tutorial. 
     
     Si no especifica ningún valor para la propiedad **end**, se calcula como "**start + 48 horas**". Para ejecutar la canalización indefinidamente, especifique **9999-09-09** como valor de propiedad **end**.
     
     En el ejemplo anterior hay 24 segmentos de datos, ya que cada segmento de datos se produce cada hora.

     Para obtener descripciones de las propiedades JSON en una definición de canalización, consulte cómo [crear canalizaciones](data-factory-create-pipelines.md). Para obtener descripciones de las propiedades JSON en una definición de actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md). Para ver las descripciones de las propiedades JSON admitidas por BlobSource, consulte el artículo sobre el [conector de Azure Blob](data-factory-azure-blob-connector.md). Para ver las descripciones de las propiedades JSON admitidas por SqlSink, consulte el artículo sobre el [conector de Azure SQL Database](data-factory-azure-sql-connector.md).

## <a name="publishdeploy-data-factory-entities"></a>Publicación e implementación de las entidades de Data Factory
En este paso, se publican las entidades de Data Factory (servicios vinculados, conjuntos de datos y canalización) que se crearon anteriormente. También se especifica el nombre de la nueva instancia de Data Factory que se va a crear para almacenar dichas entidades.  

1. Haga clic con el botón derecho en el proyecto en el Explorador de soluciones y haga clic en **Publicar**. 
2. Si ve el cuadro de diálogo **Iniciar sesión en tu cuenta Microsoft**, escriba sus credenciales para la cuenta que tiene la suscripción de Azure y haga clic en **Iniciar sesión**.
3. Debería ver el siguiente cuadro de diálogo:
   
   ![Cuadro de diálogo Publicar](./media/data-factory-copy-activity-tutorial-using-visual-studio/publish.png)
4. En la página Configure data factory (Configurar Data Factory), siga estos pasos: 
   
   1. Seleccione la opción **Crear la nueva factoría de datos** .
   2. Escriba **VSTutorialFactory** para **Nombre**.  
      
      > [!IMPORTANT]
      > El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe un error sobre el nombre de la factoría de datos cuando se publica, cámbielo (por ejemplo, sunombreVSTutorialFactory) e intente publicar de nuevo. Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.        
      > 
      > 
   3. Seleccione su suscripción de Azure en el campo **Suscripción** .
      
      > [!IMPORTANT]
      > Si no ve ninguna suscripción, asegúrese de que ha iniciado sesión con una cuenta que sea administrador o coadministrador de la suscripción.  
      > 
      > 
   4. Seleccione el **grupo de recursos** para la factoría de datos que se va a crear. 
   5. Seleccione la **Región** de la factoría de datos. La lista desplegable solo muestra las regiones que admite el servicio Data Factory.
   6. Haga clic en **Siguiente** para cambiar a la página **Publicar elementos**.
      
       ![Configurar página de Data Factory](media/data-factory-copy-activity-tutorial-using-visual-studio/configure-data-factory-page.png)   
5. En la página **Publicar elementos**, asegúrese de que todas las factorías de datos están seleccionadas y haga clic en **Siguiente** para cambiar a la página **Resumen**.
   
   ![Página Publicar elementos](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-items-page.png)     
6. Revise el resumen y haga clic en **Siguiente** para iniciar el proceso de implementación y ver el **Estado de implementación**.
   
   ![Página Publicar resumen](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-summary-page.png)
7. En la página **Estado de implementación** , debería ver el estado del proceso de implementación. Cuando se haya completado la implementación, haga clic en Finalizar.
 
   ![Página Estado de la implementación](media/data-factory-copy-activity-tutorial-using-visual-studio/deployment-status.png)

Tenga en cuenta los siguientes puntos: 

* Si recibe el error: "La suscripción no está registrada para usar el espacio de nombres Microsoft.DataFactory", realice una de las acciones siguientes e intente publicarla de nuevo: 
  
  * En Azure PowerShell, ejecute el siguiente comando para registrar el proveedor de Data Factory. 

  ```powershell    
  Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
  Puede ejecutar el comando siguiente para confirmar que se ha registrado el proveedor de Data Factory. 
    
  ```powershell
  Get-AzResourceProvider
    ```
  * Inicie sesión mediante la suscripción de Azure en el [Portal de Azure](https://portal.azure.com) y vaya a una hoja de Data Factory (o) cree una factoría de datos en el Portal de Azure. Esta acción registra automáticamente el proveedor.
* El nombre de la factoría de datos se puede registrar como un nombre DNS en el futuro y, por lo tanto, que sea visible públicamente.

> [!IMPORTANT]
> Para crear instancias de Data Factory, es preciso ser administrador o coadministrador de la suscripción de Azure

## <a name="monitor-pipeline"></a>Supervisión de la canalización
Vaya a la página principal de la factoría de datos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú izquierdo, haga clic en **Más servicios** y en **Factorías de datos**.

  ![Examinar factorías de datos](media/data-factory-copy-activity-tutorial-using-visual-studio/browse-data-factories.png)
3. Comience a escribir el nombre de la factoría de datos.

  ![Nombre de la factoría de datos](media/data-factory-copy-activity-tutorial-using-visual-studio/enter-data-factory-name.png) 
4. Haga clic en la factoría de datos en la lista de resultados para ver la página principal de la factoría de datos.

  ![Página principal Factoría de datos](media/data-factory-copy-activity-tutorial-using-visual-studio/data-factory-home-page.png)
5. Siga las instrucciones que se indican en [Supervisión de conjuntos de datos y canalizaciones](data-factory-monitor-manage-pipelines.md) para supervisar la canalización y los conjuntos de datos que ha creado en este tutorial. Actualmente, Visual Studio no permite supervisar canalizaciones de Data Factory. 

## <a name="summary"></a>Resumen
En este tutorial, ha creado una factoría de datos de Azure para copiar datos desde un blob de Azure hasta Azure SQL Database. Ha usado Visual Studio para crear la factoría de datos, los servicios vinculados, los conjuntos de datos y una canalización. Estos son los pasos de alto nivel que realizó en este tutorial:  

1. Ha creado una **factoría de datos** de Azure.
2. Ha creado **servicios vinculados**.
   1. Un servicio vinculado **Azure Storage** para vincular la cuenta de Azure Storage que contiene datos de entrada.     
   2. Un servicio vinculado **Azure SQL** para vincular la base de datos que contiene los datos de salida. 
3. Ha creado **conjuntos de datos** que describen los datos de entrada y salida para las canalizaciones.
4. Ha creado una **canalización** con una **actividad de copia** con un origen **BlobSource** y un receptor **SqlSink**. 

Para ver cómo usar una actividad de Hive en HDInsight para transformar datos mediante un clúster de Azure HDInsight, consulte el [Tutorial: Compilación de la primera canalización para transformar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

Puede encadenar dos actividades (ejecutar una después de otra) haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. Para más información, consulte [Programación y ejecución en Data Factory](data-factory-scheduling-and-execution.md). 

## <a name="view-all-data-factories-in-server-explorer"></a>Presentación de todas las factorías de datos en el explorador de servidores
En esta sección se describe cómo usar el explorador de servidores de Visual Studio para ver todas las factorías de datos de su suscripción de Azure, y cómo crear un proyecto de Visual Studio basado en una factoría de datos existente. 

1. En **Visual Studio** haga clic en **Vista** en el menú y haga clic en **Explorador de servidores**.
2. En la ventana Explorador de servidores, expanda **Azure** y **Factoría de datos**. Si aparece **Iniciar sesión en Visual Studio**, escriba la **cuenta** asociada a su suscripción de Azure y haga clic en **Continuar**. Escriba la **contraseña** y haga clic en **Iniciar sesión**. Visual Studio intenta obtener información acerca de todas las factorías de datos de Azure de su suscripción. Puede ver el estado de esta operación en la ventana **Data Factory Task List** (Lista de tareas de Data Factory).

    ![Explorador de servidores](./media/data-factory-copy-activity-tutorial-using-visual-studio/server-explorer.png)

## <a name="create-a-visual-studio-project-for-an-existing-data-factory"></a>Creación de un proyecto de Visual Studio para una factoría de datos existente

- Haga clic con el botón derecho en una factoría de datos en el Explorador de servidores y seleccione **Exportar factoría de datos al nuevo proyecto** para crear un proyecto de Visual Studio basado en una factoría de datos existente.

  ![Exportar factoría de datos a un proyecto de VS](./media/data-factory-copy-activity-tutorial-using-visual-studio/export-data-factory-menu.png)  

## <a name="update-data-factory-tools-for-visual-studio"></a>Actualización de herramientas de Factoría de datos para Visual Studio
Para actualizar las herramientas de Azure Data Factory para Visual Studio, siga estos pasos:

1. Haga clic en el menú **Herramientas** y seleccione **Extensiones y actualizaciones**. 
2. Seleccione **Actualizaciones** en el panel izquierdo y, luego, seleccione **Galería de Visual Studio**.
3. Seleccione **Herramientas de Azure Data Factory para Visual Studio** y haga clic en **Actualizar**. Si no ve esta entrada, ya tiene la versión más reciente de las herramientas. 

## <a name="use-configuration-files"></a>Uso de archivos de configuración
Puede usar archivos de configuración de Visual Studio para configurar propiedades para servicios vinculados/tablas/canalizaciones distintos para cada entorno.

Considere la siguiente definición de JSON para un servicio vinculado de Azure Storage. Con ella, podrá especificar la propiedad **connectionString** con distintos valores para accountname y accountkey basados en el entorno (desarrollo, pruebas y producción) en el que va a implementar las entidades de Data Factory. Puede conseguir este comportamiento mediante el uso del archivo de configuración independiente para cada entorno.

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="add-a-configuration-file"></a>Adición de un archivo de configuración
Agregue un archivo de configuración a cada entorno realizando los pasos siguientes:   

1. En la solución de Visual Studio, haga clic con el botón derecho en el proyecto de Data Factory, seleccione **Agregar** y haga clic en **Nuevo elemento**.
2. Seleccione **Config** en la lista de plantillas instaladas que encontrará a la izquierda, elija **Archivo de configuración**, especifique un **nombre** para este archivo y haga clic en **Agregar**.

    ![Adición de archivo de configuración](./media/data-factory-build-your-first-pipeline-using-vs/add-config-file.png)
3. Agregue parámetros de configuración y sus valores en el formato siguiente:

  ```json
  {
      "$schema": "http://datafactories.schema.management.azure.com/vsschemas/V1/Microsoft.DataFactory.Config.json",
      "AzureStorageLinkedService1": [
          {
              "name": "$.properties.typeProperties.connectionString",
              "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
          }
      ],
      "AzureSqlLinkedService1": [
          {
              "name": "$.properties.typeProperties.connectionString",
              "value":  "Server=tcp:<logical SQL server name>.database.windows.net,1433;Database=<Azure SQL datbase>;User ID=<Username>;Password=<Password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
          }
      ]
  }
    ```

    En este ejemplo se configura la propiedad connectionString de un servicio vinculado de Azure Storage y un servicio vinculado de SQL de Azure. Observe que la sintaxis para especificar el nombre es [JsonPath](https://goessner.net/articles/JsonPath/).   

    Si JSON tiene una propiedad con una matriz de valores como se muestra en el código siguiente:  

  ```json
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
    ```

    Configure las propiedades tal como se muestra en el siguiente archivo de configuración (use la indexación de base cero):

  ```json
    {
        "name": "$.properties.structure[0].name",
        "value": "FirstName"
    }
    {
        "name": "$.properties.structure[0].type",
        "value": "String"
    }
    {
        "name": "$.properties.structure[1].name",
        "value": "LastName"
    }
    {
        "name": "$.properties.structure[1].type",
        "value": "String"
    }
    ```

### <a name="property-names-with-spaces"></a>Nombres de propiedad con espacios
Si un nombre de propiedad contiene espacios, utilice corchetes como se muestra en el ejemplo siguiente (nombre de servidor de base de datos):

```json
 {
     "name": "$.properties.activities[1].typeProperties.webServiceParameters.['Database server name']",
     "value": "MyAsqlServer.database.windows.net"
 }
```

### <a name="deploy-solution-using-a-configuration"></a>Implementación de la solución mediante una configuración
Al publicar las entidades de Azure Data Factory en Visual Studio, puede especificar la configuración que desea usar para esa operación de publicación.

Para publicar entidades en un proyecto de Azure Data Factory mediante el archivo de configuración:   

1. Haga clic con el botón derecho en el proyecto de Data Factory y después haga clic en **Publicar** para abrir el cuadro de diálogo **Publish Items** (Publicar elementos).
2. Seleccione una factoría de datos existente o especifique los valores necesarios para crear una en la página **Configure data factory** (Configurar Data Factory) y haga clic en **Siguiente**.   
3. En la página **Publish Items** (Publicar elementos), verá una lista desplegable con las configuraciones disponibles para el campo **Select Deployment Config** (Seleccionar configuración de implementación).

    ![Selección de archivo de configuración](./media/data-factory-build-your-first-pipeline-using-vs/select-config-file.png)
4. Seleccione el **archivo de configuración** que desea usar y haga clic en **Siguiente**.
5. Confirme que ve el nombre del archivo JSON en la página **Resumen** y haga clic en **Siguiente**.
6. Cuando finalice la operación de implementación, haga clic en **Finalizar** .

Cuando realiza la implementación, se usan los valores del archivo de configuración para establecer los valores de las propiedades de los archivos JSON antes de que las entidades se implementen en el servicio Azure Data Factory.   

## <a name="use-azure-key-vault"></a>Uso de Azure Key Vault
No es aconsejable y, en ocasiones, contraviene la directiva de seguridad, confirmar información confidencial, como las cadenas de conexión, en el repositorio de código. Vea el ejemplo [ADF Secure Publish](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFSecurePublish) en GitHub para obtener información sobre cómo almacenar información confidencial en Azure Key Vault y usarla al publicar entidades de Data Factory. La extensión Secure Publish de Visual Studio permite almacenar los secretos en Key Vault, y solo las referencias a ellos se especifican en las configuraciones de implementación o en los servicios vinculados. Estas referencias se resuelven al publicar las entidades de Data Factory en Azure. Estos archivos se pueden confirmar en el repositorio de origen sin exponer los secretos.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha usado Azure Blob Storage como almacén de datos de origen y Azure SQL Database como almacén de datos de destino en una operación de copia. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes y destinos de la actividad de copia: 

[!INCLUDE [data-factory-supported-data-stores](includes/data-factory-supported-data-stores.md)]

Para aprender a copiar datos hacia y desde un almacén de datos, haga clic en el vínculo del almacén de datos en la tabla.
