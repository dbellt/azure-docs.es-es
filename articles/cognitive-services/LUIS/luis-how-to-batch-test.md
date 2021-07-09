---
title: 'Realización de una prueba por lotes: LUIS'
titleSuffix: Azure Cognitive Services
description: Use los conjuntos de pruebas por lotes de Language Understanding (LUIS) para buscar expresiones con intenciones y entidades incorrectas.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/18/2021
ms.openlocfilehash: b6d007d3020d3fa2b001178bd23d721ea6a6aa6b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110067841"
---
# <a name="batch-testing-with-a-set-of-example-utterances"></a>Pruebas por lotes con un conjunto de expresiones de ejemplo

Las pruebas por lotes validan la versión entrenada activa para medir la precisión de la predicción. Una prueba por lotes ayuda a ver la precisión de cada intención y entidad en la versión activa. Revise los resultados de la prueba por lotes para adoptar las medidas apropiadas para mejorar la precisión, como agregar más expresiones de ejemplo a una intención si la aplicación presenta errores frecuentes de identificación de la intención correcta o etiquetar entidades dentro de la expresión.

## <a name="group-data-for-batch-test"></a>Agrupación de datos para pruebas por lotes

Es importante que las expresiones utilizadas para las pruebas por lotes sean nuevas en LUIS. Si tiene un conjunto de datos de expresiones, divida las expresiones en tres conjuntos: expresiones de ejemplo agregadas a una intención, expresiones recibidas desde el punto de conexión publicado y expresiones utilizadas para las pruebas por lotes de LUIS después de su entrenamiento.

El archivo JSON por lotes que use debe incluir expresiones con entidades de aprendizaje automático de nivel superior etiquetadas de modo que se incluyan las posiciones de inicio y fin. Las expresiones no deben ser parte de los ejemplos que ya hay en la aplicación. Deben ser expresiones para las que desee realizar una predicción positiva de la intención y las entidades.

Puede separar las pruebas por intención o entidad o tener todas las pruebas (hasta 1000 expresiones) en el mismo archivo. 

### <a name="common-errors-importing-a-batch"></a>Errores comunes al importar un lote

Si se producen errores al cargar el archivo por lotes en LUIS, busque los siguientes problemas comunes:

* Más de 1000 expresiones en un archivo por lotes
* Un objeto JSON de expresiones que no tiene una propiedad de entidades. La propiedad puede ser una matriz vacía.
* Palabras etiquetadas en varias entidades
* Etiquetas de entidad que empiecen o acaben con un espacio.

## <a name="fixing-batch-errors"></a>Corrección de errores de lotes

Si hay errores en las pruebas por lotes, puede agregar más expresiones a una intención o etiquetar más expresiones con al entidad para ayudar a LUIS a realizar la discriminación entre intenciones. Si ha agregado expresiones y las ha etiquetado y aún obtiene errores de predicción en las pruebas por lotes, considere la adición de una característica [lista de frases](luis-concept-feature.md) con un vocabulario específico de dominios para ayudar a LUIS a aprender más rápido.


<a name="batch-testing"></a>

## <a name="batch-testing-using-the-luis-portal"></a>Prueba por lotes mediante el portal de LUIS 

### <a name="import-and-train-an-example-app"></a>Importación y entrenamiento de una aplicación de ejemplo

Importe una aplicación que tome un pedido de pizza como `1 pepperoni pizza on thin crust`.

1.  Descargue y guarde el [archivo JSON de la aplicación](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/pizza-with-machine-learned-entity.json?raw=true).

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su **Suscripción** y **Recurso de creación** para ver las aplicaciones asignadas a ese recurso de creación.
1. Seleccione la flecha situada junto a **Nueva aplicación** y haga clic en **Import as JSON** (Importar como JSON) para importar el archivo JSON a una nueva aplicación. Asigne el nombre `Pizza app` a la aplicación.


1. Seleccione **Entrenar** en la esquina superior derecha de la navegación para entrenar la aplicación.


[!INCLUDE [Entity roles in batch testing - currently not supported](../../../includes/cognitive-services-luis-roles-not-supported-in-batch-testing.md)]

### <a name="batch-test-file"></a>Archivo de prueba por lotes

El ejemplo JSON incluye una expresión con una entidad etiquetada para ilustrar el aspecto de un archivo de prueba. En sus propias pruebas, debe tener muchas expresiones con una intención correcta y una entidad de aprendizaje automático etiquetada.

1. Cree `pizza-with-machine-learned-entity-test.json` en un editor de texto o [descárguelo](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/batch-tests/pizza-with-machine-learned-entity-test.json?raw=true).

2. En el archivo por lotes con formato JSON, agregue una expresión con la **intención** que quiere predecir en la prueba.

   [!code-json[Add the intents to the batch test file](~/samples-cognitive-services-data-files/luis/batch-tests/pizza-with-machine-learned-entity-test.json "Add the intent to the batch test file")]

## <a name="run-the-batch"></a>Ejecute el lote

1. Haga clic en **Probar** en la barra de navegación superior.

2. Seleccione el **panel Prueba por lotes** en el panel de la derecha.

    ![Vínculo de Pruebas por lotes](./media/luis-how-to-batch-test/batch-testing-link.png)

3. Seleccione **Import** (Importar). En el cuadro de diálogo, seleccione **Elegir archivo** y localice un archivo JSON con el formato JSON correcto que *no contenga más de 1000* expresiones para probar.

    Los errores de importación se notifican en una barra de notificación de color rojo en la parte superior del explorador. Cuando una importación tiene errores, no se crea ningún conjunto de datos. Para obtener más información, vea [Errores comunes](#common-errors-importing-a-batch).

4. Elija la ubicación de archivo del archivo `pizza-with-machine-learned-entity-test.json`.

5. Asigne un nombre al conjunto de datos `pizza test` y seleccione **Done** (Listo).

6. Haga clic en el botón **Ejecutar**. 

7. Una vez completada la prueba por lotes, puede ver las columnas siguientes:

    | Columna | Descripción |
    | -------- | ------------- |
    | State | Estado de la prueba. El elemento **Ver resultados** solo es visible una vez completada la prueba. |
    | Nombre | Nombre que ha dado a la prueba. |
    | Size | Número de pruebas de este archivo de pruebas por lotes. |
    | Última ejecución | Fecha de la última ejecución de este archivo de pruebas por lotes. |
    | Último resultado | Número de predicciones correctas en la prueba. |

8. Para ver los resultados detallados de la prueba, seleccione **Ver resultados**. 

    > [!TIP]
    > * Al seleccionar **Descargar** se descargará el mismo archivo que ha cargado.
    > * Si ve errores en la prueba por lotes, es que al menos una intención de expresión no coincidió con la predicción.

<a name="access-batch-test-result-details-in-a-visualized-view"></a>

### <a name="review-batch-results-for-intents"></a>Revisión de los resultados de lote para determinar intenciones

Para revisar los resultados de la prueba por lotes, seleccione **Ver resultados**. Los resultado de la prueba muestran gráficamente cómo se han predicho las expresiones de prueba con respecto a la versión activa.

El gráfico por lotes muestra cuatro cuadrantes de resultados. Hay un filtro a la derecha del gráfico. El filtro contiene intenciones y entidades. Cuando se selecciona una [sección del gráfico](#review-batch-results-for-intents) o un punto dentro del gráfico, las expresiones asociadas se muestra debajo del gráfico.

Si mantiene el mouse sobre el gráfico y gira la rueda del mouse, puede agrandar o disminuir la visualización del gráfico. Esto resulta útil cuando hay muchos puntos en el gráfico estrechamente agrupados.

El gráfico está en cuatro cuadrantes, donde dos de las secciones aparecen en rojo.

1. Seleccione la intención **ModifyOrder** en la lista de filtros. La expresión se predice como un **Verdadero positivo**, lo que significa que la expresión ha coincidido correctamente con su predicción positiva indicada en el archivo por lotes.

    > [!div class="mx-imgBorder"]
    > ![La expresión ha coincidido correctamente con su predicción positiva](./media/luis-tutorial-batch-testing/intent-predicted-true-positive.png)

    Las marcas de verificación verdes de la lista de filtros también indican el éxito de la prueba para cada intención. Todas las demás intenciones se muestran con una puntuación positiva 1/1 porque la expresión se ha probado para cada intención, como una prueba negativa para cualquier intención no incluida en la prueba por lotes.

1. Seleccione la intención **Confirmation**. Esta intención no aparece en la prueba por lotes, por lo que se trata de una prueba negativa de la expresión que aparece en la prueba por lotes.

    > [!div class="mx-imgBorder"]
    > ![Predicción negativa correcta de la expresión para una intención no indicada en el archivo por lotes](./media/luis-tutorial-batch-testing/true-negative-intent.png)

    La prueba negativa se realizó correctamente, como se indica por el texto verde del filtro y la cuadrícula.

### <a name="review-batch-test-results-for-entities"></a>Revisión de los resultados de la prueba por lotes para determinar entidades

La entidad ModifyOrder, como entidad de máquina con subentidades, se muestra si la entidad de nivel superior coincide, además de cómo se predicen las subentidades.

1. Seleccione la entidad **ModifyOrder** en la lista de filtros y, a continuación, seleccione el círculo de la cuadrícula.

1. La predicción de entidad se muestra debajo del gráfico. Se muestran líneas continuas para las predicciones que coinciden con la expectativa y líneas de puntos para las predicciones que no coinciden con la expectativa.

    > [!div class="mx-imgBorder"]
    > ![Entidad principal predicha correctamente en el archivo por lotes](./media/luis-tutorial-batch-testing/labeled-entity-prediction.png)

<a name="filter-chart-results-by-intent-or-entity"></a>

#### <a name="filter-chart-results"></a>Filtrado de los resultados del gráfico

Para filtrar el gráfico por una intención o entidad específica, seleccione la intención o entidad en el panel de filtrado del lado derecho. Los puntos de datos y su distribución se actualizan en el gráfico según su selección.

![Resultado de la prueba por lotes visualizado](./media/luis-how-to-batch-test/filter-by-entity.png)

### <a name="chart-result-examples"></a>Ejemplos de resultados de gráfico

En el gráfico del portal de LUIS, se pueden llevar a cabo las siguientes acciones:
 
#### <a name="view-single-point-utterance-data"></a>Ver los datos de expresión de un único punto

En el gráfico, mantenga el puntero sobre un punto de datos para ver la puntuación de exactitud de la predicción. Seleccione un punto de datos para recuperar su expresión correspondiente en la lista de expresiones en la parte inferior de la página.

![Expresión seleccionada](./media/luis-how-to-batch-test/selected-utterance.png)


<a name="relabel-utterances-and-retrain"></a>
<a name="false-test-results"></a>

#### <a name="view-section-data"></a>Ver los datos de la sección

En el gráfico de cuatro secciones, seleccione el nombre de sección, como **Falso positivo** en la parte superior derecha del gráfico. Debajo del gráfico se muestran todas las expresiones de esa sección en una lista.

![Expresiones seleccionadas por la sección](./media/luis-how-to-batch-test/selected-utterances-by-section.png)

En la imagen anterior, la expresión `switch on` está etiquetada con la intención TurnAllOn, pero recibió la predicción de intención None. Se trata de un valor que indica que la intención TurnAllOn necesita más expresiones de ejemplo para realizar la predicción esperada.

Las dos secciones del gráfico en rojo indican las expresiones que no coincidieron con la predicción esperada. Estas indican expresiones para las que LUIS necesita más entrenamiento.

Las dos secciones del gráfico en color verde coincidieron con la predicción esperada.

## <a name="batch-testing-using-the-rest-api"></a>Prueba por lotes mediante la API de REST 

LUIS le permite realizar pruebas por lotes mediante el portal de LUIS y la API de REST. A continuación, se enumeran los puntos de conexión de la API de REST. Para obtener información sobre las pruebas por lotes mediante el portal de LUIS, vea [Tutorial: pruebas por lotes de conjuntos de datos](). Use las direcciones URL completas que se indican a continuación y reemplace los valores de marcador de posición por su propia clave de predicción de LUIS y punto de conexión. 

No olvide agregar la clave de LUIS a `Ocp-Apim-Subscription-Key` en el encabezado y establecer `Content-Type` en `application/json`.

### <a name="start-a-batch-test"></a>Inicio de una prueba por lotes

Inicie una prueba por lotes con un id. de versión de aplicación o una ranura de publicación. Envíe una solicitud **POST** a uno de los siguientes formatos de punto de conexión. Incluya el archivo por lotes en el cuerpo de la solicitud.

Ranura de publicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/slots/<YOUR-SLOT-NAME>/evaluations`

Id. de versión de aplicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/versions/<YOUR-APP-VERSION-ID>/evaluations`

Estos puntos de conexión devolverán un id. de operación que se usará para comprobar el estado y obtener los resultados. 


### <a name="get-the-status-of-an-ongoing-batch-test"></a>Obtención del estado de una prueba por lotes en curso

Use el id. de operación de la prueba por lotes que inició para obtener su estado de los siguientes formatos de punto de conexión: 

Ranura de publicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/slots/<YOUR-SLOT-ID>/evaluations/<YOUR-OPERATION-ID>/status`

Id. de versión de aplicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/versions/<YOUR-APP-VERSION-ID>/evaluations/<YOUR-OPERATION-ID>/status`

### <a name="get-the-results-from-a-batch-test"></a>Obtención de los resultados de una prueba por lotes

Use el id. de operación de la prueba por lotes que inició para obtener sus resultados de los siguientes formatos de punto de conexión: 

Ranura de publicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/slots/<YOUR-SLOT-ID>/evaluations/<YOUR-OPERATION-ID>/result`

Id. de versión de aplicación
* `<YOUR-PREDICTION-ENDPOINT>/luis/prediction/v3.0-preview/apps/<YOUR-APP-ID>/versions/<YOUR-APP-VERSION-ID>/evaluations/<YOUR-OPERATION-ID>/result`


### <a name="batch-file-of-utterances"></a>Archivo por lotes de expresiones

Envíe un archivo por lotes de expresiones, conocido como *conjunto de datos*, para las pruebas por lotes. El conjunto de datos es un archivo con formato JSON que contiene un máximo de 1000 expresiones etiquetadas. Puede probar hasta diez conjuntos de datos en una aplicación. Si necesita probar más, elimine un conjunto de datos y luego agregue uno nuevo. Todas las entidades personalizadas del modelo aparecen en el filtro de entidades de pruebas por lotes incluso si no existen las entidades correspondientes en los datos del archivo por lotes.

El archivo por lotes está compuesto de expresiones. Cada expresión tiene que tener una predicción de intenciones esperada junto con todas las [entidades de aprendizaje automático](luis-concept-entity-types.md#machine-learned-ml-entity) que espera detectar.

### <a name="batch-syntax-template-for-intents-with-entities"></a>Plantilla de sintaxis de lotes para intenciones con entidades

Use la plantilla siguiente para iniciar el archivo por lotes:

```JSON
{
    "LabeledTestSetUtterances": [
        {
            "text": "play a song",
            "intent": "play_music",
            "entities": [
                {
                    "entity": "song_parent",
                    "startPos": 0,
                    "endPos": 15,
                    "children": [
                        {
                            "entity": "pre_song",
                            "startPos": 0,
                            "endPos": 3
                        },
                        {
                            "entity": "song_info",
                            "startPos": 5,
                            "endPos": 15
                        }
                    ]
                }
            ]
        }
    ]
}

```

El archivo por lotes usa las propiedades **startPos** y **endPos** para tener en cuenta el inicio y el final de una entidad. Los valores se basan en cero y no pueden empezar ni terminar en un espacio. Esto es diferente de los registros de consultas que usan las propiedades startIndex y endIndex.

Si no quiere probar entidades, incluya la propiedad `entities` y establezca el valor como una matriz vacía, `[]`.

### <a name="rest-api-batch-test-results"></a>Resultados de la prueba por lotes de la API de REST

Hay varios objetos devueltos por la API:

* Información sobre las intenciones y los modelos de entidades, como la precisión, la coincidencia y la puntuación F.
* Información sobre los modelos de entidades, como la precisión, la recuperación y la puntuación F, para cada entidad 
  * Con la marca `verbose`, puede obtener más información sobre la entidad, como `entityTextFScore` y `entityTypeFScore`.
* Expresiones proporcionadas con los nombres de intención previstas y etiquetadas.
* Una lista de entidades con falso positivo y una lista de entidades con falso negativo.

## <a name="next-steps"></a>Pasos siguientes

Si las pruebas indican que la aplicación de LUIS no reconoce las entidades y las intenciones correctas, puede trabajar para mejorar el rendimiento de la aplicación de LUIS etiquetando más expresiones o agregando características.

* [Label suggested utterances with LUIS](luis-how-to-review-endpoint-utterances.md) (Etiquetado de expresiones sugeridas con LUIS)
* [Use features to improve your LUIS app's performance](luis-how-to-add-features.md) (Uso de características para mejorar el rendimiento de la aplicación de LUIS)
