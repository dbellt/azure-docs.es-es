---
author: glecaros
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: gelecaro
ms.openlocfilehash: c2222918106de8594881f105cc86ab1b45135027
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110164115"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para Linux.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="system-requirements"></a>Requisitos del sistema

Linux: consulte la lista de [distribuciones y arquitecturas de destino de Linux admitidas](~/articles/cognitive-services/speech-service/speech-sdk.md).

## <a name="prerequisites"></a>Requisitos previos

Para realizar este inicio rápido, necesita lo siguiente:

* [Archivo binario de Go (1.13 o posterior)](https://golang.org/dl/)
* El entorno de Linux se configura según los [requisitos del sistema y las instrucciones de configuración](~/articles/cognitive-services/speech-service/speech-sdk.md#get-the-speech-sdk).


[!INCLUDE [linux-install-sdk](linux-install-sdk.md)]


## <a name="configure-go-environment"></a>Configuración del entorno de Go

Realice los pasos siguientes para configurar el entorno de Go para buscar el SDK de Voz. En ambos pasos, reemplace `<architecture>` por la arquitectura del procesador de la CPU. Esta será `x86`, `x64`, `arm32` o `arm64`.

1. Dado que los enlaces se basan en `cgo`, es necesario que establezca las variables de entorno para que Go pueda encontrar el SDK:

   ```sh
   export CGO_CFLAGS="-I$SPEECHSDK_ROOT/include/c_api"
   export CGO_LDFLAGS="-L$SPEECHSDK_ROOT/lib/<architecture> -lMicrosoft.CognitiveServices.Speech.core"
   ```

1. Para ejecutar aplicaciones que incluyen el SDK, es necesario indicar al sistema operativo dónde encontrar las bibliotecas:

   ```sh
   export LD_LIBRARY_PATH="$SPEECHSDK_ROOT/lib/<architecture>:$LD_LIBRARY_PATH"
   ```

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list-go.md)]
