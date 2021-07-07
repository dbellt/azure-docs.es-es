---
title: Visualización de un modelo Object Anchors
description: Describa cómo visualizar un modelo convertido.
author: rgarcia
manager: vrivera
ms.author: rgarcia
ms.date: 05/28/2021
ms.topic: overview
ms.service: azure-object-anchors
ms.openlocfilehash: a9140004d9f4d79cabb9890851ba5c5cb7b815f2
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111987920"
---
# <a name="how-to-visualize-an-object-anchors-model"></a>Visualización de un modelo Object Anchors

No es necesario visualizar un modelo convertido para poder usarlo. Sin embargo, hay una manera sencilla de ver su malla antes de usarla si lo desea.

Siga los pasos del artículo de [Inicio rápido de la aplicación en Unity](quickstarts/get-started-unity-hololens.md) con un leve cambio. Al compilar la escena de ejemplo, en lugar de abrir **AOASampleScene**, abra **VisualizeScene** y agréguela a la lista de compilación de la escena. A continuación, en **Configuración de la compilación**, asegúrese de que *solo* **VisualizeScene** tenga una marca de verificación junto a ella: no se debe incluir ninguna de las demás escenas.

:::image type="content" source="../../includes/media/object-anchors-quickstarts-unity/aoa-unity-build-settings-visualize.png" alt-text="visualización de la configuración de la compilación":::

Cierre el cuadro de diálogo **Configuración de la compilación** *en lugar de* seleccionar el botón **Compilar**.

En el panel **Jerarquía**, seleccione el elemento GameObject **Visualizador**.

:::image type="content" source="../../includes/media/object-anchors-quickstarts-unity/aoa-unity-hierarchy.png" alt-text="jerarquía":::

En el panel **Inspector**, busque la propiedad **Ruta de acceso del modelo** en la sección **Mesh Loader (Script)** (Cargador de malla [script]) y escriba la ruta de acceso al archivo de modelos de Object Anchors, incluida la extensión de archivo.

:::image type="content" source="../../includes/media/object-anchors-quickstarts-unity/aoa-unity-inspector.png" alt-text="inspector":::

Seleccione el botón **Reproducir** en la parte superior del Editor de Unity y asegúrese de que la vista **Escena** esté seleccionada.

:::image type="content" source="../../includes/media/object-anchors-quickstarts-unity/aoa-unity-editor.png" alt-text="reproducir y vista de la escena":::

Con los [controles de navegación de la vista de la escena de Unity](https://docs.unity3d.com/Manual/SceneViewNavigation.html), ahora puede inspeccionar el modelo de Object Anchors.

:::image type="content" source="./media/object-anchors-visualize.png" alt-text="visualizar modelo":::
