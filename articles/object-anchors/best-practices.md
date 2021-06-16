---
title: Prácticas recomendadas
description: Procedimientos recomendados para obtener resultados mejorados
author: ariye
ms.author: crtreasu
ms.date: 03/12/2021
ms.topic: best-practice
ms.service: azure-object-anchors
ms.openlocfilehash: 0a092b45b341af37e4251951b06d1211fa6c600a
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983805"
---
# <a name="best-practices"></a>Procedimientos recomendados

Se recomienda probar algunos de estos pasos para obtener los mejores resultados.

## <a name="conversion"></a>Conversión

- Compruebe las dimensiones de los objetos físicos. Azure Object Anchors funciona mejor con los objetos cuya dimensión más pequeña se encuentra en el intervalo recomendado de 1 a 10 metros.
- Inspeccione el modelo 3D en un programa de software como [**MeshLab**](https://www.meshlab.net/) para obtener los detalles siguientes.
  - Asegúrese de que el modelo 3D tenga una malla de triángulo y que los triángulos de la superficie exterior miren hacia afuera. Es decir, los vértices deben estar orientados para que las normales sigan la regla de la mano derecha en su orientación hacia afuera.
  - Asegúrese de que el modelo 3D se especifique con las unidades de escala correctas con respecto a los objetos físicos. Las unidades deben ser: ***centímetros, decímetros, pies, pulgadas, kilómetros, metros, milímetros, yardas***.
  - Confirme la dirección de la gravedad nominal que corresponde a la orientación vertical del objeto en el mundo real. Si la gravedad o vertical descendente del objeto es -Y, use * **(0, -1, 0)** _ o _*_ (0, 0, -1)_** para -Z y, de igual modo, para cualquier otra dirección.
  - Asegúrese de que el modelo 3D esté codificado en uno de los formatos admitidos: `.glb`, `.gltf`, `.ply`, `.fbx`, `.obj`.
- Nuestro servicio de conversión de modelos podría tardar mucho tiempo en procesar un modelo de gran tamaño y con un gran nivel de detalle. Para mayor eficacia, puede preprocesar el modelo 3D para quitar las caras interiores.

## <a name="detection"></a>Detección

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Mixed-Reality/Azure-Object-Anchors-Detection-and-Alignment-Best-Practices/player]

- El SDK del entorno de ejecución proporcionado requiere una región de búsqueda proporcionada por el usuario para buscar y detectar los objetos físicos. La región de búsqueda podría ser un rectángulo delimitador, una esfera, el tronco de una vista o cualquier combinación de ellos. Para evitar una detección falsa, es preferible establecer una región de búsqueda lo suficientemente grande como para abarcar el objeto. Al usar las aplicaciones de ejemplo proporcionadas, puede pararse a un lado del objeto a aproximadamente 2 metros de la superficie más cercana e iniciar la aplicación.
- Antes de iniciar la aplicación de Azure Object Anchors en un dispositivo HoloLens 2, quite los hologramas en las proximidades de su área de trabajo a través de la configuración principal de sus dispositivos en ***Configuración -> Sistema -> Hologramas***.

  Este paso garantiza que si un nuevo objeto, como un automóvil, se encuentra en el mismo espacio que ocupaba otro objeto anteriormente, o si el objeto se ha desplazado del espacio de destino, los hologramas antiguos e irrelevantes no se conservarán ni crearán una visualización confusa para el objeto actualmente en la vista.
- Después de quitar los hologramas y antes de iniciar la aplicación, examine el objeto, por ejemplo, un automóvil, mirando al objeto y sosteniendo el dispositivo a aproximadamente 1 o 2 metros, y camine lentamente alrededor del objeto una o dos veces.

  Este paso garantiza que las estimaciones de superficies residuales creadas en el espacio por objetos y exámenes anteriores se actualicen con las superficies del objeto de destino actual con el que va a trabajar. De lo contrario, es posible que la aplicación vea superficies fantasma dobles que conduzcan a una alineación inexacta del modelo 3D y los hologramas asociados. El examen previo del objeto también reducirá en gran medida la latencia de detección de Azure Object Anchors, por ejemplo, de 30 a 5 segundos.
- En el caso de los objetos oscuros y muy reflectantes, puede que tenga que examinar el objeto a una distancia menor, y también moviendo la cabeza hacia arriba y abajo, y hacia la izquierda y la derecha para que el dispositivo vea las superficies de varios ángulos y varias distancias.
- Cuando vea una detección de objetos incorrecta, por ejemplo, que se voltee la orientación, o que la posición es incorrecta, como un modelo inclinado, debe visualizar la asignación espacial. A menudo, los resultados incorrectos se deben a una reconstrucción deficiente o incompleta de la superficie. Puede quitar los hologramas, examinar el objeto y volver a ejecutar la detección de objetos en la aplicación.
- El SDK del entorno de ejecución proporcionado ofrece algunos parámetros para que los usuarios puedan ajustar la detección, tal y como se muestra en las aplicaciones de ejemplo. Los parámetros predeterminados funcionan bien con la mayoría de los objetos. Si cree que necesita ajustarlos para objetos específicos, he aquí algunas recomendaciones:
  - Use un menor umbral de cobertura de la superficie si el objeto físico es grande, oscuro o brillante.
  - Permita un pequeño cambio de escala (por ejemplo, 0,1) para un objeto grande, como un automóvil.
  - Permita alguna desviación en grados entre la dirección vertical local del objeto y la gravedad cuando el objeto esté en una pendiente.
