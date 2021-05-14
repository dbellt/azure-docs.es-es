---
title: Creación e instalación de archivos de configuración de cliente VPN de P2S para la autenticación con certificados
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre la creación e instalación de archivos de configuración de un cliente VPN de Windows, Linux, Linux (strongSwan) y macOS X para la autenticación de certificados P2S.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/28/2021
ms.author: cherylmc
ms.openlocfilehash: 239166c872578b310fe8eb0393a7a37f4d25129f
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108202558"
---
# <a name="create-and-install-vpn-client-configuration-files-for-native-azure-certificate-authentication-p2s-configurations"></a>Creación e instalación de archivos de configuración de cliente VPN para configuraciones de punto a sitio con autenticación con certificados nativos de Azure

Los archivos de configuración de cliente VPN están contenidos en un archivo zip. Estos archivos proporcionan la configuración necesaria para que un cliente VPN nativo de Windows, Mac IKEv2 o Linux se conecte a una red virtual mediante una conexión de punto a sitio que use la autenticación con certificados de Azure nativa.

Los archivos de configuración de cliente son específicos de la configuración de VPN para la red virtual. Si después de generar el perfil de configuración de cliente VPN hay algún cambio en la configuración de VPN de punto a sitio, como el tipo de protocolo de VPN o de autenticación, debe generar archivos de configuración de cliente VPN nuevos en los dispositivos de los usuarios.

* Para más información sobre las conexiones de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).
* Para obtener instrucciones OpenVPN, consulte [Configuración de OpenVPN para la puerta de enlace de VPN de punto a sitio de Azure (versión preliminar)](vpn-gateway-howto-openvpn.md) y [Configuración de los clientes OpenVPN](vpn-gateway-howto-openvpn-clients.md).

>[!IMPORTANT]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

## <a name="generate-vpn-client-configuration-files"></a><a name="generate"></a>Generación de los archivos de configuración de cliente VPN

Antes de comenzar, asegúrese de que todos los usuarios que se vayan a conectar tienen un certificado válido instalado en el dispositivo del usuario. Para más información acerca de cómo instalar un certificado de cliente, consulte el artículo sobre la [instalación de certificados de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

Para generar archivos de configuración de cliente, puede usar PowerShell o Azure Portal. Cualquiera de los métodos devuelve el mismo archivo ZIP. Descomprima el archivo para ver las siguientes carpetas:

* **WindowsAmd64** y **WindowsX86**, que contienen los paquetes del instalador de Windows de 32 y 64 bits, respectivamente. El paquete del instalador **WindowsAmd64** es para todos los clientes de Windows de 64 bits, no solo de AMD.
* La carpeta **Genérico** contiene información general que usará para crear su propia configuración de cliente VPN, La carpeta Genérico se proporciona si se ha configurado la opción IKEv2 o SSTP + IKEv2 en la puerta de enlace. Si solo se ha configurado SSTP, la carpeta Genérico no aparece.

### <a name="generate-files-using-the-azure-portal"></a><a name="zipportal"></a>Generación de archivos mediante Azure Portal

1. En Azure Portal, navegue a la puerta de enlace de red virtual correspondiente a la red virtual a la que desea conectarse.
1. En la página de la puerta de enlace de red virtual, seleccione **Configuración de punto a sitio**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/download-client.png" alt-text="Descarga del cliente VPN":::
1. En la parte superior de la página Configuración de punto a sitio, haga clic en **Descargar cliente VPN**. La generación del paquete de configuración del cliente tarda unos minutos.
1. El explorador indica que hay disponible un archivo ZIP de configuración del cliente. Tiene el mismo nombre que su puerta de enlace. Descomprima el archivo para ver las carpetas.

### <a name="generate-files-using-powershell"></a><a name="zipps"></a>Generación de archivos mediante PowerShell

1. Al generar archivos de configuración de cliente VPN, el valor de "-AuthenticationMethod" es "EapTIs". Genere los archivos de configuración de cliente VPN con el comando siguiente:

   ```azurepowershell-interactive
   $profile=New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls"

   $profile.VPNProfileSASUrl
   ```

1. Copie la dirección URL en el explorador para descargar el archivo ZIP y, luego, descomprima el archivo para ver las carpetas.

## <a name="windows"></a><a name="installwin"></a>Windows

[!INCLUDE [Windows instructions](../../includes/vpn-gateway-p2s-client-configuration-windows.md)]

## <a name="mac-os-x"></a><a name="installmac"></a>Mac (OS X)

 Tendrá que configurar manualmente el cliente de VPN nativo IKEv2 en cada equipo Mac que se conecte a Azure. Azure no proporciona el archivo mobileconfig para realizar la autenticación de certificados nativa de Azure. La carpeta **Generic** contiene toda la información que necesita para la configuración. Si no ve la carpeta Genérico en la descarga, es posible que IKEv2 no se haya seleccionado como tipo de túnel. Tenga en cuenta que la SKU de nivel Básico de VPN Gateway no admite IKEv2. Una vez que se seleccione IKEv2, vuelva a generar el archivo ZIP para recuperar la carpeta Genérico.<br>que son los archivos siguientes:

* **VpnSettings.xml**, con configuración importante, como el tipo de túnel y la dirección del servidor. 
* **VpnServerRoot.cer**, con el certificado raíz necesario para validar la puerta de enlace de VPN de Azure durante la instalación de la conexión de punto a sitio.

Siga los pasos siguientes para configurar el cliente de VPN nativo en equipos Mac para realizar una autenticación mediante certificado. Deberá completar estos pasos en cada equipo Mac que se conecte a Azure:

1. Importe el certificado raíz **VpnServerRoot** en Mac. Para ello, sobreescriba el archivo en Mac y haga doble clic en él. Seleccione **Agregar** para importarlo.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/add-certificate.png" alt-text="Captura de pantalla que muestra la página Certificados":::
  
    >[!NOTE]
    >Al hacer doble clic en el certificado, es posible que no se muestre el cuadro de diálogo **Agregar**, pero el certificado se instalará en la tienda correcta. Puede buscar el certificado en la cadena de claves de inicio de sesión en la categoría de certificados.
    >
  
1. Compruebe que ha instalado un certificado de cliente emitido por el certificado raíz que cargó en Azure cuando estableció la configuración de P2S. Es diferente al VPNServerRoot que instaló en el paso anterior. Se necesita y se usa un certificado de cliente para la autenticación. Para más información acerca de cómo generar certificados, consulte [Generación de certificados](vpn-gateway-howto-point-to-site-resource-manager-portal.md#generatecert). Para obtener información acerca de cómo instalar un certificado de cliente, consulte [Instalación de certificados de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).
1. Abra el cuadro de diálogo **Red** en **Preferencias de red** y seleccione **"+"** para crear un nuevo perfil de conexión de cliente VPN para establecer una conexión de punto a sitio a la red virtual de Azure.

   El valor de **Interfaz** es "VPN" y el de **Tipo de VPN** es "IKEv2". Especifique un nombre para el perfil en el campo **Nombre del servicio** y seleccione **Crear** para crear el perfil de conexión de cliente VPN.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/network.png" alt-text="Captura de pantalla que muestra la ventana Red con la opción para seleccionar una interfaz, seleccionar un tipo de VPN y escribir un nombre de servicio.":::
1. En la carpeta **Genérico**, en el archivo **VpnSettings.xml**, copie el valor de la etiqueta **VpnServer**. Pegue este valor en los campos **Dirección del servidor** e **Id. remoto** del perfil.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/server.png" alt-text="Captura de pantalla que muestra información del servidor.":::
1. Seleccione **Configuración de autenticación** y **Certificado**. Para **Catalina**, seleccione **Ninguno** y luego **Certificado**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/authentication-settings.png" alt-text="Captura de pantalla que muestra la configuración de la autenticación.":::

   Para Catalina, seleccione **Ninguno** y luego **Certificado**. **Seleccione** el certificado correcto:
   
   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/catalina.png" alt-text="La captura de pantalla muestra la ventana Red con la opción Ninguna seleccionada en Configuración de autenticación y la opción Certificado también seleccionada.":::

1. Haga clic en **Seleccionar...** para elegir el certificado de cliente que quiere usar en la autenticación. Este es el certificado que instaló en el paso 2.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/certificate.png" alt-text="Captura de pantalla muestra la ventana Red con Configuración de autenticación, donde puede seleccionar un certificado.":::
1. En **Choose An Identity** (Elegir una identidad) se muestra una lista de certificados para elegir. Seleccione el certificado adecuado y luego **Continuar**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/identity.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo Choose An Identity (Elegir una identidad), donde puede seleccionar el certificado adecuado.":::

1. En el campo **Id. local**, especifique el nombre del certificado (del paso 6). En este ejemplo, es `ikev2Client.com`. A continuación, seleccione **Aplicar** para guardar los cambios.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/apply-connect.png" alt-text="Captura de pantalla que muestra Aplicar.":::
1. En el cuadro de diálogo **Red**, seleccione **Aplicar** para guardar los cambios. A continuación, seleccione **Conectar** para iniciar la conexión de punto a sitio a la red virtual de Azure.

## <a name="linux-strongswan-gui"></a><a name="linuxgui"></a>Linux (GUI de StrongSwan)

### <a name="install-strongswan"></a><a name="installstrongswan"></a>Instalación de strongSwan

[!INCLUDE [install strongSwan](../../includes/vpn-gateway-strongswan-install-include.md)]

### <a name="generate-certificates"></a><a name="genlinuxcerts"></a>Generación de certificados

Si aún no ha generado certificados, siga estos pasos:

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

### <a name="install-and-configure"></a><a name="install"></a>Instalación y configuración

Las siguientes instrucciones se crearon en Ubuntu 18.0.4. Ubuntu 16.0.10 no admite la GUI de strongSwan. Si desea usar Ubuntu 16.0.10, deberá usar la [línea de comandos](#linuxinstallcli). Los ejemplos siguientes pueden no coincidir con las pantallas que ve, en función de la versión de Linux y strongSwan que tenga.

1. Abra la herramienta **Terminal** para instalar **strongSwan** y su Network Manager (Administrador de red) ejecutando el comando del ejemplo.

   ```
   sudo apt install network-manager-strongswan
   ```
1. Seleccione **Configuración** y, a continuación, seleccione **Red**. Seleccione el botón **+** para crear una conexión.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/edit-connections.png" alt-text="Captura de pantalla que muestra la página Conexiones de red.":::

1. Seleccione **IPsec/IKEv2 (strongSwan)** en el menú y haga doble clic.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/add-connection.png" alt-text="Captura de pantalla que muestra la página para agregar VPN.":::

1. En la página **Add VPN** (Agregar VPN), agregue un nombre para la conexión VPN.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/choose-type.png" alt-text="Captura de pantalla que muestra la opción de elegir un tipo de conexión.":::
1. Abra el archivo **VpnSettings.xml** de la carpeta **Generic** (Genérico) que está incluida en los archivos de configuración de cliente descargados. Busque la etiqueta denominada **VpnServer** y copie el nombre, que comienza con "azuregateway" y termina con ".cloudapp.net".

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/vpn-server.png" alt-text="Captura de pantalla que muestra la opción de copiar datos.":::
1. Pegue este nombre en el campo **Address** (Dirección) de la nueva conexión VPN de la sección **Gateway** (Puerta de enlace). Luego, seleccione el icono de carpeta al final del campo **Certificate** (Certificado), vaya a la carpeta **Generic** (Genérico) y seleccione el archivo **VpnServerRoot**.
1. En la sección **Client** (Cliente) de la conexión, para **Authentication** (Autenticación), seleccione **Certificate/private key** (Certificado/clave privada). En **Certificate** (Certificado) y **Private key** (Clave privada), elija el certificado y la clave privada que se crearon anteriormente. En **Options** (Opciones), seleccione **Request an inner IP address** (Solicitar una dirección IP interna). Después, seleccione **Agregar**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/ip-request.png" alt-text="Captura de pantalla que muestra la opción de solicitar una dirección IP interna.":::

1. Ajuste la conexión en **Activado**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/turn-on.png" alt-text="Captura de pantalla que muestra la opción de copiar.":::

## <a name="linux-strongswan-cli"></a><a name="linuxinstallcli"></a>Linux (CLI de StrongSwan)

### <a name="install-strongswan"></a>Instalación de strongSwan

[!INCLUDE [install strongSwan](../../includes/vpn-gateway-strongswan-install-include.md)]

### <a name="generate-certificates"></a>Generación de certificados

Si aún no ha generado certificados, siga estos pasos:

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

### <a name="install-and-configure"></a>Instalar y configurar

1. Descargue el paquete de VPNClient desde Azure Portal.
1. Extrae el archivo.
1. Desde la carpeta **Generic**, copie o mueva **VpnServerRoot.cer** a **/etc/ipsec.d/cacerts**.
1. Copie o mueva **cp client.p12** a **/etc/ipsec.d/private/** . Este archivo es un certificado de cliente para la puerta de enlace de VPN.
1. Abra el archivo **VpnSettings.xml** y copie el valor `<VpnServer>`. Usará este valor en el paso siguiente.
1. Ajuste los valores en el ejemplo siguiente y después agregue el ejemplo a la configuración de **/etc/ipsec.conf**.
  
   ```
   conn azure
         keyexchange=ikev2
         type=tunnel
         leftfirewall=yes
         left=%any
         leftauth=eap-tls
         leftid=%client # use the DNS alternative name prefixed with the %
         right= Enter the VPN Server value here# Azure VPN gateway address
         rightid=% # Enter the VPN Server value here# Azure VPN gateway FQDN with %
         rightsubnet=0.0.0.0/0
         leftsourceip=%config
         auto=add
   ```
1. Agregue los siguientes valores a **/etc/ipsec.secrets**.

   ```
   : P12 client.p12 'password' # key filename inside /etc/ipsec.d/private directory
   ```

1. Ejecute los comandos siguientes:

   ```
   # ipsec restart
   # ipsec up azure
   ```

## <a name="next-steps"></a>Pasos siguientes

Vuelva al artículo original desde el que estaba trabajando y, después, complete la configuración de punto a sitio.

* [Pasos de configuración de PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md).
* [Pasos de configuración de Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md).
