---
title: 'Configuración de clientes VPN para conexiones P2S de protocolo OpenVPN: autenticación de Azure AD en macOS (versión preliminar)'
description: 'Versión preliminar: aprenda a configurar un cliente VPN de macOS para conectarse a una red virtual mediante una VPN de punto a sitio y la autenticación de Azure Active Directory.'
titleSuffix: Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 05/14/2021
ms.author: cherylmc
ms.openlocfilehash: 6027968636c5ce956189ee3f62c4defc69cafa89
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110104666"
---
# <a name="azure-active-directory-authentication-configure-vpn-clients-for-p2s-openvpn-protocol-connections---macos---preview"></a>Autenticación de Azure Active Directory: configuración de clientes VPN para conexiones P2S de protocolo OpenVPN en macOS (versión preliminar)

Este artículo le ayuda a configurar un cliente VPN en un equipo que ejecuta macOS 10.15 y versiones posteriores para conectarse a una red virtual mediante una VPN de punto a sitio y la autenticación de Azure Active Directory. Antes de poder conectarse y autenticarse con Azure AD, primero debe configurar el inquilino de Azure AD. Para más información, consulte [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md).

> [!NOTE]
> * Esta versión preliminar está disponible actualmente en la mayoría de los países (pero no en todos) debido a los requisitos normativos locales.
> * La autenticación de Azure AD solo es compatible con las conexiones del protocolo OpenVPN® y requiere el cliente VPN de Azure.
>

En cada equipo que quiera que se conecte a una red virtual mediante una conexión VPN de punto a sitio, debe hacer lo siguiente:
 
* Descargue el Cliente VPN de Azure en el equipo.
* Configure un perfil de cliente que contenga la configuración de VPN. 

Si quiere configurar varios equipos, puede crear un perfil de cliente en un equipo, exportarlo y, después, importarlo a otros equipos.

## <a name="prerequisites"></a>Requisitos previos

Antes de poder conectarse y autenticarse con Azure AD, primero debe configurar el inquilino de Azure AD. Para más información, consulte [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md).

## <a name="to-download-the-azure-vpn-client"></a><a name="download"></a>Para descargar el cliente VPN de Azure

1. Descargue el [cliente VPN de Azure](https://apps.apple.com/us/app/azure-vpn-client/id1553936137) desde la tienda Apple.
1. Instale el cliente en el equipo.

## <a name="to-import-a-connection-profile"></a><a name="import"></a>Para importar un perfil de conexión:

1. Descargue y extraiga los archivos del perfil. Para ver los pasos, consulte [Uso de archivos de perfil de cliente VPN de punto a sitio](about-vpn-profile-download.md).
1. En la página Cliente VPN de Azure, seleccione **Importar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/import-1.png" alt-text="Captura de pantalla de la selección de importación del cliente VPN de Azure.":::
1. Vaya al archivo de perfil que quiere importar, selecciónelo y haga clic en **Abrir**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/import-2.png" alt-text="Captura de pantalla de la importación del cliente VPN de Azure cuando se hace clic en Abrir.":::
1. Vea la información del perfil de conexión y, luego, haga clic en **Guardar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/import-3.png" alt-text="Captura de pantalla del cliente VPN de Azure cuando se guarda la configuración del perfil importado.":::
1. En el panel Conexiones VPN, seleccione el perfil de conexión que guardó. A continuación, haga clic en **Conectar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/import-4.png" alt-text="Captura de pantalla del cliente VPN de Azure cuando se hace clic en Conectar.":::
1. Una vez conectado, el estado cambiará a **Conectado**. Para desconectarse de la sesión, haga clic en **Desconectar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/import-5.png" alt-text="Captura de pantalla del estado Conectado y del botón de desconexión del cliente VPN de Azure.":::

## <a name="to-create-a-connection-manually"></a><a name="manual"></a>Para crear una conexión manualmente:

1. Abra el cliente de VPN de Azure. Seleccione **Agregar** para crear una conexión.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/add-1.png" alt-text="Captura de pantalla del cliente VPN de Azure cuando se selecciona Agregar.":::

1. En la página **Cliente VPN de Azure**, puede configurar los valores del perfil.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/add-2.png" alt-text="Captura de pantalla de la configuración del perfil de cliente VPN de Azure.":::

   Configure las siguientes opciones:

   * **Nombre de conexión:** : el nombre con el que quiere hacer referencia al perfil de conexión.
   * **Servidor VPN:** este es el nombre que quiere usar para hacer referencia al servidor. No es necesario que el nombre que elija aquí tenga el nombre formal de un servidor.
   * **Validación del servidor**
     * **Información del certificado:** la entidad de certificación del certificado.
     * **Secreto de servidor:** el secreto del servidor.
   * **Autenticación de cliente**
     * **Tipo de autenticación:** Azure Active Directory
     * **Inquilino:** nombre del inquilino.
     * **Emisor:** nombre del emisor.
1. Después de rellenar los campos anteriores, haga clic en **Guardar**.
1. En el panel Conexiones VPN, seleccione el perfil de conexión que configuró. A continuación, haga clic en **Conectar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/add-3.png" alt-text="Captura de pantalla de la conexión del cliente VPN de Azure.":::
1. Con sus credenciales, inicie sesión para conectarse.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/add-4.png" alt-text="Captura de pantalla del inicio de sesión del cliente VPN de Azure para conectarse.":::
1. Una vez conectado, verá el estado **Conectado**. Cuando quiera desconectarse, haga clic en **Desconectar** para desconectar la conexión.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/add-5.png" alt-text="Captura de pantalla del cliente VPN de Azure conectado y el botón Desconectar.":::

## <a name="to-remove-a-connection-profile"></a><a name="remove"></a>Para quitar un perfil de conexión:

Puede quitar el perfil de conexión VPN del equipo. 

1. Vaya al cliente VPN de Azure.
1. Seleccione la conexión VPN que quiere quitar, haga clic en la lista desplegable y seleccione **Quitar**.

   :::image type="content" source="media/openvpn-azure-ad-client-mac/remove-1.png" alt-text="Captura de pantalla de la eliminación.":::
1. En el cuadro **Remove VPN connection?** (¿Quitar conexión VPN?), haga clic en **Quitar**.
   :::image type="content" source="media/openvpn-azure-ad-client-mac/remove-2.png" alt-text="Captura de pantalla de la eliminación.":::

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [crear un inquilino de Azure Active Directory para conexiones VPN abiertas de P2S que usan Autenticación de Azure AD](openvpn-azure-ad-tenant.md).