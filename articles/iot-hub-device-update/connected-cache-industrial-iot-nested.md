---
title: Caché conectada de Microsoft en Azure IoT Edge para la configuración de IoT industrial | Microsoft Docs
description: Tutorial de configuración de Caché conectada de Microsoft en Azure IoT Edge para IoT industrial
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: 083c7bf6edc7da1fd617487e91b0a3848fb401fe
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107811826"
---
# <a name="microsoft-connected-cache-preview-deployment-scenario-sample-microsoft-connected-cache-within-an-azure-iot-edge-for-industrial-iot-configuration"></a>Ejemplo de escenario de implementación de la versión preliminar de Caché conectada de Microsoft: Caché conectada de Microsoft en un Azure IoT Edge para la configuración de IoT industrial

Las redes de fabricación suelen organizarse en capas jerárquicas que siguen el [modelo de red Purdue](https://en.wikipedia.org/wiki/Purdue_Enterprise_Reference_Architecture) (incluido en los estándares [ISA 95](https://en.wikipedia.org/wiki/ANSI/ISA-95) e [ISA 99](https://www.isa.org/standards-and-publications/isa-standards/isa-standards-committees/isa99)). En estas redes, solo la capa superior tiene conectividad con la nube y las capas inferiores de la jerarquía solo pueden comunicarse con las capas superior e inferior adyacentes.

En este ejemplo de GitHub, [Azure IOT Edge para IOT industrial](https://github.com/Azure-Samples/iot-edge-for-iiot), se implementa lo siguiente:

* Red Purdue simulada en Azure
* Recursos industriales 
* Jerarquía de puertas de enlace Azure IoT Edge
  
Estos componentes se usarán para adquirir datos industriales y cargarlos de forma segura en la nube, sin poner en peligro la seguridad de la red. La caché conectada de Microsoft se puede implementar para facilitar la descarga de contenido en todos los niveles de la red compatible con ISA 95.

La clave para configurar implementaciones de Caché conectadas de Microsoft en una red compatible con el estándar ISA 95 consiste en configurar el proxy OT *y* el host de nivel superior en la puerta de enlace IOT Edge L3.

1. Configure las implementaciones de Caché conectada de Microsoft en los niveles L5 y L4, como se describe en el ejemplo de puerta de enlace IoT Edge anidada de dos niveles. 
2. La implementación en la puerta de enlace IoT Edge L3 debe especificar:
   
   * UPSTREAM_HOST: la dirección IP/FQDN de la puerta de enlace IoT Edge L4 a la que Caché conectada de Microsoft L3 solicitará contenido.
   * UPSTREAM_PROXY: el puerto IP/FQDN del servidor proxy OT.

3. El proxy OT debe agregar la dirección IP/FQDN MCC L4 a la lista de permitidos.

Para confirmar que Caché conectada de Microsoft funciona correctamente, ejecute el siguiente comando en el terminal del dispositivo IoT Edge que hospeda el módulo, o bien en cualquier dispositivo de la red. Reemplace \<Azure IoT Edge Gateway IP\> por la dirección IP o el nombre de host de la puerta de enlace de IoT Edge (consulte los detalles de la variable de entorno para obtener información sobre la visibilidad de este informe).

```bash
    wget http://<L3 IoT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```