---
title: Preguntas más frecuentes
description: Preguntas frecuentes sobre el servicio Azure Object Anchors.
author: craigktreasure
manager: vriveras
ms.author: crtreasu
ms.date: 04/01/2020
ms.topic: overview
ms.service: azure-object-anchors
ms.openlocfilehash: 31b46475af345ec4eed3a8d5787bf859918048ef
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111985857"
---
# <a name="frequently-asked-questions-about-azure-object-anchors"></a>Preguntas más frecuentes sobre Azure Object Anchors

Azure Object Anchors permite a una aplicación detectar un objeto en el mundo físico mediante un modelo 3D y estimar su posición 6-DoF (seis grados de libertad).

Para obtener más información, consulte [Introducción a Azure Object Anchors](overview.md).

## <a name="product-faq"></a>Preguntas más frecuentes del producto
**P: ¿Qué recomendaciones hay para los objetos que se deben usar?**

**R:** Se recomiendan las siguientes propiedades para los objetos:

* De 1 a 10 metros para cada dimensión.
* No simétrica, con suficientes variaciones en la geometría.
* Baja reflectividad (superficies mate) con color brillante.
* Objetos estacionarios.
* Cantidades de articulación pequeñas o nulas.
* Fondos claros con un desorden mínimo o nulo.
* El objeto analizado debe tener una coincidencia 1:1 con el modelo con que se entrenó.

**P: ¿Cuáles son las dimensiones de objeto máximas que se pueden procesar para la conversión del modelo?**

**R:** Cada dimensión de un modelo CAD debe tener menos de 10 metros. Para más información, consulte los [requisitos de recursos](overview.md).

**P: ¿Cuál es el tamaño máximo del modelo CAD que se puede procesar para la conversión?**

**R:** El tamaño de archivo del modelo debe ser inferior a 150 MB. Para más información, consulte los [requisitos de recursos](overview.md).

**P: ¿Qué formatos CAD se admiten?**

**R:** Actualmente se admiten los tipos de archivo `fbx`, `ply`, `obj`, `glb` y `gltf`. Para más información, consulte los [requisitos de recursos](overview.md).

**P: ¿Cuál es la dirección de la gravedad y la unidad que requiere el servicio de conversión del modelo? ¿Cómo se pueden averiguar?**

**R:** La dirección de la gravedad es el vector descendente que apunta hacia la tierra. En el caso de los modelos CAD, la dirección de la gravedad suele ser la opuesta a una dirección ascendente. Por ejemplo, en muchos casos +Z representa arriba, en cuyo caso, -Z o `Vector3(0.0, 0.0, -1.0)` representarían la dirección de la gravedad. Al determinar la gravedad, no solo debe considerar el modelo, sino también la orientación en la que se verá el modelo durante el tiempo de ejecución. Si está intentando detectar una silla en el mundo real, en una superficie plana, la gravedad podría ser `Vector3(0.0, 0.0, -1.0)`. Sin embargo, si la silla está en una pendiente de 45 grados, la gravedad podría ser `Vector3(0.0, -Sqrt(2)/2, -Sqrt(2)/2)`.

La dirección de la gravedad se puede calcular con una herramienta de representación 3D, como [MeshLab](http://www.meshlab.net/).

La unidad representa la unidad de medida del modelo. Las unidades admitidas se pueden encontrar mediante la enumeración **Microsoft.Azure.ObjectAnchors.Conversion.AssetLengthUnit**.

**P: ¿Cuánto tiempo se tarda en convertir un modelo CAD?**

**R:** Para un modelo `ply`, normalmente entre 3 y 15 minutos. Si envía modelos en otros formatos, espere de 15 a 60 minutos según el tamaño de archivo.

**P: ¿Cómo me recupero de un error de conversión de modelos?**

**R:** Para más información sobre los distintos códigos de error que pueden ser el resultado de un trabajo de conversión de modelos con errores y cómo controlar cada uno de ellos, consulte la [página de códigos de error de conversión](.\model-conversion-error-codes.md).

**P: ¿Qué dispositivos admite Object Anchors?**

**A:** HoloLens 2.

**P: ¿Qué compilación del sistema operativo debe ejecutar mi HoloLens?**

**R:** Versión de SO 18363.720 o posterior, publicada después del 12 de marzo de 2020.

  Más detalles en la [actualización Windows 10, del 12 de marzo de 2020](https://support.microsoft.com/help/4551762).

**P: ¿Cuánto tiempo se tarda en detectar un objeto en HoloLens?**

**R:** Depende del tamaño del objeto y del proceso de examen. Para obtener una detección más rápida, pruebe a seguir los procedimientos recomendados para un examen exhaustivo. Para los objetos con una distancia de menos de 2 metros en cada dimensión, la detección puede producirse en pocos segundos. En el caso de objetos grandes, como un automóvil, el usuario debe recorrer un bucle completo alrededor del objeto para obtener una detección confiable, lo que significa que la detección podría tardar decenas de segundos.

**P: ¿Cuáles son los procedimientos recomendados al usar Object Anchors en una aplicación de HoloLens?**

**R:**

 1. Realice una calibración ocular para obtener una representación precisa.
 2. Asegúrese de que la sala tenga una textura visual enriquecida y una buena iluminación.
 3. Mantenga inmóvil el objeto, y alejado del desorden si es posible.
 4. De manera opcional, borre la memoria caché de [asignación espacial](/windows/mixed-reality/spatial-mapping) en el dispositivo HoloLens.
 5. Desplácese alrededor del objeto para examinarlo. Asegúrese de que se observe la mayor parte del objeto.
 6. Establezca un área de búsqueda suficientemente grande para cubrir el objeto.
 7. El objeto debe permanecer estacionario durante la detección.
 8. Inicie la detección del objeto y visualice la representación basada en la posición estimada.
 9. Para ahorrar batería, bloquee el objeto detectado o detenga el seguimiento una vez que la posición esté estable y sea precisa.

**P: ¿Qué versión de Mixed Reality Toolkit (MRTK) debe usar mi aplicación HoloLens de Unity para poder trabajar con el SDK de Unity para Object Anchors?**

**A:** El SDK de Unity para Azure Object Anchors no tiene ninguna clase de dependencia de Mixed Reality Toolkit, lo que significa que puede usar la versión que prefiera. Para más información, consulte [Introducción a MRTK para Unity](/windows/mixed-reality/develop/unity/mrtk-getting-started).

**P: ¿Cuál es la precisión de una posición calculada?**

**R:** Depende del tamaño del objeto, del material, del entorno, etc. Para objetos pequeños, la posición estimada puede tener un error de 2 cm. En el caso de objetos grandes, como un automóvil, el error puede ser de hasta 2 a 8 cm.

**P: ¿Sirve Object Anchors para objetos en movimiento?**

**R:** No se admiten objetos en **constante movimiento** ni **dinámicos**.

**P: ¿Puede Object Anchors controlar deformaciones o articulaciones?**

**R:** Parcialmente, según cuánto cambie la forma o la geometría del objeto debido a una deformación o articulación. Si la geometría del objeto cambia mucho, el usuario puede crear otro modelo para esa configuración y usarlo para la detección.

**P: ¿Cuántos objetos diferentes puede detectar Object Anchors al mismo tiempo?**

**R:** Actualmente se admite la detección de un único modelo de objetos a la vez.

**P: ¿Puede Object Anchors detectar varias instancias del mismo modelo de objetos?**

**R:** Sí, se pueden detectar hasta 3 objetos del mismo tipo de modelo. La aplicación puede llamar a `ObjectObserver.DetectAsync` varias veces con diferentes consultas para detectar varias instancias del mismo modelo.

**P: ¿Qué debo hacer si el entorno de tiempo de ejecución de Object Anchors no puede detectar el objeto?**

**R:**

* Asegúrese de que la sala tenga suficientes texturas; para ello, puede agregar algunos pósteres.
* Examine el objeto de manera más exhaustiva.
* Ajuste los parámetros del modelo como se describe a continuación.
* Proporcione un rectángulo delimitador estrecho como área de búsqueda, que incluya la totalidad o la mayor parte del objeto.
* Borre la memoria caché de asignación espacial y vuelva a examinar el objeto.
* Capture el diagnóstico y envíenos los datos.
* Ajuste la propiedad `MinSurfaceCoverage` de la clase `ObjectQuery`. Para más información, consulte [Detección de un objeto difícil](detect-difficult-object.md).

**P: ¿Cómo elegir los parámetros de la consulta de objeto?**

**R:**

* Ofrezca áreas de búsqueda estrechas para cubrir el objeto en la mayor medida posible a los efectos de mejorar la velocidad y la precisión de la detección.
* El valor predeterminado `ObjectQuery.MinSurfaceCoverage` del modelo de objetos suele ser una buena opción; de lo contrario, use un valor menor para obtener una detección más rápida.
* Use un valor pequeño para `ObjectQuery.ExpectedMaxVerticalOrientationInDegrees` si se espera que el objeto esté de pie.
* Una aplicación siempre debe usar un modelo de objetos `1:1` para la detección. De manera ideal, la escala estimada debe estar cerca de 1, con un error del 1 %. Una aplicación podría establecer `ObjectQuery.MaxScaleChange` en `0` o `0.1` para deshabilitar o habilitar la estimación de escala, y evaluar cualitativamente la posición de la instancia.
* Para más información, consulte [Detección de un objeto difícil](detect-difficult-object.md).

**P: ¿Cómo se obtienen los datos de diagnóstico de Object Anchors desde HoloLens?**

**R:** La aplicación puede especificar la ubicación de los archivos de diagnóstico. La aplicación de ejemplo de Object Anchors escribe los datos de diagnóstico en la carpeta **TempState**

**P: ¿Por qué el modelo de origen no se alinea con el objeto físico cuando se usa la posición que devuelve el SDK de Unity de Object Anchors?**

**R:** Unity puede cambiar el sistema de coordenadas al importar un modelo de objetos. Por ejemplo, el SDK de Unity de Object Anchors invierte el eje Z al convertir de un sistema de coordenadas de la mano derecha a la izquierda, pero Unity puede aplicar una rotación adicional sobre el eje X o Y. Un desarrollador puede establecer esta rotación adicional visualizando y comparando los sistemas de coordenadas.

**P: ¿Se admiten 2D?**

**R:** Puesto que se basa en geometría, solo se admiten 3D.

**P: ¿Se diferencian modelos iguales con colores diferentes?**

**R:** Dado que nuestros algoritmos se basan en geometría, los distintos colores del mismo modelo no se comportarán de manera diferente durante la detección.

**P: ¿Puedo usar Object Anchors sin conectividad a Internet?**

**R:** 
* Para la conversión y el entrenamiento de modelos, se requiere conectividad, ya que esto sucede en la nube.
* Las sesiones en tiempo de ejecución se producen totalmente en el dispositivo y no requieren conectividad, ya que todos los cálculos tienen lugar en HoloLens 2.

## <a name="privacy-faq"></a>Preguntas más frecuentes sobre privacidad
**P: ¿Cómo almacena los datos Azure Object Anchors?**

**R:** Solo almacenamos metadatos del sistema, que se cifran en reposo con una clave de cifrado de datos administrados por Microsoft.