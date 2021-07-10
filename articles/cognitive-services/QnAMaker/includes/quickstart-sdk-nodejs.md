---
title: 'Inicio rápido: Biblioteca cliente de QnA Maker para Node.js'
description: Este inicio rápido le muestra cómo empezar a trabajar con la biblioteca cliente de QnA Maker para Node.js.
ms.topic: quickstart
ms.date: 06/18/2020
ms.custom: devx-track-js
ms.openlocfilehash: c101458f9cee00fbc1f25f4c6679485439864eab
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110487341"
---
# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Use la biblioteca cliente de QnA Maker para Node.js con los siguientes fines:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Obtener la clave del punto de conexión de tiempo de ejecución de predicción
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Documentación de referencia](/javascript/api/@azure/cognitiveservices-qnamaker/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-qnamaker) | [Paquete (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-qnamaker) | [Ejemplos de Node.js](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Use la biblioteca cliente de QnA Maker para Node.js con los siguientes fines:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Documentación de referencia](/javascript/api/@azure/cognitiveservices-qnamaker/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-qnamaker) | [Paquete (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-qnamaker) | [Ejemplos de Node.js](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/preview-sdk/quickstart.js)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Requisitos previos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org).
* Cuando tenga la suscripción a Azure, cree un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) en Azure Portal para obtener la clave de creación y el recurso. Tras su implementación, seleccione **Ir al recurso**.
    * Necesitará la clave y el nombre del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el nombre del recurso en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org).
* Las preguntas y respuestas personalizadas requieren un [recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) con la característica de respuesta a preguntas personalizada habilitada para generar una clave y un punto de conexión de API.
     * Después de la implementación del recurso de Text Analytics, seleccione **Ir al recurso**. Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.

---

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir qnamaker_quickstart && cd qnamaker_quickstart
```

Ejecute el comando `npm init -y` para crear una aplicación de nodo con un archivo `package.json`.

```console
npm init -y
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Instale los siguientes paquetes NPM:

```console
npm install @azure/cognitiveservices-qnamaker
npm install @azure/cognitiveservices-qnamaker-runtime
npm install @azure/ms-rest-js
```

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Instale los siguientes paquetes NPM:

```console
npm install @azure/cognitiveservices-qnamaker
npm install @azure/ms-rest-js
```

---

El archivo `package.json` de la aplicación se actualiza con las dependencias.

Cree un archivo llamado index.js e importe las bibliotecas siguientes:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Dependencies](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Dependencies)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Dependencies](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=Dependencies)]

---

Cree una variable para la clave de Azure y el nombre del recurso.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves de QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Keys and Endpoint** (Claves y punto de conexión), en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión de QnA Maker.

 ![Punto de conexión de creación de QnA Maker](../media/keys-endpoint.png)

- El valor de QNA_MAKER_RUNTIME_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Exportar plantilla**, en **Automatización**, para buscar el punto de conexión en tiempo de ejecución.

 ![Punto de conexión en tiempo de ejecución de QnA Maker](../media/runtime-endpoint.png)
   
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-javascript[Set the resource key and resource name](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Resourcevariables)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de Text Analytics que creó en los requisitos previos. Haga clic en la página **Claves y punto de conexión**, en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión.

> [!div class="mx-imgBorder"]
> ![Punto de conexión de creación de PyR personalizado](../media/qnamaker-how-to-key-management/custom-qna-keys-and-endpoint.png)

- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-javascript[Set the resource key and resource name](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=Resourcevariables)]

---

## <a name="object-models"></a>Modelos de objetos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[QnA Maker](/javascript/api/@azure/cognitiveservices-qnamaker/) usa dos modelos de objetos diferentes:
* **[QnAMakerClient](#qnamakerclient-object-model)** , el objeto para crear, administrar, publicar y descargar la base de conocimiento.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)** , el objeto para consultar la base de conocimiento con GenerateAnswer API y enviar nuevas preguntas sugeridas mediante Train API (como parte del [aprendizaje activo](../how-to/use-active-learning.md)).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[QnA Maker](/javascript/api/@azure/cognitiveservices-qnamaker/) utiliza el siguiente modelo de objetos:
* **[QnAMakerClient](#qnamakerclient-object-model)** es el objeto para crear, administrar, publicar y descargar la base de conocimiento.

---

### <a name="qnamakerclient-object-model"></a>Modelo de objetos QnAMakerClient

El cliente de QnA Maker es un objeto [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient) que se autentica en Azure mediante sus credenciales y que contiene la clave.

Una vez creado el cliente, utilice la propiedad [knowledgebase](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient#knowledgebase) para crear, administrar y publicar la base de conocimiento.

Administre la base de conocimiento mediante el envío de un objeto JSON. En el caso de operaciones inmediatas, un método suele devolver un objeto JSON que indica el estado. En el caso de operaciones de ejecución prolongada, la respuesta es el identificador de la operación. Llame al método [client.operations.getDetails](/javascript/api/@azure/cognitiveservices-qnamaker/operations#getdetails-string--msrest-requestoptionsbase-) con el identificador de la operación para determinar el [estado de la solicitud](/javascript/api/@azure/cognitiveservices-qnamaker/operation).

### <a name="qnamakerruntimeclient-object-model"></a>Modelo de objetos QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El cliente de QnA Maker de predicción es un objeto QnAMakerRuntimeClient que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave de tiempo de ejecución de predicción que devuelve la llamada del cliente de creación, [client.EndpointKeys.getKeys](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys#getkeys-msrest-requestoptionsbase-), una vez publicada la base de conocimiento.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

La respuesta a preguntas personalizadas no requiere el uso del objeto QnAMakerRuntimeClient. En su lugar, se llama a [generate_answer](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#generateAnswer_string__QueryDTO__msRest_RequestOptionsBase_) directamente en el objeto [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient).

---

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de QnA Maker para .NET:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

* [Autenticación del cliente de creación](#authenticate-the-client-for-authoring-the-knowledge-base)
* [Creación de una base de conocimientos](#create-a-knowledge-base)
* [Actualización de una base de conocimientos](#update-a-knowledge-base)
* [Descarga de una base de conocimiento](#download-a-knowledge-base)
* [Publicación de una base de conocimientos](#publish-a-knowledge-base)
* [Eliminación de una base de conocimiento](#delete-a-knowledge-base)
* [Obtención de la clave de tiempo de ejecución de consulta](#get-query-runtime-key)
* [Obtención del estado de una operación](#get-status-of-an-operation)
* [Autenticación del cliente de tiempo de ejecución de consulta](#authenticate-the-runtime-for-generating-an-answer)
* [Generación de una respuesta de la base de conocimiento](#generate-an-answer-from-the-knowledge-base)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

* [Autenticación del cliente de creación](#authenticate-the-client-for-authoring-the-knowledge-base)
* [Creación de una base de conocimientos](#create-a-knowledge-base)
* [Actualización de una base de conocimientos](#update-a-knowledge-base)
* [Descarga de una base de conocimiento](#download-a-knowledge-base)
* [Publicación de una base de conocimientos](#publish-a-knowledge-base)
* [Eliminación de una base de conocimiento](#delete-a-knowledge-base)
* [Obtención del estado de una operación](#get-status-of-an-operation)
* [Generación de una respuesta de la base de conocimiento](#generate-an-answer-from-the-knowledge-base)

---

## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Autenticación del cliente para la creación de la base de conocimiento

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto ServiceClientCredentials con la clave y úselo con el punto de conexión para crear un objeto [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient).

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationAuthor)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=AuthorizationAuthor)]

---

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Una base de conocimiento almacena pares de preguntas y respuestas para el objeto [CreateKbDTO](/javascript/api/@azure/cognitiveservices-qnamaker/createkbdto) procedentes de tres orígenes:

* Para **contenido editorial**, use el objeto [QnADTO](/javascript/api/@azure/cognitiveservices-qnamaker/qnadto).
    * Para usar metadatos y avisos de seguimiento, utilice el contexto editorial, ya que estos datos se agregan en el nivel de un par de QnA individual.
* Para **archivos**, use el objeto [FileDTO](/javascript/api/@azure/cognitiveservices-qnamaker/filedto). El objeto FileDTO incluye el nombre de archivo así como la dirección URL pública para llegar al archivo.
* En el caso de las **direcciones URL**, use una lista de cadenas para representar las direcciones URL disponibles públicamente.

El paso de creación también incluye las propiedades de la base de conocimiento:
* `defaultAnswerUsedForExtraction`: lo que se devuelve cuando no se encuentra ninguna respuesta.
* `enableHierarchicalExtraction`: crea automáticamente relaciones de mensajes entre los pares de QnA extraídos.
* `language`: al crear la primera base de conocimiento de un recurso, establezca el lenguaje que se va a usar en el índice de Azure Search.

Llame al método [create](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#create-createkbdto--servicecallback-operation--) con la información de la base de conocimiento. La información de la base de conocimiento es básicamente un objeto JSON.

Cuando el método create realice una devolución, pase el identificador de la operación devuelta al método [wait_for_operation](#get-status-of-an-operation) para sondear el estado. El método wait_for_operation devuelve una salida cuando finaliza la operación. Analice el valor del encabezado `resourceLocation` de la operación devuelta para obtener el nuevo identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Create knowledgebase](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=CreateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Create knowledgebase](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=CreateKBMethod)]

---

Asegúrese de incluir la función [`wait_for_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de crear correctamente una base de conocimiento.

## <a name="update-a-knowledge-base"></a>Actualización de una base de conocimientos

Para actualizar una base de conocimiento, pase el identificador de la base de conocimiento y un objeto [UpdateKbOperationDTO](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto) que contenga los objetos de DTO [add](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#add), [update](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#update) y [delete](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#deleteproperty) al método [update](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#update-string--updatekboperationdto--msrest-requestoptionsbase-). Los DTO son también básicamente objetos JSON. Use el método [wait_for_operation](#get-status-of-an-operation) para determinar si la actualización se realizó correctamente.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Update a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=UpdateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Update a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=UpdateKBMethod)]

---

Asegúrese de incluir la función [`wait_for_operation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de actualizar correctamente una base de conocimiento.

## <a name="download-a-knowledge-base"></a>Descarga de una base de conocimiento

Use el método [download](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#download-string--models-environmenttype--msrest-requestoptionsbase-) para descargar la base de datos como una lista de [QnADocumentsDTO](/javascript/api/@azure/cognitiveservices-qnamaker/qnadocumentsdto). Esto _no_ equivale a las exportaciones del portal de QnA Maker desde la página **Configuración** ya que el resultado de este método no es un archivo TSV.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Download a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DownloadKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Download a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=DownloadKB)]

---

## <a name="publish-a-knowledge-base"></a>Publicación de una base de conocimientos

Publique la base de conocimiento mediante el método [publish](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#publish-string--msrest-requestoptionsbase-). Esto toma el modelo actual guardado y entrenado, al que hace referencia el identificador de la base de conocimiento, y lo publica en un punto de conexión. Compruebe el código de respuesta HTTP para validar que la publicación se haya realizado correctamente.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Publish a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=PublishKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Publish a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=PublishKB)]

---

## <a name="query-a-knowledge-base"></a>Consulta de una base de conocimiento

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

### <a name="get-query-runtime-key"></a>Obtención de la clave de tiempo de ejecución de consulta

Una vez publicada una base de conocimiento, necesita la clave de tiempo de ejecución de consulta para consultar el tiempo de ejecución. Esta clave no es la misma que la que se usa para crear el objeto de cliente original.

Use el método [EndpointKeys.getKeys](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys) para obtener la clase [EndpointKeysDTO](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeysdto).

Use cualquiera de las propiedades de clave devueltas en el objeto para consultar la base de conocimiento.

[!code-javascript[Get query runtime key](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GetQueryEndpointKey)]

### <a name="authenticate-the-runtime-for-generating-an-answer"></a>Autenticación del tiempo de ejecución para generar una respuesta

Cree un QnAMakerRuntimeClient para consultar la base de conocimiento con el fin de generar una respuesta o entrenar el aprendizaje activo.

[!code-javascript[Authenticate the runtime](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationQuery)]

Use QnAMakerRuntimeClient para obtener una respuesta de la base de conocimiento o enviar a esta nuevas preguntas sugeridas para el [aprendizaje activo](../concepts/active-learning-suggestions.md).

### <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta a partir de una base de conocimiento publicada mediante el método RuntimeClient.runtime.generateAnswer. Este método acepta el identificador de la base de conocimiento y [QueryDTO](/javascript/api/@azure/cognitiveservices-qnamaker/querydto). Acceda a las propiedades adicionales de QueryDTO, como Top y Context, que se usarán en el bot de chat.

[!code-javascript[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GenerateAnswer)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

### <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta de una base de conocimiento publicada mediante el método QnAMakerClient.knowledgebase.generateAnswer. Este método acepta el identificador de la base de conocimiento y [QueryDTO](/javascript/api/@azure/cognitiveservices-qnamaker/querydto). Acceda a las propiedades adicionales de QueryDTO, como Top y Context, que se usarán en el bot de chat.

[!code-javascript[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=GenerateAnswer)]

---

Este es un ejemplo sencillo de consulta de la base de conocimiento. Para comprender los escenarios de consulta avanzados, revise [otros ejemplos de consulta](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

Elimine la base de conocimiento con el método [delete](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#deletemethod-string--msrest-requestoptionsbase-) y el parámetro del identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Delete a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DeleteKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Delete a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=DeleteKB)]

---

## <a name="get-status-of-an-operation"></a>Obtención del estado de una operación

Algunos métodos, como Create y Update, pueden tardar bastante tiempo en que en lugar de esperar a que finalice el proceso, se devuelva una [operación](/javascript/api/@azure/cognitiveservices-qnamaker/operations). Use el [identificador](/javascript/api/@azure/cognitiveservices-qnamaker/operation#operationid) de la operación para realizar un sondeo (con lógica de reintento) para determinar el estado del método original.

La llamada _delayTimer_ del siguiente bloque de código se usa para simular la lógica de reintentos. Reemplácela por su propia lógica de reintentos.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-javascript[Monitor an operation](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=MonitorOperation)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-javascript[Monitor an operation](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=MonitorOperation)]

---

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node index.js` desde el directorio de la aplicación.

```console
node index.js
```

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/preview-sdk/quickstart.js).

---
