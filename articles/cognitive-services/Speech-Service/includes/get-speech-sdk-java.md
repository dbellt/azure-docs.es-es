---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 18d363ba216c43e01e1238ea1200db99703939d0
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110059202"
---
:::row:::
    :::column span="3":::
        El SDK de Java para Android está empaquetado como una <a href="https://developer.android.com/studio/projects/android-library" target="_blank">biblioteca de Android (AAR)</a>, que incluye las bibliotecas necesarias, así como los permisos necesarios de Android. Se hospeda en un repositorio de Maven en `https://csspeechstorage.blob.core.windows.net/maven/` como un paquete `com.microsoft.cognitiveservices.speech:client-sdk:1.17.0`. (Asegúrese de que la versión 1.17.0 sea la más reciente mediante la [búsqueda en nuestro repositorio de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/search?q=com.microsoft.cognitiveservices.speech%3Aclient-sdk)).
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Java" src="/media/logos/logo_java.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

Para consumir el paquete desde el proyecto de Android Studio, haga los siguientes cambios:

1. En el archivo *build.gradle* de nivel de proyecto, agregue lo siguiente a la sección `repositories`:
  ```gradle
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

2. En el archivo *build.gradle* de nivel de módulo, agregue lo siguiente a la sección `dependencies`:
  ```gradle
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.17.0'
  ```

El SDK de Java es parte del [SDK de dispositivos de voz](../speech-devices-sdk.md).

#### <a name="additional-resources"></a>Recursos adicionales

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android" target="_blank">Código fuente de la guía de inicio rápido de Java específico para Android </a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/jre" target="_blank">Código fuente dela guía de inicio rápido de Java Runtime (JRE) </a>
