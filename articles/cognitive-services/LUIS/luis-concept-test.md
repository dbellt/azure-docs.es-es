---
title: Probar la aplicación de LUIS
titleSuffix: Azure Cognitive Services
description: La realización de pruebas es el proceso por el cual se proporcionan expresiones de ejemplo a LUIS y se obtiene una respuesta de intenciones y entidades reconocidas por LUIS.
manager: nitinme
services: cognitive-services
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/10/2019
ms.openlocfilehash: b33f765e936d7c0db301a5b2fcf38ccaf137f771
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111352511"
---
# <a name="testing-example-utterances-in-luis"></a>Prueba de expresiones de ejemplo en LUIS

La realización de pruebas es el proceso por el cual se proporcionan expresiones de ejemplo a LUIS y se obtiene una respuesta de intenciones y entidades reconocidas por LUIS. 

Puede probar LUIS de forma interactiva, con una expresión cada vez o con un conjunto de expresiones. Durante las pruebas, puede comparar la respuesta de predicción del modelo activo actual con la respuesta de predicción del modelo publicado. 

<a name="A-test-score"></a>
<a name="Score-all-intents"></a>
<a name="E-(exponent)-notation"></a>

## <a name="what-is-a-score-in-testing"></a>¿Qué es una puntuación en las pruebas?
Vea los conceptos de [puntuación de predicción](luis-concept-prediction-score.md) para obtener más información sobre las puntuaciones de predicción.

## <a name="interactive-testing"></a>Pruebas interactivas
Las pruebas interactivas se llevan a cabo desde el panel **Test** (Prueba) del portal de LUIS. Puede introducir una expresión para ver cómo se identifican y puntúan las intenciones y las entidades. Si LUIS no predice en el panel de pruebas las intenciones y entidades tal como espera en una expresión, cópiela en la página **Intent** (Intención) como una expresión nueva. Después, etiquete las partes de esa expresión para las entidades y entrene LUIS. 

## <a name="batch-testing"></a>Pruebas por lotes
Vea [Pruebas por lotes](./luis-how-to-batch-test.md) si va a probar más de una expresión a la vez.

## <a name="endpoint-testing"></a>Pruebas de punto de conexión
Puede hacer pruebas mediante el [punto de conexión](luis-glossary.md#endpoint) con un máximo de dos versiones de la aplicación. Con la versión principal o activa de la aplicación establecida como punto de conexión de **producción**, agregue una segunda versión al punto de conexión de **ensayo**. Este enfoque le proporciona tres versiones de una expresión: el modelo actual en el panel Prueba del sitio web de [LUIS](luis-reference-regions.md) y las dos versiones en los dos puntos de conexión diferentes. 

Todas las pruebas de punto de conexión cuentan para la cuota de uso. 

## <a name="do-not-log-tests"></a>No registrar pruebas
Si hace una prueba en un punto de conexión y no quiere que la expresión quede registrada, recuerde que debe usar la configuración de cadena de consulta `logging=false`.

## <a name="where-to-find-utterances"></a>Dónde encontrar expresiones
LUIS almacena todas las expresiones registradas en el registro de consultas, que se puede descargar en la página de la lista **Apps** (Aplicaciones) del portal de LUIS, así como las [API de creación](https://go.microsoft.com/fwlink/?linkid=2092087) de LUIS. 

Todas las expresiones de las que LUIS no esté seguro aparecerán en la página **[Revisión de las expresiones de punto de conexión](luis-how-to-review-endpoint-utterances.md)** del sitio web de [LUIS](luis-reference-regions.md). 

## <a name="remember-to-train"></a>No se olvide del entrenamiento
No olvide [entrenar](luis-how-to-train.md) a LUIS después de hacer cambios en el modelo. Los cambios efectuados en la aplicación de LUIS no se ven en las pruebas hasta que se entrena la aplicación. 

## <a name="best-practices"></a>Procedimientos recomendados
Obtenga información sobre los [procedimientos recomendados](luis-concept-best-practices.md).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre las [pruebas](luis-interactive-test.md) de las expresiones.