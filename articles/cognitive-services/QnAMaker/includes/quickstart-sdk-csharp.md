---
title: 'Inicio rápido: Biblioteca cliente de QnA Maker para .NET'
description: Este inicio rápido le muestra cómo empezar a trabajar con la biblioteca cliente de QnA Maker para .NET. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.  QnA Maker le permite alimentar un servicio de preguntas y respuestas con su contenido semiestructurado, como documentos de preguntas frecuentes, direcciones URL y manuales de productos.
ms.topic: quickstart
ms.date: 04/28/2021
ms.openlocfilehash: e59f2137af303242ef2747e2b3430dd9e081c6aa
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110486641"
---
# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Use la biblioteca cliente de QnA Maker para .NET para:

 * Crear una base de conocimiento
 * Actualizar una base de conocimiento
 * Publicar una base de conocimiento
 * Obtener la clave del punto de conexión de tiempo de ejecución de predicción
 * Espera de una tarea de ejecución prolongada
 * Descargar una base de conocimiento
 * Obtener una respuesta de una base de conocimiento
 * Eliminar una base de conocimiento

[Documentación de referencia](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Knowledge.QnAMaker) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Knowledge.QnAMaker/2.0.1) | [Ejemplos de C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/QnAMaker/SDK-based-quickstart)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Use la biblioteca cliente de QnA Maker para .NET para:

 * Crear una base de conocimiento
 * Actualizar una base de conocimiento
 * Publicar una base de conocimiento
 * Espera de una tarea de ejecución prolongada
 * Descargar una base de conocimiento
 * Obtener una respuesta de una base de conocimiento
 * Eliminar una base de conocimiento

[Documentación de referencia](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Knowledge.QnAMaker) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Knowledge.QnAMaker/3.0.0-preview.1) | [Ejemplos de C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/QnAMaker/Preview-sdk-based-quickstart)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Requisitos previos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* El [IDE de Visual Studio](https://visualstudio.microsoft.com/vs/) o la versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Cuando tenga la suscripción a Azure, cree un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) en Azure Portal para obtener la clave de creación y el nombre del recurso. Tras su implementación, seleccione **Ir al recurso**.
    * Necesitará la clave y el nombre del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el nombre del recurso en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* El [IDE de Visual Studio](https://visualstudio.microsoft.com/vs/) o la versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Las preguntas y respuestas personalizadas requieren un [recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) con la característica de respuesta a preguntas personalizada habilitada para generar una clave y un punto de conexión de API.
    * Después de la implementación del recurso de Text Analytics, seleccione **Ir al recurso**. Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.

---

## <a name="setting-up"></a>Instalación

### <a name="cli"></a>CLI

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `qna-maker-quickstart`. Este comando crea un sencillo proyecto de C#, "Hola mundo", con un solo archivo de origen: *program.cs*.

```console
dotnet new console -n qna-maker-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores.

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

Dentro del directorio de aplicaciones, instale la biblioteca cliente de QnA Maker para .NET con el siguiente comando:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

```console
dotnet add package Microsoft.Azure.CognitiveServices.Knowledge.QnAMaker --version 2.0.1
```

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

```console
dotnet add package Microsoft.Azure.CognitiveServices.Knowledge.QnAMaker --version 3.0.0-preview.1
```

---

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/QnAMaker/SDK-based-quickstart/Program.cs), que contiene los ejemplos de código de este inicio rápido.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs), que contiene los ejemplos de código de este inicio rápido.

---

### <a name="using-directives"></a>Directivas Using

En el directorio del proyecto, abra el archivo *program.cs* y agregue lo siguiente mediante directivas `using`:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Dependencies](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=Dependencies)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Dependencies](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=Dependencies)]

---

### <a name="subscription-key-and-resource-endpoints"></a>Clave de suscripción y puntos de conexión de recursos

En el método `Main` de la aplicación, agregue las variables y código que se muestran en la siguiente sección, para usar las tareas comunes de este inicio rápido.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves de QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Keys and Endpoint** (Claves y punto de conexión), en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión de QnA Maker.

 ![Punto de conexión de creación de QnA Maker](../media/keys-endpoint.png)

- El valor de QNA_MAKER_RUNTIME_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Exportar plantilla**, en **Automatización**, para buscar el punto de conexión en tiempo de ejecución.

 ![Punto de conexión en tiempo de ejecución de QnA Maker](../media/runtime-endpoint.png)
      
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-csharp[Set the resource key and resource name](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=Resourcevariables)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de Text Analytics que creó en los requisitos previos. Haga clic en la página **Claves y punto de conexión**, en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión.

> [!div class="mx-imgBorder"]
> ![Punto de conexión de creación de PyR personalizado](../media/qnamaker-how-to-key-management/custom-qna-keys-and-endpoint.png)


- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-csharp[Set the resource key and resource name](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=Resourcevariables)]

---


## <a name="object-models"></a>Modelos de objetos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[QnA Maker](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker) usa dos modelos de objetos diferentes:
* **[QnAMakerClient](#qnamakerclient-object-model)** , el objeto para crear, administrar, publicar y descargar la base de conocimiento.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)** , el objeto para consultar la base de conocimiento con GenerateAnswer API y enviar nuevas preguntas sugeridas mediante Train API (como parte del [aprendizaje activo](../how-to/use-active-learning.md)).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[QnA Maker](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker) utiliza el siguiente modelo de objetos:
* **[QnAMakerClient](#qnamakerclient-object-model)** es el objeto para crear, administrar, publicar y descargar la base de conocimiento.

---

[!INCLUDE [Get KBinformation](./quickstart-sdk-cognitive-model.md)]

### <a name="qnamakerclient-object-model"></a>Modelo de objetos QnAMakerClient

El cliente de QnA Maker de creación es un objeto [QnAMakerClient](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient) que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave.

Una vez creado el cliente, utilice la propiedad [Knowledge base](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient.knowledgebase#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_QnAMakerClient_Knowledgebase) para crear, administrar y publicar la base de conocimiento.

Administre la base de conocimiento mediante el envío de un objeto JSON. En el caso de operaciones inmediatas, un método suele devolver un objeto JSON que indica el estado. En el caso de operaciones de ejecución prolongada, la respuesta es el identificador de la operación. Llame al método [client.Operations.GetDetailsAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.operationsextensions.getdetailsasync) con el identificador de la operación para determinar el [estado de la solicitud](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.operationstatetype).

### <a name="qnamakerruntimeclient-object-model"></a>Modelo de objetos QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El cliente de QnA Maker de predicción es un objeto [QnAMakerRuntimeClient](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerruntimeclient) que se autentica en Azure mediante Microsoft.Rest.ServiceClientCredentials, que contiene la clave de tiempo de ejecución de predicción, devuelta por la llamada del cliente de creación, `client.EndpointKeys.GetKeys` una vez publicada la base de conocimiento.

Use el método [GenerateAnswer](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.runtimeextensions) para obtener una respuesta del tiempo de ejecución de la consulta.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

 La respuesta a preguntas personalizadas no requiere el uso del objeto **QnAMakerRuntimeClient**. En su lugar, llame al método [QnAMakerClient.Knowledgebase](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient.knowledgebase).[GenerateAnswerAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.generateanswerasync).

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

Cree una instancia de un objeto de cliente con la clave y úsela con el recurso para construir el punto de conexión para crear un [QnAMakerClient](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient) con el punto de conexión y la clave. Cree un objeto [ServiceClientCredentials](/dotnet/api/microsoft.rest.serviceclientcredentials).

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=AuthorizationAuthor)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=AuthorizationAuthor)]

---

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Una base de conocimiento almacena pares de preguntas y respuestas para el objeto [CreateKbDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.createkbdto) procedentes de tres orígenes:

* Para **contenido editorial**, use el objeto [QnADTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.qnadto).
    * Para usar metadatos y avisos de seguimiento, utilice el contexto editorial, ya que estos datos se agregan en el nivel de un par de QnA individual.
* Para **archivos**, use el objeto [FileDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.filedto). El objeto FileDTO incluye el nombre de archivo así como la dirección URL pública para llegar al archivo.
* En el caso de las **direcciones URL**, use una lista de cadenas para representar las direcciones URL disponibles públicamente.

El paso de creación también incluye las propiedades de la base de conocimiento:
* `defaultAnswerUsedForExtraction`: lo que se devuelve cuando no se encuentra ninguna respuesta.
* `enableHierarchicalExtraction`: crea automáticamente relaciones de mensajes entre los pares de QnA extraídos.
* `language`: al crear la primera base de conocimiento de un recurso, establezca el lenguaje que se va a usar en el índice de Azure Search.

Llame al método [CreateAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.createasync) y, a continuación, pase el identificador de operación devuelto al método [MonitorOperation](#get-status-of-an-operation) para sondear el estado.

La última línea del código siguiente devuelve el identificador de la base de conocimiento de la respuesta de MonitorOperation.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Create a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=CreateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Create a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=CreateKBMethod)]

---

Asegúrese de incluir la función [`MonitorOperation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de crear correctamente una base de conocimiento.

## <a name="update-a-knowledge-base"></a>Actualización de una base de conocimientos

Puede actualizar una base de conocimiento pasando el identificador de la base de conocimiento y un [UpdatekbOperationDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdto) que contiene los objetos de DTO [add](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtoadd), [update](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtoupdate) y [delete](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.updatekboperationdtodelete) al método [UpdateAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.updateasync). Use el método [MonitorOperation](#get-status-of-an-operation) para determinar si la actualización se realizó correctamente.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Update a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=UpdateKBMethod)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Update a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=UpdateKBMethod)]

---

Asegúrese de incluir la función [`MonitorOperation`](#get-status-of-an-operation), a la que se hace referencia en el código anterior, con el fin de actualizar correctamente una base de conocimiento.

## <a name="download-a-knowledge-base"></a>Descarga de una base de conocimiento

Use el método [DownloadAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.downloadasync) para descargar la base de datos como una lista de [QnADocumentsDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.qnadocumentsdto). Esto _no_ equivale a las exportaciones del portal de QnA Maker desde la página **Configuración** ya que el resultado de este método no es un archivo.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Download a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=DownloadKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Download a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=DownloadKB)]

---

## <a name="publish-a-knowledge-base"></a>Publicación de una base de conocimientos

Publique la base de conocimiento mediante el método [PublishAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.publishasync). Este método toma el modelo actual guardado y entrenado, al que hace referencia el identificador de la base de conocimiento, y lo publica en el punto de conexión. Este es un paso necesario para consultar la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Publish a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=PublishKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Publish a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=PublishKB)]

---

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

## <a name="get-query-runtime-key"></a>Obtención de la clave de tiempo de ejecución de consulta

Una vez publicada una base de conocimiento, necesita la clave de tiempo de ejecución de consulta para consultar el tiempo de ejecución. Esta clave no es la misma que la que se usa para crear el objeto de cliente original.

Use el método [EndpointKeys](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.endpointkeys.getkeyswithhttpmessagesasync#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_EndpointKeys_GetKeysWithHttpMessagesAsync_System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_) para obtener la clase [EndpointKeysDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.endpointkeysdto).

Use cualquiera de las propiedades de clave devueltas en el objeto para consultar la base de conocimiento.

[!code-csharp[Get query runtime key](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=GetQueryEndpointKey)]

Se necesita una clave de tiempo de ejecución para consultar la base de conocimiento.

## <a name="authenticate-the-runtime-for-generating-an-answer"></a>Autenticación del tiempo de ejecución para generar una respuesta

Cree un [QnAMakerRuntimeClient](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerruntimeclient) para consultar la base de conocimiento con el fin de generar una respuesta o entrenar el aprendizaje activo.

[!code-csharp[Authenticate the runtime](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=AuthorizationQuery)]

Use QnAMakerRuntimeClient para:
* Obtener una respuesta de la base de conocimiento
* Enviar nuevas preguntas sugeridas a la base de conocimiento para el [aprendizaje activo](../concepts/active-learning-suggestions.md).

## <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta a partir de una base de conocimiento publicada mediante el método [RuntimeClient](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient.knowledgebase#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_QnAMakerClient_Knowledgebase).[GenerateAnswerAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.runtimeextensions.generateanswerasync). Este método acepta el identificador de la base de conocimiento y [QueryDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto). Obtenga acceso a las propiedades adicionales de QueryDTO, como [Top](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto.top#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_Models_QueryDTO_Top) y [Context](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto.context) que se usarán en el bot de chat.

[!code-csharp[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=GenerateAnswer)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

## <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

Genere una respuesta de una base de conocimiento publicada mediante el método [QnAMakerClient.Knowledgebase](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.qnamakerclient.knowledgebase).[GenerateAnswerAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.generateanswerasync). Este método acepta el identificador de la base de conocimiento y [QueryDTO](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto). Acceda a otras propiedades de QueryDTO, como [Top](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto.top#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_Models_QueryDTO_Top), [Context](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto.context#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_Models_QueryDTO_Context) y [AnswerSpanRequest](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.querydto.answerspanrequest#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_Models_QueryDTO_AnswerSpanRequest), que se usarán en el bot de chat.

[!code-csharp[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=GenerateAnswer)]

---

Este es un ejemplo sencillo de consulta de la base de conocimiento. Para comprender los escenarios de consulta avanzados, revise [otros ejemplos de consulta](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

Elimine la base de conocimiento mediante el método [DeleteAsync](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.knowledgebaseextensions.deleteasync) con un parámetro del identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Delete a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=DeleteKB)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Delete a knowledge base](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=DeleteKB)]

---

## <a name="get-status-of-an-operation"></a>Obtención del estado de una operación

Algunos métodos, como Create y Update, pueden tardar bastante tiempo en que en lugar de esperar a que finalice el proceso, se devuelva una [operación](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.operation). Use el [identificador](/dotnet/api/microsoft.azure.cognitiveservices.knowledge.qnamaker.models.operation.operationid#Microsoft_Azure_CognitiveServices_Knowledge_QnAMaker_Models_Operation_OperationId) de la operación para realizar un sondeo (con lógica de reintento) para determinar el estado del método original.

_loop_ y _Task.Delay_ en el siguiente bloque de código se utilizan para simular una lógica de reintentos. Estos deben reemplazarse por su propia lógica de reintentos.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-csharp[Monitor an operation](~/cognitive-services-quickstart-code/dotnet/QnAMaker/SDK-based-quickstart/Program.cs?name=MonitorOperation)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-csharp[Monitor an operation](~/cognitive-services-quickstart-code/dotnet/QnAMaker/Preview-sdk-based-quickstart/Program.cs?name=MonitorOperation)]

---

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `dotnet run` desde el directorio de la aplicación.

```dotnetcli
dotnet run
```

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/QnAMaker/SDK-based-quickstart).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/QnAMaker/Preview-sdk-based-quickstart).

---
