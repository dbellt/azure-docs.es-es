---
title: 'Revisión de expresiones del usuario: LUIS'
titleSuffix: Azure Cognitive Services
description: Revise expresiones capturadas por el aprendizaje activo para seleccionar la intención y marcar las entidades para las expresiones del mundo real; aceptar cambios, entrenar y publicar.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/05/2021
ms.openlocfilehash: 41c45b2887736847c2413c2193e57d35f49594e7
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109518107"
---
# <a name="how-to-improve-the-luis-app-by-reviewing-endpoint-utterances"></a>Cómo mejorar la aplicación LUIS revisando las expresiones de punto de conexión

El proceso de revisión de las expresiones de punto de conexión para obtener predicciones correctas se denomina [Aprendizaje activo](luis-concept-review-endpoint-utterances.md). El aprendizaje activo captura las consultas de punto de conexión y selecciona las expresiones de punto de conexión del usuario de las que no está seguro. Revise estas expresiones para seleccionar la intención y marque las entidades para estas expresiones del mundo real. Acepte estos cambios en sus expresiones de ejemplo, entrene y publique. LUIS identificará después las expresiones con mayor precisión.

## <a name="log-user-queries-to-enable-active-learning"></a>Registre consultas de usuario para habilitar el aprendizaje activo.

Para habilitar el aprendizaje activo, debe registrar las consultas de usuario. Esto se consigue llamando a la [consulta de punto de conexión](luis-get-started-create-app.md#query-the-v3-api-prediction-endpoint) con el parámetro y el valor de la cadena de consulta `log=true`.

Use el portal de LUIS para construir la consulta de punto de conexión correcta.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su **Suscripción** y **Recurso de creación** para ver las aplicaciones asignadas a ese recurso de creación.
1. Abra la aplicación mediante la selección de su nombre en la página **Mis aplicaciones**.
1. Vaya a la sección **Administrar** y seleccione **Recursos de Azure**.
1. En el recurso de predicción asignado, seleccione la opción de **cambiar parámetros de consulta**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla que muestra el vínculo Cambiar parámetros de consulta.](./media/luis-tutorial-review-endpoint-utterances/azure-portal-change-query-url-settings.png)

1. Active la opción **Guardar registros** y, a continuación, guárdelos con **Listo**.

    > [!div class="mx-imgBorder"]
    > ![Use el portal de LUIS para guardar los registros, que es necesario para el aprendizaje activo.](./media/luis-tutorial-review-endpoint-utterances/luis-portal-manage-azure-resource-save-logs.png)

     Esta acción cambia la dirección URL, ya que agrega el parámetro QueryString `log=true`. Copie y utilice la dirección URL de consulta de ejemplo modificada al realizar consultas de predicción en el punto de conexión del runtime.

## <a name="correct-predictions-to-align-utterances"></a>Corrección de las predicciones para alinear expresiones

Cada expresión tiene una intención sugerida que se muestra en la columna **Intención de predicción**, y las entidades sugeridas en los rectángulos de selección de puntos.

> [!div class="mx-imgBorder"]
> [![Revise las expresiones de punto de conexión de las que LUIS no está seguro](./media/label-suggested-utterances/review-endpoint-utterances.png)](./media/label-suggested-utterances/review-endpoint-utterances.png#lightbox)

Si está de acuerdo con la intención y las entidades de predicción, seleccione la marca de verificación situada junto a la expresión. Si la marca de verificación está deshabilitada, significa que no hay nada que confirmar.  
Si no está de acuerdo con la intención sugerida, seleccione la intención correcta en la lista desplegable Intención de predicción. Si no está de acuerdo con las entidades sugeridas, empiece a etiquetarlas.
Cuando haya terminado, seleccione la marca de verificación situada junto a la expresión para confirmar lo que ha etiquetado. Seleccione la opción para **guardar expresión** a fin de moverla de la lista de revisión y agregarla a su intención correspondiente.

> [!TIP]
> Es importante ir a la página de detalles de intención para revisar y corregir las predicciones de entidad de todas las expresiones de ejemplo de la lista **Revisar expresiones de extremo**.

## <a name="delete-utterance"></a>Eliminar la expresión

Cada expresión se puede eliminar de la lista de revisión. Una vez eliminada, no volverá a aparecer en la lista. Esto sucede incluso si el usuario escribe la misma expresión desde el punto de conexión.

Si no está seguro de si la expresión se debe eliminar, muévala a la intención None, o bien cree una intención como `miscellaneous` y mueva la expresión a esa intención.

## <a name="disable-active-learning"></a>Deshabilitación del aprendizaje activo

Para deshabilitar el aprendizaje activo, no registre las consultas de usuario. Esto se logra cambiando los parámetros de consulta como se muestra anteriormente o estableciendo el parámetro `log=false` en la consulta de punto de conexión, o bien puede omitir el parámetro `log` porque el valor predeterminado es `false`.


## <a name="next-steps"></a>Pasos siguientes

Para probar cómo mejora el rendimiento después de etiquetar las expresiones sugeridas, puede acceder a la consola de prueba si hace clic en **Test** (Prueba) en el panel superior. Para obtener instrucciones sobre cómo probar la aplicación mediante la consola de prueba, vea [Train and test your app](luis-interactive-test.md) (Entrenar y probar la aplicación).
