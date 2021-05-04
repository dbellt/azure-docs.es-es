---
title: 'Inicio rápido: Compilación de la aplicación en el portal del servicio LUIS'
description: En este inicio rápido se indica cómo crear una aplicación de LUIS que utiliza el dominio pregenerado `HomeAutomation` para encender y apagar las luces y los dispositivos. Este dominio pregenerado le proporciona intenciones, entidades y expresiones de ejemplo. Cuando termine, tendrá un punto de conexión LUIS que se ejecuta en la nube.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: quickstart
ms.date: 04/14/2021
ms.openlocfilehash: 3de83151aa00d589c470eb7ac772f4c9b5f7eda2
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107948372"
---
# <a name="quickstart-build-your-app-in-luis-portal"></a>Inicio rápido: Compilación de la aplicación en el portal del servicio LUIS

En este inicio rápido, creará una aplicación del servicio LUIS mediante el dominio pregenerado HomeAutomation para encender y apagar las luces y los dispositivos. Este dominio pregenerado le proporciona intenciones, entidades y expresiones de ejemplo. A continuación, personalice la aplicación mediante la incorporación de nuevas entidades e intenciones. Cuando termine, tendrá un punto de conexión LUIS que se ejecuta en la nube.

[!INCLUDE [Sign in to LUIS](./includes/sign-in-process.md)]

[!INCLUDE [Select authoring resource](./includes/select-authoring-resource.md)]

## <a name="create-a-new-app"></a>Creación de una nueva aplicación

Las aplicaciones se pueden crear y administrar en **My Apps** (Mis aplicaciones).

### <a name="create-an-application"></a>Crear una aplicación

Para crear una aplicación, haga clic en  **+ Nueva aplicación**. 

En la ventana que aparece, escriba la siguiente información:

|Nombre  |Descripción  |
|---------|---------|
|Nombre     | Un nombre para la aplicación. Por ejemplo, "home automation".        |
|Cultura     | El idioma que la aplicación entiende y habla.   |
|Descripción | La descripción de la aplicación.
|Recurso de predicción | El recurso de predicción que recibirá las consultas. |

Seleccione **Listo**.

>[!NOTE]
>La referencia cultural no se puede cambiar una vez creada la aplicación.

## <a name="add-prebuilt-domain"></a>Incorporación de un dominio creado previamente

LUIS ofrece un conjunto de dominios pregenerados que le ayudarán a empezar a trabajar con la aplicación. Una aplicación de dominio pregenerado ya viene con [intenciones](./luis-concept-intent.md), [entidades](./luis-concept-entity-types.md) [y expresiones](./luis-concept-utterance.md) incluidas.

1. En el panel de navegación izquierdo, seleccione **Dominios creados previamente**.
2. Busque **HomeAutomation**.
3. Seleccione **Add domain** (Agregar dominio) en la tarjeta HomeAutomation.

    > [!div class="mx-imgBorder"]
    > ![Seleccione "Dominios creados previamente" y, luego, busque "HomeAutomation". Seleccione "Agregar dominio" en la tarjeta HomeAutomation.](media/luis-quickstart-new-app/home-automation.png)

    Cuando el dominio se agrega correctamente, el cuadro de dominio pregenerado muestra un botón **Remove domain** (Quitar dominio).

## <a name="check-out-intents-and-entities"></a>Comprobación de intenciones y entidades

1. Seleccione **Intenciones** en el menú de navegación izquierdo para ver las intenciones del dominio HomeAutomation. Tiene expresiones de ejemplo, como `HomeAutomation.QueryState` y `HomeAutomation.SetDevice`.

    > [!NOTE]
    > **None** (Ninguna) es una intención que proporcionan todas las aplicaciones de LUIS. Se usa para controlar las expresiones que no corresponden a la funcionalidad que proporciona la aplicación.

2. Seleccione la intención **HomeAutomation.TurnOff**. La intención contiene una lista de expresiones de ejemplo que están etiquetadas con entidades.

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla de la intención HomeAutomation.TurnOff](media/luis-quickstart-new-app/home-automation-turnoff.png "Captura de pantalla de la intención HomeAutomation.TurnOff")](media/luis-quickstart-new-app/home-automation-turnoff.png)

3. Si desea ver las entidades de la aplicación, seleccione **Entidades**. Si hace clic en una de las entidades, como **HomeAutomation.DeviceName** verá una lista de valores que tiene asociados. 
 
    :::image type="content" source="media/luis-quickstart-new-app/entities-page.png" alt-text="Texto alternativo de la imagen" lightbox="media/luis-quickstart-new-app/entities-page.png":::

## <a name="train-the-luis-app"></a>Entrenamiento de la aplicación de LUIS
Una vez que la aplicación incluya las intenciones, entidades y expresiones, deberá entrenarla para que refleje los cambios que haya realizado.

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="test-your-app"></a>Prueba de la aplicación

Una vez que haya entrenado a su aplicación, puede probarla.

1. Select **Test** (Probar) en el panel de navegación superior derecho.

1. Escriba una expresión de prueba en el panel interactivo y presione Entrar. Por ejemplo, *Turn off the lights*.

    En este ejemplo, la expresión *Turn off the lights* (Apagar las luces) aparece correctamente identificada como la intención con una puntuación más elevada en **HomeAutomation.TurnOff**.

    :::image type="content" source="media/luis-quickstart-new-app/review-test-inspection-pane-in-portal.png" alt-text="Captura de pantalla del panel de prueba, con una expresión resaltada" lightbox="media/luis-quickstart-new-app/review-test-inspection-pane-in-portal.png":::

1. Seleccione **Inspeccionar** para ver más información sobre la predicción.

    :::image type="content" source="media/luis-quickstart-new-app/test.png" alt-text="Captura de pantalla del panel de prueba, con información de inspección" lightbox="media/luis-quickstart-new-app/test.png":::

1. Cierre el panel de prueba.

## <a name="customize-your-application"></a>Personalización de la aplicación

Aparte de los dominios pregenerados, el servicio LUIS le permite crear aplicaciones personalizadas propias o personalizar aplicaciones precompiladas.

### <a name="create-intents"></a>Creación de intenciones

Para agregar intenciones a su aplicación:

1. Seleccione **Intents** (Intenciones) en el menú de navegación izquierdo.
2. Seleccione **Crear**
3. Escriba el nombre de la intención, `HomeAutomation.AddDeviceAlias`, y seleccione Listo.

### <a name="create-entities"></a>Creación de entidades

Para agregar entidades a su aplicación:

1. Seleccione **Entidades** en el menú de navegación izquierdo.
2. Seleccione **Crear**
3. Especifique el nombre de la intención, `HomeAutomation.DeviceAlias`, seleccione un **tipo** con aprendizaje automático y, a continuación, **Create** (Crear).

### <a name="add-example-utterances"></a>Incorporación de expresiones de ejemplo

Las expresiones de ejemplo son texto que un usuario escribe en un bot de chat o en otra aplicación cliente. Asignan la intención del texto del usuario a una intención de LUIS.

En la página **Intenciones** de `HomeAutomation.AddDeviceAlias`, agregue las siguientes expresiones del ejemplo en **Expresión de ejemplo**,

|#|Expresiones de ejemplo|
|--|--|
|1|`Add alias to my fan to be wind machine`|
|2|`Alias lights to illumination`|
|3|`nickname living room speakers to our speakers a new fan`|
|4|`rename living room tv to main tv`|

[!INCLUDE [best-practice-utterances](./includes/best-practice-utterances.md)]


### <a name="label-example-utterances"></a>Expresiones de ejemplo de etiqueta

Es necesario que etiquete las expresiones porque ha agregado una entidad con aprendizaje automático (ML). La aplicación usa el etiquetado para aprender a extraer las entidades ML que ha creado.

[!INCLUDE [how-to-label](./includes/how-to-label.md)]

## <a name="create-prediction-resource"></a>Creación de un recurso de predicción
En este punto, ha terminado de crear la aplicación. Es necesario que cree un recurso de predicción para publicar la aplicación y recibir predicciones en un bot de chat u otra aplicación cliente mediante el punto de conexión de predicción.

Para crear un recurso de predicción desde el portal del servicio LUIS

[!INCLUDE [add-pred-resource-portal](./includes/add-prediction-resource-portal.md)]


## <a name="publish-the-app-to-get-the-endpoint-url"></a>Publicación de la aplicación para obtener la dirección URL del punto de conexión

[!INCLUDE [LUIS How to Publish steps](./includes/howto-publish.md)]



## <a name="query-the-v3-api-prediction-endpoint"></a>Consulta del punto de conexión de predicción de la API V3

[!INCLUDE [LUIS How to get endpoint first step](./includes/v3-prediction-endpoint.md)]

2. Asegúrese de que los valores siguientes se incluyan en la URL para la cadena de consulta, en la barra de direcciones del explorador. Si no están en la cadena de consulta, agréguelas:

    * `verbose=true`
    * `show-all-intents=true`

3. En la barra de direcciones del explorador, vaya al final de la URL y especifique *turn off the living room light* (apagar la luz de la sala de estar) para la cadena de consulta. A continuación, presione Entrar.

    ```json
    {
        "query": "turn off the living room light",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOff",
            "intents": {
                "HomeAutomation.TurnOff": {
                    "score": 0.969448864
                },
                "HomeAutomation.QueryState": {
                    "score": 0.0122336326
                },
                "HomeAutomation.TurnUp": {
                    "score": 0.006547436
                },
                "HomeAutomation.TurnDown": {
                    "score": 0.0050634006
                },
                "HomeAutomation.SetDevice": {
                    "score": 0.004951761
                },
                "HomeAutomation.TurnOn": {
                    "score": 0.00312553928
                },
                "None": {
                    "score": 0.000552945654
                }
            },
            "entities": {
                "HomeAutomation.Location": [
                    "living room"
                ],
                "HomeAutomation.DeviceName": [
                    [
                        "living room light"
                    ]
                ],
                "HomeAutomation.DeviceType": [
                    [
                        "light"
                    ]
                ],
                "$instance": {
                    "HomeAutomation.Location": [
                        {
                            "type": "HomeAutomation.Location",
                            "text": "living room",
                            "startIndex": 13,
                            "length": 11,
                            "score": 0.902181149,
                            "modelTypeId": 1,
                            "modelType": "Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceName": [
                        {
                            "type": "HomeAutomation.DeviceName",
                            "text": "living room light",
                            "startIndex": 13,
                            "length": 17,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceType": [
                        {
                            "type": "HomeAutomation.DeviceType",
                            "text": "light",
                            "startIndex": 25,
                            "length": 5,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

Más información acerca del [punto de conexión de predicción de V3](luis-migration-api-v3.md).


## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Diseño de aplicaciones iterativo](./luis-concept-app-iteration.md)
* [procedimientos recomendados](./luis-concept-best-practices.md)
