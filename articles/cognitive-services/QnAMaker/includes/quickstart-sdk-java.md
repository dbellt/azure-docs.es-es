---
title: 'Inicio rápido: Biblioteca cliente de QnA Maker para Java'
description: Este inicio rápido le muestra cómo empezar a trabajar con la biblioteca cliente de QnA Maker para Java.
author: v-jaswel
manager: chrhoder
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.date: 09/04/2020
ms.author: v-jawe
ms.openlocfilehash: dcd1a5da072954f47a21be2657bc9a4f8ae4a9cf
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110487209"
---
# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Use la biblioteca cliente de QnA Maker para Java para:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Obtener la clave del punto de conexión de tiempo de ejecución de predicción
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker) | [Paquete](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker/1.0.0-beta.1) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/qnamaker/sdk/quickstart.java)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Use la biblioteca cliente de QnA Maker para Java para:

* Crear una base de conocimiento
* Actualizar una base de conocimiento
* Publicar una base de conocimiento
* Espera de una tarea de ejecución prolongada
* Descargar una base de conocimiento
* Obtener una respuesta de una base de conocimiento
* Eliminación de una base de conocimiento

[Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker) | [Paquete](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker/1.0.0-beta.2) | [Ejemplos](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/qnamaker/sdk/preview-sdk/quickstart.java)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [JDK](https://www.oracle.com/java/technologies/javase-downloads.html)
* Cuando tenga la suscripción a Azure, cree un [recurso de QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) en Azure Portal para obtener la clave de creación y el punto de conexión. Tras su implementación, seleccione **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [JDK](https://www.oracle.com/java/technologies/javase-downloads.html)
* Las preguntas y respuestas personalizadas requieren un [recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) con la característica de respuesta a preguntas personalizada habilitada para generar una clave y un punto de conexión de API.
    * Después de la implementación del recurso de Text Analytics, seleccione **Ir al recurso**. Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a QnA Maker API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.

---

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-libraries"></a>Instalación de las bibliotecas cliente

Después de instalar Java, puede instalar las bibliotecas cliente mediante [Maven](https://maven.apache.org/) desde el [repositorio de Maven](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker).

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Cree un archivo llamado `quickstart.java` e importe las bibliotecas siguientes.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Dependencies](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=dependencies)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Dependencies](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=dependencies)]

---

Cree variables para el punto de conexión y la clave de Azure del recurso.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves de QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Keys and Endpoint** (Claves y punto de conexión), en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión de QnA Maker.

 ![Punto de conexión de creación de QnA Maker](../media/keys-endpoint.png)

- El valor de QNA_MAKER_RUNTIME_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Vaya a Azure Portal y busque el recurso de QnA Maker que creó en los requisitos previos. Haga clic en la página **Exportar plantilla**, en **Automatización**, para buscar el punto de conexión en tiempo de ejecución.

 ![Punto de conexión en tiempo de ejecución de QnA Maker](../media/runtime-endpoint.png)
   
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-java[Resource variables](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=resourceKeys)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

- Se puede usar la clave de suscripción y la clave de creación indistintamente. Para más información sobre la clave de creación, siga las indicaciones de [Claves](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- El valor de QNA_MAKER_ENDPOINT tiene el formato `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Vaya a Azure Portal y busque el recurso de Text Analytics que creó en los requisitos previos. Haga clic en la página **Claves y punto de conexión**, en **Administración de recursos** para buscar la clave de creación (suscripción) y el punto de conexión.

> [!div class="mx-imgBorder"]
> ![Punto de conexión de creación de PyR personalizado](../media/qnamaker-how-to-key-management/custom-qna-keys-and-endpoint.png)
 
- En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../key-vault/general/overview.md) proporciona almacenamiento de claves seguro.

[!code-java[Resource variables](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=resourceKeys)]

---

## <a name="object-models"></a>Modelos de objetos

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

QnA Maker usa dos modelos de objetos diferentes:
* **[QnAMakerClient](#qnamakerclient-object-model)** , el objeto para crear, administrar, publicar y descargar la base de conocimiento.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)** , el objeto para consultar la base de conocimiento con GenerateAnswer API y enviar nuevas preguntas sugeridas mediante Train API (como parte del [aprendizaje activo](../how-to/use-active-learning.md)).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

QnA Maker utiliza el siguiente modelo de objetos:
* **[QnAMakerClient](#qnamakerclient-object-model)** es el objeto para crear, administrar, publicar y descargar la base de conocimiento.

---

[!INCLUDE [Get KBinformation](./quickstart-sdk-cognitive-model.md)]

### <a name="qnamakerclient-object-model"></a>Modelo de objetos QnAMakerClient

El cliente de QnA Maker de creación es un objeto [QnAMakerClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java) que se autentica en Azure mediante MsRest::ServiceClientCredentials, que contiene la clave.

Una vez creado el cliente, use los métodos de la propiedad [Knowledgebases](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java) del cliente para crear, administrar y publicar la base de conocimiento.

En el caso de las operaciones inmediatas, un método suele devolver el resultado, si lo hay. En el caso de operaciones de ejecución prolongada, la respuesta es un objeto [Operation](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/Operation.java). Llame al método [getDetails](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Operations.java#L32) con el valor `operation.operationId` para determinar el [estado de la solicitud](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/OperationStateType.java).

### <a name="qnamakerruntimeclient-object-model"></a>Modelo de objetos QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El cliente de QnA Maker en tiempo de ejecución es un objeto [QnAMakerRuntimeClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java).

Después de publicar la base de conocimiento mediante el cliente de creación, use el método [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Runtimes.java#L36) del cliente en tiempo de ejecución para obtener una respuesta de la base de conocimiento.

Puede crear un cliente en tiempo de ejecución mediante una llamada a [QnAMakerRuntimeManager.authenticate](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerRuntimeManager.java#L29) y pasar una clave de punto de conexión en tiempo de ejecución. Para obtener la clave de punto de conexión en tiempo de ejecución, use el cliente de creación para llamar a [getKeys](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/EndpointKeys.java#L30).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

La respuesta a preguntas personalizadas no requiere el uso del objeto QnAMakerRuntimeClient. En su lugar, se llama a [generate_answer](https://github.com/Azure/azure-sdk-for-java/blob/657e9a47e4b4c7e7e7eee4100273c09468a30c63/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L308) directamente en el objeto [QnAMakerClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java).

---

## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Autenticación del cliente para la creación de la base de conocimiento

Cree una instancia de un cliente con el punto de conexión de creación y la clave de suscripción.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Authenticate](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=authenticate)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Authenticate](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=authenticate)]

---

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Una base de conocimiento almacena pares de preguntas y respuestas para el objeto [CreateKbDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/CreateKbDTO.java) procedentes de tres orígenes:

* Para **contenido editorial**, use el objeto [QnADTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QnADTO.java).
    * Para usar metadatos y avisos de seguimiento, utilice el contexto editorial, ya que estos datos se agregan en el nivel de un par de QnA individual.
* Para **archivos**, use el objeto [FileDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/FileDTO.java). El objeto FileDTO incluye el nombre de archivo así como la dirección URL pública para llegar al archivo.
* En el caso de las **direcciones URL**, use una lista de cadenas para representar las direcciones URL disponibles públicamente.

Llame al método [create](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L173) y, luego, pase la propiedad `operationId` de la operación devuelta al método [getDetails](#get-status-of-an-operation) para sondear el estado.

La última línea del código siguiente devuelve el identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Create knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=createKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Create knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=createKb)]

---

## <a name="update-a-knowledge-base"></a>Actualización de una base de conocimientos

Para actualizar una base de conocimiento, llame a [update](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L150) y pase el identificador de la base de conocimiento y un objeto [UpdateKbOperationDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTO.java). Ese objeto a su vez puede contener:
- [add](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTOAdd.java)
- [update](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTOUpdate.java)
- [delete](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTODelete.java)

Pase la propiedad `operationId` de la operación devuelta al método [getDetails](#get-status-of-an-operation) para sondear el estado.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Update knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=updateKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Update knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=updateKb)]

---

## <a name="download-a-knowledge-base"></a>Descarga de una base de conocimiento

Use el método [download](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L196) para descargar la base de datos como una lista de [QnADocumentsDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QnADocumentsDTO.java). Esto _no_ equivale a las exportaciones del portal de QnA Maker desde la página **Configuración** ya que el resultado de este método no es un archivo TSV.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Download knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=downloadKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Download knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=downloadKb)]

---

## <a name="publish-a-knowledge-base"></a>Publicación de una base de conocimientos

Publique la base de conocimiento mediante el método [publish](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L196). Esto toma el modelo actual guardado y entrenado, al que hace referencia el identificador de la base de conocimiento, y lo publica en un punto de conexión.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Publish knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=publishKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Publish knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=publishKb)]

---

## <a name="generate-an-answer-from-the-knowledge-base"></a>Generación de una respuesta de la base de conocimiento

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

Una vez publicada una base de conocimiento, necesita la clave de punto de conexión en tiempo de ejecución para consultar la base de conocimiento. Esta no es la misma que la clave de suscripción que se usa para crear el cliente de creación.

Use el método [getKeys](https://github.com/Azure/azure-sdk-for-java/blob/b637366e32edefb0fe63962983715a02c1ad2631/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/EndpointKeys.java#L30) para obtener un objeto [EndpointKeysDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/EndpointKeysDTO.java).

Cree un cliente en tiempo de ejecución mediante una llamada a [QnAMakerRuntimeManager.authenticate](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerRuntimeManager.java#L29) y pase una clave de punto de conexión en tiempo de ejecución del objeto EndpointKeysDTO.

Genere una respuesta a partir de una base de conocimiento publicada mediante el método [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Runtimes.java#L36). Este método acepta el identificador de la base de conocimiento y un objeto [QueryDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QueryDTO.java).

[!code-java[Query knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=queryKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

Genere una respuesta a partir de una base de conocimiento publicada mediante el método [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/657e9a47e4b4c7e7e7eee4100273c09468a30c63/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L308). Este método acepta el identificador de la base de conocimiento y un objeto [QueryDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QueryDTO.java).

[!code-java[Query knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=queryKb)]

---

Este es un ejemplo sencillo de consulta de una base de conocimiento. Para comprender los escenarios de consulta avanzados, revise [otros ejemplos de consulta](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

Elimine la base de conocimiento con el método [delete](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L81) y el parámetro del identificador de la base de conocimiento.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Delete knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=deleteKb)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Delete knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=deleteKb)]

---

## <a name="get-status-of-an-operation"></a>Obtención del estado de una operación

Algunos métodos, como Create y Update, pueden tardar bastante tiempo en que en lugar de esperar a que finalice el proceso, se devuelva una [operación](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/Operation.java). Use el identificador de la operación para realizar un sondeo (con lógica de reintento) para determinar el estado del método original.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Wait for operation](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=waitForOperation)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Wait for operation](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=waitForOperation)]

---

## <a name="run-the-application"></a>Ejecución de la aplicación

Este es el método principal de la aplicación.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

[!code-java[Main method](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=main)]

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

[!code-java[Main method](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=main)]

---

Ejecute la aplicación de la siguiente manera. Esto supone que el nombre de clase es `Quickstart` y que las dependencias se encuentran en una subcarpeta llamada `lib` debajo de la carpeta actual.

```console
javac Quickstart.java -cp .;lib\*
java -cp .;lib\* Quickstart
```

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/version-1)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/qnamaker/sdk/quickstart.java).

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/version-2)

El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/qnamaker/sdk/preview-sdk/quickstart.java).

---
