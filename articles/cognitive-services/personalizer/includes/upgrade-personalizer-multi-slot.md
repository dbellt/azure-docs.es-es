---
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: include file
ms.date: 03/23/2021
ms.openlocfilehash: ce9bc2807708375847594674e5c28330cbe7c23f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486438"
---
### <a name="upgrade-personalizer-instance-to-multislot"></a>Actualización de instancias de Personalizer para ser de varias ranuras

> [!NOTE]
> La personalización de varias ranuras (versión preliminar) afecta a otras funcionalidades del servicio Personalizer. Este cambio no se puede deshacer. Antes de habilitar la personalización de varias ranuras, consulte [Personalización de varias ranuras (versión preliminar)](../concept-multi-slot-personalization.md). 

En Azure Portal, en el recurso de Personalizer de la página **Configuración de modelo y aprendizaje**, haga clic en **Exportar configuración de aprendizaje**. El campo **arguments** del archivo JSON descargado comenzará por **--cb_explore_adf**. Cámbielo por **a --ccb_explore_adf** y deje tal cual el resto del archivo. Guarde el archivo. 

![Configuración de aprendizaje antes del cambio](../media/settings/learning-settings-pre-upgrade.png)

![Configuración de aprendizaje después del cambio](../media/settings/learning-settings-post-upgrade.png)

En la misma pestaña del portal, en **importar configuración de aprendizaje**, busque el archivo JSON modificado recientemente y ábralo. De este modo, su instancia de Personalizer se actualizará para ser de varias ranuras y permitir en lo sucesivo llamadas Rank y Reward de este tipo.
