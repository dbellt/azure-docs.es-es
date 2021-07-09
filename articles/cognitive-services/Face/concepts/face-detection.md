---
title: Conceptos de atributos y detección de caras
titleSuffix: Azure Cognitive Services
description: La detección de caras es la acción de búsqueda de caras humanas en una imagen y, opcionalmente, la devolución de distintos tipos de datos relacionados con las caras.
services: cognitive-services
author: PatrickFarley
manager: nitime
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: pafarley
ms.openlocfilehash: cb005621a3439d5c7976221d816536b46efb986c
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745010"
---
# <a name="face-detection-and-attributes"></a>Atributos y detección de caras

En este artículo se explican conceptos acerca de los datos de atributos y de detección de caras. La detección de caras es la acción de búsqueda de caras humanas en una imagen y, opcionalmente, la devolución de distintos tipos de datos relacionados con las caras.

La operación [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) se utiliza para detectar caras en una imagen. Como mínimo, cada cara detectada corresponde a un campo faceRectangle de la respuesta. Este conjunto de coordenadas de píxeles para la parte izquierda, la superior, el ancho y el alto marca la cara localizada. Con estas coordenadas, puede obtener la ubicación de la cara y su tamaño. En la respuesta de la API, las caras se muestran en orden de tamaño de mayor a menor.

## <a name="face-id"></a>Id. de cara

El identificador de cara es una cadena de identificador único para cada cara detectada en una imagen. Puede solicitar un identificador de cara en su llamada API [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

## <a name="face-landmarks"></a>Puntos de referencia de cara

Los puntos de referencia de cara son un conjunto de puntos fáciles de encontrar en una cara como, por ejemplo, las pupilas o la punta de la nariz. De forma predeterminada, existen 27 puntos de referencia predefinidos. La ilustración siguiente muestra los 27 puntos:

![Diagrama de cara con los 27 puntos de referencia etiquetados](../Images/landmarks.1.jpg)

Las coordenadas de los puntos se devuelven en unidades de píxeles.

## <a name="attributes"></a>Atributos

Los atributos son un conjunto de características que la API [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) puede, opcionalmente, detectar. Se pueden detectar los siguientes atributos:

* **Accesorios**. Si la cara determinada tiene accesorios. Este atributo devuelve posibles accesorios, incluidos artículos para la cabeza, gafas y mascarilla, con una puntuación de confianza entre cero y uno para cada accesorio.
* **Edad**. Edad estimada en años de una cara determinada.
* **Desenfoque**. Desenfoque de la cara de la imagen. Este atributo devuelve un valor entre cero y uno, así como una clasificación informal de bajo, medio o alto.
* **Emoción**. Lista de emociones con una confianza de detección para la cara determinada. Las puntuaciones de confianza están normalizadas y las puntuaciones de todas las emociones suman uno. Las emociones detectadas son felicidad, tristeza, neutralidad, ira, desprecio, asco, sorpresa y temor.
* **Exposición**. Exposición de la cara de la imagen. Este atributo devuelve un valor entre cero y uno, así como una clasificación informal de underExposure, goodExposure o overExposure.
* **Vello facial**. Presencia estimada de vello facial y longitud de la cara determinada.
* **Sexo**. Sexo estimado de la cara determinada. Los valores posibles son hombre, mujer y sin sexo definido.
* **Gafas**. Si la cara determinada tiene gafas. Los valores posibles son NoGlasses, ReadingGlasses, Sunglasses y Swimming Goggles.
* **Pelo**. Tipo de pelo de la cara. Este atributo muestra si el pelo está visible, así como la calvicie y el color de pelo que se detecten.
* **Posición de la cabeza**. Orientación de la cara en un espacio 3D. Este atributo se describe mediante los ángulos de giro, desviación e inclinación en grados, que se definen según la [regla de la derecha](https://en.wikipedia.org/wiki/Right-hand_rule). El orden de tres ángulos es giro-desviación-inclinación y el valor de cada ángulo oscila entre -180 y 180 grados. La orientación tridimensional de la cara se calcula mediante los ángulos de giro, desviación e inclinación en orden. Consulte el siguiente diagrama de asignaciones de ángulos:

    ![Cabeza con ejes de rotación alrededor del eje x (pitch), de rotación y de rotación alrededor del eje y (yaw) etiquetados](../Images/headpose.1.jpg)
* **Maquillaje**. Si la cara está maquillada. Este atributo devuelve un valor booleano para eyeMakeup y lipMakeup.
* **Mascarilla**.  Si la cara lleva una mascarilla. Este atributo devuelve un posible tipo de mascarilla y un valor booleano para indicar si la nariz y la boca están cubiertas.
* **Ruido**. Ruido visual detectado en la cara de la imagen. Este atributo devuelve un valor entre cero y uno, así como una clasificación informal de bajo, medio o alto.
* **Oclusión**. Si hay objetos que bloquean partes de la cara. Este atributo devuelve un valor booleano para eyeOccluded, foreheadOccluded y mouthOccluded.
* **Sonrisa**. Expresión sonriente de la cara determinada. Este valor se encuentra entre cero para no sonriente y uno para muy sonriente.

> [!IMPORTANT]
> Los atributos de caras se pueden predecir mediante algoritmos estadísticos. No obstante, es posible que no sean siempre precisos. Tenga cuidado al tomar decisiones basadas en datos de atributos.

## <a name="input-data"></a>Datos de entrada

Utilice las siguientes sugerencias para asegurarse de que las imágenes de entrada proporcionan los resultados de detección más precisos:

* Los formatos de imagen de entrada admitidos son JPEG, PNG, GIF para el primer fotograma y BMP.
* El tamaño del archivo de imagen no debe ser superior a 6 MB.
* El tamaño mínimo detectable de la cara es de 36 x 36 píxeles en una imagen no superior a 1920 x 1080 píxeles. Las imágenes de más de 1920 x 1080 píxeles tienen un tamaño mínimo detectable de la cara proporcionalmente mayor. Reducir el tamaño de la cara podría provocar que no se detecten algunas caras, aunque sean mayores que el tamaño mínimo detectable.
* El tamaño máximo de cara detectable es 4096 x 4096 píxeles.
* Las caras fuera del intervalo de tamaño de 36 x 36 a 4096 x 4096 píxeles no se detectarán.
* Es posible que no se puedan detectar algunas caras debido a desafíos técnicos. Unos ángulos de cara extremos (posición de la cabeza) o una oclusión de la cara (objetos como gafas de sol o manos que bloquean parte de la cara) pueden afectar a la detección. Las caras de frente y casi de frente proporcionan los mejores resultados.

Datos de entrada con información de orientación:
* Algunas imágenes de entrada con formato JPEG pueden contener información de orientación en metadatos de formato de archivo de imagen intercambiable (Exif). Si la orientación Exif está disponible, las imágenes se girarán automáticamente a la orientación correcta antes de su envío para la detección de caras. El rectángulo facial, los puntos de referencia y la posición de la cabeza de todas las caras detectadas se calcularán en función de la imagen girada.
* Para mostrar correctamente tanto el rectángulo facial como los puntos de referencia es preciso asegurarse de que la imagen se gira correctamente. La mayoría de las herramientas de visualización de imágenes girarán automáticamente la imagen según su orientación Exif de forma predeterminada. Em el caso de otras herramientas, es posible que tenga que usar su propio código para aplicar la rotación. En los ejemplos siguientes se muestra un rectángulo facial en una imagen girada (izquierda) y una imagen no girada (derecha).

![Dos imágenes de caras con o sin rotación](../Images/image-rotation.png)

Si está detectando caras de una fuente de vídeo, puede mejorar el rendimiento mediante el ajuste de determinados valores de la cámara de vídeo:

* **Suavizado**: muchas cámaras de vídeo aplican un efecto de suavizado. Debe desactivar esta opción, si es posible, porque crea un desenfoque entre fotogramas y reduce la claridad.
* **Velocidad de obturación**: una velocidad de obturación más rápida reduce la cantidad de movimiento entre fotogramas y hace que cada fotograma sea más claro. Se recomienda utilizar velocidades de obturación de 1/60 de segundo o más rápidas.
* **Ángulo de obturación**: algunas cámaras especifican el ángulo de obturación en lugar de la velocidad de obturación. Debe usar un ángulo de obturación inferior si es posible. Esto generará unos fotogramas de vídeo más claros.

    >[!NOTE]
    > Una cámara con un ángulo de obturación inferior recibirá menos luz en cada fotograma, por lo que la imagen será más oscura. Deberá determinar el nivel adecuado que vaya a usar.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya está familiarizado con conceptos de detección de caras, aprenda a escribir un script que detecte caras en una imagen determinada.

* [Detección de caras en una imagen](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)
