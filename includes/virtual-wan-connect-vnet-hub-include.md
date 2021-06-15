---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 05/26/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e466de84bbf26cb182135dd9a03f163633255f5c
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578282"
---
En este paso, creará la conexión entre una red virtual y el centro de conectividad. Repita estos pasos para cada red virtual que desee conectar.

1. En la página de la red WAN virtual, seleccione **Conexiones de red virtual**.
1. En la página de conexión de red virtual, seleccione **+Agregar conexión**.
1. En la página **Agregar conexión**, rellene los campos siguientes:

    * **Nombre de la conexión**: asigne un nombre a la conexión.
    * **Centros**: seleccione el concentrador que desea asociar a esta conexión.
    * **Suscripción**: compruebe la suscripción.
    * **Red virtual**: seleccione la red virtual que quiere conectar con este concentrador. La red virtual no puede tener una puerta de enlace de red virtual ya existente.
1. Seleccione **Aceptar** para crear la conexión.