---
title: Mitigación de la latencia cuando se usa el servicio Face
titleSuffix: Azure Cognitive Services
description: Obtenga información acerca de cómo mitigar la latencia cuando se usa el servicio Face.
services: cognitive-services
manager: chrhoder
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 1/5/2021
ms.openlocfilehash: a306883573387a2a5c20a53c7015c6dbd3eddf65
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107878675"
---
# <a name="how-to-mitigate-latency-when-using-the-face-service"></a>Mitigación de la latencia cuando se usa el servicio Face

Es posible encontrar latencia cuando se usa el servicio Face. La latencia hace referencia a cualquier tipo de retraso que se produce al comunicarse a través de una red. En general, entre las posibles causas de latencia se incluyen las siguientes:
- Distancia física que cada paquete debe viajar desde el origen hasta el destino.
- Problemas con el medio de transmisión.
- Errores en enrutadores o conmutadores a lo largo de la ruta de transmisión.
- Tiempo necesario para que las aplicaciones antivirus, los firewalls y otros mecanismos de seguridad inspeccionen los paquetes.
- Funcionamiento incorrecto de aplicaciones en el lado cliente o servidor.

En este tema se tratan las posibles causas de latencia específicas del uso de Azure Cognitive Services y cómo puede mitigarlas.

> [!NOTE]
> Azure Cognitive Services no proporciona ningún Acuerdo de Nivel de Servicio (SLA) con respecto a la latencia.

## <a name="possible-causes-of-latency"></a>Posibles causas de latencia

### <a name="slow-connection-between-the-cognitive-service-and-a-remote-url"></a>Conexión lenta entre la instancia de Cognitive Services y una dirección URL remota

Algunas instancias de Azure Cognitive Services proporcionan métodos que obtienen datos de una dirección URL remota especificada. Por ejemplo, al llamar al [método DetectWithUrlAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync#Microsoft_Azure_CognitiveServices_Vision_Face_FaceOperationsExtensions_DetectWithUrlAsync_Microsoft_Azure_CognitiveServices_Vision_Face_IFaceOperations_System_String_System_Nullable_System_Boolean__System_Nullable_System_Boolean__System_Collections_Generic_IList_System_Nullable_Microsoft_Azure_CognitiveServices_Vision_Face_Models_FaceAttributeType___System_String_System_Nullable_System_Boolean__System_String_System_Threading_CancellationToken_) del servicio Face, puede especificar la dirección URL de una imagen en la que el servicio intenta detectar caras.

```csharp
var faces = await client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg");
```

A continuación, el servicio Face debe descargar la imagen del servidor remoto. Si la conexión del servicio Face al servidor remoto es lenta, el tiempo de respuesta del método de detección se verá afectado.

Para mitigar esto, considere la posibilidad de [almacenar la imagen en Azure Premium Blob Storage](../../../storage/blobs/storage-upload-process-images.md?tabs=dotnet). Por ejemplo:

``` csharp
var faces = await client.Face.DetectWithUrlAsync("https://csdx.blob.core.windows.net/resources/Face/Images/Family1-Daughter1.jpg");
```

### <a name="large-upload-size"></a>Carga de gran tamaño

Algunas instancias de Azure Cognitive Services proporcionan métodos que obtienen datos de un archivo cargado. Por ejemplo, al llamar al [método DetectWithStreamAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync#Microsoft_Azure_CognitiveServices_Vision_Face_FaceOperationsExtensions_DetectWithStreamAsync_Microsoft_Azure_CognitiveServices_Vision_Face_IFaceOperations_System_IO_Stream_System_Nullable_System_Boolean__System_Nullable_System_Boolean__System_Collections_Generic_IList_System_Nullable_Microsoft_Azure_CognitiveServices_Vision_Face_Models_FaceAttributeType___System_String_System_Nullable_System_Boolean__System_String_System_Threading_CancellationToken_) del servicio Face, puede cargar una imagen en la que el servicio intenta detectar caras.

```csharp
using FileStream fs = File.OpenRead(@"C:\images\face.jpg");
System.Collections.Generic.IList<DetectedFace> faces = await client.Face.DetectWithStreamAsync(fs, detectionModel: DetectionModel.Detection02);
```

Si el archivo que se va a cargar es grande, el tiempo de respuesta del método `DetectWithStreamAsync` se verá afectado por los motivos siguientes:
- Se tarda más tiempo en cargar el archivo.
- El servicio tarda más tiempo en procesar el archivo, en proporción con el tamaño del archivo.

Mitigaciones:
- Considere la posibilidad de [almacenar la imagen en Azure Premium Blob Storage](../../../storage/blobs/storage-upload-process-images.md?tabs=dotnet). Por ejemplo:
``` csharp
var faces = await client.Face.DetectWithUrlAsync("https://csdx.blob.core.windows.net/resources/Face/Images/Family1-Daughter1.jpg");
```
- Considere la posibilidad de cargar un archivo más pequeño.
    - Consulte las instrucciones relativas a los [datos de entrada para la detección de caras](../concepts/face-detection.md#input-data) y los [datos de entrada para reconocimiento facial](../concepts/face-recognition.md#input-data).
    - En la detección de caras, cuando se usa el modelo de detección `DetectionModel.Detection01`, reducir el tamaño del archivo de imagen aumentará la velocidad de procesamiento. Al usar el modelo de detección `DetectionModel.Detection02`, reducir el tamaño del archivo de imagen solo aumentará la velocidad de procesamiento si el archivo de imagen es inferior a 1920 x 1080.
    - Para el reconocimiento facial, reducir el tamaño de la cara a 200 x 200 píxeles no afecta a la precisión del modelo de reconocimiento.
    - El rendimiento de los métodos `DetectWithUrlAsync` y `DetectWithStreamAsync` también depende de cuántas caras haya en una imagen. El servicio Face puede devolver hasta 100 caras de una imagen. Las caras se clasifican por tamaño de rectángulo facial de grande a pequeño.
    - Si necesita llamar a varios métodos de servicio, considere la posibilidad de llamarlos en paralelo si el diseño de la aplicación lo permite. Por ejemplo, si necesita detectar caras en dos imágenes para realizar una comparación de caras:
```csharp
var faces_1 = client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg");
var faces_2 = client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1NDY3OTIxMzExNzM3NjE3/john-f-kennedy---debating-richard-nixon.jpg");
Task.WaitAll (new Task<IList<DetectedFace>>[] { faces_1, faces_2 });
IEnumerable<DetectedFace> results = faces_1.Result.Concat (faces_2.Result);
```

### <a name="slow-connection-between-your-compute-resource-and-the-face-service"></a>Conexión lenta entre el recurso de proceso y el servicio Face

Si el equipo tiene una conexión lenta con el servicio Face, el tiempo de respuesta de los métodos de servicio se verá afectado.

Mitigaciones:
- Al crear la suscripción de Face, asegúrese de elegir la región más cercana a la ubicación en la que se hospede su aplicación.
- Si necesita llamar a varios métodos de servicio, considere la posibilidad de llamarlos en paralelo si el diseño de la aplicación lo permite. Vea la sección anterior para obtener un ejemplo.
- Si las latencias largas afectan al usuario, elija un umbral de tiempo de espera (por ejemplo, un máximo de 5 s) antes de volver a intentar la llamada API.

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha obtenido información acerca de cómo mitigar la latencia cuando se usa el servicio Face. A continuación, aprenderá a escalar verticalmente a partir de los objetos existentes PersonGroup y FaceList a los objetos LargePersonGroup y LargeFaceList respectivamente.

> [!div class="nextstepaction"]
> [Ejemplo: Usar la característica a gran escala](how-to-use-large-scale.md)

## <a name="related-topics"></a>Temas relacionados

- [Documentación de referencia (REST)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Documentación de referencia (SDK para .NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)
