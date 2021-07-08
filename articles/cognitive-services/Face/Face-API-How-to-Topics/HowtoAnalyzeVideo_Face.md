---
title: 'Ejemplo: Análisis de vídeo en tiempo real: Face'
titleSuffix: Azure Cognitive Services
description: Use el servicio Face para realizar el análisis casi en tiempo real de fotogramas procedentes de una secuencia de vídeo en directo.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 03/01/2018
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: 92bf7b7c1d2f24b7a858f6e9b786b84943651239
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109810739"
---
# <a name="example-how-to-analyze-videos-in-real-time"></a>Ejemplo: Análisis de vídeos en tiempo real

En esta guía se demostrará cómo realizar el análisis casi en tiempo real de fotogramas procedentes de una secuencia de vídeo en directo. Los componentes básicos en un sistema de este tipo son:

- Adquirir fotogramas desde un origen de vídeo
- Seleccionar los fotogramas que se van a analizar
- Enviar estos fotogramas a la API
- Consumir cada resultado del análisis que se devuelve de la llamada API

Estos ejemplos están escritos en C#, y el código puede encontrarse en GitHub aquí: [https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/).

## <a name="the-approach"></a>El enfoque

Hay varias formas de solucionar el problema de la ejecución del análisis casi en tiempo real en secuencias de vídeo. Empezaremos por esbozar estos tres enfoques en niveles cada vez mayores de sofisticación.

### <a name="a-simple-approach"></a>Un enfoque sencillo

El diseño más sencillo para un sistema de análisis casi en tiempo real es un bucle infinito, donde en cada iteración se toma un fotograma, se analiza y, luego, se consume el resultado:

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        AnalysisResult r = await Analyze(f);
        ConsumeResult(r);
    }
}
```

Si nuestro análisis consistió en un algoritmo de cliente ligero, este enfoque sería adecuado. Sin embargo, cuando el análisis se realiza en la nube, la latencia que interviene significa que una llamada API podría tardar varios segundos. Durante este tiempo, no se van a capturar imágenes y el subproceso será básicamente no hacer nada. La velocidad de fotogramas máxima está limitada por la latencia de las llamadas API.

### <a name="parallelizing-api-calls"></a>Paralelización de llamadas API

Si bien un bucle simple de un único subproceso tiene sentido para un algoritmo de cliente ligero, no se ajusta bien a la latencia involucrada en las llamadas API de la nube. La solución a este problema es permitir que las llamadas API de larga ejecución se ejecuten en paralelo con la captura de fotogramas. En C#, se podría lograr esto usando paralelismo basado en tareas, por ejemplo:

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        var t = Task.Run(async () => 
        {
            AnalysisResult r = await Analyze(f);
            ConsumeResult(r);
        }
    }
}
```

Este código inicia cada análisis en una tarea independiente, que se puede ejecutar en segundo plano mientras se siguen capturando fotogramas nuevos. Con este método se evita el bloqueo del subproceso principal mientras se espera a que se devuelva una llamada API, pero se pierden algunas de las garantías que proporciona la versión simple. Varias llamadas API podrían tener lugar en paralelo y es posible que los resultados se devuelvan en el orden equivocado. Además, esto podría provocar que varios subprocesos ingresaran en la función ConsumeResult() al mismo tiempo, lo que podría ser peligroso si la función no es segura para subprocesos. Por último, este código simple no realiza un seguimiento de las tareas que se crean, por lo que las excepciones desaparecerán de manera silenciosa. Por lo tanto, el paso final es agregar un subproceso "consumidor" que realice un seguimiento de las tareas de análisis, produzca excepciones, elimine tareas de larga duración y garantice que los resultados se consuman en el orden correcto.

### <a name="a-producer-consumer-design"></a>Un diseño de productor-consumidor

En el sistema final de "productor-consumidor", se cuenta con un subproceso productor que es similar al bucle infinito anterior. No obstante, en lugar de consumir los resultados del análisis en cuanto están disponibles, el productor simplemente coloca las tareas en una cola para realizarles un seguimiento.

```csharp
// Queue that will contain the API call tasks. 
var taskQueue = new BlockingCollection<Task<ResultWrapper>>();
     
// Producer thread. 
while (true)
{
    // Grab a frame. 
    Frame f = GrabFrame();
 
    // Decide whether to analyze the frame. 
    if (ShouldAnalyze(f))
    {
        // Start a task that will run in parallel with this thread. 
        var analysisTask = Task.Run(async () => 
        {
            // Put the frame, and the result/exception into a wrapper object.
            var output = new ResultWrapper(f);
            try
            {
                output.Analysis = await Analyze(f);
            }
            catch (Exception e)
            {
                output.Exception = e;
            }
            return output;
        }
        
        // Push the task onto the queue. 
        taskQueue.Add(analysisTask);
    }
}
```

También se dispone de un subproceso consumidor, que toma tareas de la cola, espera a que finalicen y muestra el resultado, o bien genera la excepción que se produjo. Mediante el uso de la cola, es posible garantizar que resultados se consuman uno a la vez, en el orden correcto, sin limitar la velocidad de fotogramas máxima del sistema.

```csharp
// Consumer thread. 
while (true)
{
    // Get the oldest task. 
    Task<ResultWrapper> analysisTask = taskQueue.Take();
 
    // Await until the task is completed. 
    var output = await analysisTask;
     
    // Consume the exception or result. 
    if (output.Exception != null)
    {
        throw output.Exception;
    }
    else
    {
        ConsumeResult(output.Analysis);
    }
}
```

## <a name="implementing-the-solution"></a>Implementación de la solución

### <a name="getting-started"></a>Introducción

Para tener su aplicación en funcionamiento lo antes posible, usará una implementación flexible del sistema que se ha descrito anteriormente. Para obtener acceso al código, vaya a [https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis).

La biblioteca contiene la clase FrameGrabber, que implementa el sistema productor-consumidor descrito anteriormente para procesar fotogramas de vídeo desde una cámara web. El usuario puede especificar la forma exacta de la llamada API, y la clase usa eventos para permitir que el código de llamada sepa cuándo se adquiere un nuevo fotograma, o cuándo hay disponible un nuevo resultado del análisis.

Para ilustrar algunas de las posibilidades, hay dos aplicaciones de ejemplo que utilizan la biblioteca. La primera es una aplicación de consola simple; una versión simplificada de ella se reproduce a continuación. Toma fotogramas de la cámara web predeterminada y los envía al servicio Face para la detección de caras.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/analyze.cs":::

La segunda aplicación de ejemplo es un poco más interesante y le permite elegir a qué API llamar en los fotogramas de vídeo. En el lado izquierdo, la aplicación muestra una vista previa del vídeo en directo; en el lado derecho, muestra el resultado más reciente de la API superpuesto con el fotograma correspondiente.

En la mayoría de los modos, habrá un retraso visible entre el vídeo en directo a la izquierda y el análisis visualizado a la derecha. Este retraso es el tiempo necesario para realizar la llamada API. Una excepción es el modo "EmotionsWithClientFaceDetect", que realiza la detección de caras localmente en el equipo cliente mediante OpenCV, antes de enviar imágenes a Cognitive Services. De esta manera, es posible visualizar la cara detectada inmediatamente y, luego, actualizar las emociones una vez que se devuelve la llamada API. Este es un ejemplo de un enfoque "híbrido", donde el cliente puede realizar algún procesamiento simple y Cognitive Services APIs puede aumentarlo con análisis más avanzados, cuando sea necesario.

![HowToAnalyzeVideo](../../Video/Images/FramebyFrame.jpg)

### <a name="integrating-into-your-codebase"></a>Integración en el código base

Para empezar a trabajar con este ejemplo, siga estos pasos:

1. Cree una [cuenta de Azure](https://azure.microsoft.com/free/cognitive-services/). Si ya tiene una, puede pasar a la siguiente sección.
2. Cree recursos para Computer Vision y Face en Azure Portal para obtener la clave y el punto de conexión. Asegúrese de seleccionar el nivel gratis (F0) durante la instalación.
   - [Computer Vision](https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision)
   - [Face](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) Después de implementar los recursos, haga clic en **Ir al recurso** para recopilar la clave y el punto de conexión de cada recurso. 
3. Clone el repositorio [Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) de GitHub.
4. Abra el ejemplo en Visual Studio y compile y ejecute las aplicaciones de ejemplo:
    - Para BasicConsoleSample, la clave de Face está codificada de forma rígida directamente en [BasicConsoleSample/Program.cs](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/blob/master/Windows/BasicConsoleSample/Program.cs).
    - Para LiveCameraSample, las claves se deben escribir en el panel de configuración de la aplicación. Se conservarán de una sesión a otra como datos de usuario.
        

Cuando esté listo para la integración, **haga referencia a la biblioteca VideoFrameAnalyzer desde sus propios proyectos**. 

## <a name="summary"></a>Resumen

En esta guía, aprendió a ejecutar análisis casi en tiempo real en secuencias de vídeo en directo mediante Face API, Computer Vision API y Emotion API, y a usar el código de ejemplo para empezar a trabajar.

No dude en enviar sus comentarios y sugerencias al [repositorio de GitHub](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) o, para enviar comentarios más amplios sobre la API, a nuestro sitio de [UserVoice](https://feedback.azure.com/forums/932041-azure-cognitive-services?category_id=395743).

## <a name="related-topics"></a>Temas relacionados
- [Detección de caras en imágenes](HowtoDetectFacesinImage.md)
