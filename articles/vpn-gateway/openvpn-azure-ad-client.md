---
title: 'Configuración de clientes VPN para conexiones P2S de protocolo OpenVPN: Autenticación de Azure AD'
description: Aprenda a configurar un cliente VPN para conectarse a una red virtual mediante una VPN de punto a sitio de VPN Gateway y la autenticación de Azure Active Directory.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 04/28/2021
ms.author: alzam
ms.openlocfilehash: ba3733f16353ad1e921430ca7699212a60d5ff2f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754962"
---
# <a name="azure-active-directory-authentication-configure-a-vpn-client-for-p2s-openvpn-protocol-connections"></a>Autenticación de Azure Active Directory: Configuración de un cliente VPN para conexiones P2S de protocolo OpenVPN

Este artículo le ayuda a configurar un cliente VPN para conectarse a una red virtual mediante una VPN de punto a sitio y la autenticación Azure Active Directory. Antes de poder conectarse y autenticarse con Azure AD, primero debe configurar el inquilino de Azure AD. Para más información, consulte [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md). Para más información sobre la ubicación de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

[!INCLUDE [OpenVPN note](../../includes/vpn-gateway-openvpn-auth-include.md)]

## <a name="working-with-client-profiles"></a><a name="profile"></a>Trabajar con perfiles de cliente

Por cada equipo que quiera conectarse a la red virtual a través del cliente VPN, debe descargar el cliente VPN de Azure para el equipo y también configurar un perfil de cliente VPN. Si quiere configurar varios equipos, puede crear un perfil de cliente en un equipo, exportarlo y, después, importarlo a otros equipos.

### <a name="to-download-the-azure-vpn-client"></a>Para descargar el cliente VPN de Azure

1. Descargue el [Cliente VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554) en el equipo.
1. Compruebe que el cliente VPN de Azure tenga permiso para ejecutarse en segundo plano. Para comprobar los permisos y habilitarlos, navegue a **Inicio -> Configuración -> Privacidad -> Aplicaciones en segundo plano**.

   * En **Aplicaciones en segundo plano**, asegúrese de que el valor **Permitir que las aplicaciones se ejecuten en segundo plano** sea **Activado**.
   * En **Elegir qué aplicaciones se pueden ejecutar en segundo plano**, cambie la configuración del **Cliente VPN de Azure** a **Activado**.

     ![Captura de pantalla de los permisos.](./media/openvpn-azure-ad-client/backgroundpermission.png)

### <a name="to-create-a-certificate-based-client-profile"></a><a name="cert"></a>Para crear un perfil de cliente basado en certificados

Cuando trabaje con un perfil basado en certificados, asegúrese de que los certificados adecuados están instalados en el equipo cliente. Para más información acerca de los certificados, consulte [Instalar certificados de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

![Captura de pantalla de la autenticación de certificado.](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="to-create-a-radius-client-profile"></a><a name="radius"></a>Para crear un perfil de cliente RADIUS

![Captura de pantalla de la autenticación RADIUS.](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> El secreto de servidor se puede exportar en el perfil de cliente de P2S VPN.  [Aquí](about-vpn-profile-download.md) encontrará instrucciones sobre cómo exportar un perfil de cliente.
>

### <a name="to-export-and-distribute-a-client-profile"></a><a name="export"></a>Para exportar y distribuir un perfil de cliente

Una vez que tenga un perfil de trabajo y necesite distribuirlo a otros usuarios, puede exportarlo mediante los siguientes pasos:

1. Resalte el perfil de cliente de VPN que quiere exportar, seleccione el **...** y,luego, seleccione **Exportar**.

    ![Captura de pantalla que muestra la página "Cliente VPN de Azure", con los puntos suspensivos seleccionados y la opción "Exportar" resaltada.](./media/openvpn-azure-ad-client/export/export1.jpg)

2. Seleccione la ubicación en la que desea guardar este perfil, deje el nombre de archivo tal cual y, a continuación, seleccione **Guardar** para guardar el archivo xml.

    ![exportar](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="to-import-a-client-profile"></a><a name="import"></a>Para importar un perfil de cliente

1. En la página, seleccione **Importar**.

    ![Captura de pantalla que muestra el botón "Agregar" seleccionado y la acción "Importar" resaltada en la parte inferior izquierda de la ventana.](./media/openvpn-azure-ad-client/import/import1.jpg)

2. Busque el archivo xml de perfil y selecciónelo. Con el archivo seleccionado, seleccione **Abrir**.

    ![Captura de pantalla que muestra un archivo de perfil XML seleccionado.](./media/openvpn-azure-ad-client/import/import2.jpg)

3. Especifique el nombre del perfil y seleccione **Guardar**.

    ![Captura de pantalla que muestra la opción "Nombre de conexión" resaltada y el botón "Guardar" seleccionado.](./media/openvpn-azure-ad-client/import/import3.jpg)

4. Seleccione **Conectar** para conectarse a la VPN.

    ![Captura de pantalla que la VPN y el botón "Conectar" seleccionados.](./media/openvpn-azure-ad-client/import/import4.jpg)

5. Una vez conectado, el icono se volverá verde y dirá **Conectado**.

    ![importación](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Para eliminar un perfil de cliente

1. Seleccione los puntos suspensivos junto al perfil de cliente que desea eliminar. Después, seleccione **Quitar**.

    ![Captura de pantalla que muestra los puntos suspensivos y la opción "Quitar" seleccionados.](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. Seleccione **Quitar** para eliminar.

    ![delete](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="create-a-connection"></a><a name="connection"></a>Crear una conexión

1. En la página, seleccione **+** , después **+ Agregar**.

    ![Captura de pantalla que muestra el botón "Agregar" seleccionado.](./media/openvpn-azure-ad-client/create/create1.jpg)

2. Rellene la información de conexión. Si no está seguro de los valores, póngase en contacto con el administrador. Después de rellenar los valores, seleccione **Guardar**.

    ![Captura de pantalla que muestra las propiedades de la conexión VPN resaltadas y el botón "Guardar" seleccionado.](./media/openvpn-azure-ad-client/create/create2.jpg)

3. Seleccione **Conectar** para conectarse a la VPN.

    ![Captura de pantalla que muestra el botón "Conectar" seleccionado.](./media/openvpn-azure-ad-client/create/create3.jpg)

4. Seleccione las credenciales adecuadas y, después, seleccione **Continuar**.

    ![Captura de pantalla que muestra las credenciales de ejemplo resaltadas y el botón "Continuar" seleccionado.](./media/openvpn-azure-ad-client/create/create4.jpg)

5. Una vez que se haya conectado correctamente, el icono se volverá verde y dirá **Conectado**.

    ![conexión](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="to-connect-automatically"></a><a name="autoconnect"></a>Para conectarse automáticamente

Estos pasos le ayudarán a configurar la conexión para que se conecte automáticamente con Always-on.

1. En la página principal del cliente VPN, seleccione **Configuración de VPN**.

    ![Captura de pantalla de la página principal de la VPN con la opción "Configuración de VPN" seleccionada.](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. Seleccione **Sí** en el cuadro de diálogo cambiar aplicaciones.

    ![Captura de pantalla del cuadro de diálogo "¿Quieres cambiar de aplicación?" con el botón "Sí" seleccionado.](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. Asegúrese de que la conexión que quiere establecer no está conectada y, luego, resalte el perfil y active la casilla **Conectar automáticamente**.

    ![Captura de pantalla de la ventana "Configuración" con la casilla "Conectar automáticamente" activada.](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. Seleccione **Conectar** para iniciar la conexión VPN.

    ![auto](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnóstico de problemas de conexión

1. Para diagnosticar problemas de conexión, puede usar la herramienta **Diagnosticar**. Seleccione el **...** junto a la conexión VPN que quiere diagnosticar para que se muestre el menú. Después, seleccione **Diagnosticar**.

    ![Captura de pantalla de los puntos suspensivos y la opción "Diagnosticar" seleccionados.](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. En la página **Propiedades de conexión**, seleccione **Ejecutar diagnóstico**.

    ![Captura de pantalla que muestra la página "Propiedades de conexión" con la opción "Ejecutar diagnóstico" seleccionada.](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. Inicie sesión con sus credenciales.

    ![Captura de pantalla que muestra el cuadro de diálogo "Vamos a iniciar su sesión" con una "cuenta profesional o educativa" seleccionada.](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. Ver los resultados del diagnóstico.

    ![diagnóstico](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>Preguntas más frecuentes

### <a name="is-the-azure-vpn-client-supported-with-windows-fips-mode"></a>¿El modo FIPS de Windows es compatible con Cliente VPN de Azure?

Sí, con la revisión [KB4577063](https://support.microsoft.com/help/4577063/windows-10-update-kb4577063).

### <a name="how-do-i-add-dns-suffixes-to-the-vpn-client"></a>¿Cómo agrego sufijos DNS al cliente VPN?

Puede modificar el archivo XML del perfil descargado y agregar las etiquetas **\<dnssuffixes>\<dnssufix> \</dnssufix>\</dnssuffixes>** .

```
<azvpnprofile>
<clientconfig>

    <dnssuffixes>
          <dnssuffix>.mycorp.com</dnssuffix>
          <dnssuffix>.xyz.com</dnssuffix>
          <dnssuffix>.etc.net</dnssuffix>
    </dnssuffixes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-add-custom-dns-servers-to-the-vpn-client"></a>¿Cómo agrego servidores DNS personalizados al cliente VPN?

Puede modificar el archivo XML del perfil descargado y agregar las etiquetas **\<dnsservers>\<dnsserver> \</dnsserver>\</dnsservers>** .

```
<azvpnprofile>
<clientconfig>

    <dnsservers>
        <dnsserver>x.x.x.x</dnsserver>
        <dnsserver>y.y.y.y</dnsserver>
    </dnsservers>
    
</clientconfig>
</azvpnprofile>
```

> [!NOTE]
> El cliente OpenVPN de Azure AD usa las entradas de la Tabla de directivas de resolución de nombres (NRPT) de DNS, lo que significa que los servidores DNS no se mostrarán en la salida de `ipconfig /all`. Para confirmar la configuración de DNS en uso, consulte [Get-DnsClientNrptPolicy](/powershell/module/dnsclient/get-dnsclientnrptpolicy) en PowerShell.
>

### <a name="can-i-configure-split-tunneling-for-the-vpn-client"></a><a name="split"></a>¿Puedo configurar la tunelización dividida para el cliente VPN?

La tunelización dividida está configurada de forma predeterminada para el cliente VPN.

### <a name="how-do-i-direct-all-traffic-to-the-vpn-tunnel-forced-tunneling"></a><a name="forced-tunnel"></a>¿Cómo puedo dirigir todo el tráfico al túnel VPN (tunelización forzada)?

Puede configurar la tunelización forzada mediante dos métodos distintos: el anuncio de rutas personalizadas o la modificación del archivo XML de perfil.    

> [!NOTE]
> No se proporciona conectividad a Internet a través de VPN Gateway. Como resultado, se anula todo el tráfico enlazado a Internet.
>

* **Anunciar las rutas personalizadas**: puede anunciar las rutas personalizadas 0.0.0.0/1 y 128.0.0.0/1. Para obtener más información, consulte [Anuncio de rutas personalizadas para clientes VPN de P2S](vpn-gateway-p2s-advertise-custom-routes.md).

* **XML de perfil**: puede modificar el archivo XML de perfil descargado para agregar las etiquetas **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>** .


    ```
    <azvpnprofile>
    <clientconfig>
          
        <includeroutes>
            <route>
                <destination>0.0.0.0</destination><mask>1</mask>
            </route>
            <route>
                <destination>128.0.0.0</destination><mask>1</mask>
            </route>
        </includeroutes>
           
    </clientconfig>
    </azvpnprofile>
    ```


### <a name="how-do-i-add-custom-routes-to-the-vpn-client"></a>¿Cómo agrego rutas personalizadas al cliente VPN?

Puede modificar el archivo XML del perfil descargado y agregar las etiquetas **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>** .

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-block-exclude-routes-from-the-vpn-client"></a>¿Cómo bloqueo (excluyo) las rutas del cliente VPN?

Puede modificar el archivo XML del perfil descargado y agregar las etiquetas **\<excluderoutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</excluderoutes>** .

```
<azvpnprofile>
<clientconfig>

    <excluderoutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </excluderoutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="can-i-import-the-profile-from-a-command-line-prompt"></a>¿Puedo importar el perfil desde un símbolo de la línea de comandos?

Para importar el perfil desde un símbolo de la línea de comandos, coloque el archivo **azurevpnconfig.xml** descargado en la carpeta **%userprofile%\AppData\Local\Packages\Microsoft.AzureVpn_8wekyb3d8bbwe\LocalState** y ejecute el comando siguiente:

```
azurevpn -i azurevpnconfig.xml 
```
Para forzar la importación, use el modificador **-f**.


## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [crear un inquilino de Azure Active Directory para conexiones VPN abiertas de P2S que usan Autenticación de Azure AD](openvpn-azure-ad-tenant.md).