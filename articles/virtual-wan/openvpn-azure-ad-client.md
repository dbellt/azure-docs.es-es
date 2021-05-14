---
title: 'VPN Gateway: cliente VPN para conexiones de punto a sitio con el protocolo OpenVPN: Autenticación de Azure AD'
description: Aprenda a usar la P2S VPN para conectarse a la red virtual con la autenticación de Azure AD.
services: vpn-gateway
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 04/26/2021
ms.author: cherylmc
ms.openlocfilehash: 13587862f079c028a9848e36afcff74a9c725547
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108207508"
---
# <a name="configure-a-vpn-client-for-p2s-openvpn-protocol-connections-azure-ad-authentication"></a>Configuración de un cliente VPN para conexiones P2S de protocolo OpenVPN: Autenticación de Azure AD

Este artículo le ayuda a configurar un cliente VPN para conectarse a una red virtual mediante una VPN de punto a sitio y la autenticación Azure Active Directory. Antes de poder conectarse y autenticarse con Azure AD, primero debe configurar el inquilino de Azure AD. Para más información, consulte [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md).

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN®.
>

## <a name="working-with-client-profiles"></a><a name="profile"></a>Trabajar con perfiles de cliente

Por cada equipo que quiera conectarse a la red virtual a través del cliente VPN, debe descargar el cliente VPN de Azure para el equipo y también configurar un perfil de cliente VPN. Si quiere configurar varios equipos, puede crear un perfil de cliente en un equipo, exportarlo y, después, importarlo a otros equipos.

### <a name="to-download-the-azure-vpn-client"></a>Para descargar el cliente VPN de Azure

1. Descargue el [Cliente VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554) en el equipo.
1. Compruebe que el cliente VPN de Azure tenga permiso para ejecutarse en segundo plano. Para comprobar los permisos y habilitarlos, navegue a **Inicio -> Configuración -> Privacidad -> Aplicaciones en segundo plano**.

   * En **Aplicaciones en segundo plano**, asegúrese de que el valor **Permitir que las aplicaciones se ejecuten en segundo plano** sea **Activado**.
   * En **Elegir qué aplicaciones se pueden ejecutar en segundo plano**, cambie la configuración del **Cliente VPN de Azure** a **Activado**.

     ![Captura de pantalla que muestra las aplicaciones en segundo plano.](./media/openvpn-azure-ad-client/backgroundpermission.png)

### <a name="to-create-a-certificate-based-client-profile"></a><a name="cert"></a>Para crear un perfil de cliente basado en certificados

Cuando trabaje con un perfil basado en certificados, asegúrese de que los certificados adecuados están instalados en el equipo cliente. Para más información acerca de los certificados, consulte [Instalar certificados de cliente](certificates-point-to-site.md).

![Captura de pantalla que muestra la autenticación de certificados.](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="to-create-a-radius-client-profile"></a><a name="radius"></a>Para crear un perfil de cliente RADIUS

![Captura de pantalla que muestra la información del cliente de la conexión RADIUS.](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> El secreto de servidor se puede exportar en el perfil de cliente de P2S VPN. Para exportar un perfil de cliente, consulte [Perfiles de cliente de VPN de usuario](about-vpn-profile-download.md).
>

### <a name="to-export-and-distribute-a-client-profile"></a><a name="export"></a>Para exportar y distribuir un perfil de cliente

Una vez que tenga un perfil de trabajo y necesite distribuirlo a otros usuarios, puede exportarlo mediante los siguientes pasos:

1. Resalte el perfil de cliente de VPN que quiere exportar, seleccione el **...** y,luego, seleccione **Exportar**.

    ![Captura de pantalla que muestra la opción Exportar seleccionada en el menú.](./media/openvpn-azure-ad-client/export/export1.jpg)

2. Seleccione la ubicación en la que desea guardar este perfil, deje el nombre de archivo tal cual y, a continuación, seleccione **Guardar** para guardar el archivo xml.

    ![Captura de pantalla que muestra el cuadro de diálogo Guardar como donde puede especificar un nombre de archivo.](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="to-import-a-client-profile"></a><a name="import"></a>Para importar un perfil de cliente

1. En la página, seleccione **Importar**.

    ![Captura de pantalla que muestra la opción Importar seleccionada en el menú Más.](./media/openvpn-azure-ad-client/import/import1.jpg)

2. Busque el archivo xml de perfil y selecciónelo. Con el archivo seleccionado, seleccione **Abrir**.

    ![Captura de pantalla que muestra un cuadro de diálogo Abrir en el que puede seleccionar un archivo.](./media/openvpn-azure-ad-client/import/import2.jpg)

3. Especifique el nombre del perfil y seleccione **Guardar**.

    ![Captura de pantalla que muestra la opción Nombre de conexión agregada y el botón Guardar seleccionado.](./media/openvpn-azure-ad-client/import/import3.jpg)

4. Seleccione **Conectar** para conectarse a la VPN.

    ![Captura de pantalla que muestra el botón Conectar para la conexión que acaba de crear.](./media/openvpn-azure-ad-client/import/import4.jpg)

5. Una vez conectado, el icono se volverá verde y dirá **Conectado**.

    ![Captura de pantalla que muestra la conexión con el estado Conectado y la opción para desconectarla.](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Para eliminar un perfil de cliente

1. Seleccione los puntos suspensivos junto al perfil de cliente que desea eliminar. Después, seleccione **Quitar**.

    ![Captura de pantalla que muestra la opción Quitar seleccionada en el menú.](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. Seleccione **Quitar** para eliminar.

    ![Captura de pantalla que muestra un cuadro de diálogo de confirmación con la opción Quitar o Cancelar.](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="create-a-connection"></a><a name="connection"></a>Crear una conexión

1. En la página, seleccione **+** , después **+ Agregar**.

    ![Captura de pantalla que muestra la opción Agregar selección en el menú Más.](./media/openvpn-azure-ad-client/create/create1.jpg)

2. Rellene la información de conexión. Si no está seguro de los valores, póngase en contacto con el administrador. Después de rellenar los valores, seleccione **Guardar**.

    ![Captura de pantalla que muestra el panel donde puede especificar los valores necesarios.](./media/openvpn-azure-ad-client/create/create2.jpg)

3. Seleccione **Conectar** para conectarse a la VPN.

    ![Captura de pantalla que muestra el botón Conectar de su conexión.](./media/openvpn-azure-ad-client/create/create3.jpg)

4. Seleccione las credenciales adecuadas y, después, seleccione **Continuar**.

    ![Captura de pantalla que muestra el cuadro de diálogo Iniciar sesión.](./media/openvpn-azure-ad-client/create/create4.jpg)

5. Una vez que se haya conectado correctamente, el icono se volverá verde y dirá **Conectado**.

    ![Captura de pantalla que muestra la conexión con el estado Conectado.](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="to-connect-automatically"></a><a name="autoconnect"></a>Para conectarse automáticamente

Estos pasos le ayudarán a configurar la conexión para que se conecte automáticamente con Always-on.

1. En la página principal del cliente VPN, seleccione **Configuración de VPN**.

    ![La captura de pantalla muestra las conexiones VPN donde puede seleccionar la configuración de VPN.](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. Seleccione **Sí** en el cuadro de diálogo cambiar aplicaciones.

    ![Captura de pantalla que muestra un mensaje de comprobación sobre cómo cambiar aplicaciones.](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. Asegúrese de que la conexión que quiere establecer no está conectada y, luego, resalte el perfil y active la casilla **Conectar automáticamente**.

    ![Captura de pantalla que muestra un cuadro de diálogo Configuración en el que puede seleccionar Conectar automáticamente.](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. Seleccione **Conectar** para iniciar la conexión VPN.

    ![Captura de pantalla que muestra el botón Conectar.](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnóstico de problemas de conexión

1. Para diagnosticar problemas de conexión, puede usar la herramienta **Diagnosticar**. Seleccione el **...** junto a la conexión VPN que quiere diagnosticar para que se muestre el menú. Después, seleccione **Diagnosticar**.

    ![Captura de pantalla que muestra la opción de diagnóstico de la selección en el menú.](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. En la página **Propiedades de conexión**, seleccione **Ejecutar diagnóstico**.

    ![Captura de pantalla que muestra el botón Ejecutar diagnóstico para una conexión.](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. Inicie sesión con sus credenciales.

    ![Captura de pantalla que muestra el cuadro de diálogo Iniciar sesión para esta acción.](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. Ver los resultados del diagnóstico.

    ![Captura de pantalla que muestra los resultados del diagnóstico.](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>Preguntas más frecuentes

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

### <a name="how-do-i-direct-all-traffic-to-the-vpn-tunnel-force-tunnel"></a><a name="force-tunneling"></a>¿Cómo dirijo todo el tráfico al túnel VPN (forzar túnel)?

Puede modificar el archivo XML del perfil descargado y agregar las etiquetas **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>** .

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