---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/20/2020
ms.author: glenga
ms.openlocfilehash: 7d1bf8dd2d1c8feab8b051a8edad7d5e570ee11b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026923"
---
La zona horaria predeterminada que se usa con las expresiones CRON es la Hora universal coordinada (UTC). Para que la expresión CRON se base en otra zona horaria, cree una configuración de aplicación para la aplicación de función denominada `WEBSITE_TIME_ZONE`. 

El valor de esta opción de configuración depende del sistema operativo y del plan en el que se ejecuta la aplicación de funciones.

|Sistema operativo |Plan |Value |
|-|-|-|
| **Windows** |All | Establezca como valor el nombre de la zona horaria que desea, tal como aparece en la segunda línea de los pares que proporciona el comando de Windows `tzutil.exe /L` |
| **Linux** |Premium<br/>Dedicado |Establezca el valor en el nombre de la zona horaria deseada como se muestra en [tz database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). |

> [!NOTE]
> El plan de Consumo para Linux actualmente no admite `WEBSITE_TIME_ZONE`.

Por ejemplo, la hora del este de EE. UU. (que se representa mediante `Eastern Standard Time` en Windows o `America/New_York` en Linux) usa actualmente UTC-05:00 mientras está vigente la hora estándar y UTC-04:00 durante el horario de verano. Para que un desencadenador de temporizador lo ponga en marcha a las 10:00 AM, hora del este, todos los días,cree una configuración de aplicación para la aplicación de funciones denominada `WEBSITE_TIME_ZONE`, establezca el valor en `Eastern Standard Time` (Windows) o `America/New_York` (Linux) y, a continuación, use la siguiente expresión NCRONTAB: 

```
"0 0 10 * * *"
``` 

Si usa `WEBSITE_TIME_ZONE`, la hora se ajusta a los cambios de hora de la zona horaria concreta, lo que incluye el horario de verano, y los cambios en la hora estándar.
