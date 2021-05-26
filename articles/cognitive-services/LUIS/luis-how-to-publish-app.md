---
title: 'Publicar aplicación: LUIS'
titleSuffix: Azure Cognitive Services
description: Cuando termine de compilar y probar la aplicación de LUIS activa, haga que esté disponible para la aplicación cliente mediante su publicación en el punto de conexión.
services: cognitive-services
author: aahill
manager: nitinme
ms.author: aahi
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 01/12/2021
ms.openlocfilehash: 53549869098504396d1fa1039ed43b5ff02d2a9b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110083645"
---
# <a name="publish-your-active-trained-app-to-a-staging-or-production-endpoint"></a>Publicación de la aplicación activa y entrenada en un punto de conexión de almacenamiento provisional o de producción

Cuando termine de compilar, entrenar y probar la aplicación de LUIS activa, haga que esté disponible para la aplicación cliente mediante su publicación en el punto de conexión.

## <a name="publishing"></a>Publicación
1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su **Suscripción** y **Recurso de creación** para ver las aplicaciones asignadas a ese recurso de creación.
1. Abra la aplicación mediante la selección de su nombre en la página **Mis aplicaciones**.
1. Para publicar en el punto de conexión, seleccione **Publicar** en el panel superior derecho.

    ![Botón para publicar que se encuentra en la parte superior de la barra de navegación derecha](./media/luis-how-to-publish-app/publish-top-nav-bar.png)

1. Seleccione la configuración del punto de conexión de predicción publicado y, a continuación, seleccione **Publicar**.

    ![Seleccione la configuración de publicación y, a continuación, haga clic en el botón de publicar.](./media/luis-how-to-publish-app/publish-pop-up.png)

### <a name="publishing-slots"></a>Ranuras de publicación

Seleccione la ranura correcta cuando se muestre la ventana emergente:

* Ensayo
* Producción

Si usa las dos ranuras de publicación, puede tener dos versiones distintas de la aplicación en los puntos de conexión publicados, o la misma versión en dos puntos de conexión diferentes.

### <a name="publishing-regions"></a>Regiones de publicación

La aplicación se publica en todas las regiones asociadas a los recursos del punto de conexión de predicción de LUIS agregados en el portal de LUIS desde la página **Administrar** ->  **[Recursos de Azure](luis-how-to-azure-subscription.md#assign-luis-resources)** .

Por ejemplo, para una aplicación creada en [www.luis.ai](https://www.luis.ai), si crea un recurso de LUIS en dos regiones, **westus** y **eastus**, y agrega los recursos creados a la aplicación como recursos, la aplicación se publicará en ambas regiones. Para obtener más información sobre las regiones de LUIS, consulte [Regiones](luis-reference-regions.md).

> [!TIP]
> Hay tres regiones de creación. Debe crear en la región en la que desea publicar. Si tiene que publicar en todas las regiones, debe administrar el proceso de creación y el modelo entrenado resultante en las tres regiones de creación.


## <a name="configuring-publish-settings"></a>Configuración de los ajustes de publicación

Después de seleccionar la ranura, configure las opciones de publicación para realizar lo siguiente:

* análisis de opiniones
* Preparación para la voz

Después de la publicación, esta configuración estará disponible para su revisión en la página de **configuración de publicación** de la sección **Administrar**. La configuración se puede cambiar en cada publicación. Si cancela una publicación, también se cancelarán los cambios realizados durante la misma.

### <a name="when-your-app-is-published"></a>Cuando se publique la aplicación

Cuando la aplicación se publica correctamente, aparece un cuadro de notificación en la parte superior del explorador. La notificación también incluye un vínculo a los puntos de conexión.

Si necesita la dirección URL del punto de conexión, seleccione el vínculo. También puede obtener las direcciones URL del punto de conexión seleccionando **Administrar** en el menú superior y, a continuación, **Recursos de Azure** en el menú de la izquierda.

## <a name="sentiment-analysis"></a>análisis de opiniones

<a name="enable-sentiment-analysis"></a>

El análisis de opiniones permite que LUIS se integre con [Text Analytics](https://azure.microsoft.com/services/cognitive-services/text-analytics/) para proporcionar un análisis de frases clave y opiniones.

No tiene que proporcionar una clave de Text Analytics y no se carga ningún costo de facturación a su cuenta de Azure por este servicio.

Los datos de opinión son una puntuación entre 1 y 0 que indica el valor de opinión positiva (más cercano a 1) o negativa (más cercano a 0) de los datos. La etiqueta de opinión de `positive`, `neutral` y `negative` es por referencia cultural admitida. Actualmente, solo se admites etiquetas de opinión en inglés.

Para obtener más información acerca de la respuesta del punto de conexión JSON con análisis de sentimiento, consulte [Análisis de sentimiento](luis-reference-prebuilt-sentiment.md).

## <a name="speech-priming"></a>Preparación para la voz

La preparación para la voz es un proceso que usa el envío del modelo LUIS a los servicios de voz antes de realizar la conversión de texto a voz. Esto permite que el servicio de voz proporcione la conversión de voz de forma más precisa para el modelo. Esto permite obtener solicitudes y respuestas de voz y de LUIS en una sola llamada, cuando se realiza una llamada de voz y se devuelve una respuesta de LUIS. Asimismo, proporciona menos latencia en general.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Administrar claves](./luis-how-to-azure-subscription.md) para agregar claves a la clave de suscripción de Azure a LUIS y para obtener información sobre cómo establecer la clave de Bing Spell Check e incluir todas las intenciones en los resultados.
* Consulte [Entrenar y probar la aplicación](luis-interactive-test.md) para obtener instrucciones sobre cómo probar la aplicación publicada en la consola de prueba.

