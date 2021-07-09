---
title: Detección de un objeto difícil
description: Se describen los mecanismos que se pueden configurar para detectar objetos difíciles.
author: rgarcia
manager: vrivera
ms.author: rgarcia
ms.date: 05/28/2021
ms.topic: overview
ms.service: azure-object-anchors
ms.openlocfilehash: f7ebff2a9fb45831842b9390e3e72cdbfe58e794
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111987941"
---
# <a name="how-to-detect-a-difficult-object"></a>Detección de un objeto difícil

A veces, un objeto puede resultar más difícil de detectar. Por ejemplo:

- Cuando no se puede acceder a un área expuesta grande porque el objeto está contra una pared
- Cuando un objeto es demasiado grande y tarda demasiado tiempo en recorrerlo
- Cuando los sensores de dispositivo no detectan la superficie del objeto

Algunos mecanismos que ofrece el SDK de Azure Object Anchors que pueden ayudar en estas situaciones son:

- Propiedad `ObjectQuery.MinSurfaceCoverage` Representa la proporción mínima de cobertura de superficie necesaria para considerar que una instancia de objeto es un verdadero positivo. Permite un intervalo de 0 a 1,0 (que representa entre el 0 % y el 100 %). La configuración predeterminada varía entre objetos (cuanto mayor sea el área expuesta, menor será la cobertura mínima necesaria). Funcionará tal cual en la mayoría de las situaciones. Sin embargo, cuando se enfrenta a objetos difíciles, la recomendación es reducir el valor de esta propiedad, para que se requiera menos cobertura de superficie para detectar el objeto.

- Propiedad `ObjectQuery.MaxScaleChange` Si el modelo original no tiene una escala `1:1` hacia el objeto que se detecta, se puede ajustar esta configuración. Permite un intervalo de 0 a 1,0 (que representa entre el 0 % y el 100 %). La configuración predeterminada, en 0, deshabilita la estimación de escala, que requiere una asignación de escala `1:1`. Establecer esta propiedad en 10 %, por ejemplo, habilitaría la estimación de escala y permitiría cierta flexibilidad en los casos en los que la escala del modelo no tiene una coincidencia `1:1` con el objeto.

- Propiedad `ObjectQuery.ExpectedMaxVerticalOrientationInDegrees` Representa el ángulo máximo, en grados, entre la dirección ascendente del objeto y la gravedad. Oscila entre 0 y 180. En otras palabras, representa la inclinación del objeto con respecto al modelo original. La configuración predeterminada, a 3 grados, se puede aumentar para permitir más flexibilidad en los casos en los que la inclinación del objeto no coincide con el modelo original.

- Propiedad `ObjectQuery.IsExpectedToBeStandingOnGroundPlane` Es un valor booleano que representa si se espera que el objeto esté a nivel del suelo o no. El valor predeterminado es "false". Se puede cambiar a true para acelerar la detección en los casos en los que el objeto está a nivel del suelo.

- Propiedad `ObjectQuery.SearchAreas` Representa una colección de regiones para buscar objetos. Proporcionar áreas de búsqueda limitadas, a la vez que cubre todo o la mayor parte del objeto, mejora la velocidad y la precisión de la detección. Puede seleccionar:

  - Un rectángulo de selección orientado, mediante `ObjectSearchArea.FromOriented`.
  - Un campo visual, mediante `ObjectSearchArea.FromFieldOfView`.
  - Una ubicación, mediante `ObjectSearchArea.FromLocation`.
  - Una esfera, mediante `ObjectSearchArea.FromSphere`.

Para más información, vea la clase `ObjectQuery` para [Unity](/dotnet/api/microsoft.azure.objectanchors.objectquery) o [HoloLens C++/WinRT](/cpp/api/object-anchors/winrt/objectquery).
