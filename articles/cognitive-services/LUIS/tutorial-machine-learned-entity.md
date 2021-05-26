---
title: 'Tutorial: Extracción de datos estructurados con una entidad de aprendizaje automático: LUIS'
description: Extraiga los datos estructurados de una expresión mediante la entidad de aprendizaje automático. Para aumentar la precisión de la extracción, agregue subentidades con características.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 04/28/2020
ms.openlocfilehash: b3a41ec4eb10ccf1f1b9e80315f554f72a5c8cfe
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110095918"
---
# <a name="tutorial-extract-structured-data-from-user-utterance-with-machine-learning-entities-in-language-understanding-luis"></a>Tutorial: Extracción de datos estructurados de una expresión de usuario con entidades de aprendizaje automático en Language Understanding (LUIS)

En este tutorial, se extraen datos estructurados de una expresión mediante la entidad de aprendizaje automático.

La entidad con aprendizaje automático admite el [concepto de descomposición del modelo](luis-concept-model.md#v3-authoring-model-decomposition), ya que proporciona entidades de subentidad con [características](luis-concept-feature.md).

**En este tutorial, aprenderá a:**

> [!div class="checklist"]
> * Importar la aplicación de ejemplo
> * Agregar la entidad de aprendizaje automático
> * Agregar una subentidad y una característica
> * Entrenar, probar y publicar una aplicación
> * Obtener la predicción de la entidad a partir del punto de conexión

<!---
[!INCLUDE [LUIS Free account](includes/quickstart-tutorial-use-free-starter-key.md)]
-->

## <a name="why-use-a-machine-learning-entity"></a>¿Por qué usar una entidad de aprendizaje automático?

En este tutorial se agrega una entidad de aprendizaje automático para extraer datos de una expresión de usuario.

La entidad define los datos que se van a extraer de la expresión. Esto incluye asignar a los datos un nombre, un tipo (si es posible), cualquier resolución de los datos si existe ambigüedad y el texto exacto que compone los datos.

Para definir los datos, debe:
* Crear la entidad
* Etiquetar el texto, dentro de las expresiones de ejemplo, que representa la entidad. Estos ejemplos etiquetados enseñan a LUIS qué es la entidad y dónde se puede encontrar en una expresión.

## <a name="entity-decomposability-is-important"></a>La descomponibilidad de la entidad es importante

La descomponibilidad de la entidad es importante tanto para la predicción de la intención como para la extracción de los datos con la entidad.

Comience con una entidad de aprendizaje automático, que es la entidad de inicio y de nivel superior para la extracción de datos. Después, descomponga la entidad en subentidades.

Aunque es posible que no sepa qué detalle quiere que tenga la entidad al comenzar la aplicación, un procedimiento recomendado es empezar con una entidad de aprendizaje automático y, luego, descomponer en subentidades a medida que la aplicación crece.

En este tutorial, creará una entidad de aprendizaje automático para representar una aplicación de pedido de pizzas. La entidad extraerá texto relacionado con el pedido, como el tamaño y la cantidad.

Una expresión de `Please deliver one large cheese pizza to me` debe extraer `one large cheese pizza` como el pedido y también `1` para la cantidad y `large` para el tamaño.

## <a name="download-json-file-for-app"></a>Descarga de un archivo JSON para la aplicación

Descargue y guarde el [archivo JSON de la aplicación](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/tutorials/machine-learned-entity/pizza-intents-only.json).

## <a name="import-json-file-for-app"></a>Importación de un archivo JSON para la aplicación

[!INCLUDE [Import app steps](includes/import-app-steps.md)]

## <a name="create-machine-learned-entity"></a>Crear una entidad con aprendizaje automático

Para extraer detalles sobre un pedido de pizza, cree un nivel superior, una entidad `Order` de aprendizaje automático.

1. En la página **Intenciones**, seleccione la intención **OrderPizza**.

1. En la lista de expresiones de ejemplo, seleccione la expresión siguiente.

    |Expresión de pedido de ejemplo|
    |--|
    |`pickup a cheddar cheese pizza large with extra anchovies`|

    Haga clic y arrastre el cursor sobre el texto de la primera intención del ejemplo. En el menú que aparece, escriba el nombre de la entidad como `Order`. A continuación, seleccione `Order Create new entity` en la lista.

    ![Etiquetar el comienzo y el final del texto para el pedido completo](media/tutorial-machine-learned-entity/mark-complete-order.png)

    > [!NOTE]
    > Una entidad no siempre será toda la expresión. En este caso concreto, `pickup` indica cómo se va a recibir el pedido. Desde una perspectiva conceptual, `pickup` debe formar parte de la entidad etiquetada para el pedido.

1. En el cuadro **Choose an entity type** (Elegir un tipo de entidad), seleccione **Add Structure** (Agregar estructura) y, a continuación, seleccione **Siguiente**. La estructura es necesaria para agregar subentidades, como el tamaño y la cantidad.

    ![La captura de pantalla muestra la ventana de elección de un tipo de entidad, con la opción Agregar estructura seleccionada.](media/tutorial-machine-learned-entity/add-structure-to-entity.png)

1. En el cuadro **Add subentities (optional)** (Agregar subentidades [opcional]), seleccione **+** en la fila `Order`, después agregue `Size` y `Quantity` como subentidades y, a continuación, seleccione **Crear**.

    > [!div class="mx-imgBorder"]
    > ![La captura de pantalla muestra la ventana para agregar subentidades (opcional), con subentidades resaltadas.](media/tutorial-machine-learned-entity/add-subentities-when-creating-machine-learned-entity.png)

## <a name="edit-subentities-to-improve-extraction"></a>Edición de subentidades para mejorar la extracción

En los pasos anteriores se ha creado la entidad y la subentidad. Para mejorar la extracción, agregue ahora características a las subentidades.

### <a name="improve-size-extraction-with-phrase-list"></a>Mejora de la extracción de tamaño con una lista de frases

1. Seleccione **Entidades** en el menú de la izquierda y, a continuación, seleccione la entidad **Order** (Pedido).

1. En la pestaña **Schema and features** (Esquema y características), seleccione la subentidad **Size** (Tamaño) y, a continuación, seleccione **+ Add feature** (Agregar característica).

1. Seleccione **Crear nueva lista de frases** en el menú desplegable.

1. En el cuadro **Crear lista de frases**, escriba el nombre `SizePhraselist` y, a continuación, los valores `small`, `medium` y `large`. Cuando se rellene el cuadro **Sugerencias**, seleccione `extra large` y `xl`. Seleccione **Crear** para crear la nueva lista de frases.

    Esta característica lista de frases ayuda a la subentidad `Size` a buscar palabras relacionadas con el tamaño al proporcionarle palabras de ejemplo. Esta lista de frases no necesita incluir cada palabra de tamaño, pero debe incluir palabras que se espera que indiquen tamaño.

### <a name="add-sizelist-entity"></a>Incorporación de la entidad SizeList

Agregar una lista de tamaños conocidos que la aplicación cliente reconoce, ayudará también a la extracción.

1. Seleccione **Entidades** en el menú de la izquierda y, a continuación, seleccione **+ Crear**.

1. Establezca el nombre de la entidad como `SizeListentity` y el tipo como **Lista** para que sea fácil de identificar cuando se compare con la lista `SizePhraselist` creada en la sección anterior.

1. Agregue los tamaños que espera la aplicación cliente: `Small`, `Medium`, `Large` y `XLarge`, a continuación, agregue sinónimos para cada uno. Los sinónimos deben ser los términos que un usuario escribe en el bot de chat. La entidad se extrae con una entidad de lista cuando coincide exactamente con el valor normalizado o con los sinónimos.

    |Valor normalizado|Sinónimos|
    |--|--|
    |Pequeña|P, pq, mini, mínima|
    |Media|M, md, normal, mediana|
    |grande|G, gr, maxi|
    |XGrande|XG, familiar, extra|


    > [!div class="mx-imgBorder"]
    > ![La captura de pantalla muestra la ventana SizeList y elementos de lista, con el valor XGrande seleccionado.](media/tutorial-machine-learned-entity/size-list-entity-with-synonyms.png)

### <a name="add-feature-of-sizelist-entity"></a>Incorporación de la característica de la entidad SizeList

1. Seleccione **Entidades** en el menú de la izquierda para volver a la lista de entidades.

1. Seleccione **Order** (Pedido) en la lista de entidades.

1. En la pestaña **Schema and features** (Esquema y características), seleccione la entidad **Size** (Tamaño) y, a continuación, seleccione **+ Add feature** (Agregar característica).

1. Seleccione **@ SizeListentity** en la lista desplegable.

<a name="add-prebuilt-number-entity-to-app"></a>

### <a name="add-prebuilt-number-entity"></a>Incorporación de una entidad numérica precompilada

Agregar una entidad de número precompilada también ayudará a la extracción.

1. Seleccione **Entidades** en el menú de la izquierda y, a continuación, seleccione **+ Agregar entidad predefinida**.

1. Seleccione **Número** en la lista y, a continuación, seleccione **Listo**.

1. Seleccione **Entidades** en el menú de la izquierda para volver a la lista de entidades.

### <a name="add-feature-of-prebuilt-number-entity"></a>Incorporación de la característica de entidad de número precompilada

1. Seleccione **Order** (Pedido) en la lista de entidades.

1. En la pestaña **Schema and features** (Esquema y características), seleccione la entidad **Quantity** (Cantidad) y, a continuación, seleccione **+ Add feature** (Agregar característica).

1. Seleccione **@ number** en la lista desplegable.

<a name="create-subcomponent-entity-with-constraint-to-help-extract-data"></a>

## <a name="configure-required-features"></a>Configuración de las características necesarias

En la página de detalles de la entidad para la entidad **Order** (Pedido), seleccione el asterisco, `*`, para las características **@ SizeList** y **@ number**. El asterisco aparece en la misma etiqueta que el nombre de la característica.

> [!div class="mx-imgBorder"]
> ![La captura de pantalla muestra la característica @SizeList, con el asterisco y la advertencia de obligatorio.](media/tutorial-machine-learned-entity/set-required-feature-on-subentity.png)

<a name="label-text-as-entities-in-example-utterances"></a>
<a name="label-example-utterance-to-teach-luis-about-the-entity"></a>

## <a name="label-example-utterances"></a>Expresiones de ejemplo de etiqueta

Se crea la entidad de aprendizaje automático, y las subentidades tienen características. Para completar la mejora de la extracción, las expresiones de ejemplo tienen que etiquetarse con las subentidades.

1. Seleccione **Intenciones** desde el panel de navegación de la izquierda y, a continuación, seleccione la intención **OrderPizza**.

1. Para abrir la **Paleta de entidades**, seleccione el símbolo **@** en la barra de herramientas contextual.

1. Seleccione cada fila de entidad en la paleta y, a continuación, use el cursor de la paleta para seleccionar la entidad en cada expresión de ejemplo. Cuando haya terminado, la lista de entidades debe tener un aspecto similar al de la siguiente imagen.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla parcial de la configuración de la característica necesaria](media/tutorial-machine-learned-entity/labeled-example-utterances-for-machine-learned-entity.png)

## <a name="train-the-app"></a>Entrenamiento de la aplicación

Para entrenar la aplicación, seleccione **Entrenar**. El entrenamiento aplica los cambios, como las nuevas entidades y las expresiones etiquetadas, al modelo activo.

## <a name="add-a-new-example-utterance"></a>Incorporación de una nueva expresión de ejemplo

1. Después del entrenamiento, agregue una nueva expresión de ejemplo a la intención `OrderPizza` para ver el grado en que LUIS entiende la entidad de aprendizaje automático.

    |Expresión de pedido de ejemplo|
    |--|
    |`I need a large pepperoni pizza`|

    La entidad superior general, `Order`, está etiquetada, y la subentidad `Size` también está etiquetada con líneas punteadas.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla parcial de la nueva expresión de ejemplo prevista con la entidad](media/tutorial-machine-learned-entity/new-example-utterance-predicted-with-entity.png)

    La línea de puntos indica la predicción basada en la aplicación entrenada actual.

1. Para cambiar la predicción a una entidad etiquetada, seleccione la marca de verificación en la misma fila.

    > [!div class="mx-imgBorder"]
    > ![La captura de pantalla muestra una expresión de ejemplo, con la marca de verificación resaltada.](media/tutorial-machine-learned-entity/confirm-entity-prediction-for-new-example-utterance-added.png)

    En este momento, la entidad de aprendizaje automático funciona porque puede encontrar la entidad dentro de una nueva expresión de ejemplo. Al agregar expresiones de ejemplo, si la entidad no se predice correctamente, etiquete la entidad y las subentidades. Si la entidad se predice correctamente, asegúrese de confirmar las predicciones.


## <a name="train-the-app-to-apply-the-entity-changes-to-the-app"></a>Entrenamiento de la aplicación para aplicar los cambios de la entidad a la aplicación

Seleccione **Entrenar** para entrenar la aplicación con esta nueva expresión.

En este momento, el pedido tiene algunos detalles que se pueden extraer (tamaño, cantidad y texto total del pedido). Hay más refinamientos de la entidad `Order`, como los ingredientes de la pizza, el tipo de corteza y los pedidos secundarios. Cada uno debe crearse como subentidad de la entidad `Order`.

## <a name="test-the-app-to-validate-the-changes"></a>Prueba de la aplicación para validar los cambios

Pruebe la aplicación mediante el panel **Prueba** interactivo. Este proceso permite especificar una nueva expresión y, a continuación, ver los resultados de la predicción para ver el funcionamiento de la aplicación entrenada en activo. La predicción de intención debe ser bastante segura (por encima del 60 %), y la extracción de entidades debe recoger al menos la entidad `Order`. Puede que falten los detalles de la entidad Order porque estas pocas expresiones no bastan para administrar cada caso.

1. Seleccione **Test** (Prueba) en la barra de navegación superior.
1. Escriba la expresión `2 small cheese pizzas for pickup` y presione Entrar. El modelo activo predijo la intención correcta con más del 60 % de confianza.


1. Seleccione **Inspeccionar** para ver las predicciones de la entidad.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla parcial de vista de las predicciones de la entidad en el panel de prueba interactivo.](media/tutorial-machine-learned-entity/interactive-test-panel-with-first-utterance-and-entity-predictions.png)

<a name="publish-with-sentiment-analysis"></a>

## <a name="publish-the-app-to-access-it-from-the-http-endpoint"></a>Publicación de la aplicación para tener acceso a ella desde el punto de conexión HTTP

Para recibir una predicción de LUIS en un bot de chat u otra aplicación cliente, tiene que publicar la aplicación en el punto de conexión.

1. Seleccione **Publish** (Publicar) en el panel de navegación superior derecho.

    ![Captura de pantalla del botón de publicación en el punto de conexión de LUIS del menú superior derecho](./media/howto-publish/publish-button.png)

1. Seleccione el espacio de **Producción** y luego **Cambiar la configuración**, seleccione **Análisis de sentimiento** y, a continuación, **Listo**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la publicación de LUIS en el punto de conexión](./media/tutorial-machine-learned-entity/publish-with-sentiment-analysis.png)

1. Seleccione el vínculo **Access your endpoint URLs** (Acceder a las direcciones URL del punto de conexión) de la notificación para ir a la página **Azure Resources** (Recursos de Azure). Las direcciones URL del punto de conexión se muestran como **Example Query** (Ejemplo de consulta).

## <a name="get-intent-and-entity-prediction-from-http-endpoint"></a>Obtención de la intención y la predicción de la entidad desde el punto de conexión HTTP

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Vaya al final de la dirección URL en la barra de direcciones sustituya _YOUR_QUERY_HERE_ por la misma consulta que especificó en el panel de prueba interactivo.

    `2 small cheese pizzas for pickup`

    El último parámetro de la cadena de consulta es `query`, la expresión **query**.

    ```json
    {
        "query": "2 small cheese pizzas for pickup",
        "prediction": {
            "topIntent": "OrderPizza",
            "intents": {
                "OrderPizza": {
                    "score": 0.7812769
                },
                "None": {
                    "score": 0.0314020254
                },
                "Confirm": {
                    "score": 0.009299271
                },
                "Greeting": {
                    "score": 0.007551549
                }
            },
            "entities": {
                "Order": [
                    {
                        "Size": [
                            "small"
                        ],
                        "Quantity": [
                            2
                        ]
                    }
                ]
            }
        },
        "sentimentAnalysis":{
            "label":"neutral",
            "score":0.98
       }
    }
    ```


[!INCLUDE [LUIS How to clean up resources](includes/quickstart-tutorial-cleanup-resources.md)]

## <a name="related-information"></a>Información relacionada

* [Tutorial: Intenciones](./tutorial-intents-only.md)
* [Concepto: Entidades](luis-concept-entity-types.md) información conceptual
* [Concepto: Características](luis-concept-feature.md) información conceptual
* [Cómo se realiza el entrenamiento](luis-how-to-train.md)
* [Publicación](luis-how-to-publish-app.md)
* [Prueba de la aplicación el portal de LUIS](luis-interactive-test.md)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, la aplicación usa una entidad de aprendizaje automático para encontrar la intención de una expresión de usuario y extraer los detalles de esa expresión. El uso de la entidad de aprendizaje automático permite descomponer los detalles de la entidad.

> [!div class="nextstepaction"]
> [Incorporación de una entidad keyphrase precompilada](./luis-reference-prebuilt-keyphrase.md)
