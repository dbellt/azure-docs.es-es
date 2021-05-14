---
title: 'Intenciones y entidades: LUIS'
titleSuffix: Azure Cognitive Services
description: Una intención individual representa una tarea o acción que el usuario quiere realizar. Es una finalidad o un objetivo que se expresa en la expresión de un usuario. Defina un conjunto de intenciones que se correspondan con las acciones que los usuarios quieren realizar en la aplicación.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/13/2021
ms.openlocfilehash: 8e76e3e7683d43a7a39bc0c168a29016a988c705
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107499417"
---
# <a name="intents-in-your-luis-app"></a>Intenciones en la aplicación de LUIS

Una intención representa una tarea o acción que el usuario quiere realizar. Es una finalidad o un objetivo que se expresa en la [expresión](luis-concept-utterance.md) de un usuario.

Defina un conjunto de intenciones que se correspondan con las acciones que los usuarios quieren realizar en la aplicación. Por ejemplo, una aplicación de viajes define varias intenciones:

Intenciones de la aplicación de viajes   |   Expresiones de ejemplo   |
------|------|
 BookFlight     |   "Resérvame un vuelto a Río la próxima semana" <br/> "Llévame a Río en avión el 24" <br/> "Necesito un billete de avión para el próximo domingo a Río de Janeiro"    |
 Saludo     |   "Buenas" <br/>"Hello" <br/>"Buenos días"  |
 CheckWeather | "¿Qué tiempo hace en Boston?" <br/> "Muéstrame la previsión de este fin de semana" |
 None         | "Dame una receta de galletas"<br>"¿Ganaron los Lakers?" |

Todas las aplicaciones vienen con la intención predefinida "[None](#none-intent)", que es la intención de reserva.

## <a name="prebuilt-domains-provide-intents"></a>Los dominios creados previamente proporcionan intenciones
Además de las intenciones que defina, puede usar intenciones creadas previamente desde uno de los [dominios creados previamente](./howto-add-prebuilt-models.md).

## <a name="return-all-intents-scores"></a>Devolver las puntuaciones de todas las intenciones
Asigne una expresión a una única intención. Cuando LUIS recibe una expresión en el punto de conexión, de manera predeterminada, devuelve la intención superior para esa expresión.

Si quiere puntuaciones para todas las intenciones de la expresión, puede proporcionar una marca en la cadena de consulta de la API Prediction.

|API Prediction|Marca|
|--|--|
|V2|`verbose=true`|
|V3|`show-all-intents=true`|

## <a name="intent-compared-to-entity"></a>Comparación de intención con entidad
La intención representa la acción que la aplicación debe realizar para el usuario y se basa en la expresión completa. Una expresión solo puede tener una intención de puntuación superior, pero puede tener muchas entidades.

<a name="how-do-intents-relate-to-entities"></a>

 Cree una intención cuando la _intención_ del usuario desencadenaría una acción en la aplicación cliente, como una llamada a la función checkweather(). Después, cree entidades para representar los parámetros necesarios para ejecutar la acción.

|Intención   | Entidad | Expresión de ejemplo   |
|------------------|------------------------------|------------------------------|
| CheckWeather | { "type": "location", "entity": "Seattle" }<br>{ "type": "builtin.datetimeV2.date","entity": "tomorrow","resolution":"2018-05-23" } | What's the weather like in `Seattle` `tomorrow`? |
| CheckWeather | { "type": "date_range", "entity": "this weekend" } | Show me the forecast for `this weekend` (Muéstrame la previsión de este fin de semana) |
||||

## <a name="prebuilt-domain-intents"></a>Intenciones de dominio creados previamente

Los [dominios creados previamente](./howto-add-prebuilt-models.md) proporcionan intenciones con expresiones.

## <a name="none-intent"></a>Intención None

La intención **None** se ha creado y dejado en blanco de forma intencionada. La intención **None** es obligatoria y no se puede eliminar ni se puede cambiar su nombre. Rellénela con expresiones que se encuentren fuera de su dominio.

La intención **None** es la intención de reserva, importante en todas las aplicaciones, y debe tener un 10 % del total de expresiones. Se usa para enseñar a LUIS expresiones que no son importantes en el dominio de la aplicación (área de asunto). Si no agrega ninguna expresión a la intención **None**, LUIS fuerza una expresión que está fuera del dominio en una de las intenciones del dominio. Esto sesgará las puntuaciones de la predicción al enseñar a LUIS la intención incorrecta para la expresión.

Cuando una expresión se ha predicho como la intención None, la aplicación cliente puede hacer más preguntas o proporcionar un menú para dirigir al usuario a opciones válidas.

## <a name="negative-intentions"></a>Intenciones negativas
Si quiere determinar intenciones positivas y negativas, como "**Quiero** un coche" y "**No** quiero un coche", puede crear dos intenciones (una positiva y otra negativa) y agregar expresiones adecuadas para cada una. O bien, puede crear una única intención y marcar los dos términos positivos y negativos diferentes como una entidad.

## <a name="intents-and-patterns"></a>Intenciones y patrones

Si tiene expresiones de ejemplo, que se pueden definir parcial o totalmente como una expresión regular, considere el uso de la [entidad de expresión regular](luis-concept-entity-types.md#regex-entity) emparejada con un [patrón](luis-concept-patterns.md).

El uso de una entidad de expresión regular garantiza la extracción de datos para buscar la coincidencia con el patrón. La coincidencia de patrones garantiza que se devuelve una intención exacta.

## <a name="intent-balance"></a>Equilibrio entre las intenciones
Las intenciones del dominio de la aplicación deben tener un equilibrio entre las expresiones de cada intención. No tenga una intención con 10 expresiones y otra con 500, ya que no están equilibradas. Si se encuentra en esta situación, revise la intención con 500 expresiones para ver si muchas de ellas se pueden reorganizar en un [patrón](luis-concept-patterns.md).

La intención **None** no se incluye en este equilibrio. Esa intención debe contener un 10 % de las expresiones totales de la aplicación.

## <a name="intent-limits"></a>Límites de las intenciones
Consulte los [límites](luis-limits.md#model-boundaries) para comprender cuántas intenciones puede agregar a un modelo.

### <a name="if-you-need-more-than-the-maximum-number-of-intents"></a>Si necesita más del número máximo de intenciones
En primer lugar, evalúe si el sistema usa demasiadas intenciones.

### <a name="can-multiple-intents-be-combined-into-single-intent-with-entities"></a>Varias intenciones se pueden combinar en una única intención con entidades
Si hay intenciones muy similares, esto puede dificultar que LUIS las distinga. Las intenciones deben ser lo suficientemente diferentes como para capturar las tareas principales que pregunta el usuario, pero no tienen que capturar cada ruta que realice el código. Por ejemplo, es posible que "BookFlight" y "FlightCustomerService" deban ser intenciones diferentes en una aplicación de viajes, pero "BookInternationalFlight" y "BookDomesticFlight" son demasiado parecidas. Si el sistema necesita distinguirlas, use entidades u otra lógica en lugar de intenciones.

### <a name="dispatcher-model"></a>Modelo de distribuidor
Obtenga más información sobre cómo combinar LUIS y aplicaciones de QnA Maker con el [modelo de distribución](luis-concept-enterprise.md#when-you-need-to-combine-several-luis-and-qna-maker-apps).

### <a name="request-help-for-apps-with-significant-number-of-intents"></a>Solicitar ayuda para las aplicaciones con un número considerable de intenciones
Si reducir el número de intenciones o dividir las intenciones en varias aplicaciones no funciona, póngase en contacto con el soporte técnico. Si la suscripción a Azure incluye servicios de soporte técnico, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre las [entidades](luis-concept-entity-types.md), que son palabras importantes y relevantes para las intenciones.
* Obtenga información sobre cómo [agregar y administrar intenciones](luis-how-to-add-intents.md) en su aplicación de LUIS.
* Revise los [procedimientos recomendados](luis-concept-best-practices.md) de las intenciones.