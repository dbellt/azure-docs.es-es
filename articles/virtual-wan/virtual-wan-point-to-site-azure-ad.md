---
title: 'Configuración de la autenticación de Azure AD para la conexión VPN de usuario: Red WAN virtual'
description: Aprenda a configurar la autenticación de Azure Active Directory para una VPN de usuario de Virtual WAN (punto a sitio).
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 10/14/2020
ms.author: alzam
ms.openlocfilehash: f16a7675805fa2665c25b5d4a9c3847b710ec71b
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164208"
---
# <a name="configure-azure-active-directory-authentication-for-user-vpn"></a>Configuración de la autenticación de Azure Active Directory para una VPN de usuario

En este tutorial se muestra cómo configurar la autenticación de Azure AD para una VPN de usuario en Virtual WAN para conectarse a los recursos de Azure mediante una conexión de VPN OpenVPN. La autenticación de Azure Active Directory solo está disponible para puertas de enlace que usan el protocolo OpenVPN y los clientes que ejecutan Windows.

Este tipo de conexión requiere que se configure un cliente en el equipo cliente. Para más información sobre Virtual WAN, consulte la [Introducción a Virtual WAN](virtual-wan-about.md).

En este artículo aprenderá a:

* Crear una instancia de Virtual WAN
* Crear concentrador virtual
* Crear una configuración de VPN de usuario
* Descargar un perfil de VPN de usuario de WAN virtual
* Aplicar la configuración de VPN de usuario a un centro de conectividad virtual
* Conectar una red virtual a un concentrador virtual
* Descargar y aplicar la configuración de cliente VPN
* Ver la instancia de Virtual WAN

![Diagrama de Virtual WAN](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Tiene una red virtual a la que quiere conectarse. Compruebe que ninguna de las subredes de sus redes locales se superpone a las redes virtuales a las que quiere conectarse. Para crear una red virtual en Azure Portal consulte este [Inicio rápido](../virtual-network/quick-create-portal.md).

* Su red virtual no tiene ninguna puerta de enlace de red virtual. Si la red virtual tiene alguna puerta de enlace (ya sea VPN o ExpressRoute), tiene que quitarla. Esta configuración requiere que las redes virtuales estén conectadas a la puerta de enlace del centro de conectividad de Virtual WAN.

* Obtenga un intervalo de direcciones IP para la región del concentrador. El centro de conectividad es una red virtual que Virtual WAN crea y usa. El intervalo de direcciones que especifique para el centro de conectividad no se puede superponer a ninguna de las redes virtuales existentes a las que ya esté conectado. Igualmente no se puede superponer a los intervalos de direcciones con las que esté conectadas en el entorno local. Si no está familiarizado con los intervalos de direcciones IP ubicados en la configuración de la red local, póngase de acuerdo con alguien que pueda proporcionarle estos detalles.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Creación de una instancia de Virtual WAN

Desde un explorador, navegue al [Portal de Azure](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

1. Vaya a la página de Virtual WAN. En el portal, haga clic en **+Crear un recurso**. Escriba **Virtual WAN** en el cuadro de búsqueda y seleccione ENTRAR.
2. Seleccione **Virtual WAN** en los resultados. En la página Virtual WAN, haga clic en **Crear** para abrir la página Crear una red WAN.
3. Dentro de la página **Crear una red WAN**, en la pestaña **Aspectos básicos**, rellene los campos siguientes:

   ![Red WAN virtual](./media/virtual-wan-point-to-site-azure-ad/vwan.png)

   * **Suscripción**: seleccione la suscripción que quiere usar.
   * **Grupo de recursos**: cree uno nuevo o utilice uno ya existente.
   * **Ubicación del grupo de recursos**: elija una ubicación para los recursos en la lista desplegable. Una red WAN es un recurso global y no reside en una región determinada. De todas formas, tiene que seleccionar una región con el fin de administrar y ubicar más fácilmente el recurso WAN que cree.
   * **Nombre**: escriba el nombre que desea dar a la WAN.
   * **Tipo:** Estándar. Si crea una WAN básica, solo puede crear un centro de conectividad básico. Los centros de conectividad básicos solo pueden tener conectividad VPN de sitio a sitio.
4. Cuando termine de rellenar los campos, haga clic en **Revisión y creación**.
5. Una vez que se haya superado la validación, seleccione **Crear** para crear la red WAN virtual.

## <a name="create-an-empty-virtual-hub"></a><a name="site"></a>Creación de un centro virtual vacío

1. En la red WAN virtual, seleccione Centros de conectividad y haga clic en **+Nuevo centro de conectividad**.

   ![Captura de pantalla que muestra el cuadro de diálogo Configuración de centros de conectividad con la opción Nuevo centro de conectividad seleccionada.](media/virtual-wan-point-to-site-azure-ad/hub1.jpg)
2. En la página Crear centro de conectividad virtual, complete los siguientes campos.

   **Región**: seleccione la región en la que quiere implementar el centro de conectividad virtual.

   **Nombre**: escriba el nombre que quiere asignar al centro de conectividad virtual.

   **Espacio de direcciones privadas del centro de conectividad**: intervalo de direcciones del centro de conectividad en la notación CIDR.

   ![Captura de pantalla que muestra el panel Crear centro de conectividad virtual, donde puede escribir valores.](media/virtual-wan-point-to-site-azure-ad/hub2.jpg)  
3. Haga clic en **Revisar + crear**.
4. En la página **Validación superada**, haga clic en **Crear**.

## <a name="create-a-new-user-vpn-configuration"></a><a name="site"></a> Creación de una configuración de VPN de usuario

Una configuración de VPN de usuario define los parámetros para conectarse a los clientes remotos.

1. En la red WAN virtual, seleccione **Configuraciones de VPN de usuario**.

   ![Captura de pantalla que muestra el elemento de menú Configuraciones de VPN de usuario seleccionado.](media/virtual-wan-point-to-site-azure-ad/aadportal1.jpg)

2. Haga clic en **+Create user VPN config** (+Crear configuración de VPN de usuario).

   ![Captura de pantalla que muestra el vínculo Crear configuración de VPN de usuario.](media/virtual-wan-point-to-site-azure-ad/aadportal2.jpg)

3. Escriba la información y haga clic en **Crear**.

   * **Nombre de la configuración**: escriba el nombre que desea asignar a la configuración de VPN de usuario.
   * **Tipo de túnel**: seleccione OpenVPN.
   * **Método de autenticación**: seleccione Azure Active Directory.
   * **Público**: escriba el identificador de aplicación de la [aplicación de Azure VPN](openvpn-azure-ad-tenant.md) Enterprise registrada en el inquilino de Azure AD. 
   * **Emisor** - `https://sts.windows.net/<your Directory ID>/`
   * **Inquilino de AAD** - `https://login.microsoftonline.com/<your Directory ID>`
  
   ![Captura de pantalla que muestra el panel Crear configuración de VPN de usuario, donde puede especificar los valores.](media/virtual-wan-point-to-site-azure-ad/aadportal3.jpg)

## <a name="edit-hub-assignment"></a><a name="hub"></a>Edición de la asignación del concentrador

1. Vaya a la hoja **Centros de conectividad** de la red virtual WAN.
2. Seleccione el concentrador al que desea asociar la configuración del servidor VPN y haga clic en los puntos suspensivos (...).

   ![Captura de pantalla que muestra la opción Editar centro de conectividad virtual seleccionada en el menú.](media/virtual-wan-point-to-site-azure-ad/p2s4.jpg)
3. Haga clic en **Editar centro de conectividad virtual**.
4. Active la casilla **Incluir puerta de enlace de punto a sitio** y elija la **unidad de escalado de puerta de enlace** que quiera.

   ![Captura de pantalla que muestra el cuadro de diálogo Editar centro de conectividad virtual, donde puede seleccionar la unidad de escala de la puerta de enlace.](media/virtual-wan-point-to-site-azure-ad/p2s2.jpg)
5. Especifique el **Grupo de direcciones** del que se asignarán direcciones IP a los clientes VPN.
6. Haga clic en **Confirmar**.
7. La operación puede tardar hasta 30 minutos en completarse.

## <a name="download-user-vpn-profile"></a><a name="device"></a>Descarga de un perfil de VPN de usuario

Use el perfil de VPN para configurar los clientes.

1. En la página de su red WAN virtual, haga clic en **Configuraciones de VPN de usuario**.
2. En la parte superior de la página, haga clic en **Download user VPN config** (Descargar configuración de VPN de usuario).
3. Una vez el archivo se haya terminado de crear, puede hacer clic en el vínculo para descargarlo.
4. Use el archivo de perfil para configurar los clientes de VPN.

## <a name="configure-user-vpn-clients"></a>Configuración de clientes de VPN de usuario

Para conectarse, debe descargar el cliente de VPN de Azure e importar el perfil de cliente de VPN que descargó en los pasos anteriores en todos los equipos que quieran conectarse a la red virtual.

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN&reg;.
>

#### <a name="to-download-the-azure-vpn-client"></a>Para descargar el cliente VPN de Azure

Use este [vínculo](https://www.microsoft.com/p/azure-vpn-client-preview/9np355qt2sqb?rtc=1&activetab=pivot:overviewtab) para descargar el cliente VPN de Azure.

#### <a name="to-import-a-client-profile"></a><a name="import"></a>Para importar un perfil de cliente

1. En la página, seleccione **Importar**.

    ![Captura de pantalla que muestra la opción Importar seleccionada en el menú Más.](./media/virtual-wan-point-to-site-azure-ad/import/import1.jpg)

2. Busque el archivo xml de perfil y selecciónelo. Con el archivo seleccionado, seleccione **Abrir**.

    ![Captura de pantalla que muestra un cuadro de diálogo Abrir en el que puede seleccionar un archivo.](./media/virtual-wan-point-to-site-azure-ad/import/import2.jpg)

3. Especifique el nombre del perfil y seleccione **Guardar**.

    ![Captura de pantalla que muestra la opción Nombre de conexión agregada y el botón Guardar seleccionado.](./media/virtual-wan-point-to-site-azure-ad/import/import3.jpg)

4. Seleccione **Conectar** para conectarse a la VPN.

    ![Captura de pantalla que muestra el botón Conectar para la conexión que acaba de crear.](./media/virtual-wan-point-to-site-azure-ad/import/import4.jpg)

5. Una vez conectado, el icono se volverá verde y dirá **Conectado**.

    ![Captura de pantalla que muestra la conexión con el estado Conectado y la opción para desconectarla.](./media/virtual-wan-point-to-site-azure-ad/import/import5.jpg)

#### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Para eliminar un perfil de cliente

1. Seleccione los puntos suspensivos junto al perfil de cliente que quiera eliminar. Después, seleccione **Quitar**.

    ![Captura de pantalla que muestra la opción Quitar seleccionada en el menú.](./media/virtual-wan-point-to-site-azure-ad/delete/delete1.jpg)

2. Seleccione **Quitar** para eliminar.

    ![Captura de pantalla que muestra un cuadro de diálogo de confirmación con la opción Quitar o Cancelar.](./media/virtual-wan-point-to-site-azure-ad/delete/delete2.jpg)

#### <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnóstico de problemas de conexión

1. Para diagnosticar problemas de conexión, puede usar la herramienta **Diagnosticar**. Seleccione los puntos suspensivos (...) junto a la conexión de VPN que desea diagnosticar para que se muestre el menú. Después, seleccione **Diagnosticar**.

    ![Captura de pantalla que muestra la opción de diagnóstico de la selección en el menú.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose1.jpg)

2. En la página **Propiedades de conexión**, seleccione **Ejecutar diagnóstico**.

    ![Captura de pantalla que muestra el botón Ejecutar diagnóstico para una conexión.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose2.jpg)

3. Inicie sesión con sus credenciales.

    ![Captura de pantalla que muestra el cuadro de diálogo Iniciar sesión para esta acción.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose3.jpg)

4. Ver los resultados del diagnóstico.

    ![Captura de pantalla que muestra los resultados del diagnóstico.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose4.jpg)

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Visualización de la instancia de Virtual WAN

1. Vaya a la instancia de Virtual WAN.
2. En la página Información general, cada punto del mapa representa un concentrador.
3. En la sección de concentradores y conexiones, puede ver estado del concentrador, sitio, región, estado de la conexión VPN y bytes de entrada y salida.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Limpieza de recursos

Cuando ya no necesite estos recursos, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene. Reemplace "myResourceGroup" con el nombre del grupo de recursos y ejecute el siguiente comando de PowerShell:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte la página [Introducción a Virtual WAN](virtual-wan-about.md).
