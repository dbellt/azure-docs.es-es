---
title: Configuración de un túnel de usuario de VPN Always On
titleSuffix: Azure VPN Gateway
description: Aprenda a configurar un túnel de usuario de VPN Always On en su instancia de VPN Gateway.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/29/2021
ms.author: cherylmc
ms.openlocfilehash: 59c12333c6cf8ea9d251ce6387e3dce4ff23468b
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287327"
---
# <a name="configure-an-always-on-vpn-user-tunnel"></a>Configuración de un túnel de usuario de VPN para Grupos de disponibilidad AlwaysOn

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

## <a name="configure-the-gateway"></a>Configuración de la puerta de enlace

 Use las instrucciones del artículo [Configuración de una conexión VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md) para configurar la puerta de enlace de VPN para que use IKEv2 y la autenticación basada en certificados.

## <a name="configure-a-user-tunnel"></a>Configuración de un túnel de usuario

[!INCLUDE [user configuration](../../includes/vpn-gateway-vwan-always-on-user.md)]

## <a name="to-remove-a-profile"></a>Eliminación de un perfil

Para quitar un perfil, use estos pasos:

1. Ejecute el siguiente comando:

   ```powershell
   C:\> Remove-VpnConnection UserTest  
   ```

1. Desconecte la conexión y desactive la opción **Conectar automáticamente**.

   ![Limpieza](./media/vpn-gateway-howto-always-on-user-tunnel/disconnect.jpg)

## <a name="next-steps"></a>Pasos siguientes

Para solucionar los problemas de conexión que puedan producirse, consulte [Problemas de conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
