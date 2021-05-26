---
title: 'Uso del aprendizaje activo con la base de conocimiento: QnA Maker'
description: Aprenda a mejorar la calidad de la base de conocimiento con el aprendizaje activo. Revise, acepte o rechace, o agregue sin quitar ni cambiar las preguntas existentes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: 4d346610ecb4e0552a52775c086eeb16e6e11680
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110373022"
---
# <a name="active-learning"></a>Aprendizaje activo

La característica _Sugerencias de aprendizaje activo_ le permiten mejorar la calidad de la base de conocimiento al sugerir preguntas alternativas, en función de los envíos del usuario, para el par de preguntas y respuestas. Usted revisa las sugerencias, ya sea al agregarlas a preguntas existentes o rechazarlas.

La base de conocimiento no cambia automáticamente. Debe aceptar las sugerencias para que cualquier cambio surta efecto. Estas sugerencias agregan preguntas, pero no cambian ni quitan preguntas existentes.

## <a name="what-is-active-learning"></a>¿Qué es el aprendizaje activo?

QnA Maker aprende las nuevas variaciones de las preguntas con comentarios implícitos y explícitos.

* [Comentarios implícitos:](#how-qna-makers-implicit-feedback-works) el clasificador entiende cuando la pregunta de un usuario tiene varias respuestas con puntuaciones muy similares y lo considera un comentario. No es necesario hacer nada para que esto ocurra.
* [Comentarios explícitos:](#how-you-give-explicit-feedback-with-the-train-api) cuando se devuelven desde la base de conocimiento varias respuestas con poca variación en las puntuaciones, la aplicación cliente le pregunta al usuario cuál es la pregunta correcta. Los comentarios explícitos del usuario se envían a QnA Maker con la [API Train](../How-To/improve-knowledge-base.md#train-api).

Ambos métodos proporcionan al clasificador consultas similares que se agrupan en clústeres.

## <a name="how-active-learning-works"></a>Cómo funciona el aprendizaje activo

El aprendizaje activo se desencadena en función de las puntuaciones de las mejores respuestas devueltas por QnA Maker. Si las diferencias de puntuación entre los pares de QnA que coinciden con la consulta se encuentran dentro de un intervalo pequeño, la consulta se considera una posible sugerencia (como una pregunta alternativa) para cada uno de los pares de QnA posibles. Una vez que acepta la pregunta sugerida para un par específico de QnA, se rechaza para los otros pares. Debe recordar, guardar y entrenar, después de aceptar las sugerencias.

El aprendizaje activo ofrece las mejores sugerencias posibles en los casos donde los puntos de conexión reciben una cantidad razonable y una variedad de consultas de uso. Cuando 5 o más consultas similares se agrupan en clústeres, cada 30 minutos, QnA Maker sugiere las preguntas basadas en el usuario al diseñador de la base de conocimiento para que las acepte o rechace. Todas las sugerencias se agrupan en clústeres por similitud, y las sugerencias principales para preguntas alternativas se muestran según la frecuencia de las consultas en particular por los usuarios finales.

Una vez que se sugieren preguntas en el portal de QnA Maker, deberá revisar y aceptar o rechazar las sugerencias. No hay ninguna API para administrar las sugerencias.

## <a name="how-qna-makers-implicit-feedback-works"></a>Cómo funcionan los comentarios implícitos de QnA Maker

Los comentarios implícitos de QnA Maker usan un algoritmo para determinar la proximidad de la puntuación y, después, hacen sugerencias de aprendizaje activo. El algoritmo para determinar la proximidad no es un cálculo sencillo. Los intervalos en el ejemplo siguiente no están diseñados para ser fijos, pero deben usarse como guía para comprender el impacto del algoritmo únicamente.

Cuando la puntuación de una pregunta tiene una confianza alta, por ejemplo, un 80 %, el intervalo de puntuaciones que se considera para el aprendizaje activo es amplio, dentro del 10 % aproximadamente. A medida que la puntuación de confianza se reduce, por ejemplo, un 40 %, el intervalo de puntuaciones también disminuye aproximadamente dentro del 4 %.

En la respuesta JSON siguiente de una consulta a generateAnswer de QnA Maker, las puntuaciones de A, B y C son cercanas y se podrían considerar como sugerencias.

```json
{
  "activeLearningEnabled": true,
  "answers": [
    {
      "questions": [
        "Q1"
      ],
      "answer": "A1",
      "score": 80,
      "id": 15,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q2"
      ],
      "answer": "A2",
      "score": 78,
      "id": 16,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q3"
      ],
      "answer": "A3",
      "score": 75,
      "id": 17,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q4"
      ],
      "answer": "A4",
      "score": 50,
      "id": 18,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    }
  ]
}
```

QnA Maker no sabrá qué respuesta es la mejor respuesta. Use la lista de sugerencias del portal de QnA Maker para seleccionar la mejor respuesta y entrenar nuevamente.


## <a name="how-you-give-explicit-feedback-with-the-train-api"></a>Cómo proporcionar comentarios explícitos con Train API

QnA Maker necesita comentarios explícitos sobre cuál de las respuestas era la mejor. La forma de determinar la mejor respuesta depende de usted y puede incluir:

* Comentarios del usuario, seleccione una de las respuestas.
* Lógica de negocios, como determinar un intervalo de puntuación aceptable.
* Una combinación de los comentarios del usuario y la lógica de negocios.

Use [Train API](/rest/api/cognitiveservices/qnamaker4.0/runtime/train) para enviar la respuesta correcta a QnA Maker, después de que el usuario la seleccione.

## <a name="upgrade-runtime-version-to-use-active-learning"></a>Actualización de la versión en tiempo de ejecución para usar el aprendizaje activo

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

El aprendizaje activo se admite en el tiempo de ejecución versión 4.4.0 y superior. Si la base de conocimiento se creó en una versión anterior, [actualice el tiempo de ejecución](configure-QnA-Maker-resources.md#get-the-latest-runtime-updates) para usar esta característica.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

En la respuesta a preguntas personalizada (versión preliminar), no es necesario actualizar el entorno de ejecución manualmente.

---

## <a name="turn-on-active-learning-for-alternate-questions"></a>Activación del aprendizaje activo para preguntas alternativas

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

El aprendizaje activo está desactivado de forma predeterminada. Actívelo para ver preguntas sugeridas. Una vez activado el aprendizaje activo, deberá enviar información desde la aplicación cliente a QnA Maker. Para obtener más información, consulte [Flujo arquitectónico para usar GenerateAnswer y Train API desde un bot](improve-knowledge-base.md#architectural-flow-for-using-generateanswer-and-train-apis-from-a-bot).

1. Seleccione **Publicar** para publicar la base de conocimiento. Las consultas de aprendizaje activo se recopilan solo desde el punto de conexión de predicción de GenerateAnswer API. Las consultas en el panel de prueba del portal de QnA Maker no afectarán al aprendizaje activo.

1. Para activar el aprendizaje activo en el portal de QnA Maker, vaya a la esquina superior derecha, seleccione su **Nombre** y vaya a [**Configuración del servicio**](https://www.qnamaker.ai/UserSettings).

    ![Active las preguntas sugeridas alternativas de aprendizaje activo desde la página Configuración del servicio. Seleccione su nombre de usuario en el menú superior derecho y seleccione Continuación del servicio.](../media/improve-knowledge-base/Endpoint-Keys.png)


1. Busque el servicio QnA Maker, y active **Aprendizaje activo**.

    > [!div class="mx-imgBorder"]
    > [![En la página Configuración del servicio, active la característica Aprendizaje activo. Si no es capaz de activar o desactivar la característica, deberá actualizar el servicio.](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png)](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png#lightbox)

    > [!Note]
    > La versión exacta de la imagen anterior se muestra solo como un ejemplo. Su versión puede ser diferente.

    Una vez que **Aprendizaje activo** está habilitado, la base de conocimiento sugiere nuevas preguntas a intervalos regulares según las preguntas enviadas por el usuario. Para deshabilitar **Aprendizaje activo**, vuelva a cambiar la configuración.
    
# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

De forma predeterminada, el aprendizaje activo está **activado** en la respuesta a preguntas personalizada (versión preliminar). Para ver las preguntas alternativas sugeridas, [use las opciones de vista](../How-To/improve-knowledge-base.md#view-suggested-questions) en la página de edición.

---

## <a name="review-suggested-alternate-questions"></a>Revisión de preguntas alternativas sugeridas

[Revise las preguntas sugeridas alternativas](improve-knowledge-base.md) en la página **Editar** de cada base de conocimiento.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una base de conocimientos](./manage-knowledge-bases.md)
