---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/16/2019
ms.author: alkohli
ms.openlocfilehash: f79081d506db6225b9ebef25674aad136e342ecf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96465889"
---
Para los datos en tránsito:

- Se utiliza Seguridad de la capa de transporte 1.2 estándar para los datos que circulan entre el dispositivo y Azure. No hay ninguna reserva para TLS 1.1 y versiones anteriores. Si no se admite TLS 1.2, se bloqueará la comunicación del agente. TLS 1.2 también es necesario para la administración del portal y SDK.
- Cuando los clientes acceden al dispositivo mediante la interfaz web local de un explorador, se utiliza TLS 1.2 Estándar como protocolo seguro predeterminado.

  - El procedimiento recomendado consiste en configurar el explorador para usar TLS 1.2.
  - El dispositivo solo admite TLS 1.2 y no es compatible con las versiones anteriores TLS 1.1 ni TLS 1.0.
- Se recomienda utilizar SMB 3.0 con cifrado para proteger los datos cuando se copien de los servidores de datos.
