---
title: Administración de Wi-Fi de Azure Stack Edge Mini R
description: Se describe cómo usar Azure Portal para administrar Wi-Fi en Azure Stack Edge.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 03/24/2021
ms.author: alkohli
ms.openlocfilehash: a2cc0707c344c3ca537795666a3f60f648026596
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105043774"
---
# <a name="use-the-local-web-ui-to-manage-wireless-connectivity-on-your-azure-stack-edge-mini-r"></a>Uso de la interfaz de usuario web local para administrar la conectividad inalámbrica en su instancia de Azure Stack Edge Mini R

En este artículo se describe cómo administrar la conectividad de red inalámbrica en el dispositivo Azure Stack Edge Mini R. Puede usar la interfaz de usuario web local en el dispositivo Azure Stack Edge Mini R a través de la incorporación, la eliminación y la conexión a perfiles de Wi-Fi.

## <a name="about-wi-fi"></a>Acerca de Wi-Fi

Su dispositivo Azure Stack Edge Mini R puede funcionar al conectarse a la red o a través de una red inalámbrica. El dispositivo tiene un puerto Wi-Fi que debe habilitarse para permitir al dispositivo su conexión a una red inalámbrica. 

Su dispositivo tiene cinco puertos, PUERTO 1 a PUERTO 4 y un quinto puerto Wi-Fi. Este es un diagrama del backplane de un dispositivo Mini R al conectarse a una red inalámbrica.

![Cableado para Wi-Fi](./media/azure-stack-edge-mini-r-deploy-install/wireless-cabled.png)


## <a name="add-connect-to-wi-fi-profile"></a>Incorporación y conexión a un perfil de Wi-Fi

Siga los pasos que se describen a continuación en la interfaz de usuario local del dispositivo para agregar un perfil de Wi-Fi y conectarse a él.

1. Vaya a la página **Introducción** en la interfaz de usuario web local del dispositivo. En el icono **Red**, seleccione **Configurar**.  
    
    En el dispositivo físico hay cinco interfaces de red. PUERTO 1 y PUERTO 2 son interfaces de red de 1 Gbps. El PUERTO 3 y el PUERTO 4 son interfaces de red de 10 Gbps. El quinto puerto es el puerto Wi-Fi. 

    [![Página "Network settings" (Configuración de red) de la interfaz de usuario web local 1](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-1.png#lightbox)  
    
    Seleccione el puerto Wi-Fi y establezca la configuración del puerto. 
    
    > [!IMPORTANT]
    > Se recomienda que configure una dirección IP estática para el puerto Wi-Fi.  

    ![Página 2 "Configuración de red" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-2.png)

    La página **Network** (Red)se actualiza después de aplicar la configuración del puerto Wi-Fi.

    ![Página "Network settings" (Configuración de red) de la interfaz de usuario web local 3](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-4.png)

   
2. Seleccione **Add Wi-Fi profile** (Agregar perfil de Wi-Fi) y cargue el perfil de Wi-Fi. 

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 1](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-1.png)
    
    Un perfil de red inalámbrica contiene el SSID (nombre de red), la clave de contraseña y la información de seguridad para poder conectarse a una red inalámbrica. Puede obtener el perfil de Wi-Fi de su entorno del administrador de red.

    Para obtener información sobre la preparación de los perfiles de Wi-Fi, consulte [Uso de perfiles de Wi-Fi con dispositivos de Azure Stack Edge Mini R](azure-stack-edge-mini-r-use-wifi-profiles.md).

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 2](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-2.png)

    Una vez agregado el perfil, se actualiza la lista de perfiles de Wi-Fi para reflejar el nuevo perfil. El perfil debe mostrar el campo **Connection status** (Estado de la conexión) como **Disconnected** (Desconectado). 

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 3](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-3.png)
    
3. Una vez que el perfil de red inalámbrica se haya cargado correctamente, conéctese a este perfil. Seleccione **Connect to Wi-Fi profile** (Conectarse a perfil de Wi-Fi). 

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 4](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-4.png)

4. Seleccione el perfil de Wi-Fi que agregó en el paso anterior y seleccione **Aplicar**. 

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 5](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-5.png)

    El campo **Connection status** (Estado de la conexión) se debe actualizar a **Conectado** (Conectado). La intensidad de la señal se actualiza para indicar la calidad de la señal. 

    !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 6](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-6.png)

    > [!NOTE]
    > Para transferir grandes cantidades de datos, recomendamos que use una conexión con cable en lugar de una red inalámbrica. 


## <a name="download-wi-fi-profile"></a>Descarga de un perfil de Wi-Fi

Puede descargar un perfil de Wi-Fi que utilice para la conectividad de red inalámbrica.

1. En la interfaz de usuario web local del dispositivo, vaya a **Configuración > Red**. 

2. En la configuración de perfil de Wi-Fi, seleccione **Descargar perfil**. Esto debería descargar el perfil de Wi-Fi que utiliza actualmente.


## <a name="delete-wi-fi-profile"></a>Eliminación de un perfil de Wi-Fi

Puede eliminar un perfil de Wi-Fi que utilice para la conectividad de red inalámbrica.


1. En la interfaz de usuario web local del dispositivo, vaya a **Configuración > Red**. 

2. En la configuración del perfil de Wi-Fi, seleccione **Delete Wi-Fi profile** (Eliminar perfil de Wi-Fi).

3. En la hoja **Delete Wi-Fi profile** (Eliminar perfil de Wi-Fi), elija el perfil que desea eliminar. Seleccione **Aplicar**.


## <a name="configure-cisco-wi-fi-profile"></a>Configuración del perfil de Wi-Fi de Cisco

A continuación se ofrecen instrucciones sobre cómo administrar y configurar un punto de acceso y un controlador inalámbrico de Cisco en el dispositivo. 

### <a name="dhcp-bridging-mode"></a>Modo puente DHCP

Para usar un controlador inalámbrico de Cisco para el dispositivo, debe habilitar el modo puente del protocolo de configuración dinámica de host (DHCP) en el controlador de LAN inalámbrico (WLC).

Para obtener más información, consulte el apartado sobre el [modo puente DHCP](https://www.cisco.com/c/en/us/support/docs/wireless/4400-series-wireless-lan-controllers/110865-dhcp-wlc.html#anc9).

#### <a name="bridging-configuration-example"></a>Ejemplo de configuración de puente

Para habilitar la funcionalidad de puente DHCP en el controlador, debe deshabilitar la característica de proxy DHCP en el controlador. Para habilitar el puente DHCP mediante la línea de comandos:

```powershell
(Cisco Controller) > config dhcp proxy disable
(Cisco Controller) > show dhcp proxy
DHCP Proxy Behaviour: disabled
```

Si el servidor DHCP no existe en la misma red de nivel 2 (L2) que el cliente, la difusión debe reenviarse al servidor DHCP en la puerta de enlace cliente mediante un asistente IP. Este es un ejemplo de esta configuración:

```powershell
Switch#conf t
Switch(config)#interface vlan <client vlan #>
Switch(config-if)#ip helper-address <dhcp server IP>
```

La característica de puente DHCP es una configuración global, por lo que afecta a todas las transacciones DHCP dentro del controlador. Debe agregar instrucciones del asistente IP en la infraestructura cableada para todas las redes de área local virtual (VLAN) necesarias en el controlador.

### <a name="enable-the-passive-client-for-wlan"></a>Habilitación del cliente pasivo para WLAN

Para habilitar la característica de cliente pasivo para la red de área local inalámbrica (WLAN) en un controlador inalámbrico de Cisco:

* La interfaz asociada a la WLAN debe tener habilitada un etiquetado de VLAN.
* La VLAN de multidifusión debe estar habilitada para la WLAN.
* El reenvío de GARP debe habilitarse en el WLC.

Para obtener más información, consulte la [información de VLAN de multidifusión sobre la optimización de multidifusión](https://www.cisco.com/c/en/us/td/docs/wireless/controller/8-5/config-guide/b_cg85/wlan_interfaces.html).

### <a name="troubleshoot"></a>Solución de problemas

Si tiene problemas con las asignaciones de direcciones IP en máquinas virtuales que se ejecutan en un dispositivo Azure Stack Edge Mini R, se deben validar las opciones de configuración anteriores en el entorno de red.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información acerca de cómo [implementar el dispositivo Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-prep.md).
