---
title: 'Lenguaje de marcado de síntesis de voz (SSML): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Uso del lenguaje de marcado de síntesis de voz para controlar la pronunciación y la prosodia en la conversión de texto a voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: trbye
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 2c66a7e3bf9e417b47d08e50e21c08625e9d0549
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210217"
---
# <a name="improve-synthesis-with-speech-synthesis-markup-language-ssml"></a>Mejora de la síntesis con el Lenguaje de marcado de síntesis de voz (SSML)

El lenguaje de marcado de síntesis de voz (SSML) es un lenguaje de marcado basado en XML que permite a los desarrolladores especificar cómo se convierte el texto de entrada en una voz sintetizada mediante el servicio de texto a voz. En comparación con el texto sin formato, SSML permite a los desarrolladores ajustar el tono, la pronunciación, la velocidad del habla, el volumen y muchas cosas más en la salida de texto a voz. La puntuación normal, como hacer una pausa después de un punto o usar la entonación correcta cuando una oración termina con un signo de interrogación, se administra automáticamente.

La implementación del servicios de voz de SSML se basa en la [versión 1.0 del lenguaje de marcado de síntesis de voz](https://www.w3.org/TR/speech-synthesis) del World Wide Web Consortium.

> [!IMPORTANT]
> Los caracteres en chino, japonés y coreano se cuentan como dos caracteres para la facturación. Para obtener más información, consulte el apartado [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

## <a name="neural-and-custom-voices"></a>Voces neuronales y personalizadas

Use una voz neuronal de tipo humano o cree su propia voz personalizada única para su producto o marca. Para obtener una lista completa de los idiomas compatibles, las configuraciones regionales y las voces, consulte [compatibilidad con idiomas](language-support.md). Para más información sobre las voces neuronales y personalizadas, consulte la [introducción de Texto a voz](text-to-speech.md).


> [!NOTE]
> Puede escuchar voces en distintos estilos y hacer que se lea texto de ejemplo mediante [la página de Text to Speech](https://azure.microsoft.com/services/cognitive-services/text-to-speech/#features).


## <a name="special-characters"></a>Caracteres especiales

Al usar SSML, recuerde que los caracteres especiales, como comillas, apóstrofos y corchetes, deben ser de escape. Para más información, consulte [Lenguaje de marcado extensible (XML) 1.0: Apéndice D](https://www.w3.org/TR/xml/#sec-entexpand).

## <a name="supported-ssml-elements"></a>Elementos SSML admitidos

Cada documento SSML se crea con los elementos SSML (o etiquetas). Estos elementos se utilizan para ajustar el tono, la prosodia, el volumen y mucho más. En las siguientes secciones se detallan cómo se utiliza cada elemento y cuándo es necesario u opcional.

> [!IMPORTANT]
> No se olvide de utilizar comillas dobles alrededor de los valores de atributo. Las normas para un XML válido y bien formado requieren que los valores de los atributos estén rodeados de comillas dobles. Por ejemplo, `<prosody volume="90">` es un elemento válido y bien formado, pero `<prosody volume=90>` no. Es posible que SSML no reconozca valores de atributos que no estén entre comillas.

## <a name="create-an-ssml-document"></a>Creación de un documento SSML

`speak` es el elemento raíz y se **necesita** para todos los documentos SSML. El elemento `speak` contiene información importante, como la versión, el idioma y la definición del vocabulario de marcado.

**Sintaxis**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="string"></speak>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `version` | Indica la versión de la especificación SSML utilizada para interpretar el marcado del documento. La versión actual es 1.0. | Obligatorio |
| `xml:lang` | Especifica el idioma del documento raíz. El valor puede contener un código de idioma en minúsculas y de dos letras (por ejemplo, `en`) o el código de idioma y el país o región en mayúscula (por ejemplo, `en-US`). | Obligatorio |
| `xmlns` | Especifica el URI del documento que define el vocabulario de marcado (los tipos de elementos y nombres de atributos) del documento SSML. El identificador URI actual es http://www.w3.org/2001/10/synthesis. | Obligatorio |

## <a name="choose-a-voice-for-text-to-speech"></a>Elección de una voz para la conversión de texto a voz

El elemento `voice` es obligatorio. Sirve para especificar la voz que se usa en la conversión de texto a voz.

**Sintaxis**

```xml
<voice name="string">
    This text will get converted into synthesized speech.
</voice>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `name` | Identifica la voz que se usa para la salida de texto a voz. Para ver una lista completa de voces compatibles, consulte [Compatibilidad con idiomas](language-support.md#text-to-speech). | Obligatorio |

**Ejemplo**

> [!NOTE]
> En este ejemplo se usa la voz `en-US-JennyNeural`. Para ver una lista completa de voces compatibles, consulte [Compatibilidad con idiomas](language-support.md#text-to-speech).

```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        This is the text that is spoken.
    </voice>
</speak>
```

## <a name="use-multiple-voices"></a>Uso de varias voces

Dentro del elemento `speak`, puede especificar varias voces para la salida de texto a voz. Estas voces pueden estar en diferentes idiomas. Para cada voz, el texto se debe encapsular en un elemento `voice`.

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `name` | Identifica la voz que se usa para la salida de texto a voz. Para ver una lista completa de voces compatibles, consulte [Compatibilidad con idiomas](language-support.md#text-to-speech). | Obligatorio |

> [!IMPORTANT]
> La característica de límite de palabra no admite varias voces. La característica de límite de palabra debe deshabilitarse para poder usar varias voces.

### <a name="disable-word-boundary"></a>Deshabilitar el límite de palabra

Dependiendo del lenguaje del SDK de Voz, establecerá la propiedad `"SpeechServiceResponse_Synthesis_WordBoundaryEnabled"` en `false` en una instancia del objeto `SpeechConfig`.

# <a name="c"></a>[C#](#tab/csharp)

Para más información, consulte <a href="/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.setproperty" target="_blank"> `SetProperty` </a>.

```csharp
speechConfig.SetProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="c"></a>[C++](#tab/cpp)

Para más información, consulte <a href="/cpp/cognitive-services/speech/speechconfig#setproperty" target="_blank"> `SetProperty` </a>.

```cpp
speechConfig->SetProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="java"></a>[Java](#tab/java)

Para más información, consulte <a href="/java/api/com.microsoft.cognitiveservices.speech.speechconfig.setproperty#com_microsoft_cognitiveservices_speech_SpeechConfig_setProperty_String_String_" target="_blank"> `setProperty` </a>.

```java
speechConfig.setProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="python"></a>[Python](#tab/python)

Para más información, consulte <a href="/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#set-property-by-name-property-name--str--value--str-" target="_blank"> `set_property_by_name` </a>.

```python
speech_config.set_property_by_name(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Para más información, consulte <a href="/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig#setproperty-string--string-" target="_blank"> `setProperty`</a>.

```javascript
speechConfig.setProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="objective-c"></a>[Objective-C](#tab/objectivec)

Para más información, consulte <a href="/objectivec/cognitive-services/speech/spxspeechconfiguration#setpropertytobyname" target="_blank"> `setPropertyTo` </a>.

```objectivec
[speechConfig setPropertyTo:@"false" byName:@"SpeechServiceResponse_Synthesis_WordBoundaryEnabled"];
```

# <a name="swift"></a>[Swift](#tab/swift)

Para más información, consulte <a href="/objectivec/cognitive-services/speech/spxspeechconfiguration#setpropertytobyname" target="_blank"> `setPropertyTo` </a>.

```swift
speechConfig!.setPropertyTo(
    "false", byName: "SpeechServiceResponse_Synthesis_WordBoundaryEnabled")
```

---

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        Good morning!
    </voice>
    <voice name="en-US-GuyNeural">
        Good morning to you too Jenny!
    </voice>
</speak>
```

## <a name="adjust-speaking-styles"></a>Ajuste de los estilos de habla

De forma predeterminada, el servicio de texto a voz sintetiza el texto mediante un estilo de habla neutro para voces neuronales. Es posible adaptar el estilo del habla para expresar diferentes emociones, como alegría, empatía y serenidad, o bien modular la voz para diferentes escenarios, como un servicio al cliente, la transmisión de noticias o un asistente de voz, mediante el elemento `mstts:express-as`. Se trata de un elemento opcional único para el servicio de voz.

Actualmente, los ajustes del estilo de habla son compatibles con las siguientes voces neuronales:
* `en-US-AriaNeural`
* `en-US-JennyNeural`
* `en-US-GuyNeural`
* `pt-BR-FranciscaNeural`
* `zh-CN-XiaoxiaoNeural`
* `zh-CN-YunyangNeural`
* `zh-CN-YunyeNeural`
* `zh-CN-YunxiNeural` (versión preliminar)
* `zh-CN-XiaohanNeural` (versión preliminar)
* `zh-CN-XiaomoNeural` (versión preliminar)
* `zh-CN-XiaoxuanNeural` (versión preliminar)
* `zh-CN-XiaoruiNeural` (versión preliminar)

La intensidad del estilo de habla se puede cambiar aún más para ajustarse mejor a su caso de uso. Puede especificar un estilo más seguro o más suave con `styledegree` para que la voz sea más expresiva o moderada. Actualmente, se admiten ajustes del estilo de habla de las voces neuronales en chino (mandarín y simplificado).

Además de ajustar los estilos de habla y el grado de estilo, también puede ajustar el parámetro `role` para que la voz parezca que es de alguien de otra edad y género. Por ejemplo, una voz masculina puede subir el tono y cambiar la entonación para que parezca femenina, pero el nombre de la voz no cambiará. Actualmente, se admiten ajustes del rol de estas voces neuronales en chino (mandarín y simplificado):
* `zh-CN-XiaomoNeural`
* `zh-CN-XiaoxuanNeural`

Los cambios anteriores se aplican en el nivel de la oración y tanto los estilos como las representaciones varían según la voz. Si no se admite un estilo o representación, el servicio devolverá la voz con la forma de habla neutra predeterminada. Los estilos y roles que se admiten para cada voz se pueden ver mediante la [API de la lista de voces](rest-text-to-speech.md#get-a-list-of-voices), o bien mediante la plataforma [Audio Content Creation](https://aka.ms/audiocontentcreation) sin código.

**Sintaxis**

```xml
<mstts:express-as style="string"></mstts:express-as>
```
```xml
<mstts:express-as style="string" styledegree="value"></mstts:express-as>
```
```xml
<mstts:express-as role="string" style="string"></mstts:express-as>
```
> [!NOTE]
> En este momento, `styledegree` solo admite voces neuronales en chino (mandarín y simplificado). `role` solo admite zh-CN-XiaomoNeural y zh-CN-XiaoxuanNeural.

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `style` | Especifica el estilo de habla. Actualmente, los estilos de habla son específicos de la voz. | Se necesita si se ajusta el estilo de habla para una voz neuronal. Si se usa `mstts:express-as`, se debe especificar el estilo. Si se proporciona un valor no válido, se omitirá este elemento. |
| `styledegree` | Especifica la intensidad del estilo de voz. **Valores aceptados**: 0,01 a 2, inclusivo. El valor predeterminado es 1, que señala la intensidad de estilo predefinida. La unidad mínima es 0,01, que da como resultado una ligera tendencia hacia el estilo de destino. Un valor de 2, como resultado, duplica la intensidad de estilo predeterminada.  | Opcional (en este momento, `styledegree` solo admite voces neuronales en chino [mandarín y simplificado]).|
| `role` | Especifica la representación del habla. La voz actuará de una edad y un sexo diferentes, pero su nombre no se cambiará.  | Opcional (en este momento, `role` solo admite zh-CN-XiaomoNeural y zh-CN-XiaoxuanNeural).|

Utilice esta tabla para determinar qué estilos de habla son compatibles para cada voz neuronal.

| Voz                   | Estilo                     | Descripción                                                 |
|-------------------------|---------------------------|-------------------------------------------------------------|
| `en-US-AriaNeural`      | `style="newscast-formal"` | Expresa un tono formal, seguro y autoritario para la difusión informativa. |
|                         | `style="newscast-casual"` | Expresa un tono versátil e informal para la difusión de noticias generales.        |
|                         | `style="narration-professional"` | Exprese un tono objetivo y profesional para la lectura de contenido        |
|                         | `style="customerservice"` | Expresa un tono amistoso y servicial para atender a clientes  |
|                         | `style="chat"`            | Expresa un tono casual y relajado                         |
|                         | `style="cheerful"`        | Expresa un tono positivo y feliz                         |
|                         | `style="empathetic"`      | Expresa un sentimiento de cuidado y comprensión.               |
| `en-US-JennyNeural`     | `style="customerservice"` | Expresa un tono amistoso y servicial para atender a clientes  |
|                         | `style="chat"`            | Expresa un tono casual y relajado                         |
|                         | `style="assistant"`       | Expresa un tono cálido y relajado para asistentes digitales    |
|                         | `style="newscast"`        | Expresa un tono versátil e informal para la difusión de noticias generales.   |
| `en-US-GuyNeural`       | `style="newscast"`        | Expresa un tono formal y profesional para narrar noticias |
| `pt-BR-FranciscaNeural` | `style="calm"`            | Expresa una actitud interesante, recolectada y compuesta al hablar. El tono, la intensidad y la prosodia son mucho más uniformes en comparación con otros tipos de voz.                                |
| `zh-CN-XiaoxiaoNeural`  | `style="newscast"`        | Expresa un tono formal y profesional para narrar noticias |
|                         | `style="customerservice"` | Expresa un tono amistoso y servicial para atender a clientes  |
|                         | `style="assistant"`       | Expresa un tono cálido y relajado para asistentes digitales    |
|                         | `style="chat"`            | Expresa un tono casual y relajado para charlar.           |
|                         | `style="calm"`            | Expresa una actitud interesante, recolectada y compuesta al hablar. El tono, la intensidad y la prosodia son mucho más uniformes en comparación con otros tipos de voz.                                |
|                         | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="sad"`             | Expresa un tono afligido, agudo, con menos intensidad y poco enérgico. Los indicadores comunes de esta emoción serían lloriqueos o gimoteos al hablar.            |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.          |
|                         | `style="affectionate"`    | Expresa un tono cálido y afectuoso, agudo y con una energía vocal alta. El hablante está en un estado que atrae la atención de su interlocutor. La "personalidad" del hablante suele ser simpática por naturaleza.          |
|                         | `style="gentle"`          | Expresa un tono dulce, educado y agradable, con un tono más grave y energía vocal.         |
|                         | `style="lyrical"`         | Expresa emociones de forma melódica y sentimental         |
| `zh-CN-YunyangNeural`   | `style="customerservice"` | Expresa un tono amistoso y servicial para atender a clientes  |
| `zh-CN-YunyeNeural`     | `style="calm"`            | Expresa una actitud interesante, recolectada y compuesta al hablar. El tono, la intensidad y la prosodia son mucho más uniformes en comparación con otros tipos de voz.    |
|                         | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="sad"`             | Expresa un tono afligido, agudo, con menos intensidad y poco enérgico. Los indicadores comunes de esta emoción serían lloriqueos o gimoteos al hablar.            |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.          |
| `zh-CN-YunxiNeural`     | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="sad"`             | Expresa un tono afligido, agudo, con menos intensidad y poco enérgico. Los indicadores comunes de esta emoción serían lloriqueos o gimoteos al hablar.            |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.    |
|                         | `style="depressed"`       | Expresa un tono melancólico y pesimista, con un timbre bajo y poca energía    |
|                         | `style="embarrassed"`     | Expresa un tono incierto y dubitativo cuando quien habla se siente incómodo   |
| `zh-CN-XiaohanNeural`   | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="sad"`             | Expresa un tono afligido, agudo, con menos intensidad y poco enérgico. Los indicadores comunes de esta emoción serían lloriqueos o gimoteos al hablar.            |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.    |
|                         | `style="embarrassed"`     | Expresa un tono vacilante e inseguro cuando quien habla se siente incómodo   |
|                         | `style="affectionate"`    | Expresa un tono cálido y afectuoso, agudo y con una energía vocal alta. El hablante está en un estado que atrae la atención de su interlocutor. La "personalidad" del hablante suele ser simpática por naturaleza.          |
|                         | `style="gentle"`          | Expresa un tono dulce, educado y agradable, con un tono más grave y energía vocal.         |
| `zh-CN-XiaomoNeural`    | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.    |
|                         | `style="depressed"`       | Expresa un tono melancólico y pesimista, con un timbre bajo y poca energía    |
|                         | `style="gentle"`          | Expresa un tono dulce, educado y agradable, con un tono más grave y energía vocal.         |
| `zh-CN-XiaoxuanNeural`  | `style="cheerful"`        | Expresa un tono animado y entusiasta, agudo y enérgico.                         |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |
|                         | `style="disgruntled"`     | Expresa desdén y un tono de queja. La voz de esta emoción muestra desagrado y desprecio.              |
|                         | `style="serious"`         | Expresa un tono estricto e imponente. A menudo, el hablante suena rígido y mucho menos relajado, con una cadencia firme.    |
|                         | `style="depressed"`       | Expresa un tono melancólico y pesimista, con un timbre bajo y poca energía    |
|                         | `style="gentle"`          | Expresa un tono dulce, educado y agradable, con un tono más grave y energía vocal.         |
| `zh-CN-XiaoruiNeural`    | `style="sad"`             | Expresa un tono afligido, agudo, con menos intensidad y poco enérgico. Los indicadores comunes de esta emoción serían lloriqueos o gimoteos al hablar.            |
|                         | `style="angry"`           | Expresa un tono enfadado y molesto, grave, con mayor intensidad y energía vocal más alta. El hablante está en un estado de ira, disgustado y ofendido.       |
|                         | `style="fearful"`         | Expresa un tono asustado y nervioso, agudo, con energía vocal alta y más velocidad. El hablante está en un estado de tensión y ansiedad.                          |

Use esta tabla para comprobar los roles admitidos y sus definiciones.

|Role                     | Descripción                |
|-------------------------|----------------------------|
|`role="Girl"`            | La voz imita a la de una chica. |
|`role="Boy"`             | La voz imita a la de un chico. |
|`role="YoungAdultFemale"`| La voz imita la de una mujer adulta joven.|
|`role="YoungAdultMale"`  | La voz imita la de un hombre adulto joven.|
|`role="OlderAdultFemale"`| La voz imita a la de una mujer adulta anciana.|
|`role="OlderAdultMale"`  | La voz imita a la de un hombre adulto anciano.|
|`role="SeniorFemale"`    | La voz imita la de una mujer anciana.|
|`role="SeniorMale"`      | La voz imita la de un hombre anciano.|


**Ejemplo**

Este fragmento de SSML ilustra cómo se utiliza el elemento `<mstts:express-as>` para cambiar el estilo de habla a `cheerful`.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        <mstts:express-as style="cheerful">
            That'd be just amazing!
        </mstts:express-as>
    </voice>
</speak>
```

Este fragmento de código SSML muestra cómo se usa el atributo `styledegree` para cambiar la intensidad del estilo de habla de XiaoxiaoNeural.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="zh-CN">
    <voice name="zh-CN-XiaoxiaoNeural">
        <mstts:express-as style="sad" styledegree="2">
            快走吧，路上一定要注意安全，早去早回。
        </mstts:express-as>
    </voice>
</speak>
```

Este fragmento de código SSML muestra cómo se usa el atributo `role` para cambiar la representación de XiaomoNeural.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="zh-CN">
    <voice name="zh-CN-XiaomoNeural">
        女儿看见父亲走了进来，问道：
        <mstts:express-as role="YoungAdultFemale" style="calm">
            “您来的挺快的，怎么过来的？”
        </mstts:express-as>
        父亲放下手提包，说：
        <mstts:express-as role="OlderAdultMale" style="calm">
            “刚打车过来的，路上还挺顺畅。”
        </mstts:express-as>
    </voice>
</speak>
```

## <a name="add-or-remove-a-breakpause"></a>Adición o supresión de una pausa

Utilice el elemento `break` para insertar las pausas entre palabras, o para evitar pausas agregadas automáticamente por el servicio de texto a voz.

> [!NOTE]
> Utilice este elemento para invalidar el comportamiento predeterminado de texto a voz de una palabra o frase si el habla sintetizada de esa palabra o frase no suena natural. Establezca `strength` en `none` para evitar una pausa prosódica, que inserta automáticamente el servicio de texto a voz.

**Sintaxis**

```xml
<break strength="string" />
<break time="string" />
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `strength` | Especifica la duración relativa de una pausa mediante uno de los valores siguientes:<ul><li>None</li><li>x-weak</li><li>weak</li><li>medium (default)</li><li>strong</li><li>x-strong</li></ul> | Opcional |
| `time` | Especifica la duración absoluta de una pausa, en segundos o milisegundos. Este valor debe ser inferior a 5000 ms. Los ejemplos de valores válidos son `2s` y `500ms` | Opcional |

| Intensidad                      | Descripción |
|-------------------------------|-------------|
| Ninguno, o si no se ha proporcionado ningún valor | 0 ms        |
| x-weak                        | 250 ms      |
| weak                          | 500 ms      |
| medio                        | 750 ms      |
| strong                        | 1000 ms     |
| x-strong                      | 1250 ms     |

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        Welcome to Microsoft Cognitive Services <break time="100ms" /> Text-to-Speech API.
    </voice>
</speak>
```
## <a name="add-silence"></a>Incorporación del silencio

Use el elemento `mstts:silence` para insertar pausas antes o después del texto, o bien entre dos oraciones adyacentes.

> [!NOTE]
>La diferencia entre `mstts:silence` y `break` es que `break` se puede agregar en cualquier lugar del texto, pero el silencio solo funciona al principio o al final del texto de entrada, o entre dos oraciones adyacentes.


**Sintaxis**

```xml
<mstts:silence  type="string"  value="string"/>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `type` | Especifica la ubicación en que se debe agregar el silencio: <ul><li>`Leading`: al principio del texto </li><li>`Tailing`: al final del texto </li><li>`Sentenceboundary`: entre oraciones adyacentes </li></ul> | Obligatorio |
| `Value` | Especifica la duración absoluta de una pausa, en segundos o milisegundos. Este valor debe ser inferior a 5000 ms. Los ejemplos de valores válidos son `2s` y `500ms` | Obligatorio |

**Ejemplo** En este ejemplo, `mtts:silence` se usa para agregar 200 ms de silencio entre dos frases.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
<voice name="en-US-AriaNeural">
<mstts:silence  type="Sentenceboundary" value="200ms"/>
If we’re home schooling, the best we can do is roll with what each day brings and try to have fun along the way.
A good place to start is by trying out the slew of educational apps that are helping children stay happy and smash their schooling at the same time.
</voice>
</speak>
```

## <a name="specify-paragraphs-and-sentences"></a>Especificación de párrafos y oraciones

Los elementos `p` y `s` se utilizan para indicar párrafos y oraciones, respectivamente. En ausencia de estos elementos, el servicio de texto a voz determina automáticamente la estructura del documento SSML.

El elemento `p` puede contener texto y los siguientes elementos: `audio`, `break`, `phoneme`, `prosody`, `say-as`, `sub`, `mstts:express-as` y `s`.

El elemento `s` puede contener texto y los siguientes elementos: `audio`, `break`, `phoneme`, `prosody`, `say-as`, `mstts:express-as` y `sub`.

**Sintaxis**

```XML
<p></p>
<s></s>
```

**Ejemplo**

```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <p>
            <s>Introducing the sentence element.</s>
            <s>Used to mark individual sentences.</s>
        </p>
        <p>
            Another simple paragraph.
            Sentence structure in this paragraph is not explicitly marked.
        </p>
    </voice>
</speak>
```

## <a name="use-phonemes-to-improve-pronunciation"></a>Uso de fonemas para mejorar la pronunciación

El elemento `ph` se utiliza para la pronunciación fonética en los documentos SSML. El elemento `ph` solo puede contener texto, no otros elementos. Proporcione siempre una voz natural como reserva.

Los alfabetos fonéticos se componen de segmentos acústicos, que se componen de letras, números o caracteres, a veces en combinación. Cada segmento acústico describe un sonido de voz único. Esto contrasta con el alfabeto latino, donde cualquier letra puede representar múltiples sonidos hablados. Tenga en cuenta las pronunciaciones diferentes de la letra "c" en las palabras "casa" y "cese" o, en inglés, las distintas pronunciaciones de la combinación de letras "th" en las palabras "thing" y "those".

> [!NOTE]
> Actualmente, la etiqueta de fonemas no se admite en estas cinco voces (et-EE-AnuNeural, ga-IE-OrlaNeural, lt-LT-OnaNeural, lv-LV-EveritaNeural y mt-MT-GarceNeural).

**Sintaxis**

```XML
<phoneme alphabet="string" ph="string"></phoneme>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `alphabet` | Especifica el alfabeto fonético que se utilizará al sintetizar la pronunciación de la cadena en el atributo `ph`. La cadena que especifica el alfabeto debe especificarse en minúsculas. Estos son los posibles alfabetos que puede especificar.<ul><li>`ipa` &ndash; <a href="https://en.wikipedia.org/wiki/International_Phonetic_Alphabet" target="_blank">Alfabeto fonético internacional</a></li><li>`sapi` &ndash; [Alfabeto fonético del servicio de voz](speech-ssml-phonetic-sets.md)</li><li>`ups` &ndash;<a href="https://documentation.help/Microsoft-Speech-Platform-SDK-11/17509a49-cae7-41f5-b61d-07beaae872ea.htm" target="_blank"> Conjunto de segmentos acústicos universal</a></li></ul><br>El alfabeto solo se aplica a `phoneme` en el elemento. | Opcional |
| `ph` | Una cadena que contiene los segmentos acústicos que especifican la pronunciación de la palabra en el elemento `phoneme`. Si la cadena especificada contiene segmentos acústicos no reconocidos, el servicio de texto a voz rechaza todo el documento SSML y no produce ninguna de las salidas de voz especificadas en el documento. | Obligatorio si usa fonemas. |

**Ejemplos**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <phoneme alphabet="ipa" ph="t&#x259;mei&#x325;&#x27E;ou&#x325;"> tomato </phoneme>
    </voice>
</speak>
```

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <phoneme alphabet="sapi" ph="iy eh n y uw eh s"> en-US </phoneme>
    </voice>
</speak>
```

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <s>His name is Mike <phoneme alphabet="ups" ph="JH AU"> Zhou </phoneme></s>
    </voice>
</speak>
```

## <a name="use-custom-lexicon-to-improve-pronunciation"></a>Uso de léxico personalizado para mejorar la pronunciación

A veces, el servicio de conversión de texto a voz no puede pronunciar correctamente una palabra. Por ejemplo, el nombre de una empresa o un término médico. Los desarrolladores pueden definir cómo se leen las entidades únicas en SSML mediante las etiquetas `phoneme` y `sub`. Sin embargo, si tiene que definir cómo se leen varias entidades, puede crear un lexicón personalizado mediante la etiqueta `lexicon`.

> [!NOTE]
> El lexicón personalizado actualmente admite la codificación UTF-8.

> [!NOTE]
> Actualmente, el léxico personalizado no se admite en estas cinco voces (et-EE-AnuNeural, ga-IE-OrlaNeural, lt-LT-OnaNeural, lv-LV-EveritaNeural y mt-MT-GarceNeural).


**Sintaxis**

```XML
<lexicon uri="string"/>
```

**Atributos**

| Atributo | Descripción                               | Obligatorio u opcional |
|-----------|-------------------------------------------|---------------------|
| `uri`     | Dirección del documento PLS externo. | Necesario.           |

**Uso**

Para definir el modo en que se leen varias entidades, puede crear un lexicón personalizado, que se almacena como archivo .xml o .pls. El siguiente es un archivo .xml de ejemplo.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0"
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="ipa" xml:lang="en-US">
  <lexeme>
    <grapheme>BTW</grapheme>
    <alias>By the way</alias>
  </lexeme>
  <lexeme>
    <grapheme> Benigni </grapheme>
    <phoneme> bɛˈniːnji</phoneme>
  </lexeme>
</lexicon>
```

El elemento `lexicon` contiene al menos un elemento `lexeme`. Cada elemento `lexeme` contiene al menos un elemento `grapheme` y uno o varios elementos `grapheme`, `alias` y `phoneme`. El elemento `grapheme` contiene texto que describe la <a href="https://www.w3.org/TR/pronunciation-lexicon/#term-Orthography" target="_blank">ortografía de </a>. Los elementos `alias` se usan para indicar la pronunciación de un acrónimo o un término abreviado. El elemento `phoneme` proporciona texto que describe cómo se pronuncia `lexeme`.

Es importante destacar que no se puede establecer directamente la pronunciación de una frase mediante el lexicón personalizado. Si tiene que configurar la pronunciación de un acrónimo o un término abreviado, primero indique un `alias` y, a continuación, asocie el `phoneme` con dicho `alias`. Por ejemplo:

```xml
  <lexeme>
    <grapheme>Scotland MV</grapheme>
    <alias>ScotlandMV</alias>
  </lexeme>
  <lexeme>
    <grapheme>ScotlandMV</grapheme>
    <phoneme>ˈskɒtlənd.ˈmiːdiəm.weɪv</phoneme>
  </lexeme>
```

También puede proporcionar directamente el `alias` esperado para el acrónimo o el término abreviado. Por ejemplo:
```xml
  <lexeme>
    <grapheme>Scotland MV</grapheme>
    <alias>Scotland Media Wave</alias>
  </lexeme>
```

> [!IMPORTANT]
> El elemento `phoneme` no puede contener espacios en blanco al usar el Alfabeto Fonético Internacional (IPA).

Para obtener más información sobre el archivo de lexicón personalizado, consulte [Pronunciation Lexicon Specification (PLS) Version 1.0](https://www.w3.org/TR/pronunciation-lexicon/).

A continuación, publique el archivo de lexicón personalizado. Si bien no existen restricciones sobre dónde se puede almacenar este archivo, se recomienda usar [Azure Blob Storage](../../storage/blobs/storage-quickstart-blobs-portal.md).

Una vez que haya publicado el lexicón personalizado, puede hacer referencia a él desde su SSML.

> [!NOTE]
> El elemento `lexicon` debe estar dentro del elemento `voice`.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
          xmlns:mstts="http://www.w3.org/2001/mstts"
          xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <lexicon uri="http://www.example.com/customlexicon.xml"/>
        BTW, we will be there probably at 8:00 tomorrow morning.
        Could you help leave a message to Robert Benigni for me?
    </voice>
</speak>
```

Al usar este lexicón personalizado, "BTW" se leerá como "By the way". "Benigni" se leerá con el IPA indicado "bɛˈniːnji".

**Limitaciones**
- Tamaño de archivo: el límite máximo del tamaño de archivo de léxico personalizado es de 100 KB. Si supera este tamaño, se producirá un error en la solicitud de síntesis.
- Actualización de la caché de léxico: el léxico personalizado se almacenará en caché con el URI como clave en el servicio TTS cuando se cargue por primera vez. El léxico con el mismo URI no se volverá a cargar durante 15 minutos, por lo que el cambio del léxico personalizado tardará un máximo de 15 minutos en aplicarse.

**Conjuntos fonéticos del servicio de voz**

En el ejemplo anterior, usamos el alfabeto fonético internacional, conocido también como conjunto de fonos IPA. Se recomienda que los desarrolladores usen este conjunto de fonos, ya que es el estándar internacional. En algunos caracteres del IPA, tienen la versión "precompuesta" y "descompuesta" cuando se representan con Unicode. El lexicón personalizado solo admite caracteres Unicode descompuestos.

Dado que este alfabeto no es fácil de recordar, el servicio de voz define un conjunto fonético para siete idiomas (`en-US`, `fr-FR`, `de-DE`, `es-ES`, `ja-JP`, `zh-CN` y `zh-TW`).

`sapi` se puede usar como valor para el atributo `alphabet` con léxicos personalizados, como se muestra a continuación:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0"
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="sapi" xml:lang="en-US">
  <lexeme>
    <grapheme>BTW</grapheme>
    <alias> By the way </alias>
  </lexeme>
  <lexeme>
    <grapheme> Benigni </grapheme>
    <phoneme> b eh 1 - n iy - n y iy </phoneme>
  </lexeme>
</lexicon>
```

Para más información sobre el alfabeto fonético del servicio de voz detallado, consulte los [conjuntos fonéticos del servicio de voz](speech-ssml-phonetic-sets.md).

## <a name="adjust-prosody"></a>Ajuste de la prosodia

El elemento `prosody` se utiliza para especificar los cambios en el tono, la curva melódica, el rango, la velocidad, la duración y el volumen de la salida de texto a voz. El elemento `prosody` puede contener texto y los siguientes elementos: `audio`, `break`, `p`, `phoneme`, `prosody`, `say-as`, `sub` y `s`.

Dado que los valores de los atributos prosódicos pueden variar en un amplio rango, el reconocedor de voz interpreta los valores asignados como una sugerencia de cuáles deben ser los valores prosódicos reales de la voz seleccionada. El servicio de texto a voz limita o sustituye valores que no son compatibles. Ejemplos de valores no compatibles son un tono de 1 MHz o un volumen de 120.

**Sintaxis**

```XML
<prosody pitch="value" contour="value" range="value" rate="value" duration="value" volume="value"></prosody>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `pitch` | Indica el tono de la línea de referencia del texto. Puede expresar el tono como:<ul><li>Un valor absoluto, expresado como un número seguido de "Hz" (Hercios). Por ejemplo, `<prosody pitch="600Hz">some text</prosody>`.</li><li>Un valor relativo, expresado como un número precedido por "+" o "-" y seguido por "Hz" o "st", que especifica una cantidad para cambiar el tono. Por ejemplo: `<prosody pitch="+80Hz">some text</prosody>` o `<prosody pitch="-2st">some text</prosody>`. La "st" indica que la unidad de cambio es un semitono, que es la mitad de un tono (medio paso) en la escala diatónica estándar.</li><li>Un valor constante:<ul><li>x-low</li><li>low</li><li>medio</li><li>high</li><li>x-high</li><li>default</li></ul></li></ul> | Opcional |
| `contour` |El contorno ahora admite voces neuronales y estándar. El contorno representa los cambios en el tono. Estos cambios se representan como una matriz de objetivos en posiciones de tiempo específicas en la salida de voz. Cada destino se define por conjuntos de pares de parámetros. Por ejemplo: <br/><br/>`<prosody contour="(0%,+20Hz) (10%,-2st) (40%,+10Hz)">`<br/><br/>El primer valor de cada conjunto de parámetros especifica la ubicación del cambio de tono como porcentaje de la duración del texto. El segundo valor especifica la cantidad para subir o bajar el tono, mediante un valor relativo o un valor de enumeración para el tono (consulte `pitch`). | Opcional |
| `range` | Un valor que representa el rango del tono para el texto. Puede expresar `range` mediante los mismos valores absolutos, valores relativos o valores de enumeración usados para describir `pitch`. | Opcional |
| `rate` | Indica la velocidad de habla del texto. Puede expresar `rate` como:<ul><li>Un valor relativo, expresado como un número que actúa como multiplicador del valor predeterminado. Por ejemplo, un valor de *1* no da como resultado ningún cambio en la velocidad. Un valor de *0,5* da como resultado la mitad de la velocidad. Un valor de *3* da como resultado el triple de la velocidad.</li><li>Un valor constante:<ul><li>x-slow</li><li>lento</li><li>medio</li><li>fast</li><li>x-fast</li><li>default</li></ul></li></ul> | Opcional |
| `duration` | El período de tiempo que debe transcurrir mientras el servicio de síntesis de voz lee el texto, en segundos o milisegundos. Por ejemplo, *2 s* o *1800 ms*. La duración solo admite voces estándar.| Opcional |
| `volume` | Indica el nivel de volumen de la voz. Puede expresar el volumen como:<ul><li>Un valor absoluto, expresado como un número en el rango de 0,0 a 100,0, de *más silencioso* a *más alto*. Por ejemplo, 75. El valor predeterminado es 100.0.</li><li>Un valor relativo, expresado como un número precedido por "+" o "-" que especifica una cantidad para cambiar el volumen. Por ejemplo, +10 o -5,5.</li><li>Un valor constante:<ul><li>silent</li><li>x-soft</li><li>soft</li><li>medio</li><li>loud</li><li>x-loud</li><li>default</li></ul></li></ul> | Opcional |

### <a name="change-speaking-rate"></a>Cambio de la velocidad de habla

La velocidad de habla puede aplicarse tanto a voces neuronales como a voces estándar en el nivel de palabra o frase.

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-GuyNeural">
        <prosody rate="+30.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-volume"></a>Cambio de volumen

Los cambios de volumen pueden aplicarse a voces estándar en el nivel de palabra o frase. Sin embargo, solo pueden aplicarse a voces neuronales en el nivel de frase.

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <prosody volume="+20.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-pitch"></a>Cambio de tono

Los cambios de tono pueden aplicarse a voces estándar en el nivel de palabra o frase. Sin embargo, solo pueden aplicarse a voces neuronales en el nivel de frase.

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        Welcome to <prosody pitch="high">Microsoft Cognitive Services Text-to-Speech API.</prosody>
    </voice>
</speak>
```

### <a name="change-pitch-contour"></a>Cambio de curva melódica

> [!IMPORTANT]
> Los cambios de curva melódica ahora son compatibles con las voces neuronales.

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        <prosody contour="(60%,-60%) (100%,+80%)" >
            Were you the only person in the room?
        </prosody>
    </voice>
</speak>
```
## <a name="say-as-element"></a>Elemento Say-as

`say-as` es un elemento opcional que indica el tipo de contenido (por ejemplo, el número o la fecha) del texto del elemento. De esta forma se proporcionan instrucciones al motor de síntesis de voz sobre cómo pronunciar el texto.

**Sintaxis**

```XML
<say-as interpret-as="string" format="digit string" detail="string"> <say-as>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `interpret-as` | Indica el tipo de contenido del texto del elemento. Para ver una lista de tipos, consulte la tabla siguiente. | Obligatorio |
| `format` | Proporciona información adicional sobre el formato preciso del texto del elemento para los tipos de contenido que pueden tener formatos ambiguos. SSML define formatos para los tipos de contenido que los usan (vea la tabla siguiente). | Opcional |
| `detail` | Indica el nivel de detalle con que se va a hablar. Por ejemplo, este atributo puede solicitar que el motor de síntesis de voz pronuncie signos de puntuación. No hay valores estándar definidos para `detail`. | Opcional |

<!-- I don't understand the last sentence. Don't we know which one Cortana uses? -->

A continuación se muestran los tipos de contenido admitidos para los atributos `interpret-as` y `format`. Incluya el atributo `format` solo si `interpret-as` está establecido en la fecha y hora.

| interpret-as | format | Interpretación |
|--------------|--------|----------------|
| `address` | | El texto se pronuncia como una dirección. El motor de síntesis de voz pronuncia:<br /><br />`I'm at <say-as interpret-as="address">150th CT NE, Redmond, WA</say-as>`<br /><br />Como "Estoy en el número 150 de Court North East Redmond, en Washington". |
| `cardinal`, `number` | | El texto se pronuncia como un número cardinal. El motor de síntesis de voz pronuncia:<br /><br />`There are <say-as interpret-as="cardinal">3</say-as> alternatives`<br /><br />Como "Hay tres alternativas". |
| `characters`, `spell-out` | | El texto se pronuncia como letras individuales (deletreadas). El motor de síntesis de voz pronuncia:<br /><br />`<say-as interpret-as="characters">test</say-as>`<br /><br />Como "T E S T". |
| `date` | dmy, mdy, ymd, ydm, ym, my, md, dm, d, m, y | El texto se pronuncia como una fecha. El atributo `format` especifica el formato de la fecha (*d=día, m=mes, y=año)* . El motor de síntesis de voz pronuncia:<br /><br />`Today is <say-as interpret-as="date" format="mdy">10-19-2016</say-as>`<br /><br />Como "Hoy es el diecinueve de octubre de 2016". |
| `digits`, `number_digit` | | El texto se pronuncia como una secuencia de dígitos individuales. El motor de síntesis de voz pronuncia:<br /><br />`<say-as interpret-as="number_digit">123456789</say-as>`<br /><br />Como "1 2 3 4 5 6 7 8 9". |
| `fraction` | | El texto se pronuncia como un número fraccionario. El motor de síntesis de voz pronuncia:<br /><br /> `<say-as interpret-as="fraction">3/8</say-as> of an inch`<br /><br />Como "tres octavos de pulgada". |
| `ordinal` | | El texto se pronuncia como un número ordinal. El motor de síntesis de voz pronuncia:<br /><br />`Select the <say-as interpret-as="ordinal">3rd</say-as> option`<br /><br />Como "Seleccionar la tercera opción". |
| `telephone` | | El texto se pronuncia como un número de teléfono. El atributo `format` puede contener dígitos que representan un código de país. Por ejemplo, "1" para Estados Unidos o "39" para Italia. El motor de síntesis de voz puede utilizar esta información para orientar la pronunciación de un número de teléfono. El número de teléfono también puede incluir el código de país y, si es así, tiene prioridad sobre el código de país de `format`. El motor de síntesis de voz pronuncia:<br /><br />`The number is <say-as interpret-as="telephone" format="1">(888) 555-1212</say-as>`<br /><br />Como "Mi número es el código de área ocho ocho ocho cinco cinco cinco uno dos uno dos". |
| `time` | hms12, hms24 | El texto se pronuncia como una hora. El atributo `format` especifica si la hora se especifica mediante un reloj de 12 horas (hms12) o de 24 horas (hms24). Use un signo de dos puntos para separar los números que representan las horas, los minutos y los segundos. Los siguientes son ejemplos de horas válidos: 12:35, 1:14:32, 08:15 y 02:50:45. El motor de síntesis de voz pronuncia:<br /><br />`The train departs at <say-as interpret-as="time" format="hms12">4:00am</say-as>`<br /><br />Como "El tren sale a las cuatro A. M.". |

**Uso**

El elemento `say-as` puede contener solo texto.

**Ejemplo**

El motor de síntesis de voz pronuncia el ejemplo siguiente como "Su primera solicitud fue de una habitación el diecinueve de octubre de dos mil diez con llegada a las diecisiete treinta y cinco".

```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <p>
        Your <say-as interpret-as="ordinal"> 1st </say-as> request was for <say-as interpret-as="cardinal"> 1 </say-as> room
        on <say-as interpret-as="date" format="mdy"> 10/19/2010 </say-as>, with early arrival at <say-as interpret-as="time" format="hms12"> 12:35pm </say-as>.
        </p>
    </voice>
</speak>
```

## <a name="add-recorded-audio"></a>Adición del audio grabado

`audio` es un elemento opcional que le permite insertar audio MP3 en un documento SSML. El cuerpo del elemento de audio puede contener texto sin formato o marcado SSML que se lee en voz alta si el archivo de audio no está disponible o no se puede reproducir. Además, el elemento `audio` puede contener texto y los siguientes elementos: `audio`, `break`, `p`, `s`, `phoneme`, `prosody`, `say-as` y `sub`.

Cualquier audio incluido en el documento SSML debe cumplir estos requisitos:

* El archivo MP3 debe estar hospedado en un punto de conexión HTTPS accesible desde Internet. Se debe usar HTTPS y el dominio que hospeda el archivo MP3 debe presentar un certificado TLS/SSL válido y de confianza.
* El archivo MP3 debe ser válido (MPEG V2).
* La velocidad de bits debe ser de 48 kbps.
* La frecuencia de muestreo debe ser de 16 000 Hz.
* El tiempo total combinado de todos los archivos de texto y audio de una sola respuesta no puede superar los noventa (90) segundos.
* El archivo MP3 no debe contener ninguna información específica del cliente ni otra que sea confidencial.

**Sintaxis**

```xml
<audio src="string"/></audio>
```

**Atributos**

| Atributo | Descripción                                   | Obligatorio u opcional                                        |
|-----------|-----------------------------------------------|------------------------------------------------------------|
| `src`     | Especifica la ubicación o la URL del archivo de audio. | Es obligatorio si se usa el elemento de audio en el documento SSML. |

**Ejemplo**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <p>
            <audio src="https://contoso.com/opinionprompt.wav"/>
            Thanks for offering your opinion. Please begin speaking after the beep.
            <audio src="https://contoso.com/beep.wav">
                Could not play the beep, please voice your opinion now.
            </audio>
        </p>
    </voice>
</speak>
```

## <a name="add-background-audio"></a>Adición de audio de fondo

El elemento `mstts:backgroundaudio` permite agregar audio de fondo a los documentos SSML (o mezclar un archivo de audio con el servicio de texto a voz). Con `mstts:backgroundaudio`, puede reproducir en bucle un archivo de audio de fondo, hacer que aparezca gradualmente al principio de la conversión de texto a voz y quitarlo gradualmente al final de la conversión de texto a voz.

Si el audio de fondo proporcionado dura menos que la conversión de texto a voz o el fundido de salida, se reproducirá en bucle. Si dura más que la conversión de texto a voz, se detendrá cuando finalice el fundido de salida.

Solo se permite un archivo de audio de fondo por cada documento SSML. Sin embargo, puede intercalar etiquetas `audio` dentro del elemento `voice` para agregar audio adicional al documento SSML.

**Sintaxis**

```XML
<mstts:backgroundaudio src="string" volume="string" fadein="string" fadeout="string"/>
```

**Atributos**

| Atributo | Descripción | Obligatorio u opcional |
|-----------|-------------|---------------------|
| `src` | Especifica la ubicación o la URL del archivo de audio de fondo. | Es obligatorio si se usa el audio de fondo en el documento SSML. |
| `volume` | Especifica el volumen del archivo de audio de fondo. **Valores aceptados**: de `0` a `100`, ambos incluidos. El valor predeterminado es `1`. | Opcional |
| `fadein` | Especifica la duración del "fundido" de entrada del audio de fondo en milisegundos. El valor predeterminado es `0`, que equivale a ningún fundido de entrada. **Valores aceptados**: de `0` a `10000`, ambos incluidos.  | Opcional |
| `fadeout` | Especifica la duración del fundido de salida del audio de fondo en milisegundos. El valor predeterminado es `0`, que equivale a ningún fundido de salida. **Valores aceptados**: de `0` a `10000`, ambos incluidos.  | Opcional |

**Ejemplo**

```xml
<speak version="1.0" xml:lang="en-US" xmlns:mstts="http://www.w3.org/2001/mstts">
    <mstts:backgroundaudio src="https://contoso.com/sample.wav" volume="0.7" fadein="3000" fadeout="4000"/>
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, JennyNeural)">
        The text provided in this document will be spoken over the background audio.
    </voice>
</speak>
```

## <a name="bookmark-element"></a>Elemento marcador

El elemento marcador permite insertar marcadores personalizados en SSML para obtener el desplazamiento de cada marcador en la secuencia de audio.
No se leerán los elementos de marcador.
El elemento de marcador se puede usar para hacer referencia a una ubicación específica en la secuencia de texto o etiqueta.

> [!NOTE]
> Por ahora, el elemento `bookmark` solo funciona para la voz `en-US-AriaNeural`.

**Sintaxis**

```xml
<bookmark mark="string"/>
```

**Atributos**

| Atributo | Descripción                                   | Obligatorio u opcional                                        |
|-----------|-----------------------------------------------|------------------------------------------------------------|
|  `mark`   | Especifica el texto de referencia del elemento `bookmark`. | Necesario. |

**Ejemplo**

Por ejemplo, puede que desee conocer el desplazamiento de tiempo de cada palabra de flores como se indica a continuación

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        We are selling <bookmark mark='flower_1'/>roses and <bookmark mark='flower_2'/>daisies.
    </voice>
</speak>
```

### <a name="get-bookmark-using-speech-sdk"></a>Obtención de un marcador mediante el SDK de voz

Puede suscribirse al evento `BookmarkReached` de SDK de voz para obtener los desplazamientos del marcador.

> [!NOTE]
> El evento `BookmarkReached` solo está disponible desde la versión 1.16.0 del SDK de voz.

Los eventos `BookmarkReached` se generan cuando los datos de audio de salida están disponibles, lo que será más rápido que la reproducción en un dispositivo de salida.

* `AudioOffset` informa del tiempo transcurrido del audio de salida entre el inicio de la síntesis y el elemento de marcador. Se mide en unidades de centenares de nanosegundos (SNP)y 10 000 SNP equivalen a 1 milisegundo.
* `Text` es el texto de referencia del elemento de marcador, que es la cadena que se establece en el atributo `mark`.

# <a name="c"></a>[C#](#tab/csharp)

Para más información, consulte <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer.bookmarkreached" target="_blank"> `BookmarkReached` </a>.

```csharp
synthesizer.BookmarkReached += (s, e) =>
{
    // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
    Console.WriteLine($"Bookmark reached. Audio offset: " +
        $"{e.AudioOffset / 10000}ms, bookmark text: {e.Text}.");
};
```

En el ejemplo de SSML anterior, el evento `BookmarkReached` se desencadenará dos veces y el resultado de la consola será
```text
Bookmark reached. Audio offset: 825ms, bookmark text: flower_1.
Bookmark reached. Audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="c"></a>[C++](#tab/cpp)

Para más información, consulte <a href="https://docs.microsoft.com/cpp/cognitive-services/speech/speechsynthesizer#bookmarkreached" target="_blank"> `BookmarkReached` </a>.

```cpp
synthesizer->BookmarkReached += [](const SpeechSynthesisBookmarkEventArgs& e)
{
    cout << "Bookmark reached. "
        // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
        << "Audio offset: " << e.AudioOffset / 10000 << "ms, "
        << "bookmark text: " << e.Text << "." << endl;
};
```

En el ejemplo de SSML anterior, el evento `BookmarkReached` se desencadenará dos veces y el resultado de la consola será
```text
Bookmark reached. Audio offset: 825ms, bookmark text: flower_1.
Bookmark reached. Audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="java"></a>[Java](#tab/java)

Para más información, consulte <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechsynthesizer.bookmarkReached#com_microsoft_cognitiveservices_speech_SpeechSynthesizer_BookmarkReached" target="_blank"> `BookmarkReached` </a>.

```java
synthesizer.BookmarkReached.addEventListener((o, e) -> {
    // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
    System.out.print("Bookmark reached. Audio offset: " + e.getAudioOffset() / 10000 + "ms, ");
    System.out.println("bookmark text: " + e.getText() + ".");
});
```

En el ejemplo de SSML anterior, el evento `BookmarkReached` se desencadenará dos veces y el resultado de la consola será
```text
Bookmark reached. Audio offset: 825ms, bookmark text: flower_1.
Bookmark reached. Audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="python"></a>[Python](#tab/python)

Para más información, consulte <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesizer#bookmark-reached" target="_blank"> `bookmark_reached` </a>.

```python
# The unit of evt.audio_offset is tick (1 tick = 100 nanoseconds), divide it by 10,000 to convert to milliseconds.
speech_synthesizer.bookmark_reached.connect(lambda evt: print(
    "Bookmark reached: {}, audio offset: {}ms, bookmark text: {}.".format(evt, evt.audio_offset / 10000, evt.text)))
```

En el ejemplo de SSML anterior, el evento `bookmark_reached` se desencadenará dos veces y el resultado de la consola será
```text
Bookmark reached, audio offset: 825ms, bookmark text: flower_1.
Bookmark reached, audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Para más información, consulte <a href="https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer#bookmarkReached" target="_blank"> `bookmarkReached`</a>.

```javascript
synthesizer.bookmarkReached = function (s, e) {
    window.console.log("(Bookmark reached), Audio offset: " + e.audioOffset / 10000 + "ms, bookmark text: " + e.text);
}
```

En el ejemplo de SSML anterior, el evento `bookmarkReached` se desencadenará dos veces y el resultado de la consola será
```text
(Bookmark reached), Audio offset: 825ms, bookmark text: flower_1.
(Bookmark reached), Audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="objective-c"></a>[Objective-C](#tab/objectivec)

Para más información, consulte <a href="https://docs.microsoft.com/objectivec/cognitive-services/speech/spxspeechsynthesizer#addbookmarkreachedeventhandler" target="_blank"> `addBookmarkReachedEventHandler` </a>.

```objectivec
[synthesizer addBookmarkReachedEventHandler: ^ (SPXSpeechSynthesizer *synthesizer, SPXSpeechSynthesisBookmarkEventArgs *eventArgs) {
    // The unit of AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to converted to milliseconds.
    NSLog(@"Bookmark reached. Audio offset: %fms, bookmark text: %@.", eventArgs.audioOffset/10000., eventArgs.text);
}];
```

En el ejemplo de SSML anterior, el evento `BookmarkReached` se desencadenará dos veces y el resultado de la consola será
```text
Bookmark reached. Audio offset: 825ms, bookmark text: flower_1.
Bookmark reached. Audio offset: 1462.5ms, bookmark text: flower_2.
```

# <a name="swift"></a>[Swift](#tab/swift)

Para obtener más información, vea <a href="https://docs.microsoft.com/swift/cognitive-services/speech/spxspeechsynthesizer#addbookmarkreachedeventhandler" target="_blank"> `addBookmarkReachedEventHandler` </a>.

---

## <a name="next-steps"></a>Pasos siguientes

* [Compatibilidad de idiomas: voces, configuraciones regionales e idiomas](language-support.md)
