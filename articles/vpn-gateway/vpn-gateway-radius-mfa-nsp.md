---
title: Integración de NPS con la autenticación RADIUS de VPN Gateway para MFA
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo integrar la autenticación RADIUS de Azure VPN Gateway con el servidor NPS para Multi-Factor Authentication.
services: vpn-gateway
author: ahmadnyasin
manager: dcscontentpm
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/16/2019
ms.author: genli
ms.openlocfilehash: 6cfc6ea361dddf703741baa1a990018186b402e4
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289487"
---
# <a name="integrate-azure-vpn-gateway-radius-authentication-with-nps-server-for-multi-factor-authentication"></a>Autenticación RADIUS de Azure VPN Gateway de integración con el servidor NPS para Multi-Factor Authentication 

En el artículo se describe cómo integrar Servidor de directivas de redes (NPS) con la autenticación RADIUS de Azure VPN Gateway a fin de ofrecer Multi-Factor Authentication (MFA) a las conexiones VPN de punto a sitio. 

## <a name="prerequisite"></a>Requisito previo

Para habilitar MFA, los usuarios deben estar en Azure Active Directory (Azure AD), que debe sincronizarse desde el entorno local o de nube. Además, el usuario ya debe haber completado el proceso de inscripción automática para MFA.  Para obtener más información, consulte [Configuración de mi cuenta para la comprobación en dos pasos](../active-directory/user-help/multi-factor-authentication-end-user-first-time.md)

## <a name="detailed-steps"></a>Pasos detallados

### <a name="step-1-create-a-virtual-network-gateway"></a>Paso 1: Creación de una puerta de enlace de red virtual

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la red virtual que hospedará la puerta de enlace de red virtual, seleccione **Subredes** y, a continuación, **Subred de puerta de enlace** para crear una subred. 

    ![La imagen sobre cómo agregar una subred de puerta de enlace](./media/vpn-gateway-radiuis-mfa-nsp/gateway-subnet.png)
3. Cree una puerta de enlace de red virtual especificando la siguiente configuración:

    - **Tipo de puerta de enlace**: seleccione **VPN**.
    - **Tipo de VPN**: seleccione **Basada en rutas**.
    - **SKU**: seleccione un tipo de SKU basada en sus requisitos.
    - **Red virtual**: seleccione la red virtual en la que creó la subred de puerta de enlace.

        ![La imagen sobre la configuración de puerta de enlace de red virtual](./media/vpn-gateway-radiuis-mfa-nsp/create-vpn-gateway.png)


 
### <a name="step-2-configure-the-nps-for-azure-ad-mfa"></a>Paso 2: Configuración del NPS para Azure AD MFA

1. En el servidor NPS, [instale la extensión de NPS para Azure AD MFA](../active-directory/authentication/howto-mfa-nps-extension.md#install-the-nps-extension).
2. Abra la consola de NPS, haga clic con el botón derecho en **Clientes RADIUS** y seleccione **Nuevo**. Cree el cliente RADIUS especificando la siguiente configuración:

    - **Nombre descriptivo**: escriba cualquier nombre.
    - **Dirección (IP o DNS)** : escriba la subred de puerta de enlace que creó en el Paso 1.
    - **Secreto compartido**: escriba cualquier clave secreta y recuérdela para su uso posterior.

      ![Imagen de la configuración del cliente RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client1.png)

 
3.  En la pestaña **Avanzado**, establezca el nombre del proveedor en **RADIUS estándar** y asegúrese de que la casilla **Opciones adicionales** no está activada.

    ![Imagen de la configuración avanzada del cliente RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client2.png)

4. Vaya a **Directivas** > **Directivas de red**, haga doble clic en la directiva **Conexiones al servidor de enrutamiento y acceso remoto de Microsoft**, seleccione **Conceder acceso** y, a continuación, haga clic en **Aceptar**.

### <a name="step-3-configure-the-virtual-network-gateway"></a>Paso 3: Configuración de la puerta de enlace de red virtual

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Abra la puerta de enlace de red virtual que creó. Asegúrese de que el tipo de puerta de enlace se establece en **VPN** y de que el tipo de VPN está **basado en rutas**.
3. Haga clic en **Configuración de punto a sitio** > **Configurar ahora** y, a continuación, especifique la siguiente configuración:

    - **Grupo de direcciones**: escriba la subred de puerta de enlace que creó en el paso 1.
    - **Tipo de autenticación**: seleccione **Autenticación RADIUS**.
    - **Dirección IP del servidor**: escriba la dirección IP del servidor NPS.

      ![La imagen sobre la configuración de punto a sitio](./media/vpn-gateway-radiuis-mfa-nsp/configure-p2s.png)

## <a name="next-steps"></a>Pasos siguientes

- [Azure AD Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md)
- [Integración de la infraestructura existente de NPS con Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-nps-extension.md)