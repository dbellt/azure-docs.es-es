---
title: Procedimiento para detectar personajes animados con Video Indexer
titleSuffix: Azure Media Services
description: En este procedimiento se muestra cómo usar la detección de personajes animados con Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.custom: references_regions
ms.topic: how-to
ms.date: 12/07/2020
ms.author: juliako
ms.openlocfilehash: e80699a043d4c18a1bc7ba75ce58c6972fae0fad
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531562"
---
# <a name="use-the-animated-character-detection-preview-with-portal-and-api"></a>Uso de la detección de personajes animados (versión preliminar) con el portal y la API 

Video Indexer de Azure Media Services admite la detección, la agrupación y el reconocimiento de personajes en contenido animado. Esta funcionalidad está disponible a través de Azure Portal y la API. Revise el tema de [información general](animated-characters-recognition.md).

En este artículo se muestra cómo usar la detección de personajes animados con Azure Portal y Video Indexer API.

## <a name="use-the-animated-character-detection-with-portal"></a>Uso de la detección de personajes animados con el portal 

En las cuentas de evaluación la integración de Custom Vision se administra mediante Video Indexer, puede empezar a crear y usar el modelo de personajes animados. Si usa la cuenta de prueba, puede omitir la siguiente sección ("Conexión de una cuenta de Custom Vision").

### <a name="connect-your-custom-vision-account-paid-accounts-only"></a>Conexión de la cuenta de Custom Vision (solo cuentas de pago)

Si es propietario de una cuenta de pago de Video Indexer, primero debe conectar una cuenta de Custom Vision. Si aún no tiene una cuenta de Custom Vision, cree una. Para obtener más información, consulte [Custom Vision](../../cognitive-services/custom-vision-service/overview.md).

> [!NOTE]
> Las dos cuentas deben estar en la misma región. La integración de Custom Vision no se admite actualmente en la región de Japón.

Las cuentas de pago que tienen acceso a su cuenta de Custom Vision pueden ver los modelos y las imágenes etiquetadas allí. Obtenga más información sobre cómo  [mejorar su clasificador en Custom Vision](../../cognitive-services/custom-vision-service/getting-started-improving-your-classifier.md). 

Tenga en cuenta que el entrenamiento del modelo debe realizarse exclusivamente a través de Video Indexer, no del sitio web de Custom Vision. 

#### <a name="connect-a-custom-vision-account-with-api"></a>Conexión de una cuenta de Custom Vision con la API 

Siga estos pasos para conectar su cuenta de Custom Vision a Video Indexer, o bien para cambiar la cuenta de Custom Vision que está conectada actualmente a Video Indexer:

1. Vaya a [www.customvision.ai](https://www.customvision.ai) e inicie sesión.
1. Copie las claves de los recursos de Entrenamiento y Predicción:

    > [!NOTE]
    > Para proporcionar todas las claves, debe tener dos recursos independientes en Custom Vision, uno para el entrenamiento y otro para la predicción.
1. Especifique otra información:

    * Punto de conexión 
    * Identificador del recurso de predicción
1. Inicie sesión en [Video Indexer](https://vi.microsoft.com/).
1. Haga clic en el signo de interrogación de la esquina superior derecha de la página y elija **API Reference** (Referencia de la API).
1. Asegúrese de que está suscrito a API Management haciendo clic en la pestaña **Products** (Productos). Si tiene una API conectada, puede continuar con el paso siguiente; de lo contrario, suscríbase. 
1. En el portal para desarrolladores, haga clic en **Complete API Reference** (Completar referencia de la API) y vaya a **Operations** (Operaciones).  
1. Seleccione **Connect Custom Vision Account (PREVIEW)** [Conectar cuenta de Custom Vision (VERSIÓN PRELIMINAR)] y haga clic en **Try it** (Probar).
1. Rellene los campos obligatorios, así como el token de acceso y haga clic en **Send** (Enviar). 

    Para obtener más información sobre cómo obtener el token de Video Indexer, vaya al [portal para desarrolladores](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Account-Access-Token) y consulte la [documentación pertinente](video-indexer-use-apis.md#obtain-access-token-using-the-authorization-api).  
1. Cuando la llamada devuelva la respuesta 200 OK, significa que la cuenta está conectada.
1. Para comprobar la conexión, vaya al portal de [Video Indexer](https://vi.microsoft.com/):
1. Haga clic en el botón **Content model customization** Personalización del modelo de contenido() en la esquina superior derecha de la página.
1. Vaya a la pestaña **Animated characters** (Personajes animados).
1. Una vez que haga clic en Manage models (Administrar modelos) en Custom Vision, se le transferirá a la cuenta de Custom Vision que acaba de conectar.

> [!NOTE]
> Actualmente, solo se admiten los modelos creados a través de Video Indexer. Los modelos que se crean a través de Custom Vision no estarán disponibles. Además, el procedimiento recomendado es editar los modelos creados a través de Video Indexer solo con la plataforma de Video Indexer, ya que los cambios realizados a través de Custom Vision pueden provocar resultados no deseados.

### <a name="create-an-animated-characters-model"></a>Creación de un modelo de personajes animados

1. Vaya al sitio web de [Video Indexer](https://vi.microsoft.com/) e inicie sesión.
1. Para personalizar un modelo en su cuenta, seleccione el botón **Content model customization** (Personalización del modelo de contenido) a la izquierda de la página.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/content-model-customization/content-model-customization.png" alt-text="Personalización de un modelo de contenido en Video Indexer":::
1. Vaya a la pestaña **Animated characters** (Personajes animados) en la sección de personalización de modelos.
1. Haga clic en **Agregar modelo**.
1. Asigne un nombre al modelo y haga clic en Entrar para guardarlo.

> [!NOTE]
> El procedimiento recomendado es tener un modelo de Custom Vision para cada serie animada. 

### <a name="index-a-video-with-an-animated-model"></a>Indexación de un vídeo con un modelo animado

Para el entrenamiento inicial, cargue al menos dos vídeos. Para lograr un buen modelo de reconocimiento, lo deseable es que ninguno de ellos dure menos de 15 minutos. Si tiene episodios más cortos, es aconsejable cargar al menos 30 minutos de contenido de vídeo antes del entrenamiento. Esto le permitirá fusionar mediante combinación grupos que pertenezcan al mismo personaje desde diferentes escenas y orígenes, lo que, por consiguiente, aumenta la posibilidad de que detecte el personaje en los episodios siguientes que se indexen. Para entrenar un modelo en varios vídeos (episodios), debe indexar todos con el mismo modelo de animación. 

1. Haga clic en el botón **Upload** (Cargar).
1. Elija un vídeo para cargar (desde un archivo o una dirección URL).
1. Haga clic en **Opciones avanzadas**.
1. En **People / Animated characters** (Personas/personajes animados), elija **Animation models** (Modelos de animación).
1. Si tiene un modelo, se elegirá automáticamente y, si tiene varios modelos, puede elegir el que proceda del menú desplegable.
1. Haga clic en el botón para cargar.
1. Una vez que se indexa el vídeo, verá los personajes detectados en la sección **Animated characters** (Personajes animados) del panel **Información**.

Antes de etiquetar y entrenar el modelo, todos los personajes animados se denominarán "Unknown #X" (Desconocido X). Después de entrenar el modelo, también se reconocerán.

### <a name="customize-the-animated-characters-models"></a>Personalización de los modelos de personajes animados

1. Asigne nombres a los personajes en Video Indexer.

    1. Después del grupo de personajes creado por el modelo, se recomienda examinar estos grupos en Custom Vision. 
    1. Para etiquetar un personaje animado en el vídeo, vaya a la pestaña  **Insights**  (Información) y haga clic en el botón  **Edit**  (Editar) de la esquina superior derecha de la ventana. 
    1. En el panel  **Información**  (Información), haga clic en cualquiera de los personajes animados detectados y cambie sus nombres de "Unknown #X" (Desconocido X) por un nombre temporal (o el nombre que se asignara previamente al personaje). 
    1. Después de escribir el nuevo nombre, haga clic en el icono de verificación junto a él. Esto guarda el nuevo nombre en el modelo en Video Indexer. 
1. Solo las cuentas de pago: examine los grupos en Custom Vision. 

    > [!NOTE]
    > Las cuentas de pago que tienen acceso a su cuenta de Custom Vision pueden ver los modelos y las imágenes etiquetadas allí. Obtenga más información sobre cómo  [mejorar su clasificador en Custom Vision](../../cognitive-services/custom-vision-service/getting-started-improving-your-classifier.md). Es importante tener en cuenta que el entrenamiento del modelo debe realizarse exclusivamente a través de Video Indexer (como se ha descrito en este tema), no del sitio web de Custom Vision. 

    1. Vaya a la página **Custon Models** (Modelos personalizados) de Video Indexer y elija la pestaña **Animated characters** (Personajes animados). 
    1. Haga clic en el botón Edit (Editar) del modelo en el que trabaja para administrarlo en Custom Vision. 
    1. Examine todos los grupos de personajes: 

        * Si el grupo contiene imágenes no relacionadas, se recomienda eliminarlas en el sitio web de Custom Vision. 
        * Si hay imágenes que pertenezcan a otro personaje, cambie la etiqueta en dichas imágenes concretas; para ello, haga clic en la imagen, agregue la etiqueta correcta y elimine la etiqueta equivocada. 
        * Si el grupo no es correcto, es decir, que contiene principalmente imágenes que no son del personaje o imágenes de varios personajes, puede eliminarlo en el sitio web de Custom Vision o en la información de Video Indexer. 
        * A veces, el algoritmo de agrupación dividirá los personajes en grupos diferentes. Por consiguiente, se recomienda asignar el mismo nombre a todos los grupos que pertenecen al mismo personaje (en la pestaña de información de Video Indexer), lo que hará que de inmediato todos estos grupos aparezcan como en el sitio web de Custom Vision. 
    1. Una vez que se refine el grupo, asegúrese de que el nombre inicial con el que lo etiquetó refleja el carácter del mismo. 
1. Entrenamiento del modelo 

    1. Una vez que haya terminado de editar todos los nombres que desee, debe entrenar el modelo. 
    1. Una vez que se entrena un personaje en el modelo, se reconocerá en el siguiente vídeo indexado con ese modelo. 
    1. Abra la página de personalización y haga clic en la pestaña  **Animated characters**  (Personajes animados) y haga clic en el botón  **Train** (Entrenar) para entrenar el modelo. Para mantener la conexión entre Video 
    
Indexer y el modelo, no entrene el modelo en el sitio web Custom Vision (las cuentas de pago tienen acceso al sitio web de Custom Vision), solo en Video Indexer. Una vez entrenado, cualquier vídeo que se indexe o se vuelva a indexar con ese modelo reconocerá los personajes entrenados. 

## <a name="delete-an-animated-character-and-the-model"></a>Eliminación de un personaje animado y el modelo

1. Elimine un personaje animado.

    1. Para eliminar un personaje animado de la información del vídeo, vaya a la pestaña **Información** y haga clic en el botón **Editar** en la esquina superior derecha de la ventana.
    1. Elija el personaje animado y, a continuación, haga clic en el botón **Eliminar** bajo su nombre.

    > [!NOTE]
    > Esto eliminará la información de este vídeo, pero no afectará al modelo.
1. Elimine un modelo.

    1. Haga clic en el botón de **personalización del modelo de contenido** en el menú superior y haga clic en la pestaña **Animated characters** (Personajes animados).
    1. Haga clic en el icono de puntos suspensivos a la derecha del modelo que desea eliminar y, a continuación, en el botón para eliminar.
    
    * Cuenta de pago: el modelo se desconectará de Video Indexer y no podrá volver a conectarlo.
    * Cuenta de prueba: el modelo también se eliminará de Custom Vision. 
    
        > [!NOTE]
        > En una cuenta de prueba, solo tiene un modelo que puede usar. Después de eliminarlo, no puede entrenar otros modelos.

## <a name="use-the-animated-character-detection-with-api"></a>Uso de la detección de personajes animados con API 

1. Conecte una cuenta de Custom Vision.

    Si es propietario de una cuenta de pago de Video Indexer, primero debe conectar una cuenta de Custom Vision. <br/>
    Si aún no tiene una cuenta de Custom Vision, cree una. Para obtener más información, consulte [Custom Vision](../../cognitive-services/custom-vision-service/overview.md).

    [Conecte la cuenta de Custom Vision con API](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Connect-Custom-Vision-Account).
1. Cree un modelo de personajes animados.

    Use la API de [creación de modelo de animación](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Create-Animation-Model).
1. Indexe o vuelva a indexar un vídeo.

    Use la API de [reindexación](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Re-Index-Video). 
1. Personalice los modelos de personajes animados.

    Use la API de [entrenamiento de modelo de animación](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Train-Animation-Model).

### <a name="view-the-output"></a>Visualización de la salida

Vea los personajes animados en el archivo JSON generado.

```json
"animatedCharacters": [
    {
    "videoId": "e867214582",
    "confidence": 0,
    "thumbnailId": "00000000-0000-0000-0000-000000000000",
    "seenDuration": 201.5,
    "seenDurationRatio": 0.3175,
    "isKnownCharacter": true,
    "id": 4,
    "name": "Bunny",
    "appearances": [
        {
            "startTime": "0:00:52.333",
            "endTime": "0:02:02.6",
            "startSeconds": 52.3,
            "endSeconds": 122.6
        },
        {
            "startTime": "0:02:40.633",
            "endTime": "0:03:16.6",
            "startSeconds": 160.6,
            "endSeconds": 196.6
        },
    ]
    },
]
```

## <a name="limitations"></a>Limitaciones

* Actualmente, la funcionalidad de "identificación de animación" no se admite en la región de Asia Oriental.
* Es posible que los personajes que aparezcan pequeños o lejanos en el vídeo no se identifiquen correctamente si la calidad del vídeo es deficiente.
* Se recomienda usar un modelo por conjunto de personajes animados (por ejemplo, por serie animada).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Video Indexer](video-indexer-overview.md)