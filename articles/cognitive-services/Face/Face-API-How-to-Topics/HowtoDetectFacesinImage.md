---
title: 'Detección de caras en una imagen: Face'
titleSuffix: Azure Cognitive Services
description: Esta guía muestra cómo usar la detección de caras para extraer atributos como el sexo, la edad o la postura de una imagen determinada.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: 71e98b735b4aa4631d73f8730a48c56a8c7585ab
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107497649"
---
# <a name="get-face-detection-data"></a>Obtención de los datos de detección de caras

Esta guía muestra cómo usar la detección de caras para extraer atributos como el sexo, la edad o la postura de una imagen determinada. Los fragmentos de código de esta guía están escritos en C# con la biblioteca cliente Face de Azure Cognitive Services. La misma funcionalidad está disponible mediante la [API REST](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

Esta guía le muestra cómo:

- Obtener las ubicaciones y las dimensiones de las caras de una imagen.
- Obtener las ubicaciones de diversos puntos de referencia faciales, como las pupilas, nariz y boca, de una imagen.
- Adivinar el sexo, edad, emoción y otros atributos de una cara detectada.

## <a name="setup"></a>Configurar

En esta guía se da por supuesto que ya ha construido un objeto [FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) llamado `faceClient`, con una clave de suscripción y una dirección URL de punto de conexión de Face. A partir de aquí, puede usar la característica de detección de caras mediante una llamada a [DetectWithUrlAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync), que se usa en esta guía o una llamada a [DetectWithStreamAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync). Para obtener instrucciones sobre cómo configurar esta característica, siga uno de los inicios rápidos.

Esta guía se centra en los detalles de la llamada de detección, como los argumentos que se pueden pasar y lo que puede hacer con los datos devueltos. Se recomienda consultar solo las características que necesita. Cada operación tarda más tiempo en completarse.

## <a name="get-basic-face-data"></a>Obtención de los datos básicos de la cara

Para encontrar caras y obtener sus ubicaciones en una imagen, llame al método [DetectWithUrlAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync) o [DetectWithStreamAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync) con el parámetro _returnFaceId_ establecido en **true**. Esta es la configuración predeterminada.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic1":::

Puede consultar los objetos [DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface) devueltos para obtener sus identificadores únicos y un rectángulo que proporciona las coordenadas en píxeles de la cara.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic2":::

Para obtener información sobre cómo analizar la ubicación y las dimensiones de la cara, consulte [FaceRectangle](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.facerectangle). Normalmente, este rectángulo contiene los ojos, las cejas, la nariz y la boca. La parte superior de la cabeza, las orejas y la barbilla no están incluidas necesariamente. Para usar el rectángulo facial para recortar una cabeza completa u obtener un retrato intermedio, quizás para una imagen de tipo foto de carnet, puede expandir el rectángulo en cada dirección.

## <a name="get-face-landmarks"></a>Obtención de los puntos de referencia faciales

Los [puntos de referencia faciales](../concepts/face-detection.md#face-landmarks) son un conjunto de puntos fáciles de encontrar en una cara como, por ejemplo, las pupilas o la punta de la nariz. Para obtener los datos de los puntos de referencia faciales, establezca el parámetro _detectionModel_ en **DetectionModel.Detection01** y el parámetro _returnFaceLandmarks_ en **true**.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks1":::

El código siguiente muestra cómo podría recuperar las ubicaciones de la nariz y las pupilas:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks2":::

También puede usar los datos de los puntos de referencia faciales para calcular con precisión la dirección de la cara. Por ejemplo, se puede definir la rotación de la cara como un vector desde el centro de la boca hasta el centro de los ojos. El código siguiente calcula este vector:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="direction":::

Cuando conoce la dirección de la cara, puede girar el marco rectangular de la cara para alinearlo más correctamente. Para recortar las caras de una imagen, puede girar mediante programación la imagen para que las caras siempre aparezcan verticales.

## <a name="get-face-attributes"></a>Obtención de los atributos de la cara

Además de los rectángulos y los puntos de referencia faciales, la API de detección de caras puede analizar varios atributos conceptuales de una cara. Para obtener una lista completa, consulte la sección conceptual [Atributos de una cara](../concepts/face-detection.md#attributes).

Para analizar los atributos de una cara, establezca el parámetro _detectionModel_ en **DetectionModel.Detection01** y el parámetro _returnFaceAttributes_ en una lista de valores [FaceAttributeType Enum](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype).

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes1":::

A continuación, puede obtener referencias a los datos devueltos y realizar más operaciones según sus necesidades.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes2":::

Para más información acerca de cada uno de los atributos, consulte la guía conceptual [Detección de caras y atributos](../concepts/face-detection.md).

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido cómo utilizar las diversas funcionalidades de la detección de caras. A continuación, integre estas características en una aplicación para agregar datos faciales de los usuarios.

- [Tutorial: Incorporación de usuarios a un servicio de Face](../enrollment-overview.md)

## <a name="related-topics"></a>Temas relacionados

- [Documentación de referencia (REST)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Documentación de referencia (SDK para .NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)
