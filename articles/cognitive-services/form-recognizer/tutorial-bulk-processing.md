---
title: 'Tutorial: Extracción masiva de datos de formularios con Azure Data Factory: Form Recognizer'
titleSuffix: Azure Applied AI Services
description: Configure actividades de Azure Data Factory que desencadenen el entrenamiento y la ejecución de modelos de Form Recognizer y digitalice una gran cantidad de trabajo pendiente de documentos.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 01/04/2021
ms.author: lajanuar
ms.openlocfilehash: ddb4ec251cc6d630ba8161a6f25bce33223c75cf
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377875"
---
# <a name="tutorial-extract-form-data-in-bulk-by-using-azure-data-factory"></a>Tutorial: Extracción masiva de datos de formularios mediante Azure Data Factory

En este tutorial se explica cómo usar los servicios de Azure para ingerir un gran lote de formularios en medios digitales. En el tutorial se mostrará cómo automatizar la ingesta de datos de un lago de datos de documentos de Azure en una base de datos de Azure SQL. Podrá entrenar rápidamente modelos y procesar nuevos documentos con solo unos pocos clics.

## <a name="business-need"></a>Necesidad empresarial

La mayoría de las organizaciones ya son conscientes del valor de los datos que tienen en distintos formatos (PDF, imágenes o vídeos). Buscan los procedimientos recomendados y las maneras más rentables de digitalizar esos recursos.

Además, nuestros clientes a menudo tienen distintos tipos de formularios que proceden a su vez de sus propios clientes. A diferencia de los [inicios rápidos](./quickstarts/client-library.md), en este tutorial se muestra cómo entrenar automáticamente un modelo con nuevos y diferentes tipos de formularios utilizando un enfoque basado en metadatos. Si existe ningún modelo para el tipo de formulario dado, el sistema lo creará y le proporcionará el identificador del mismo. 

Mediante la extracción de los datos de los formularios y su combinación con los sistemas operativos y almacenes de datos existentes, las empresas pueden obtener información y aportar valor a sus clientes y usuarios empresariales.

Azure Form Recognizer ayuda a las organizaciones a usar sus datos, automatizar procesos (pago de facturas, procesamiento de impuestos, etc.), ahorrar tiempo y dinero, y disfrutar de una mejor precisión de los datos.

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Configurar Azure Data Lake para almacenar los formularios.
> * Usar una base de datos de Azure para crear una tabla de parametrización.
> * Usar Azure Key Vault para almacenar credenciales confidenciales.
> * Entrenar el modelo de Form Recognizer en un cuaderno de Azure Databricks.
> * Extraer datos del formulario mediante un cuaderno de Databricks.
> * Automatizar el entrenamiento y la extracción de formularios mediante Azure Data Factory.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. [cree una de forma gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Después de que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Creación de un recurso de Form Recognizer"  target="_blank">cree un recurso de Form Recognizer </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez implementado el recurso, seleccione **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a la API Form Recognizer. Tanto la clave como el punto de conexión los pegará en el código más adelante.
    * Puede usar el plan de tarifa gratuito (F0) para probar el servicio. Luego, puede realizar la actualización a un nivel de pago para producción.
* Un conjunto de al menos cinco formularios del mismo tipo. Idealmente, este flujo de trabajo está pensado para admitir grandes conjuntos de documentos. En [Creación de un conjunto de datos de entrenamiento](./build-training-data-set.md) encontrará sugerencias y opciones para reunir un conjunto de datos de entrenamiento. En este tutorial puede usar los archivos de la carpeta Train del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2128080).


## <a name="project-architecture"></a>Arquitectura de proyecto 

Este proyecto representa un conjunto de canalizaciones de Azure Data Factory que desencadenan cuadernos de Python que entrenan, analizan y extraen datos de los documentos de una cuenta de Azure Data Lake Storage.

Form Recognizer API REST requiere algunos parámetros como entrada. Por motivos de seguridad, algunos de estos parámetros se almacenarán en Azure Key Vault. Otros parámetros menos confidenciales, como el nombre de la carpeta del blob de almacenamiento, se almacenarán en la tabla de parametrización de una base de datos de Azure SQL.

Para cada tipo de formulario que se vaya a analizar, los ingenieros de datos o los científicos de datos rellenarán una fila de la tabla de parámetros. Después, usarán Azure Data Factory para iterar por la lista de tipos de formulario detectados y pasar los parámetros pertinentes a un cuaderno de Databricks para entrenar o volver a entrenar los modelos de Form Recognizer. También se puede usar una función de Azure aquí.

Luego, el cuaderno de Azure Databricks usa los modelos entrenados para extraer los datos del formulario. Exporta los datos a una base de datos de Azure SQL.

:::image type="content" source="./media/tutorial-bulk-processing/architecture.png" alt-text="Diagrama que muestra la arquitectura del proyecto.":::


## <a name="set-up-azure-data-lake"></a>Configuración de Azure Data Lake Storage

El trabajo pendiente de los formularios puede estar en el entorno local o en uno o varios servidores FTP. En este tutorial se usan los formularios en una cuenta de Azure Data Lake Storage Gen2. Parea transferir los archivos allí, utilice Azure Data Factory, el Explorador de Azure Storage o AzCopy. Los conjuntos de datos de entrenamiento y puntuación pueden estar en diferentes contenedores, pero los conjuntos de datos de entrenamiento de todos los tipos de formularios deben estar en el mismo contenedor (aunque pueden estar en carpetas diferentes).

Para crear un lago de datos, siga las instrucciones del artículo [Creación de una cuenta de almacenamiento para su uso con Azure Data Lake Storage Gen2 habilitado](../../storage/blobs/create-data-lake-storage-account.md).

## <a name="create-a-parameterization-table"></a>Creación de una tabla de parametrización

A continuación, vamos a crear una tabla de metadatos en una base de datos de Azure SQL. Esta tabla contendrá los datos no confidenciales que requiere la API REST Form Recognizer. Siempre que haya un nuevo tipo de formulario en el conjunto de datos, se insertará un nuevo registro en esta tabla y se desencadenarán las canalizaciones de entrenamiento y puntuación (que se implementarán más adelante).

En la tabla se usarán estos campos:

* **form_description**. No es necesario como parte del entrenamiento. Proporciona una descripción del tipo de formulario para el que se va a entrenar el modelo (por ejemplo, los "formularios del cliente A" o los "formularios del Hotel B").
* **training_container_name**. Nombre del contenedor de la cuenta de almacenamiento donde se ha almacenado el conjunto de información de entrenamiento. Puede ser el mismo contenedor que el de **scoring_container_name**.
* **training_blob_root_folder**. la carpeta de la cuenta de almacenamiento donde se almacenarán los archivos para el entrenamiento del modelo.
* **scoring_container_name**. Nombre del contenedor de la cuenta de almacenamiento en que se han almacenado los archivos de los que se desea extraer los pares clave-valor. Puede ser el mismo contenedor que el de **training_container_name**.
* **scoring_input_blob_folder**. la carpeta de la cuenta de almacenamiento donde se almacenarán los archivos de los que se van a extraer los datos.
* **model_id**. Identificador del modelo que se desea volver a entrenar. En la primera ejecución, el valor debe establecerse en -1, lo que hará que el cuaderno de entrenamiento cree un modelo personalizado para entrenar. El cuaderno de entrenamiento devolverá el nuevo identificador de modelo a la instancia de Azure Data Factory. Mediante el uso de una actividad de procedimiento almacenado, se actualizará este valor en la base de datos Azure SQL.

  Cada vez que desee ingerir un nuevo tipo de formulario, deberá restablecer manualmente el identificador del modelo en -1 antes de entrenar el modelo.

* **file_type**. Los tipos de formulario admitidos son `application/pdf`, `image/jpeg`, `image/png` y `image/tif`.

  Si tiene formularios en otros tipos de archivo, deberá cambiar este valor y el de **model_id** al entrenar un nuevo tipo de formulario.
* **form_batch_group_id**. Con el tiempo, es posible que tenga varios tipos de formulario que entrena con el mismo modelo. El campo **form_batch_group_id** le permitirá especificar todos los tipos de formulario que se han entrenado con un modelo concreto.

### <a name="create-the-table"></a>Creación de la tabla


[Cree una base de datos de Azure SQL](https://ms.portal.azure.com/#create/Microsoft.SQLDatabase) y, a continuación, ejecute este script SQL en el [editor de consultas](../../azure-sql/database/connect-query-portal.md) para crear la tabla necesaria:


```sql
CREATE TABLE dbo.ParamFormRecogniser(
    form_description varchar(50) NULL,
  training_container_name varchar(50) NOT NULL,
    training_blob_root_folder varchar(50) NULL,
    scoring_container_name varchar(50) NOT NULL,
    scoring_input_blob_folder varchar(50) NOT NULL,
    scoring_output_blob_folder varchar(50) NOT NULL,
    model_id varchar(50) NULL,
    file_type varchar(50) NULL
) ON PRIMARY
GO
```

Ejecute este script para crear el procedimiento que actualiza automáticamente **model_id** después del entrenamiento.

```SQL
CREATE PROCEDURE [dbo].[update_model_id] ( @form_batch_group_id  varchar(50),@model_id varchar(50))
AS
BEGIN 
    UPDATE [dbo].[ParamFormRecogniser]   
        SET [model_id] = @model_id  
    WHERE form_batch_group_id =@form_batch_group_id
END
```

## <a name="use-azure-key-vault-to-store-sensitive-credentials"></a>Usar Azure Key Vault para almacenar credenciales confidenciales

Por motivos de seguridad, no queremos almacenar cierta información confidencial en la tabla de parametrización de la base de datos de Azure SQL. Almacenaremos los parámetros confidenciales como secretos de Azure Key Vault.

### <a name="create-an-azure-key-vault"></a>Creación de una instancia de Azure Key Vault

[Creación de un recurso de Key Vault](https://ms.portal.azure.com/#create/Microsoft.KeyVault). Vaya al recurso de Key Vault una vez que se cree y, en la sección **Configuración**, seleccione **Secretos** para agregar los parámetros.

Aparecerá una ventana nueva. Seleccione **Generar o importar**. Escriba el nombre del parámetro y su valor, y seleccione **Crear**. Complete estos pasos para los siguientes parámetros:

* **CognitiveServiceEndpoint**. La dirección URL del punto de conexión de la API Form Recognizer.
* **CognitiveServiceSubscriptionKey**. La clave de acceso para el servicio Form Recognizer. 
* **StorageAccountName**. La cuenta de almacenamiento donde se almacenan el conjunto de datos de entrenamiento y los formularios de los que desea extraer los pares clave-valor. Si estos elementos se encuentran en cuentas diferentes, escriba cada nombre de cuenta como un secreto independiente. Recuerde que los conjuntos de datos de entrenamiento deben estar en el mismo contenedor para todos los tipos de formulario. Pueden estar en carpetas diferentes.
* **StorageAccountSasKey**. La firma de acceso compartido (SAS) de la cuenta de almacenamiento. Para recuperar la dirección URL de SAS, vaya al recurso de almacenamiento. En la pestaña **Explorador de Azure Storage**, vaya al contenedor, haga clic con el botón derecho y seleccione **Obtener firma de acceso compartido**. Es importante obtener la firma de acceso compartido para el contenedor, no para la propia cuenta de almacenamiento. Asegúrese de que los permisos de **lectura** y **enumeración** están seleccionados y haga clic en **Crear**. A continuación, copie el valor en la sección **URL**. Debe tener este formato: `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.

## <a name="train-your-form-recognizer-model-in-a-databricks-notebook"></a>Entrenar el modelo de Form Recognizer en un cuaderno de Databricks

Usará Azure Databricks para almacenar y ejecutar el código de Python que interactúa con el servicio Form Recognizer.

### <a name="create-a-notebook-in-databricks"></a>Creación de un cuaderno en Databricks

[Cree un recurso de Azure Databricks](https://ms.portal.azure.com/#create/Microsoft.Databricks) en Azure Portal. Una vez que haya creado el recurso vaya a él y abra el área de trabajo.

### <a name="create-a-secret-scope-backed-by-azure-key-vault"></a>Creación de un ámbito de secreto respaldado por Azure Key Vault


Para hacer referencia a los secretos de la instancia de Azure Key Vault que ha creado anteriormente, tendrá que crear un ámbito secreto en Databricks. Siga los pasos de este artículo: [Creación de un ámbito de secreto compatible con Azure Key Vault](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope).

### <a name="create-a-databricks-cluster"></a>Creación de un clúster de Databricks

Un clúster es una colección de recursos de cálculo de Databricks. Para crear un clúster:

1. En el panel izquierdo, seleccione el botón **Clústeres**.
1. En la página **Clústeres**, seleccione **Crear clúster**.
1. En la página **Crear clúster**, especifique un nombre de clúster y seleccione **7.2 (Scala 2.12, Spark 3.0.0)** en la lista **Databricks Runtime Version** (Versión del runtime de Databricks).
1. Seleccione **Crear clúster**.

### <a name="write-a-settings-notebook"></a>Escritura de un cuaderno de configuración

Ya está listo para agregar cuadernos de Python. En primer lugar, cree un cuaderno denominado **Settings**. Este cuaderno asignará los valores de la tabla de parametrización a las variables del script. Posteriormente, Azure Data Factory pasará los valores como parámetros. También se asignarán los valores de los secretos del almacén de claves a variables. 

1. Para crear el cuaderno **Settings**, seleccione el botón **Área de trabajo**. En la pestaña nueva, seleccione la lista desplegable, luego seleccione **Crear** y, finalmente, **Cuaderno**.
1. En la ventana emergente, escriba el nombre del cuaderno y seleccione **Python** como lenguaje predeterminado. Seleccione un clúster de Databricks y, después, seleccione **Crear**.
1. En la primera celda del cuaderno se recuperan los parámetros que ha pasado Azure Data Factory:

    ```python 
    dbutils.widgets.text("form_batch_group_id", "","")
    dbutils.widgets.get("form_batch_group_id")
    form_batch_group_id = getArgument("form_batch_group_id")
    
    dbutils.widgets.text("model_id", "","")
    dbutils.widgets.get("model_id")
    model_id = getArgument("model_id")
    
    dbutils.widgets.text("training_container_name", "","")
    dbutils.widgets.get("training_container_name")
    training_container_name = getArgument("training_container_name")
    
    dbutils.widgets.text("training_blob_root_folder", "","")
    dbutils.widgets.get("training_blob_root_folder")
    training_blob_root_folder=  getArgument("training_blob_root_folder")
    
    dbutils.widgets.text("scoring_container_name", "","")
    dbutils.widgets.get("scoring_container_name")
    scoring_container_name=  getArgument("scoring_container_name")
    
    dbutils.widgets.text("scoring_input_blob_folder", "","")
    dbutils.widgets.get("scoring_input_blob_folder")
    scoring_input_blob_folder=  getArgument("scoring_input_blob_folder")
    
    
    dbutils.widgets.text("file_type", "","")
    dbutils.widgets.get("file_type")
    file_type = getArgument("file_type")
    
    dbutils.widgets.text("file_to_score_name", "","")
    dbutils.widgets.get("file_to_score_name")
    file_to_score_name=  getArgument("file_to_score_name")
    ```

1. En la segunda celda, se recuperan los secretos de Key Vault y se asignan a variables:

    ```python 
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    training_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    ScoredFile = file_to_score_name+ "_output.json"
    training_storage_account_url="https://"+training_storage_account_name+".blob.core.windows.net/"+training_container_name+storage_account_sas_key 
    ```

### <a name="write-a-training-notebook"></a>Escritura de un cuaderno de entrenamiento

Ahora que hemos completado la creación del cuaderno **Settings**, se puede crear un cuaderno para entrenar el modelo. Como ya se ha indicado, usaremos los archivos almacenados en una carpeta en una cuenta de almacenamiento de Azure Data Lake Storage Gen2 (**training_blob_root_folder**). El nombre de la carpeta se ha pasado como una variable. Cada conjunto de tipos de formulario estará en la misma carpeta. Al recorrer la tabla de parámetros, entrenaremos el modelo, para lo que usaremos todos los tipos de formulario. 

1. Cree un cuaderno llamado **TrainFormRecognizer**. 
1. En la primera celda, ejecute el cuaderno **Settings**:

    ```python
    %run "./Settings"
    ```

1. En la celda siguiente, asigne las variables del archivo Settings y entrene dinámicamente el modelo para cada tipo de formulario, aplicando el código del [inicio rápido de REST](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-train-extract.md#get-training-results%20).

    ```python
    import json
    import time
    from requests import get, post
    
    post_url = cognitive_service_endpoint + r"/formrecognizer/v2.0/custom/models"
    source = training_storage_account_url
    prefix= training_blob_root_folder
    
    includeSubFolders=True
    useLabelFile=False
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    body =     {
        "source": source
        ,"sourceFilter": {
            "prefix": prefix,
            "includeSubFolders": includeSubFolders
       },
    }
    if model_id=="-1": # If you don't already have a model you want to retrain. In this case, we create a model and use it to extract the key/value pairs.
      try:
          resp = post(url = post_url, json = body, headers = headers)
          if resp.status_code != 201:
              print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
              quit()
          print("POST model succeeded:\n%s" % resp.headers)
          get_url = resp.headers["location"]
          model_id=get_url[get_url.index('models/')+len('models/'):]     
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    else :# If you already have a model you want to retrain, we reuse it and (re)train with the new form types.  
      try:
        get_url =post_url+r"/"+model_id
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    ```

1. El último paso del proceso de entrenamiento es obtener el resultado del entrenamiento en formato JSON:

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
        try:
            resp = get(url = get_url, headers = headers)
            resp_json = resp.json()
            print (resp.status_code)
            if resp.status_code != 200:
                print("GET model failed (%s):\n%s" % (resp.status_code, json.dumps(resp_json)))
                n_try += 1
                quit()
            model_status = resp_json["modelInfo"]["status"]
            print (model_status)
            if model_status == "ready":
                print("Training succeeded:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            if model_status == "invalid":
                print("Training failed. Model is invalid:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            # Training still running. Wait and retry.
            time.sleep(wait_sec)
            n_try += 1
            wait_sec = min(2*wait_sec, max_wait_sec)     
            print (n_try)
        except Exception as e:
            msg = "GET model failed:\n%s" % str(e)
            print(msg)
            quit()
    print("Train operation did not complete within the allocated time.")
    ```

## <a name="extract-form-data-by-using-a-notebook"></a>Extracción de datos de formularios mediante un cuaderno

### <a name="mount-the-azure-data-lake-storage"></a>Montaje de Azure Data Lake Storage

El siguiente paso es puntuar los diferentes formularios que tenemos, para lo que se usa el modelo entrenado. Se montará la cuenta de Azure Data Lake Storage en Databricks y se hará referencia al montaje durante el proceso de ingesta.

Tal como se hizo en la fase de entrenamiento, se usará Azure Data Factory para invocar la extracción de los pares clave-valor de los formularios. Se recorrerán los formularios de las carpetas que se especifican en la tabla de parámetros.

1. Cree el cuaderno para montar la cuenta de almacenamiento en Databricks. Llámelo **MountDataLake**. 
1. Debe llamar primero al cuaderno **Settings**:

    ```python
    %run "./Settings"
    ```

1. En la segunda celda, se definen variables para los parámetros confidenciales, los cuales se recuperarán de nuestros secretos de Key Vault:

    ```python
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    scoring_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    scoring_storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    scoring_mount_point = "/mnt/"+scoring_container_name 
    scoring_source_str = "wasbs://{container}@{storage_acct}.blob.core.windows.net/".format(container=scoring_container_name, storage_acct=scoring_storage_account_name) 
    scoring_conf_key = "fs.azure.sas.{container}.{storage_acct}.blob.core.windows.net".format(container=scoring_container_name, storage_acct=scoring_storage_account_name)
    
    ```

1. Intente desmontar la cuenta de almacenamiento, en caso de que se hubiera montado previamente:

    ```python
    try:
      dbutils.fs.unmount(scoring_mount_point) # Use this to unmount as needed
    except:
      print("{} already unmounted".format(scoring_mount_point))
    
    ```

1. Monte la cuenta de almacenamiento:


    ```python
    try: 
      dbutils.fs.mount( 
        source = scoring_source_str, 
        mount_point = scoring_mount_point, 
        extra_configs = {scoring_conf_key: scoring_storage_account_sas_key} 
      ) 
    except Exception as e: 
      print("ERROR: {} already mounted. Run previous cells to unmount first".format(scoring_mount_point))
    
    ```

    > [!NOTE]
    > Se ha montado solo la cuenta de almacenamiento de entrenamiento. En este caso, los archivos de entrenamiento y los archivos de los que se quieren extraer pares clave-valor se encuentran en la misma cuenta de almacenamiento. Si las cuentas de almacenamiento de puntuación y entrenamiento son diferentes, tendrá que montar ambas. 

### <a name="write-the-scoring-notebook"></a>Escritura del cuaderno de puntuación

Ahora se puede crear un cuaderno de puntuación. Se va a llevar a cabo un proceso similar al que se realizó en el cuaderno de entrenamiento: se usarán los archivos almacenados en las carpetas de la cuenta de Azure Data Lake Storage que se acaba de montar. El nombre de la carpeta se pasa como una variable. Se recorren todos los formularios de la carpeta especificada y se extraen los pares clave/valor de ellos. 

1. Cree un cuaderno y llámelo **ScoreFormRecognizer**. 
1. Ejecute los cuadernos **Settings** y **MountDataLake**:

    ```python
    %run "./Settings"
    %run "./MountDataLake"
    ```

1. Agregue este código, que llama a [Analyze](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm) API:

    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post 
    
    
    #prefix= TrainingBlobFolder
    post_url = cognitive_service_endpoint + "/formrecognizer/v2.0/custom/models/%s/analyze" % model_id
    source = r"/dbfs/mnt/"+scoring_container_name+"/"+scoring_input_blob_folder+"/"+file_to_score_name
    output = r"/dbfs/mnt/"+scoring_container_name+"/scoringforms/ExtractionResult/"+os.path.splitext(os.path.basename(source))[0]+"_output.json"
    
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```

1. En la celda siguiente, se obtendrán los resultados de la extracción de los pares clave/valor. Esta celda generará el resultado. Queremos el resultado en formato JSON para poder procesarlo en Azure SQL Database o Azure Cosmos DB. Por consiguiente, escribiremos el resultado en un archivo .json. El nombre del archivo de salida será el nombre del archivo puntuado, al que se habrá agregado "_output.json". El archivo se almacenará en la misma carpeta que el archivo de origen.

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
       try:
           resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": cognitive_service_subscription_key})
           resp_json = resp.json()
           if resp.status_code != 200:
               print("GET analyze results failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           status = resp_json["status"]
           if status == "succeeded":
               print("Analysis succeeded:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           if status == "failed":
               print("Analysis failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           # Analysis still running. Wait and retry.
           time.sleep(wait_sec)
           n_try += 1
           wait_sec = min(2*wait_sec, max_wait_sec)     
       except Exception as e:
           msg = "GET analyze results failed:\n%s" % str(e)
           print(msg)
           n_try += 1
           print("Analyze operation did not complete within the allocated time.")
           quit()
    
    ```
1. Realice el procedimiento de escritura de archivo en una nueva celda:

    ```python
    import requests
    file = open(output, "w")
    file.write(str(resp_json))
    file.close()
    ```

## <a name="automate-training-and-scoring-by-using-azure-data-factory"></a>Automatización del entrenamiento y la puntuación mediante Azure Data Factory

Solo queda configurar el servicio Azure Data Factory (ADF) para automatizar los procesos de entrenamiento y puntuación. En primer lugar, siga los pasos que se indican en [Crear una factoría de datos](../../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory). Después de crear el recurso de Azure Data Factory, tiene que crear tres canalizaciones: una para el entrenamiento y dos para la puntuación (se explicará más adelante).

### <a name="training-pipeline"></a>Canalización de entrenamiento

La primera actividad de la canalización de entrenamiento es una búsqueda para leer y devolver los valores de la tabla de parametrización en la base de datos de Azure SQL. Todos los conjuntos de datos de formación estarán en la misma cuenta de almacenamiento y contenedor (pero potencialmente en distintas carpetas). Por consiguiente, se conservará el valor del atributo predeterminado **First row only** (Solo la primera fila) en la configuración de la actividad de búsqueda. Para todos los tipos de formulario en los que se entrena el modelo, se utilizarán todos los archivos de **training_blob_root_folder**.

:::image type="content" source="./media/tutorial-bulk-processing/training-pipeline.png" alt-text="Captura de pantalla que muestra una canalización de entrenamiento en Data Factory.":::

El procedimiento almacenado toma dos parámetros: **model_id** y **form_batch_group_id**. El código para devolver el identificador del modelo del cuaderno de Databricks es `dbutils.notebook.exit(model_id)`. El código para leer el código de la actividad de procedimiento almacenado en Data Factory es `@activity('GetParametersFromMetadaTable').output.firstRow.form_batch_group_id`.

### <a name="scoring-pipelines"></a>Canalizaciones de puntuación

Para extraer los pares clave/valor, analizaremos todas las carpetas de la tabla de parametrización. Se extraerán los pares clave/valor de todos los archivos de todas las carpetas. Actualmente, Azure Data Factory no admite bucles ForEach anidados. En su lugar, se crearán dos canalizaciones. La primera canalización realizará la búsqueda en la tabla de parametrización y pasará la lista de carpetas como un parámetro a la segunda canalización.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1a.png" alt-text="Captura de pantalla que muestra la primera canalización de puntuación de Data Factory.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1b.png" alt-text="Captura de pantalla que muestra los detalles de la primera canalización de puntuación de Data Factory.":::

La segunda canalización usará una actividad GetMeta para obtener la lista de los archivos de la carpeta y pasarla como un parámetro al cuaderno de puntuación de Databricks.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2a.png" alt-text="Captura de pantalla que muestra la segunda canalización de puntuación de Data Factory.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2b.png" alt-text="Captura de pantalla que muestra los detalles de la segunda canalización de puntuación de Data Factory.":::

### <a name="specify-a-degree-of-parallelism"></a>Especificación de un grado de paralelismo

Tanto en la canalización de entrenamiento como en la de puntuación se puede especificar el grado de paralelismo, con el fin de que se puedan procesar varios formularios simultáneamente.

Para establecer el grado de paralelismo en la canalización de Azure Data Factory:

1. Seleccione la actividad **ForEach**.
1. Desactive la casilla **Secuencial**.
1. Establezca el grado de paralelismo en el cuadro **Batch count** (Número de lotes). Se recomienda un número máximo de lotes de 15 para la puntuación.

:::image type="content" source="./media/tutorial-bulk-processing/parallelism.png" alt-text="Captura de pantalla que muestra la configuración del paralelismo para la actividad de puntuación de Azure Data Factory.":::

## <a name="how-to-use-the-pipeline"></a>Procedimientos para usar la canalización

Ahora tiene una canalización automatizada para digitalizar el trabajo pendiente de formularios y ejecutar algunos análisis. Cuando agregue formularios nuevos de un tipo que sea familiar a una carpeta de almacenamiento existente, no tiene más que volver a ejecutar las canalizaciones de puntuación. Estas actualizarán todos los archivos de salida, incluidos los de los nuevos formularios. 

Si agrega formularios de un nuevo tipo, también tendrá que cargar un conjunto de datos de entrenamiento en el contenedor adecuado. Después, agregue una nueva fila a la tabla de parametrización y especifique las ubicaciones de los nuevos documentos y su conjunto de datos de entrenamiento. Escriba el valor -1 para **model_ID**, con el fin de indicar que es preciso entrenar un nuevo modelo para los formularios. A continuación, ejecute la canalización de entrenamiento en Azure Data Factory. Esta realizará la lectura de la tabla, entrenará un modelo y sobrescribirá el identificador del modelo en la tabla. Después, puede llamar a las canalizaciones de puntuación para empezar a escribir los archivos de salida.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado canalizaciones de Azure Data Factory que desencadenan el entrenamiento y la ejecución de modelos de Form Recognizer para digitalizar una gran cantidad de trabajo pendiente de archivos. A continuación, explore la API Form Recognizer para ver qué más puede hacer.

* [API REST Form Recognizer](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeBusinessCardAsync)
