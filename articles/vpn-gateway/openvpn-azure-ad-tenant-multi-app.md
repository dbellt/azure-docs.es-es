---
title: 'Creación de un inquilino de Azure AD para conexiones de protocolo OpenVPN de P2S: autenticación de Azure AD'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo configurar un inquilino de Azure AD para la autenticación de OpenVPN de P2S, y cómo registrar varias aplicaciones en Azure AD a fin de permitir un acceso diferente a los distintos usuarios y grupos.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 05/05/2021
ms.author: cherylmc
ms.openlocfilehash: f5b67a1843dd363c727122d77c4c9d574e90a4ab
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108760488"
---
# <a name="create-an-active-directory-ad-tenant-for-p2s-openvpn-protocol-connections"></a>Creación de un inquilino de Active Directory (AD) para conexiones del protocolo P2S OpenVPN

Al conectarse a la red virtual mediante un protocolo de punto a sitio, tiene la opción de elegir qué protocolo usar. El protocolo que use determina las opciones de autenticación que tiene a su disposición. Si quiere usar la autenticación de Azure Active Directory, puede hacerlo cuando use el protocolo OpenVPN. Si quiere que un conjunto de usuarios diferente pueda conectarse a diferentes puertas de enlace de VPN, puede registrar varias aplicaciones en AD y vincularlas a diferentes puertas de enlace de VPN. Este artículo le ayuda a configurar un inquilino de Azure AD para OpenVPN de P2S y a crear y registrar varias aplicaciones en Azure AD a fin de permitir un acceso distinto a los diferentes usuarios y grupos. Para más información sobre los protocolos y la autenticación de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

[!INCLUDE [OpenVPN note](../../includes/vpn-gateway-openvpn-auth-include.md)]

[!INCLUDE [create](../../includes/openvpn-azure-ad-tenant-multi-app.md)]

## <a name="6-enable-authentication-on-the-gateway"></a><a name="enable-authentication"></a>6. Habilitación de la autenticación en la puerta de enlace

En este paso, habilitará la autenticación de Azure AD en la puerta de enlace VPN.

1. Habilite la autenticación de Azure AD en la puerta de enlace de VPN. Para ello, navegue hasta **Configuración de punto a sitio** y seleccione **OpenVPN (SSL)** como **Tipo de túnel**. Seleccione **Azure Active Directory** como **Tipo de autenticación**, a continuación, rellene la información de la sección **Azure Active Directory**.

    ![Visualización de Azure Portal](./media/openvpn-azure-ad-tenant-multi-app/azure-ad-auth-portal.png)

    > [!NOTE]
    > No use el identificador de la aplicación del cliente VPN de Azure: Concederá acceso a todos los usuarios a la puerta de enlace VPN. Use el identificador de las aplicaciones que registró.

2. Para crear y descargar el perfil, haga clic en el vínculo **Descargar cliente VPN**.

3. Extraiga el archivo zip descargado.

4. Busque la carpeta "AzureVPN" descomprimida.

5. Anote la ubicación del archivo "azurevpnconfig. xml". Azurevpnconfig. xml contiene la configuración de la conexión VPN y se puede importar directamente en la aplicación del cliente VPN Azure. También puede distribuir este archivo a todos los usuarios que necesiten conectarse a través del correo electrónico u otros medios. El usuario necesitará credenciales de Azure AD válidas para conectarse correctamente.

## <a name="next-steps"></a>Pasos siguientes

Para conectarse a la red virtual, debe crear y configurar un perfil de cliente de VPN. Consulte [Configurar un cliente VPN para conexiones P2S VPN](openvpn-azure-ad-client.md).
