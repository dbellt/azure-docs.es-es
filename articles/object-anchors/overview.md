---
title: Información general de Azure Object Anchors
description: Obtenga información sobre cómo Azure Object Anchors lo ayuda a detectar objetos en el mundo físico.
author: craigktreasure
manager: vriveras
ms.author: crtreasu
ms.date: 03/02/2021
ms.topic: overview
ms.service: azure-object-anchors
ms.openlocfilehash: 84a69c46e9d7587ed87d8288fca5e7d8ef25f546
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983601"
---
# <a name="azure-object-anchors-overview"></a>Información general de Azure Object Anchors

Azure Object Anchors permite a una aplicación detectar un objeto en el mundo físico mediante un modelo 3D y estimar su posición 6-DoF (seis grados de libertad). La posición 6-DoF (seis grados de libertad) se define como una rotación y traslación entre un modelo 3D y su contraparte física, el objeto real.

Azure Object Anchors consta de un servicio para la conversión de modelos y un SDK de cliente de entorno de ejecución para HoloLens. El servicio acepta un modelo de objetos 3D y genera un modelo de Azure Object Anchors. El modelo de Azure Object Anchors se usa junto con el SDK de entorno de ejecución para permitir que una aplicación de HoloLens cargue un modelo de objetos y detecte y haga el seguimiento de las instancias de ese modelo en el mundo físico.

:::image type="content" source="./media/object-anchors-overview.jpg" alt-text="Azure Object Anchors en acción":::

## <a name="examples"></a>Ejemplos

Entre los casos de uso de ejemplos habilitados por Azure Object Anchors se incluyen los siguientes:

- **Entrenamiento**. Cree experiencias de aprendizaje de Mixed Reality para sus trabajadores, sin necesidad de colocar marcadores o dedicar tiempo para ajustar manualmente la alineación de los hologramas. Si quiere aumentar las experiencias de aprendizaje de Mixed Reality con detección y seguimiento automáticos, ingiera su modelo en el servicio de Azure Object Anchors y estará un paso más cerca de disfrutar de una experiencia sin marcadores.

- **Guía para las tareas**. Guiar a los empleados por un conjunto de tareas puede ser mucho más sencillo con Mixed Reality. La superposición de instrucciones y procedimientos recomendados digitales aplicada al objeto físico, que puede ser una pieza de maquinaria en una planta de producción o una cafetera en la cocina del equipo, puede disminuir considerablemente la dificultad de completar el conjunto de tareas. Por lo general, desencadenar estas experiencias requiere algún tipo de marcador o alineación manual, pero con Azure Object Anchors, puede crear una experiencia que detecte automáticamente el objeto relacionado con la tarea a mano. Luego, siga sin problemas por las directrices de Mixed Reality sin marcadores ni alineación manual.

- **Búsqueda de recursos**. Si ya tiene un modelo 3D de algún objeto en el espacio físico, Azure Object Anchors puede permitirle localizar instancias de ese objeto en el entorno físico y hacer su seguimiento.

## <a name="usage-flow"></a>Flujo de uso

En primer lugar, cargue el recurso 3d en nuestro servicio de conversión de Azure Object Anchors. Puede seguir los pasos de uno de nuestros inicios rápidos:

  - [HoloLens en Unity](quickstarts/get-started-unity-hololens.md)
  - [HoloLens para Unity con MRTK](quickstarts/get-started-unity-hololens-mrtk.md)
  - [HoloLens DirectX](quickstarts/get-started-hololens-directx.md)

A continuación, el servicio convertirá el recurso en un modelo de Azure Object Anchors. Descargue el modelo convertido, para que pueda [visualizar su malla](visualize-converted-model.md) si lo desea. Por último, copie el modelo en un dispositivo HoloLens que tenga el SDK del entorno de ejecución para [Unity](/dotnet/api/Microsoft.Azure.ObjectAnchors) u [HoloLens C++/WinRT](/cpp/api/object-anchors/winrt): ahora puede detectar objetos físicos que coincidan con el modelo original.

:::image type="content" source="./media/object-anchors-flow.png" alt-text="Flujo de uso":::

## <a name="asset-requirements"></a>Requisitos de los recursos

Cada dimensión de un recurso debe tener entre 1 y 10 metros, y el tamaño del archivo debe ser inferior a 150 MB.

Los formatos de recurso admitidos actualmente son: `fbx`, `ply`, `obj`, `glb` y `gltf`.

## <a name="next-steps"></a>Pasos siguientes

En las secciones siguientes se proporciona información sobre cómo comenzar a usar y compilar aplicaciones con Azure Object Anchors.

> [!div class="nextstepaction"]
> [Ingesta de modelos](quickstarts/get-started-model-conversion.md)

> [!div class="nextstepaction"]
> [HoloLens en Unity](quickstarts/get-started-unity-hololens.md)

> [!div class="nextstepaction"]
> [HoloLens para Unity con MRTK](quickstarts/get-started-unity-hololens-mrtk.md)

> [!div class="nextstepaction"]
> [HoloLens DirectX](quickstarts/get-started-hololens-directx.md)
