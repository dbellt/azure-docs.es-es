---
title: Conexión de ExpressRoute a la puerta de enlace de red virtual
description: Pasos para conectar ExpressRoute a la puerta de enlace de red virtual.
ms.topic: include
ms.date: 12/08/2020
ms.openlocfilehash: 115e8829723c25fb1644f3f5652fbace529a05cb
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945702"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-configure-networking.md -->


1. Solicite una clave de autorización de ExpressRoute:

   [!INCLUDE [request-authorization-key](request-authorization-key.md)]

1. Acceda a la puerta de enlace de red virtual que va a usar y seleccione **Conexiones** >  **+ Agregar**.

1. En la página **Agregar conexión**, proporcione valores para los campos y seleccione **Aceptar**. 

   | Campo | Valor |
   | --- | --- |
   | **Nombre**  | Escriba un nombre para la conexión.  |
   | **Tipo de conexión**  | seleccione **ExpressRoute**.  |
   | **Canjear autorización**  | Asegúrese de que este cuadro está seleccionado.  |
   | **Puerta de enlace de red virtual** | La puerta de enlace de red virtual que tiene pensado usar.  |
   | **Clave de autorización**  | Pegue la clave de autorización que copió anteriormente. |
   | **URI de circuito del mismo nivel**  | Pegue el identificador de ExpressRoute que copió anteriormente.  |

   :::image type="content" source="../media/expressroute-global-reach/expressroute-add-connection.png" alt-text="Captura de pantalla del panel Agregar conexión para conectar ExpressRoute a la puerta de enlace de red virtual.":::

Se crea la conexión entre el circuito ExpressRoute y la red virtual.

:::image type="content" source="../media/expressroute-global-reach/virtual-network-gateway-connections.png" alt-text="Captura de pantalla de las conexiones de puerta de enlace de red virtual.":::