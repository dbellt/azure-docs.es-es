---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: e177fabdad7a1517a8371fe1d3483cb5c9310d2c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96581381"
---
Para los datos en tránsito:

- Se utiliza TLS 1.2 Estándar para los datos que circulan entre el dispositivo y Azure. No hay ninguna reserva para TLS 1.1 y versiones anteriores. Si no se admite TLS 1.2, se bloqueará la comunicación del agente. TLS 1.2 también es necesario para la administración del portal y SDK.
- Cuando los clientes acceden al dispositivo mediante la interfaz web local de un explorador, se utiliza TLS 1.2 Estándar como protocolo seguro predeterminado.

    - El procedimiento recomendado consiste en configurar el explorador para usar TLS 1.2.
    - Si el explorador no admite TLS 1.2, puede usar TLS 1.1 o TLS 1.0.
- Se recomienda utilizar SMB 3.0 con cifrado para proteger los datos cuando se copien de los servidores de datos.
