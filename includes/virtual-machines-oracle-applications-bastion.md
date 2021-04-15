---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/10/2019
ms.author: danlep
ms.openlocfilehash: 35f506235f698fbcf42308e6f0b0f400e925df29
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "68361268"
---
### <a name="bastion-tier"></a>Nivel de bastión

El host bastión es un componente opcional que puede usar como un servidor de salto para acceder a las instancias de aplicación y de base de datos. La máquina virtual host bastión puede tener asignada una dirección IP pública, a pesar de que la recomendación es configurar una VPN de sitio a sitio o una conexión de ExpressRoute con la red local para un acceso seguro. Además, solo se debe abrir SSH (puerto 22, Linux) o RDP (puerto 3389, Windows Server) para el tráfico entrante. Para lograr alta disponibilidad, implemente un host bastión en dos zonas de disponibilidad o en un conjunto de disponibilidad único.

También puede habilitar el desvío del agente SSH en las máquinas virtuales, lo que le permitirá acceder a otras máquinas virtuales de la red virtual si desvía las credenciales desde el host bastión. O bien, use la tunelización de SSH para acceder a otras instancias.

Este es un ejemplo de desvío del agente:

```
ssh -A -t user@BASTION_SERVER_IP ssh -A root@TARGET_SERVER_IP`
```

Este comando se conecta al bastión y, luego, de inmediato ejecuta `ssh` nuevamente, por lo que se obtiene un terminal en la instancia de destino. Es posible que tenga que especificar un usuario distinto de la raíz en la instancia de destino si el clúster se configuró de manera distinta. El argumento `-A` desvía la conexión del agente para que la clave privada de la máquina local se use automáticamente. Tenga en cuenta que el desvío del agente es una cadena, por lo que el segundo comando `ssh` también incluye `-A` para que toda conexión SSH subsiguiente iniciada desde la instancia de destino también use la clave privada local.