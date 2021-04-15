---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: ebe112103bc3eb30239e80095db9bb91a33bebf3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186510"
---
## <a name="troubleshooting-update-failures"></a>Solución de errores en actualización
**¿Qué ocurre si ve una notificación que indica que se ha producido un error en las comprobaciones previas a la actualización?**

Si se produce un error en la comprobación previa, asegúrese de que ha examinado la barra de notificación detallada en la parte inferior de la página. Esto proporciona una guía para saber en qué comprobación previa se ha producido el error. Por ejemplo, recibe una notificación que le indica que ni la comprobación de mantenimiento del controlador ni la comprobación de mantenimiento de los componentes de hardware han resultado satisfactorias. Vaya a **Supervisar > Mantenimiento de hardware**. Es preciso asegurarse de que ambos controladores funcionan correctamente y están en línea. También necesitará asegurarse de que todos los componentes de hardware del dispositivo StorSimple están en buen estado en esta página. A continuación, puede intentar instalar las actualizaciones. Si no puede corregir los problemas de los componentes de hardware, deberá ponerse en contacto con el servicio de soporte técnico de Microsoft para los pasos siguientes.

**¿Qué sucede si recibe el mensaje de error "Las actualizaciones no se pudieron instalar" y la recomendación es hacer referencia a la guía de solución de problemas de actualización para determinar la causa del error?**

Una causa probable podría ser que no tiene conectividad con los servidores de Microsoft Update. Esta es una comprobación manual que se debe realizar. Si se pierde la conexión con servidor de actualizaciones, se producirá un error en el trabajo de actualización. Para comprobar la conectividad, ejecute el siguiente cmdlet desde la interfaz de Windows PowerShell del dispositivo StorSimple:

 `Test-Connection -Source <Fixed IP of your device controller> -Destination <Any IP or computer name outside of datacenter>`

Ejecute el cmdlet en ambos controladores.

Si ha comprobado que existe conectividad y sigue apareciendo este problema, póngase en contacto con servicio de soporte técnico de Microsoft para los pasos siguientes.

**¿Qué ocurre si ve un error de actualización al actualizar el dispositivo a la actualización 4 y ambos controladores ejecutan la actualización 4?**

A partir de la actualización 4, si ambos controladores ejecutan la misma versión de software y si se produce un error de actualización, los controladores no entran en modo de recuperación. Esta situación puede ocurrir si la revisión de software de dispositivo (actualización de primer orden) se aplica a ambos controladores correctamente pero otras revisiones (de segundo y tercer orden) todavía tienen que aplicarse. Desde la actualización 4, los controladores pasarán al modo de recuperación solo si los dos controladores ejecutan diferentes versiones de software. 

Si el usuario ve un error de actualización cuando ejecutan ambos controladores la actualización 4, se recomienda esperar unos minutos y, después, volver a intentar la actualización. Si el reintento no funciona, debe ponerse en contacto con el soporte técnico de Microsoft.
