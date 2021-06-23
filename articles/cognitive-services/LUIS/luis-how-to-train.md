---
title: 'Entrenamiento de aplicaciones: LUIS'
titleSuffix: Azure Cognitive Services
description: El entrenamiento es el proceso de enseñar a la versión de la aplicación de Language Understanding (LUIS) a mejorar su comprensión del lenguaje natural. Entrene la aplicación de LUIS después de realizar actualizaciones en el modelo, como la adición, la edición, el etiquetado o la eliminación de entidades, intenciones o expresiones.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 11/15/2019
ms.openlocfilehash: 0ff8376eea2eca459b93adbf46b81dbeb7639d40
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111954330"
---
# <a name="train-your-active-version-of-the-luis-app"></a>Entrenamiento de la versión activa de la aplicación de LUIS

El entrenamiento es el proceso de enseñar a la aplicación Language Understanding (LUIS) para mejorar su comprensión del lenguaje natural. Entrene la aplicación de LUIS después de realizar actualizaciones en el modelo, como la adición, la edición, el etiquetado o la eliminación de entidades, intenciones o expresiones.

El entrenamiento y la [prueba](./luis-interactive-test.md) de una aplicación es un proceso iterativo. Tras entrenar la aplicación de LUIS, pruébela con expresiones de ejemplo para ver si las entidades e intenciones se reconocen correctamente. En caso contrario, realice actualizaciones en la aplicación de LUIS, entrénela y pruébela de nuevo.

El entrenamiento se aplica a la versión activa en el portal de LUIS.

## <a name="how-to-train-interactively"></a>Cómo entrenar interactivamente

Para iniciar el proceso iterativo en el [portal de LUIS](https://www.luis.ai), primero debe entrenar la aplicación de LUIS al menos una vez. Asegúrese de que cada intención tiene al menos una expresión antes del entrenamiento.

1. Acceda a la aplicación mediante la selección de su nombre en la página **Mis aplicaciones**.

1. En la aplicación, seleccione **Entrenar** en el panel superior.

1. Una vez completado el entrenamiento, aparece una notificación en la parte superior del explorador.

## <a name="training-date-and-time"></a>Fecha y hora del entrenamiento

La fecha y la hora del entrenamiento son GMT + 2.

## <a name="train-with-all-data"></a>Entrenamiento con todos los datos

El entrenamiento usa un pequeño porcentaje de muestreo negativo. En su lugar, puede usar todos los datos disponibles mediante el portal o la API. 

### <a name="using-the-luis-portal"></a>Mediante el portal de LUIS

Inicie sesión en el [portal de LUIS](https://www.luis.ai/) y haga clic en la aplicación. Seleccione **Administrar** en la parte superior de la pantalla y, después, seleccione **Configuración** y habilite o deshabilite la opción **Use non-deterministic training** (Usar aprendizaje no determinista). Cuando está deshabilitada, el aprendizaje usará todos los datos disponibles.

![Botón para habilitar o deshabilitar el aprendizaje no determinista](./media/non-determinstic-training.png)

### <a name="using-the-version-settings-api"></a>Uso de la API de configuración de versión

Use la [API de configuración de versión](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) con el elemento `UseAllTrainingData` establecido en true para desactivar esta característica.

## <a name="unnecessary-training"></a>Entrenamiento innecesario

No es necesario entrenar después de realizar un cambio. El entrenamiento debe realizarse después de que se aplique un grupo de cambios al modelo y el siguiente paso que debe hacer es probar o publicar. Si no necesita probar ni publicar, el entrenamiento no es necesario.

## <a name="training-with-the-rest-apis"></a>Entrenamiento con las API REST

Para el entrenamiento en el portal de LUIS, solo debe pulsar el botón **Entrenar**. El proceso de entrenamiento con las API REST consta de dos pasos. El primero consiste en [solicitar entrenamiento](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c45) con HTTP POST. A continuación, debe solicitar el [estado del entrenamiento](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c46) con HTTP Get.

Para saber cuando concluirá el entrenamiento, debe sondear el estado hasta que todos los modelos estén completamente entrenados.

## <a name="next-steps"></a>Pasos siguientes

* [Pruebas interactivas](luis-interactive-test.md)
* [Pruebas por lotes](luis-how-to-batch-test.md)