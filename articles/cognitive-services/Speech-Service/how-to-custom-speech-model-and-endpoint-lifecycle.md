---
title: Ciclo de vida de los modelos y los puntos de conexión del servicio de voz de Custom Speech
titleSuffix: Azure Cognitive Services
description: Custom Speech proporciona modelos base adaptables y permite crear modelos personalizados a partir de los datos. En este artículo se describen las escalas de tiempo de los modelos y los puntos de conexión que utilizan estos modelos.
services: cognitive-services
author: heikora
manager: dongli
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/2/2021
ms.author: heikora
ms.openlocfilehash: b82a732533c3d069b519b07c3209d4b96c472900
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106385032"
---
# <a name="model-and-endpoint-lifecycle"></a>Ciclo de vida del modelo y el punto de conexión

Nuestra voz estándar (no personalizada) se basa en modelos de IA que llamamos modelos base. En la mayoría de los casos, entrenamos un modelo base diferente para cada idioma hablado que admitimos.  Actualizamos el servicio de voz con nuevos modelos base cada pocos meses para mejorar la precisión y la calidad.  
Con Habla personalizada, los modelos personalizados se crean mediante la adaptación de un modelo base elegido con los datos de un escenario de cliente determinado. Una vez que se crea un modelo personalizado, ese modelo no se actualizará ni cambiará, aunque el modelo base correspondiente a partir del cual se haya adaptado se actualice en el servicio de voz estándar.  
Esta directiva le permite seguir usando un modelo personalizado determinado durante mucho tiempo después de tener un modelo personalizado que satisfaga sus necesidades.  Pero se recomienda volver a crear periódicamente el modelo personalizado para que pueda adaptarse al modelo base más reciente y aprovechar las ventajas de la precisión y la calidad mejoradas.

Otros términos clave relacionados con el ciclo de vida del modelo incluyen:

* **Adaptación**: tomar un modelo base y personalizarlo para el dominio o escenario propios mediante datos de texto o datos de audio.
* **Descodificación**: usar un modelo y realizar el reconocimiento de voz (descodificar audio en texto).
* **Punto de conexión**: una implementación específica del usuario, bien de un modelo base, bien de un modelo personalizado al que *solo* puede acceder un usuario dado.

### <a name="expiration-timeline"></a>Escala de tiempo de expiración

A medida que los nuevos modelos y la nueva funcionalidad pasan a estar disponibles, se retiran los modelos más antiguos y menos precisos. Consulte las siguientes escalas de tiempo de expiración del modelo y el punto de conexión:

**Modelos base** 

* Adaptación: disponibles durante un año. Una vez que se importa el modelo, está disponible durante un año para crear modelos personalizados. Después de un año, deben crearse nuevos modelos personalizados a partir de una versión más reciente del modelo base.  
* Descodificación: disponibles durante dos años después de la importación. Por consiguiente, puede crear un punto de conexión y usar la transcripción por lotes durante dos años con este modelo. 
* Puntos de conexión: disponibles según la misma escala de tiempo que la descodificación.

**Modelos personalizados**

* Descodificación: disponible durante dos años después de la creación del modelo. Por consiguiente, el modelo personalizado se puede usar durante dos años (lote, tiempo real y pruebas) una vez creado. Después de dos años, *debe volver a entrenar el modelo*, ya que la mayoría de las veces el modelo base habrá quedado en desuso para la adaptación.  
* Puntos de conexión: disponibles según la misma escala de tiempo que la descodificación.

Cuando expira un modelo base o un modelo personalizado, siempre se revierte a la *versión más reciente del modelo base*. Por tanto, la implementación nunca se interrumpirá, pero puede ser menos precisa para *sus datos específicos* si los modelos personalizados llegan a expirar. Puede ver la expiración de un modelo en los siguientes lugares del portal de Habla personalizada de Speech Studio:

* Resumen del entrenamiento del modelo
* Detalles del entrenamiento del modelo
* Resumen de implementación
* Detalles de la implementación

Este es un formulario de ejemplo del resumen del entrenamiento del modelo:

![Resumen de entrenamiento del modelo ](media/custom-speech/custom-speech-model-training-with-expiry.png) Y también de la página de detalles de entrenamiento del modelo:

![Detalles del entrenamiento del modelo](media/custom-speech/custom-speech-model-details-with-expiry.png)

También puede comprobar las fechas de expiración mediante las API [`GetModel`](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetModel) y [`GetBaseModel`](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetBaseModel) de habla personalizada, en la propiedad `deprecationDates` de la respuesta JSON.

Este es un ejemplo de los datos de expiración de la llamada de API GetModel. **DEPRECATIONDATES** se muestra cuando el modelo expira: 
```json
{
    "SELF": "HTTPS://WESTUS2.API.COGNITIVE.MICROSOFT.COM/SPEECHTOTEXT/V3.0/MODELS/{id}",
    "BASEMODEL": {
    "SELF": HTTPS://WESTUS2.API.COGNITIVE.MICROSOFT.COM/SPEECHTOTEXT/V3.0/MODELS/BASE/{id}
    },
    "DATASETS": [
    {
        "SELF": https://westus2.api.cognitive.microsoft.com/speechtotext/v3.0/datasets/{id}
    }
    ],
    "LINKS": {
    "MANIFEST": "HTTPS://WESTUS2.API.COGNITIVE.MICROSOFT.COM/SPEECHTOTEXT/V3.0/MODELS/{id}/MANIFEST",
    "COPYTO": https://westus2.api.cognitive.microsoft.com/speechtotext/v3.0/models/{id}/copyto
    },
    "PROJECT": {
        "SELF": https://westus2.api.cognitive.microsoft.com/speechtotext/v3.0/projects/{id}
    },
    "PROPERTIES": {
    "DEPRECATIONDATES": {
        "ADAPTATIONDATETIME": "2022-01-15T00:00:00Z",     // last date the base model can be used for adaptation
        "TRANSCRIPTIONDATETIME": "2023-03-01T21:27:29Z"   // last date this model can be used for decoding
    }
    },
    "LASTACTIONDATETIME": "2021-03-01T21:27:40Z",
    "STATUS": "SUCCEEDED",
    "CREATEDDATETIME": "2021-03-01T21:27:29Z",
    "LOCALE": "EN-US",
    "DISPLAYNAME": "EXAMPLE MODEL",
    "DESCRIPTION": "",
    "CUSTOMPROPERTIES": {
    "PORTALAPIVERSION": "3"
    }
}
```
Tenga en cuenta que puede actualizar el modelo en un punto de conexión de habla personalizada sin tiempo de inactividad si cambia el modelo usado por el punto de conexión en la sección de implementación de Speech Studio o por medio de la API de habla personalizada.

## <a name="what-happens-when-models-expire-and-how-to-update-them"></a>Qué ocurre cuando los modelos expiran y cómo se actualizan
Lo que sucede cuando un modelo expira y cómo se actualiza dependen de cómo se use.
### <a name="batch-transcription"></a>Transcripción de Azure Batch
Si expira un modelo que se usa con solicitudes de [transcripción por lotes](batch-transcription.md), se producirá un error 4xx. Para evitar esto, actualice el parámetro `model` en el archivo JSON enviado en el cuerpo de la solicitud **Crear transcripción** para que apunte a un modelo base más reciente o a un modelo personalizado más reciente. También puede quitar la entrada `model` del archivo JSON para usar siempre el modelo base más reciente.
### <a name="custom-speech-endpoint"></a>Punto de conexión de Habla personalizada
Si expira un modelo que usa un [punto de conexión de Habla personalizada](how-to-custom-speech-train-model.md), el servicio revertirá automáticamente al uso del modelo base más reciente para el idioma que esté usando. Puede seleccionar **Implementación** en el menú de **Habla personalizada** en la parte superior de la página y, a continuación, hacer clic en el nombre del punto de conexión para ver sus detalles. En la parte superior de la página de detalles, verá un botón **Actualizar modelo** que le permite actualizar sin problemas el modelo usado por este punto de conexión sin ningún tiempo de inactividad. También puede hacer este cambio mediante programación con la API REST de [**actualización del modelo**](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/UpdateModel).

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento e implementación de un modelo](how-to-custom-speech-train-model.md)

## <a name="additional-resources"></a>Recursos adicionales

* [Preparación y prueba de los datos](./how-to-custom-speech-test-and-train.md)
* [Inspección de los datos](how-to-custom-speech-inspect-data.md)