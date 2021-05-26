---
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: include file
ms.date: 03/23/2021
ms.openlocfilehash: b61e9ba2374286a313444de7ab1e74b17c0a7af5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382339"
---
### <a name="upgrade-personalizer-instance-to-multislot"></a>Actualización de instancias de Personalizer para ser de varias ranuras

En Azure Portal, en el recurso de Personalizer de la página **Configuración de modelo y aprendizaje**, haga clic en **Exportar configuración de aprendizaje**. El campo **arguments** del archivo JSON descargado comenzará por **--cb_explore_adf**. Cámbielo por **a --ccb_explore_adf** y deje tal cual el resto del archivo. Guarde el archivo. 

![Configuración de aprendizaje antes del cambio](../media/settings/learning-settings-pre-upgrade.png)

![Configuración de aprendizaje después del cambio](../media/settings/learning-settings-post-upgrade.png)

En la misma pestaña del portal, en **importar configuración de aprendizaje**, busque el archivo JSON modificado recientemente y ábralo. De este modo, su instancia de Personalizer se actualizará para ser de varias ranuras y permitir en lo sucesivo llamadas Rank y Reward de este tipo.