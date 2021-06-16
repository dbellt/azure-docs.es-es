---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/26/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: f6576a6aa4be42705ce8b8c12d7cee2bdb87bfd0
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579615"
---
## <a name="windows-clients"></a><a name="windows"></a>Clientes Windows

1. Descargue e instale el cliente OpenVPN (versión 2.4 o superior) desde el [sitio web de OpenVPN](https://openvpn.net/index.php/open-source/downloads.html) oficial.
2. Descargue el perfil de VPN para la puerta de enlace. Esto puede hacerse desde la pestaña de configuración de punto a sitio de Azure Portal o mediante "New-AzVpnClientConfiguration" en PowerShell.
3. Descomprima el perfil. A continuación, abra el archivo de configuración *vpnconfig.ovpn* desde la carpeta OpenVPN del Bloc de notas.
4. Exporte el certificado de cliente de punto a sitio que ha creado y cargado en la configuración de P2S en la puerta de enlace. Utilice los vínculos de los artículos siguientes:

   * Instrucciones de [VPN Gateway](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport)
   
   * Instrucciones de [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#clientexport)
5. Extraiga la clave privada y la huella digital de base64 del archivo *.pfx*. Hay varias formas de hacerlo. Una de ellas es usar OpenSSL en su máquina. El archivo *profileinfo.txt* contiene la clave privada y la huella digital de la entidad de certificación y el certificado de cliente. Asegúrese de usar la huella digital del certificado de cliente.

   ```
   openssl pkcs12 -in "filename.pfx" -nodes -out "profileinfo.txt"
   ```
6. Abra *profileinfo.txt* en el Bloc de notas. Para obtener la huella digital del certificado (secundario) de cliente, seleccione el texto (incluido y entre) "---BEGIN CERTIFICATE---" y "---END CERTIFICATE---" para el certificado secundario y cópielo. Puede identificar el certificado secundario si examina la línea subject=/.
7. Cambie al archivo *vpnconfig.ovpn* que abrió en el Bloc de notas en el paso 3. Busque la sección que se muestra a continuación y reemplace todo lo que hay entre "cert" y "/cert".

   ```
   # P2S client certificate
   # please fill this field with a PEM formatted cert
   <cert>
   $CLIENTCERTIFICATE
   </cert>
   ```
8. Abra el archivo *profileinfo.txt* en el Bloc de notas. Para obtener la clave privada, seleccione el texto (incluido y entre) "-----BEGIN PRIVATE KEY-----" y "-----END PRIVATE KEY-----" y cópielo.
9. Vuelva al archivo vpnconfig.ovpn en el Bloc de notas y busque esta sección. Pegue la clave privada y reemplace todo lo que hay entre "key" y "/key".

   ```
   # P2S client root certificate private key
   # please fill this field with a PEM formatted key
   <key>
   $PRIVATEKEY
   </key>
   ```
10. No cambie los demás campos. Use los datos de la configuración de entrada del cliente para conectarse a la VPN.
11. Copie el archivo vpnconfig.ovpn en la carpeta C:\Program Files\OpenVPN\config.
12. Haga clic con el botón derecho en l icono de OpenVPN en la bandeja del sistema y, después, haga clic en Conectar.

## <a name="mac-clients"></a><a name="mac"></a>Clientes Mac

1. Descargue e instale un cliente OpenVPN, como [TunnelBlick](https://tunnelblick.net/downloads.html). 
2. Descargue el perfil de VPN para la puerta de enlace. Esto puede hacerse desde la pestaña de configuración de punto a sitio de Azure Portal o mediante "New-AzVpnClientConfiguration" en PowerShell.
3. Descomprima el perfil. Abra el archivo de configuración vpnconfig.ovpn desde la carpeta OpenVPN en un editor de texto.
4. Rellene la sección de certificado cliente de P2S con la clave pública del certificado cliente de P2S en Base64. En los certificados con formato PEM, puede abrir el archivo .cer y copiar la clave de base64 entre los encabezados de certificados. Use los vínculos de los artículos siguientes para obtener información acerca de cómo exportar un certificado para obtener la clave pública codificada:

   * Instrucciones de [VPN Gateway](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#cer) 
   
   * Instrucciones de [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#cer)
5. Rellene la sección de la clave privada con la clave privada del certificado cliente de P2S en Base64. Consulte [Exportación de la clave privada](https://openvpn.net/community-resources/how-to/#pki) en el sitio de OpenVPN para obtener información sobre cómo extraer una clave privada.
6. No cambie los demás campos. Use los datos de la configuración de entrada del cliente para conectarse a la VPN.
7. Haga doble clic en el archivo de perfil para crear el perfil en Tunnelblick.
8. Inicie Tunnelblick desde la carpeta de aplicaciones.
9. Haga clic en el icono de Tunneblick en la bandeja del sistema y seleccione Conectar.

> [!IMPORTANT]
>Solo iOS 11.0 y MacOS 10.13 (y sus respectivas versiones posteriores) son compatibles con el protocolo OpenVPN.
>
## <a name="ios-clients"></a><a name="iOS"></a>Clientes iOS

1. Instale el cliente de OpenVPN (versión 2,4 o posterior) desde App Store.
2. Descargue el perfil de VPN para la puerta de enlace. Esto puede hacerse desde la pestaña de configuración de punto a sitio de Azure Portal o mediante "New-AzVpnClientConfiguration" en PowerShell.
3. Descomprima el perfil. Abra el archivo de configuración vpnconfig.ovpn desde la carpeta OpenVPN en un editor de texto.
4. Rellene la sección de certificado cliente de P2S con la clave pública del certificado cliente de P2S en Base64. En los certificados con formato PEM, puede abrir el archivo .cer y copiar la clave de base64 entre los encabezados de certificados. Use los vínculos de los artículos siguientes para obtener información acerca de cómo exportar un certificado para obtener la clave pública codificada:

   * Instrucciones de [VPN Gateway](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#cer) 
   
   * Instrucciones de [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#cer)
5. Rellene la sección de la clave privada con la clave privada del certificado cliente de P2S en Base64. Consulte [Exportación de la clave privada](https://openvpn.net/community-resources/how-to/#pki) en el sitio de OpenVPN para obtener información sobre cómo extraer una clave privada.
6. No cambie los demás campos.
7. Envíe por correo electrónico el archivo de perfil (.ovpn) a su cuenta de correo electrónico configurada en la aplicación de correo en su iPhone. 
8. Abra el correo electrónico en la aplicación de correo del iPhone y pulse en el archivo adjunto.

    ![Correo electrónico abierto](./media/vpn-gateway-vwan-config-openvpn-clients/ios2.png)

9. Pulse en **More** (Más) si no ve la opción **Copy to OpenVPN** (Copiar a OpenVPN).

    ![Más](./media/vpn-gateway-vwan-config-openvpn-clients/ios3.png)

10. Pulse en **Copiar a OpenVPN**. 

    ![Copia a OpenVPN](./media/vpn-gateway-vwan-config-openvpn-clients/ios4.png)

11. Pulse en **ADD** en la página **Import Profile** (Importar perfil).

    ![Sumar](./media/vpn-gateway-vwan-config-openvpn-clients/ios5.png)

12. Pulse en **ADD** en la página **Imported Profile** (Perfil importado).

    ![Pulsar ADD](./media/vpn-gateway-vwan-config-openvpn-clients/ios6.png)

13. Inicie la aplicación OpenVPN y deslice el conmutador de la página **Profiles** (Perfiles) a la derecha para conectar.

    ![Conectar](./media/vpn-gateway-vwan-config-openvpn-clients/ios8.png)


## <a name="linux-clients"></a><a name="linux"></a>Clientes Linux

1. Abra una nueva sesión de terminal. Para abrir una nueva sesión, presione "CTRL + ALT + T" al mismo tiempo.
2. Escriba el siguiente comando para instalar los componentes necesarios:

   ```
   sudo apt-get install openvpn
   sudo apt-get -y install network-manager-openvpn
   sudo service network-manager restart
   ```
3. Descargue el perfil de VPN para la puerta de enlace. Esto se puede hacer desde la pestaña de configuración de punto a sitio en Azure Portal.
4. Exporte el certificado de cliente P2P que ha creado y cargado en la configuración de P2S en la puerta de enlace. Utilice los vínculos de los artículos siguientes:

   * Instrucciones de [VPN Gateway](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport) 
   
   * Instrucciones de [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#clientexport)
5. Extraiga la clave privada y la huella digital de base64 del archivo .pfx. Hay varias formas de hacerlo. Una de ellas es usar OpenSSL en su equipo.

    ```
    openssl pkcs12 -in "filename.pfx" -nodes -out "profileinfo.txt"
    ```
   El archivo *profileinfo.txt* contendrá la clave privada y la huella digital de la entidad de certificación y el certificado de cliente. Asegúrese de usar la huella digital del certificado de cliente.

6. Abra *profileinfo.txt* en un editor de texto. Para obtener la huella digital del certificado (secundario) de cliente, seleccione el texto (incluido y entre) "---BEGIN CERTIFICATE---" y "---END CERTIFICATE---" para el certificado secundario y cópielo. Puede identificar el certificado secundario si examina la línea subject=/.

7. Abra el archivo *vpnconfig.ovpn* y busque la sección que se muestra a continuación. Reemplace todo lo que aparece entre "cert" y "/cert".

   ```
   # P2S client certificate
   # please fill this field with a PEM formatted cert
   <cert>
   $CLIENTCERTIFICATE
   </cert>
   ```
8. Abra el archivo profileinfo.txt en un editor de texto. Para obtener la clave privada, seleccione el texto incluido y entre "-----BEGIN PRIVATE KEY-----" y "-----END PRIVATE KEY-----" y cópielo.

9. Abra el archivo vpnconfig.ovpn en un editor de texto y busque esta sección. Pegue la clave privada y reemplace todo lo que hay entre "key" y "/key".

   ```
   # P2S client root certificate private key
   # please fill this field with a PEM formatted key
   <key>
   $PRIVATEKEY
   </key>
   ```

10. No cambie los demás campos. Use los datos de la configuración de entrada del cliente para conectarse a la VPN.
11. Para conectarse mediante la línea de comandos, escriba el siguiente comando:
  
    ```
    sudo openvpn --config <name and path of your VPN profile file>&
    ```
12. Para conectarse mediante la GUI, vaya a la configuración del sistema.
13. Haga clic en **+** para agregar una nueva conexión VPN.
14. En **Agregar VPN**, seleccione **Importar desde archivo...** .
15. Vaya al archivo de perfil y haga doble clic o seleccione **Abrir**.
16. Haga clic en **Agregar** en la ventana **Agregar VPN**.
  
    ![Importar desde archivo](./media/vpn-gateway-vwan-config-openvpn-clients/import.png)
17. Puede conectarse mediante la **activación** de VPN en la página **Configuración de red** o en el icono de red de la bandeja del sistema.
