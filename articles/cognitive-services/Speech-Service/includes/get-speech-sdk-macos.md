---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: b68fb8cf5458081f96febbac75fd393a80345f60
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105043246"
---
Al desarrollar para macOS, hay tres SDK de voz disponibles.

- El SDK de voz de Objective-C está disponible de forma nativa como un paquete de CocoaPod.
- El SDK de voz de .NET podría usarse con **Xamarin.Mac**, ya que implementa .NET Standard 2.0.
- El SDK de voz de Python está disponible como un módulo de PyPI.

> [!TIP]
> Para obtener más información sobre el uso del SDK de Voz de Objective-C con Swift, consulte <a href="https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift" target="_blank">Importación de Objective-C en Swift </a>.

### <a name="system-requirements"></a>Requisitos del sistema

- Una versión macOS 10.13 o posterior

# <a name="xcode"></a>[Xcode](#tab/mac-xcode)

:::row:::
    :::column span="3":::
        El paquete CocoaPod de macOS está disponible para su descarga y uso con el entorno de desarrollo integrado (IDE) de <a href="https://apps.apple.com/us/app/xcode/id497799835" target="_blank">Xcode 9.4.1 o posterior </a>. En primer lugar, <a href="https://aka.ms/csspeech/macosbinary" target="_blank">descargue el archivo binario de CocoaPod </a>. Extraiga el pod en el mismo directorio para su uso previsto, cree un *Podfile* y enumere `pod` como `target`.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xcode" src="https://docs.microsoft.com/media/logos/logo_xcode.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

```
platform :ios, '9.3'
use_frameworks!

target 'MyApp' do
  pod 'MicrosoftCognitiveServicesSpeech', '~> 1.15.0'
end
```

# <a name="xamarinmac"></a>[Xamarin.Mac](#tab/mac-xamarin)

:::row:::
    :::column span="3":::
        Xamarin.Mac expone el SDK de macOS completo para que los desarrolladores de .NET puedan compilar aplicaciones Mac nativas mediante C#. Para más información, vea <a href="/xamarin/mac/" target="_blank">Xamarin.Mac </a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xamarin" src="https://docs.microsoft.com/media/logos/logo_xamarin.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

# <a name="python"></a>[Python](#tab/mac-python)

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

---

#### <a name="additional-resources"></a>Recursos adicionales

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/macos" target="_blank">Código fuente de Objective-C en la guía de inicio rápido del SDK de voz de macOS </a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/macos" target="_blank">Código fuente de Swift en la guía de inicio rápido del SDK de voz de macOS </a>