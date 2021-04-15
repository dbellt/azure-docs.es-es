---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/10/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 91dc6ba0bcd455aefe9de2efdff2feb20938152d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100376399"
---
### <a name="step-1-navigate-to-the-virtual-network-gateway"></a>Paso 1: Navegación a la puerta de enlace de red virtual

1. En [Azure Portal](https://portal.azure.com), vaya a **Todos los recursos**. 
2. Para abrir la página de la puerta de enlace de red virtual, vaya a la puerta de enlace de red virtual que quiere eliminar y haga clic para seleccionarla. 

### <a name="step-2-delete-connections"></a>Paso 2: Eliminación de conexiones

1. En la página de la puerta de enlace de red virtual, haga clic en **Conexiones** para ver todas las conexiones a la puerta de enlace.
2. Haga clic en los puntos suspensivos **"..."** en la fila del nombre de la conexión y seleccione **Eliminar** en la lista desplegable.
3. Haga clic en **Sí** para confirmar que quiere eliminar la conexión. Si tiene varias conexiones, elimine cada conexión.

### <a name="step-3-delete-the-virtual-network-gateway"></a>Paso 3: Eliminación de la puerta de enlace de red virtual

Tenga en cuenta que si tiene una configuración P2S en esta red virtual además de la configuración S2S, al eliminar la puerta de enlace de la red virtual se desconectarán automáticamente todos los clientes P2S sin previo aviso.

1. En la página de la puerta de enlace de la red virtual, haga clic en **Introducción**.
2. En la página **Información general**, haga clic en **Eliminar** para eliminar la puerta de enlace.
