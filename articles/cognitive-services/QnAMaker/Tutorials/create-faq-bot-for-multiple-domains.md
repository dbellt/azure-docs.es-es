---
title: 'Tutorial: Creación de un bot de preguntas frecuentes para varios dominios con Azure Bot Service'
description: En este tutorial se crea un bot de preguntas frecuentes sin código para casos de uso en producción, con QnA Maker y Azure Bot Service.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.author: diagarw
ms.date: 03/31/2021
ms.openlocfilehash: d79eed22d441949810cfc1738f3af2c0f8703adc
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110116476"
---
# <a name="add-multiple-domains-to-your-faq-bot"></a>Incorporación de varios dominios al bot de preguntas frecuentes

Al crear un bot de preguntas frecuentes, puede encontrar casos de uso que requieran que aborde consultas en varios dominios. Supongamos que el equipo de marketing de Microsoft quiere crear un bot de ayuda al cliente que responda a las consultas comunes del usuario sobre varios productos de Surface. Para simplificar el ejercicio, usaremos una URL de preguntas frecuentes sobre el [lápiz de Surface](https://support.microsoft.com/surface/how-to-use-your-surface-pen-8a403519-cd1f-15b2-c9df-faa5aa924e98) y los [auriculares de Surface](https://support.microsoft.com/surface/use-surface-earbuds-aea108c3-9344-0f11-e5f5-6fc9f57b21f9) para crear la base de conocimientos.

Puede diseñar el bot para controlar las consultas en varios dominios con QnA Maker de las siguientes maneras:

* Cree una única base de conocimiento y etiquete los pares de QnA en dominios distintos con metadatos.
* Cree una base de conocimientos independiente para cada dominio.
* Cree un recurso de QnA Maker independiente para cada dominio.

## <a name="create-a-single-knowledge-base-and-tag-qna-pairs-into-distinct-domains-with-metadata"></a>Cree una única base de conocimiento y etiquete los pares de QnA en dominios distintos con metadatos.

Los autores de contenido pueden usar documentos para extraer preguntas y respuestas o agregar preguntas y respuestas personalizadas a la base de conocimiento. Para agrupar estas preguntas y respuestas en dominios o categorías específicos, puede agregar [metadatos](../How-To/query-knowledge-base-with-metadata.md) a los pares de preguntas y respuestas.

En cuanto al bot de productos Surface, puede realizar los pasos siguientes para crear un bot que responda a las consultas de ambos tipos de productos:

1. Agregue las siguientes direcciones URL de preguntas frecuentes en los productos de Surface en el PASO 3 de la página Crear KB y haga clic en "Crear la base de conocimiento". Después de extraer los pares de preguntas y respuestas de estos orígenes, se crea una nueva base de conocimiento. 
   
   [Preguntas frecuentes del lápiz de Surface](https://support.microsoft.com/surface/how-to-use-your-surface-pen-8a403519-cd1f-15b2-c9df-faa5aa924e98)<br>[Preguntas frecuentes sobre los auriculares de Surface](https://support.microsoft.com/surface/use-surface-earbuds-aea108c3-9344-0f11-e5f5-6fc9f57b21f9)
 
2. Después de crear la base de conocimiento, puede ir a **Ver opciones** y hacer clic en **Mostrar metadatos**. Se abrirá una columna de metadatos para las preguntas y respuestas.

   >[!div class="mx-imgBorder"]
   >[![Mostrar metadatos]( ../media/qnamaker-tutorial-updates/show-metadata.png)]( ../media/qnamaker-tutorial-updates/expand/show-metadata.png#lightbox)


3. En esta base de conocimiento, tenemos las preguntas y respuestas en dos productos y nos gustaría distinguirlas de forma que podamos buscar respuestas entre las preguntas y respuestas de un producto determinado. Para ello, debemos actualizar el campo de metadatos para los pares de preguntas y respuestas en consecuencia. 

   Como puede ver en el ejemplo siguiente, hemos agregado metadatos con el **producto** como clave y **surface_pen** o **surface_earbuds** como valores, siempre que corresponda. Puede ampliar este ejemplo para extraer datos de varios productos y agregar un valor diferente para cada producto.

   >[!div class="mx-imgBorder"]
   >[![Metadatos]( ../media/qnamaker-tutorial-updates/metadata-example-2.png) ]( ../media/qnamaker-tutorial-updates/expand/metadata-example-2.png#lightbox)

4. Ahora, para restringir el sistema para buscar la respuesta en un producto determinado, tendrá que pasar ese producto como filtro estricto en la API Generate Answer.

    Aprenda a [usar la API GenerateAnswer](../How-To/metadata-generateanswer-usage.md). La dirección URL de GenerateAnswer tiene el formato siguiente:
    ```
    https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
    ```

    En el cuerpo JSON de la llamada API, hemos pasado *surface_pen* como valor para el *producto* de metadatos. Por lo tanto, el sistema solo buscará la respuesta entre los pares de preguntas y respuestas con los mismos metadatos. 

    ```json
    {
        "question": "What is the price?",
        "top": 6,
        "isTest": true,
        "scoreThreshold": 30,
        "rankerType": ""  // values: QuestionOnly
        "strictFilters": [
        {
            "name": "product",
            "value": "surface_pen"
        }],
        "userId": "sd53lsY="
    }
    ```

    Puede obtener el valor de los metadatos en función de la entrada de usuario de las maneras siguientes: 

    * Tome explícitamente el dominio como entrada de usuario a través del cliente bot. Por ejemplo, tal como se muestra a continuación, puede tomar la categoría de producto como entrada de usuario cuando se inicia la conversación.

      ![Toma de la entrada de metadatos](../media/qnamaker-tutorial-updates/expand/explicit-metadata-input.png)

    * Identifique implícitamente el dominio en función del contexto del bot. Por ejemplo, en caso de que la pregunta anterior se realizara en un producto de Surface determinado, el cliente puede guardarla como contexto. Si el usuario no especifica el producto en la siguiente consulta, puede pasar el contexto del bot como metadatos a la API Generate Answer.

      ![Pasar contexto]( ../media/qnamaker-tutorial-updates/expand/extract-metadata-from-context.png)

    * Extraiga la entidad de la consulta de usuario para identificar el dominio que se usará en el filtro de metadatos. Puede usar otras instancias de Cognitive Services como Text Analytics y LUIS para la extracción de entidades.

      ![Extracción de metadatos de la consulta]( ../media/qnamaker-tutorial-updates/expand/extract-metadata-from-query.png)

### <a name="how-large-can-our-knowledge-bases-be"></a>¿Qué tamaño pueden tener nuestras bases de conocimiento? 

Puede agregar hasta 50 000 pares de preguntas y respuestas a una única base de conocimiento. Si los datos superan los 50 000 pares de preguntas y respuestas, debe considerar la posibilidad de dividir la base de conocimiento.

## <a name="create-a-separate-knowledge-base-for-each-domain"></a>Creación de una base de conocimiento independiente para cada dominio

También puede crear una base de conocimiento para cada dominio y mantener cada base por separado. Todas las API requieren que el usuario pase el id. de knowledge base para realizar cualquier actualización a la base de conocimiento o capturar una respuesta a la pregunta del usuario.  

Cuando el servicio reciba la pregunta del usuario, deberá pasar el id. de KB en el punto de conexión "Generar respuesta" que se mostró anteriormente para obtener una respuesta de la base de conocimiento correspondiente. Puede encontrar el id. de knowledge base en la sección de la página de **publicación**, tal como se muestra a continuación.

>[!div class="mx-imgBorder"]
>![Captura del id. de KB](../media/qnamaker-tutorial-updates/fetch-kb-id.png)

## <a name="create-a-separate-qna-maker-resource-for-each-domain"></a>Cree un recurso de QnA Maker independiente para cada dominio.

Supongamos que el equipo de marketing de Microsoft quiere crear un bot de ayuda al cliente que responda a las consultas del usuario sobre varios productos de Surface y Xbox. Para ello, planea asignar distintos equipos para acceder a las bases de conocimiento en Surface y Xbox. En este caso, se recomienda crear dos recursos de QnA Maker: uno para Surface y otro para Xbox. Sin embargo, puede definir roles distintos para los usuarios que acceden al mismo recurso. Obtenga más información sobre el [control de acceso basado en roles](../How-To/manage-qna-maker-app.md). 
