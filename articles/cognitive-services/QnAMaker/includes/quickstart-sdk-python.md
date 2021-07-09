---
title: 'Inicio rápido: Biblioteca cliente de QnA Maker para Python'
description: En este inicio rápido se muestra cómo empezar a trabajar con la biblioteca cliente de QnA Maker para Python.
ms.topic: include
ms.date: 06/18/2020
ms.openlocfilehash: d64946d60162f322d940d4bee7203bfd4b77a4f5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110486834"
---
# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Use la biblioteca cliente de QnA Maker para Python para:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Obtener la clave del punto de conexión de tiempo de ejecución de predicción
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Documentación de referencia](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-knowledge-qnamaker) | [Paquete (PyPi)](https://pypi.org/project/azure-cognitiveservices-knowledge-qnamaker/0.2.0/) | [Ejemplos de Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/QnAMaker/sdk/quickstart.py)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Use la biblioteca cliente de QnA Maker para Python para:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Documentación de referencia](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-knowledge-qnamaker) | [Paquete (PyPi)](https://pypi.org/project/azure-cognitiveservices-knowledge-qnamaker/) | [Ejemplos de Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/QnAMaker/sdk/preview-sdk/quickstart.py)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Requisitos previos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [Python 3.x](https://www.python.org/)
* Cuando tenga la suscripción a Azure, cree un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) en Azure Portal para obtener la clave de creación y el punto de conexión. Tras su implementación, seleccione **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [Python 3.x](https://www.python.org/)
* * Las preguntas y respuestas personalizadas requieren un [recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) con la característica de respuesta a preguntas personalizada habilitada para generar una clave y un punto de conexión de API.
    * Después de la implementación del recurso de Text Analytics, seleccione **Ir al recurso**. Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.

---

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Después de instalar Python, puede instalar la biblioteca cliente con:

```console
pip install azure-cognitiveservices-knowledge-qnamaker==0.2.0
```

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Después de instalar Python, puede instalar la biblioteca cliente con:

```console
pip install azure-cognitiveservices-knowledge-qnamaker
```

---

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree un archivo de Python denominado `quickstart-file.py` e importe las bibliotecas siguientes.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Dependencies](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=Dependencies)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Dependencies](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=Dependencies)]

---

Cree variables para el punto de conexión y la clave de Azure del recurso.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves de QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Keys and Endpoint** (Claves y punto de conexión), en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión de QnA Maker.

 ![Punto de conexión de creación de QnA Maker](../media/keys-endpoint.png)

- El valor de QNA_MAKER_RUNTIME_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Exportar plantilla**, en **Automatización**, para buscar el punto de conexión en tiempo de ejecución.

 ![Punto de conexión en tiempo de ejecución de QnA Maker](../media/runtime-endpoint.png)
   
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-python[Resource variables](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=Resourcevariables)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de Text Analytics que creó en los requisitos previos. Haga clic en la página **Claves y punto de conexión**, en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión.

> [!div class="mx-imgBorder"]
> ![Punto de conexión de creación de PyR personalizado](../media/qnamaker-how-to-key-management/custom-qna-keys-and-endpoint.png)
 
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-python[Resource variables](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=Resourcevariables)]

---

## <a name="object-models"></a>Modelos de objetos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[QnA Maker](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker) usa dos modelos de objetos diferentes:
* **[QnAMakerClient](#qnamakerclient-object-model)** , el objeto para crear, administrar, publicar y descargar la base de conocimiento.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)** , el objeto para consultar la base de conocimiento con GenerateAnswer API y enviar nuevas preguntas sugeridas mediante Train API (como parte del [aprendizaje activo](../how-to/use-active-learning.md)).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[QnA Maker](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker) utiliza el siguiente modelo de objetos:
* **[QnAMakerClient](#qnamakerclient-object-model)** es el objeto para crear, administrar, publicar y descargar la base de conocimiento.

---

[!INCLUDE [Get KBinformation](./quickstart-sdk-cognitive-model.md)]

### <a name="qnamakerclient-object-model"></a>Modelo de objetos QnAMakerClient

El cliente de QnA Maker de creación es un objeto [QnAMakerClient](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.qn_amaker_client.qnamakerclient) que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave.

Una vez creado el cliente, utilice la propiedad [Knowledge base](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebase_operations.knowledgebaseoperations) para crear, administrar y publicar la base de conocimiento.

Administre la base de conocimiento mediante el envío de un objeto JSON. En el caso de operaciones inmediatas, un método suele devolver un objeto JSON que indica el estado. En el caso de operaciones de ejecución prolongada, la respuesta es el identificador de la operación. Llame al método [operations.get_details](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebase_operations.knowledgebaseoperations#get-details-kb-id--custom-headers-none--raw-false----operation-config-) con el identificador de la operación para determinar el [estado de la solicitud](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.operationstatetype).

### <a name="qnamakerruntimeclient-object-model"></a>Modelo de objetos QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El cliente de QnA Maker de predicción es un objeto `QnAMakerRuntimeClient` que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave de tiempo de ejecución de predicción que devuelve la llamada del cliente de creación, [client.EndpointKeysOperations.get_keys](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.endpoint_keys_operations.endpointkeysoperations#get-keys-custom-headers-none--raw-false----operation-config-), una vez publicada la base de conocimiento.

Use el método `generate_answer` para obtener una respuesta del tiempo de ejecución de la consulta.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

La respuesta a preguntas personalizadas no requiere el uso del objeto QnAMakerRuntimeClient. En su lugar, se llama a [generate_answer](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebase_operations.knowledgebaseoperations#generate-answer-kb-id--generate-answer-payload--custom-headers-none--raw-false----operation-config-) directamente en el objeto [QnAMakerClient](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.qn_amaker_client.qnamakerclient).

---

## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Autenticación del cliente para la creación de la base de conocimiento

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto CognitiveServicesCredentials con la clave y úselo con el punto de conexión para crear un objeto [QnAMakerClient](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.qn_amaker_client.qnamakerclient).

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Authorization to resource key](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=AuthorizationAuthor)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Authorization to resource key](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=AuthorizationAuthor)]

---

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Use el objeto de cliente para obtener un objeto de [operaciones de base de conocimiento](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations).

Una base de conocimiento almacena pares de preguntas y respuestas para el objeto [CreateKbDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.createkbdto) procedentes de tres orígenes:

* Para **contenido editorial**, use el objeto [QnADTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.qnadto).
    * Para usar metadatos y avisos de seguimiento, utilice el contexto editorial, ya que estos datos se agregan en el nivel de un par de QnA individual.
* Para **archivos**, use el objeto [FileDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.filedto). El objeto FileDTO incluye el nombre de archivo así como la dirección URL pública para llegar al archivo.
* En el caso de las **direcciones URL**, use una lista de cadenas para representar las direcciones URL disponibles públicamente.

Llame al método [create](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations#create-create-kb-payload--custom-headers-none--raw-false----operation-config-) y, luego, pase el identificador de operación devuelto al método [Operations.getDetails](#get-status-of-an-operation) para sondear el estado.

La última línea del código siguiente devuelve el identificador de la base de conocimiento de la respuesta de MonitorOperation.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Create knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=CreateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Create knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=CreateKBMethod)]

---

Asegúrese de incluir la función [`_monitor_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de crear correctamente una base de conocimiento.

## <a name="update-a-knowledge-base"></a>Actualización de una base de conocimientos

Para actualizar una base de conocimiento, pase el identificador de la base de conocimiento y un objeto [UpdateKbOperationDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdto) que contenga los objetos de DTO [add](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtoadd), [update](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtoupdate) y [delete](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtodelete) al método [update](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebase_operations.knowledgebaseoperations). Use el método [Operation.getDetail](#get-status-of-an-operation) para determinar si la actualización se realizó correctamente.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Update a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=UpdateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Update a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=UpdateKBMethod)]

---

Asegúrese de incluir la función [`_monitor_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de actualizar correctamente una base de conocimiento.

## <a name="download-a-knowledge-base"></a>Descarga de una base de conocimiento

Use el método [download](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations) para descargar la base de datos como una lista de [QnADocumentsDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.qnadocumentsdto). Esto _no_ equivale a las exportaciones del portal de QnA Maker desde la página **Configuración** ya que el resultado de este método no es un archivo TSV.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Download a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=DownloadKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Download a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=DownloadKB)]

---

## <a name="publish-a-knowledge-base"></a>Publicación de una base de conocimientos

Publique la base de conocimiento mediante el método [publish](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations#publish-kb-id--custom-headers-none--raw-false----operation-config-). Esto toma el modelo actual guardado y entrenado, al que hace referencia el identificador de la base de conocimiento, y lo publica en un punto de conexión.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Publish a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=PublishKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Publish a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=PublishKB)]

---

## <a name="query-a-knowledge-base"></a>Consulta de una base de conocimiento

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

### <a name="get-query-runtime-key"></a>Obtención de la clave de tiempo de ejecución de consulta

Una vez publicada una base de conocimiento, necesita la clave de tiempo de ejecución de consulta para consultar el tiempo de ejecución. Esta clave no es la misma que la que se usa para crear el objeto de cliente original.

Use el método [EndpointKeysOperations.get_keys](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.endpointkeysoperations#get-keys-custom-headers-none--raw-false----operation-config-) para obtener la clase [EndpointKeysDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.endpointkeysdto).

Use cualquiera de las propiedades de clave devueltas en el objeto para consultar la base de conocimiento.

[!code-python[Get query runtime key](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=GetQueryEndpointKey)]

### <a name="authenticate-the-runtime-for-generating-an-answer"></a>Autenticación del tiempo de ejecución para generar una respuesta

Cree un [QnAMakerRuntimeClient](/javascript/api/@azure/cognitiveservices-qnamaker-runtime/qnamakerruntimeclient) para consultar la base de conocimiento con el fin de generar una respuesta o entrenar el aprendizaje activo.

[!code-python[Authenticate the runtime](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=AuthorizationQuery)]

Use QnAMakerRuntimeClient para obtener una respuesta de la base de conocimiento o enviar a esta nuevas preguntas sugeridas para el [aprendizaje activo](../concepts/active-learning-suggestions.md).

### <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta a partir de una base de conocimiento publicada mediante el método QnAMakerRuntimeClient.runtime.generate_answer. Este método acepta el identificador de la base de conocimiento y [QueryDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.querydto). Acceda a las propiedades adicionales de QueryDTO, como Top y Context, que se usarán en el bot de chat.

[!code-python[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=GenerateAnswer)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

### <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta a partir de una base de conocimiento publicada mediante el método [generate_answer](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations#generate-answer-kb-id--generate-answer-payload--custom-headers-none--raw-false----operation-config-). Este método acepta el identificador de la base de conocimiento y [QueryDTO](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.querydto). Acceda a las propiedades adicionales de QueryDTO, como Top y Context, que se usarán en el bot de chat.

[!code-python[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=GenerateAnswer)]

---

Este es un ejemplo sencillo de consulta de la base de conocimiento. Para comprender los escenarios de consulta avanzados, revise [otros ejemplos de consulta](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

Elimine la base de conocimiento con el método [delete](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.operations.knowledgebaseoperations#delete-kb-id--custom-headers-none--raw-false----operation-config-) y el parámetro del identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Delete a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=DeleteKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Delete a knowledge base](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=DeleteKB)]

---

## <a name="get-status-of-an-operation"></a>Obtención del estado de una operación

Algunos métodos, como Create y Update, pueden tardar bastante tiempo en que en lugar de esperar a que finalice el proceso, se devuelva una [operación](/python/api/azure-cognitiveservices-knowledge-qnamaker/azure.cognitiveservices.knowledge.qnamaker.models.operation(class)). Use el identificador de la operación para realizar un sondeo (con lógica de reintento) para determinar el estado del método original.

La llamada _setTimeout_ del siguiente bloque de código se usa para simular código asincrónico. Reemplácela por la lógica de reintento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-python[Monitor an operation](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/quickstart.py?name=MonitorOperation)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-python[Monitor an operation](~/cognitive-services-quickstart-code/python/QnAMaker/sdk/preview-sdk/quickstart.py?name=MonitorOperation)]

---

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando de python en el archivo de inicio rápido.

```console
python quickstart-file.py
```

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/QnAMaker/sdk/quickstart.py).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/QnAMaker/sdk/preview-sdk/quickstart.py).

---
