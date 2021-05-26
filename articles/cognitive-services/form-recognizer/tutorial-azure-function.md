---
title: 'Tutorial: Uso de una instancia de Azure Functions para procesar documentos almacenados'
titleSuffix: Azure Applied AI Services
description: En esta guía se muestra cómo usar una función de Azure para desencadenar el procesamiento de documentos que se han cargado en un contenedor de Azure Blob Storage.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 03/19/2021
ms.author: lajanuar
ms.openlocfilehash: 6a8f3cf5b98361f96df26da923ff6bff7a604d23
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374005"
---
# <a name="tutorial-use-an-azure-function-to-process-stored-documents"></a>Tutorial: Uso de una instancia de Azure Functions para procesar documentos almacenados

Puede usar Form Recognizer como parte de una canalización de procesamiento de datos automatizada que haya creado con Azure Functions. En esta guía se muestra cómo usar una función de Azure para procesar documentos que se han cargado en un contenedor de Azure Blob Storage. Este flujo de trabajo extrae los datos de tabla de los documentos almacenados mediante el servicio de diseño de Form Recognizer y guarda los datos de la tabla en un archivo .csv en Azure. A continuación, puede mostrar los datos mediante Microsoft Power BI (no se explica en este documento).

> [!div class="mx-imgBorder"]
> ![diagrama del flujo de trabajo del servicio de Azure](./media/tutorial-azure-function/workflow-diagram.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una cuenta de Azure Storage
> * Creación de un proyecto de Azure Functions
> * Extraer datos del diseño de los formularios cargados
> * Cargar datos del diseño en Azure Storage

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Creación de un recurso de Form Recognizer"  target="_blank">cree un recurso de Form Recognizer <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión de Form Recognizer. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a la API Form Recognizer. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Un documento PDF local que analizará. Puede descargar y usar este [documento de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/sample-layout.pdf).
* [Python 3.8.x](https://www.python.org/downloads/) instalado.
* [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) instalado.
* [Azure Functions Core Tools](../../azure-functions/functions-run-local.md?tabs=windows%2ccsharp%2cbash#install-the-azure-functions-core-tools) instalado.
* Visual Studio Code con las siguientes extensiones instaladas:
  * [Extensión de Azure Functions](/azure/developer/python/tutorial-vs-code-serverless-python-01#visual-studio-code-python-and-the-azure-functions-extension)
  * [Extensión de Python](https://code.visualstudio.com/docs/python/python-tutorial#_install-visual-studio-code-and-the-python-extension)

## <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

[Cree una cuenta de Azure Storage](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM) en Azure Portal. Seleccione **StorageV2** como el Tipo de cuenta.

En el panel izquierdo, seleccione la pestaña **CORS** y quite la directiva CORS existente, si hay alguna.

Una vez finalizada la implementación, cree dos contenedores de almacenamiento de blobs vacíos, con los nombres **test** y **output**.

## <a name="create-an-azure-functions-project"></a>Creación de un proyecto de Azure Functions

Abra Visual Studio Code. Si ha instalado la extensión de Azure Functions, debería ver un logotipo de Azure en el panel de navegación de la izquierda. Selecciónelo. Cree un nuevo proyecto y, cuando se le pida, cree una carpeta local denominada **coa_new** para el proyecto.

![Botón Crear función de VS Code](./media/tutorial-azure-function/vs-code-create-function.png)


Se le pedirá que configure una serie de valores:
* En la solicitud **Seleccionar un lenguaje**, seleccione Python.
* En la solicitud **Seleccionar una plantilla**, seleccione el desencadenador de Azure Blob Storage. A continuación, asigne un nombre al desencadenador predeterminado.
* En la solicitud **Seleccionar configuración**, elija crear una nueva configuración de aplicación local.
* Seleccione la suscripción a Azure que contiene la cuenta de almacenamiento que creó. A continuación, debe escribir el nombre del contenedor de almacenamiento (en este caso, `test/{name}`).
* Elija abrir el proyecto en la ventana actual. 

![Ejemplo de solicitud de creación de VS Code](./media/tutorial-azure-function/vs-code-prompt.png)

Cuando haya completado estos pasos, VS Code agregará un nuevo proyecto de Azure Functions con un script de Python *\_\_init\_\_.py*. Este script se desencadenará cuando se cargue un archivo en el contenedor de almacenamiento **test**, pero no hará nada.

## <a name="test-the-function"></a>Probar la función

Presione F5 para ejecutar la función básica. VS Code le pedirá que seleccione una cuenta de almacenamiento con la que interactuar. Seleccione la cuenta de almacenamiento que ha creado y continúe.

Abra el Explorador de Azure Storage y cargue un documento PDF de ejemplo en el contenedor **Test**. A continuación, compruebe el terminal de VS Code. El script debe registrar que la carga del archivo PDF lo desencadenó.

![Prueba de terminal de VS Code](./media/tutorial-azure-function/vs-code-terminal-test.png)


Detenga el script antes de continuar.

## <a name="add-form-processing-code"></a>Adición de código de procesamiento de formularios

A continuación, agregará su propio código al script de Python para llamar al servicio de Form Recognizer y analizar los documentos cargados mediante la [API de diseño](concept-layout.md) de Form Recognizer.

En VS Code, navegue hasta el archivo *requirements.txt* de la función. Este define las dependencias para el script. Agregue los siguientes paquetes de Python al archivo:

```
cryptography
azure-functions
azure-storage-blob
azure-identity
requests
pandas
numpy
```

A continuación, abra el script *\_\_init\_\_.py*. Agregue las instrucciones siguientes `import` :

```Python
import logging
from azure.storage.blob import BlobServiceClient
import azure.functions as func
import json
import time
from requests import get, post
import os
from collections import OrderedDict
import numpy as np
import pandas as pd
```

Puede dejar la función `main` generada tal cual está. Agregará el código personalizado dentro de esta función.

```python
# This part is automatically generated
def main(myblob: func.InputStream):
    logging.info(f"Python blob trigger function processed blob \n"
    f"Name: {myblob.name}\n"
    f"Blob Size: {myblob.length} bytes")
```

El siguiente bloque de código llama a la API de [análisis de diseño](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1/operations/AnalyzeLayoutAsync) de Form Recognizer en el documento cargado. Rellene los valores de punto de conexión y clave. 


# <a name="version-20"></a>[versión 2.0](#tab/2-0)

```Python
# This is the call to the Form Recognizer endpoint
    endpoint = r"Your Form Recognizer Endpoint"
    apim_key = "Your Form Recognizer Key"
    post_url = endpoint + "/formrecognizer/v2.0/Layout/analyze"
    source = myblob.read()

    headers = {
    # Request headers
    'Content-Type': 'application/pdf',
    'Ocp-Apim-Subscription-Key': apim_key,
        }

    text1=os.path.basename(myblob.name)
```

# <a name="version-21"></a>[versión 2.1](#tab/2-1)

```Python
# This is the call to the Form Recognizer endpoint
    endpoint = r"Your Form Recognizer Endpoint"
    apim_key = "Your Form Recognizer Key"
    post_url = endpoint + "/formrecognizer/v2.1/Layout/analyze"
    source = myblob.read()

    headers = {
    # Request headers
    'Content-Type': 'application/pdf',
    'Ocp-Apim-Subscription-Key': apim_key,
        }

    text1=os.path.basename(myblob.name)
```
---

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Form Recognizer que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**. 
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad de Cognitive Services](../cognitive-services-security.md).

A continuación, agregue el código para consultar al servicio y obtener los datos devueltos. 


```Python
resp = requests.post(url = post_url, data = source, headers = headers)
    if resp.status_code != 202:
        print("POST analyze failed:\n%s" % resp.text)
        quit()
    print("POST analyze succeeded:\n%s" % resp.headers)
    get_url = resp.headers["operation-location"]

    wait_sec = 25
    
    time.sleep(wait_sec)
    # The layout API is async therefore the wait statement
    
    resp =requests.get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
    
    resp_json = json.loads(resp.text)
    
    
    status = resp_json["status"]
    
    
    if status == "succeeded":
        print("Layout Analysis succeeded:\n%s")
        results=resp_json
    else:
        print("GET Layout results failed:\n%s")
        quit()

    results=resp_json
```

Después, agregue el siguiente código para conectarse al contenedor **output** de Azure Storage. Escriba sus propios valores para el nombre y la clave de la cuenta de almacenamiento. Puede obtener la clave en la pestaña **Claves de acceso** del recurso de almacenamiento en Azure Portal.

```Python
# This is the connection to the blob storage, with the Azure Python SDK
    blob_service_client = BlobServiceClient.from_connection_string("DefaultEndpointsProtocol=https;AccountName="Storage Account Name";AccountKey="storage account key";EndpointSuffix=core.windows.net")
    container_client=blob_service_client.get_container_client("output")
```

El código siguiente analiza la respuesta que devuelve Form Recognizer, crea un archivo. csv y lo carga en el contenedor **output**. 


> [!IMPORTANT]
> Probablemente tendrá que editar este código para que coincida con la estructura de sus documentos de formulario.

```Python
# The code below is how I extract the json format into tabular data 
    # Please note that you need to adjust the code below to your form structure
    # It probably won't work out-of-box for your specific form
    pages = results["analyzeResult"]["pageResults"]

    def make_page(p):
        res=[]
        res_table=[]
        y=0
        page = pages[p]
        for tab in page["tables"]:
            for cell in tab["cells"]:
                res.append(cell)
                res_table.append(y)
            y=y+1
    
        res_table=pd.DataFrame(res_table)
        res=pd.DataFrame(res)
        res["table_num"]=res_table[0]
        h=res.drop(columns=["boundingBox","elements"])
        h.loc[:,"rownum"]=range(0,len(h))
        num_table=max(h["table_num"])
        return h, num_table, p

    h, num_table, p= make_page(0)   

    for k in range(num_table+1):
        new_table=h[h.table_num==k]
        new_table.loc[:,"rownum"]=range(0,len(new_table))
        row_table=pages[p]["tables"][k]["rows"]
        col_table=pages[p]["tables"][k]["columns"]
        b=np.zeros((row_table,col_table))
        b=pd.DataFrame(b)
        s=0
        for i,j in zip(new_table["rowIndex"],new_table["columnIndex"]):
            b.loc[i,j]=new_table.loc[new_table.loc[s,"rownum"],"text"]
            s=s+1

```

Por último, el último bloque de código carga la tabla y los datos de texto que se extrajeron en el elemento de almacenamiento de blobs.

```Python
    # Here is the upload to the blob storage
    tab1_csv=b.to_csv(header=False,index=False,mode='w')
    name1=(os.path.splitext(text1)[0]) +'.csv'
    container_client.upload_blob(name=name1,data=tab1_csv)
```

## <a name="run-the-function"></a>Ejecución de la función

Presione F5 para ejecutar la función nuevamente. Use el Explorador de Azure Storage para cargar un formulario PDF de ejemplo en el contenedor de almacenamiento **test**. Esta acción debe desencadenar la ejecución del script y, a continuación, se deberá mostrar el archivo .csv resultante (mostrado como una tabla) en el contenedor **output**.

Puede conectar este contenedor a Power BI para crear visualizaciones enriquecidas de los datos que contiene.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a usar una función de Azure escrita en Python para procesar automáticamente los documentos PDF cargados y devolver el contenido con un formato más fácil de usar. A continuación, obtenga información sobre cómo usar Power BI para mostrar esos datos.

> [!div class="nextstepaction"]
> [Microsoft Power BI](https://powerbi.microsoft.com/integrations/azure-table-storage/)

* [¿Qué es Form Recognizer?](overview.md)
* Más información acerca de la [API de diseño](concept-layout.md).