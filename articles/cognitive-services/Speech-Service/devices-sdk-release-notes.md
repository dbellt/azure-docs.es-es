---
title: Documentación de Speech Devices SDK
titleSuffix: Azure Cognitive Services
description: Las notas de la versión proporcionan un registro de actualizaciones, mejoras, correcciones de errores y cambios en el SDK de dispositivos de voz. Este artículo se actualiza con cada versión del SDK de los dispositivos de voz.
services: cognitive-services
author: wsturman
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: wellsi
ms.openlocfilehash: 5214d914c104fdf6df302c7879230bba2b3d2928
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891849"
---
# <a name="release-notes-speech-devices-sdk"></a>Notas de la versión: SDK de dispositivos de voz

En las siguientes secciones se indican los cambios en las versiones más recientes.

## <a name="speech-devices-sdk-1160"></a>SDK de dispositivos de voz 1.16.0:

- Se ha corregido el [problema de Github n.º 22](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/issues/22).
- Se ha actualizado el componente [SDK de voz](./speech-sdk.md) a la versión 1.16.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-1150"></a>SDK de dispositivos de Voz 1.15.0:

- Se ha actualizado a la nueva pila de audio de Microsoft (MAS) con formación de haces y reducción de ruido mejorada para la voz.
- Se ha reducido el tamaño binario hasta un 70 % según el destino.
- Se ha añadido la compatibilidad con [Azure Percept Audio](../../azure-percept/overview-azure-percept-audio.md) con [binario](https://aka.ms/sdsdk-download-APAudio).
- Se ha actualizado el componente [SDK de Voz](./speech-sdk.md) a la versión 1.15.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-1110"></a>SDK de dispositivos de voz 1.11.0:

- Compatibilidad con [geometrías de matriz de micrófonos arbitrarias](how-to-devices-microphone-array-configuration.md) y establecimiento del ángulo de trabajo a través de un [archivo de configuración](https://aka.ms/sdsdk-micarray-json).
- Compatibilidad con [Urbetter DDK](http://www.urbetter.com/products_56/278.html).
- Se han publicado los binarios para [GGEC Speaker](https://aka.ms/sdsdk-download-speaker) usados en nuestro [ejemplo del Asistente de voz](https://aka.ms/sdsdk-speaker).
- Se han publicado los binarios para [Linux ARM32](https://aka.ms/sdsdk-download-linux-arm32) y [Linux ARM 64](https://aka.ms/sdsdk-download-linux-arm64) para Raspberry Pi y dispositivos similares.
- Se ha actualizado el componente [SDK de Voz](./speech-sdk.md) a la versión 1.11.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-190"></a>SDK de dispositivos de voz 1.9.0:

- Se proporcionan archivos binarios iniciales para [Urbetter DDK](https://aka.ms/sdsdk-download-urbetter) (Linux ARM64).
- Ahora, Roobo v1 utiliza Maven para Speech SDK
- Se ha actualizado el componente [SDK de Voz](./speech-sdk.md) a la versión 1.9.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-170"></a>SDK de dispositivos de voz 1.7.0:

- Ahora se admite Linux ARM.
- Se proporcionan archivos binarios iniciales para [Roobo v2 DDK](https://aka.ms/sdsdk-download-roobov2) (Linux ARM64).
- Los usuarios de Windows pueden usar `AudioConfig.fromDefaultMicrophoneInput()` o `AudioConfig.fromMicrophoneInput(deviceName)` para especificar el micrófono que se utilizará.
- Se ha optimizado el tamaño de la biblioteca.
- Compatibilidad con el reconocimiento de múltiples turnos mediante el mismo objeto reconocedor de voz/intención.
- Corrige un problema ocasional en el que el proceso dejaba de responder y se detenía el reconocimiento.
- Las aplicaciones de ejemplo ahora contienen un archivo participants.properties de ejemplo para demostrar el formato del archivo.
- Se ha actualizado el componente [SDK de Voz](./speech-sdk.md) a la versión 1.7.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-160"></a>SDK de dispositivos de voz 1.6.0:

- Compatibilidad con [Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/) en Windows y Linux con una [aplicación de ejemplo](./speech-devices-sdk.md) común
- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.6.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-151"></a>SDK de dispositivos de voz 1.5.1:

- Incluya [transcripción de conversaciones](./conversation-transcription.md) en la aplicación de ejemplo.
- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.5.1. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-150-2019-may-release"></a>SDK de dispositivos de voz 1.5.0: Versión de mayo de 2019

- SDK de dispositivos de voz es ahora disponibilidad general y ya no está en versión preliminar controlada.
- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.5.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).
- La nueva tecnología de palabra clave proporciona importantes mejoras de calidad; consulte Cambios importantes.
- Nueva canalización de procesamiento de audio para el reconocimiento mejorado de campo lejano.

**Cambios importantes**

- debido a la nueva tecnología de palabra clave, todas las palabras clave deben volver a crearse en nuestro portal mejorado de palabras clave. Para quitar completamente las palabras clave antiguas del dispositivo, desinstale la aplicación antigua.
  - adb uninstall com.microsoft.cognitiveservices.speech.samples.sdsdkstarterapp

## <a name="speech-devices-sdk-140-2019-apr-release"></a>SDK de dispositivos de voz 1.4.0: Versión de abril de 2019

- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.4.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).

## <a name="speech-devices-sdk-131-2019-mar-release"></a>SDK de dispositivos de voz 1.3.1: Versión de marzo de 2019

- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.3.1. Para más información, consulte sus [notas de la versión](./releasenotes.md).
- Administración de palabras clave; consulte Cambios importantes.
- La aplicación de ejemplo agrega la opción de idioma para el reconocimiento y la traducción de voz.

**Cambios importantes**

- La [instalación de una palabra clave](./custom-keyword-basics.md) se ha simplificado, y ahora es parte de la aplicación (no necesita una instalación independiente en el dispositivo).
- El reconocimiento de la palabra clave ha cambiado, y se admiten dos eventos.
  - `RecognizingKeyword,` indica que el resultado de voz contiene texto de palabras clave (sin verificar).
  - `RecognizedKeyword` indica que el reconocimiento de la palabra clave terminó de reconocer la palabra clave especificada.

## <a name="speech-devices-sdk-110-2018-nov-release"></a>SDK de dispositivos de voz 1.1.0: Versión de noviembre de 2018

- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.1.0. Para más información, consulte sus [notas de la versión](./releasenotes.md).
- Ha aumentado la precisión del reconocimiento de voz a gran distancia gracias a nuestro algoritmo mejorado de procesamiento de audio.
- La aplicación de ejemplo ahora es compatible con el reconocimiento de voz en chino.

## <a name="speech-devices-sdk-101-2018-oct-release"></a>SDK de dispositivos de voz 1.0.1: versión de octubre de 2018

- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 1.0.1. Para más información, consulte sus [notas de la versión](./releasenotes.md).
- La precisión del reconocimiento de voz se verá mejorada gracias a nuestro algoritmo mejorado de procesamiento de audio.
- Se ha corregido un error en la sesión de audio de reconocimiento continuo.

**Cambios importantes**

- Con esta versión se presentan una serie de cambios importantes. Consulte [esta página](https://aka.ms/csspeech/breakingchanges_1_0_0) para obtener detalles relacionados con las API.
- Los archivos de modelo de reconocimiento de palabras clave no son compatibles con el SDK de dispositivos de voz 1.0.1. Los archivos existentes de palabras clave se eliminarán después de que los nuevos se escriban en el dispositivo.

## <a name="speech-devices-sdk-050-2018-aug-release"></a>SDK de dispositivos de voz 0.5.0: versión de agosto de 2018

- Se ha mejorado la precisión del reconocimiento de voz mediante la corrección de un error en el código de procesamiento de audio.
- Se ha actualizado el componente [Speech SDK](./speech-sdk.md) a la versión 0.5.0. Para más información, consulte sus [notas de la versión](releasenotes.md#cognitive-services-speech-sdk-050-2018-july-release).

## <a name="speech-devices-sdk-0212733-2018-may-release"></a>SDK de dispositivos de voz 0.2.12733: Versión de mayo de 2018

La primera versión preliminar pública de Speech Devices SDK de Cognitive Services.