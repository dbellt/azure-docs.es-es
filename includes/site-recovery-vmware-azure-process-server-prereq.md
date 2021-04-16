---
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 11/28/2019
ms.author: raynew
ms.openlocfilehash: de15e3028cf22cdd03ce29385278fc5e2babaa9b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96008499"
---
En este artículo se supone que

1. Ya se ha establecido una **VPN de sitio a sitio** o una conexión de **Express Route** entre su red local y Azure Virtual Network.
2. Su cuenta de usuario tiene permisos para crear una nueva máquina virtual en la suscripción de Azure en la que las máquinas virtuales se han conmutado por error.
3. La suscripción tiene un mínimo de 8 núcleos disponibles para poner en marcha una nueva máquina virtual de servidor de procesos.
4. Dispone de la **frase de contraseña del servidor de configuración**.

> [!TIP]
> Asegúrese de que se puede conectar al puerto 443 del servidor de configuración (que se ejecuta de forma local) desde la instancia de Azure Virtual Network en la que las máquinas virtuales se han conmutado por error.
