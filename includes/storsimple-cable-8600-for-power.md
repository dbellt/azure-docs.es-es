---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 9b9922602218280d58331a755ed0dfed7df96f40
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186351"
---
#### <a name="to-cable-your-device-for-power"></a>Cableado del dispositivo para tener alimentación
> [!NOTE]
> Los dos alojamientos del dispositivo StorSimple incluyen PCM redundantes. Para los dos alojamientos, los PCM tienen que estar instalados y conectados a diferentes fuentes de alimentación para garantizar una alta disponibilidad.
> 
> 

1. Asegúrese de que el interruptor de alimentación de cada uno de los PCM se encuentran en la posición OFF.
2. Para el alojamiento principal, conecta los cables de alimentación a ambos PCM. Los cables de alimentación se identifican en rojo en el diagrama de cables de alimentación siguiente.
3. Asegúrate de que los dos PCM del receptáculo principal usan fuentes de alimentación independientes.
4. Adjunta los cables de alimentación a las unidades de distribución de energía del bastidor tal y como se muestra en el diagrama de cables de alimentación.
5. Repita los pasos del 2 al 4 para el receptáculo EBOD.
6. Enciende el alojamiento de EBOD colocando cada interruptor de alimentación de PCM en la posición de encendido.
7. Verifica que el alojamiento de EBOD está activado comprobando que los LED verdes de la parte posterior del EBOD están encendidos.
8. Después enciende el alojamiento principal colocando todos los interruptores de PCM en la posición de encendido.
9. Comprueba que el sistema está operativo asegurándote de que los LED del controlador de dispositivo están encendidos.
10. Verifica que la conexión entre el controlador EBOD y el controlador del dispositivo esté activa comprobando que los cuatro LED situados junto al puerto SAS del controlador EBOD están en verde.
    
    > [!IMPORTANT]
    > Para garantizar una alta disponibilidad del sistema, se recomienda cumplir estrictamente el esquema de cableado de potencia que se muestra en el siguiente diagrama de cableado.
    > 
    > 
    
    ![Cableado del dispositivo 4U para alimentación](./media/storsimple-cable-8600-for-power/HCSCableYour4UDeviceforPower.png)
    
    **Cables de alimentación**
    
    | Etiqueta | Descripción |
    |:--- |:--- |
    | 1 |Receptáculo principal |
    | 2 |PCM 0 |
    | 3 |PCM 1 |
    | 4 |Controlador 0 |
    | 5 |Controlador 1 |
    | 6 |Controlador EBOD 0 |
    | 7 |Controlador EBOD 1 |
    | 8 |Receptáculo EBOD |
    | 9 |PDU |

