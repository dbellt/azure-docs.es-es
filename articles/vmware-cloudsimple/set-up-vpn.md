---
title: 'Azure VMware Solution by CloudSimple: configuración de VPN entre la red local y la nube privada'
description: Describe cómo configurar una conexión VPN de sitio a sitio o de punto a sitio entre la red local y la nube privada de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 49dc804e4dc482f49ca21354d594a84092b8868b
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108180903"
---
# <a name="configure-a-vpn-connection-to-your-cloudsimple-private-cloud"></a>Configuración de una conexión VPN en la nube privada de CloudSimple

Las puertas de enlace de VPN le permiten conectarse a la red de CloudSimple desde la red local y desde un equipo cliente de forma remota.  En este artículo, puede encontrar información sobre cómo configurar las puertas de enlace de VPN desde el portal de CloudSimple.  Una conexión VPN entre la red local y la red de CloudSimple proporciona acceso a vCenter y a cargas de trabajo en la nube privada. CloudSimple admite puertas de enlace de VPN de sitio a sitio y de punto a sitio.

## <a name="vpn-gateway-types"></a>Tipos de puertas de enlace de VPN

* La conexión **VPN de punto a sitio** es la manera más sencilla de conectarse a la nube privada desde su equipo. Use la conectividad de VPN de punto a sitio para conectarse a la nube privada de forma remota.
* La conexión **VPN de sitio a sitio** le permite configurar las cargas de trabajo de la nube privada para acceder a los servicios locales. También puede usar Active Directory local como origen de identidad para autenticarse en el servidor vCenter de la nube privada.  Actualmente, se admite el tipo de **VPN basada en directivas**.

En una región, puede crear una puerta de enlace de VPN de punto a sitio y una puerta de enlace de VPN de sitio a sitio.

## <a name="point-to-site-vpn"></a>VPN de punto a sitio

Para crear una puerta de enlace de VPN de punto a sitio, consulte [Creación de puerta de enlace de VPN de punto a sitio](vpn-gateway.md#create-point-to-site-vpn-gateway).

### <a name="connect-to-cloudsimple-using-point-to-site-vpn"></a>Conexión a CloudSimple mediante VPN de punto a sitio

Para conectarse a CloudSimple desde su equipo, es necesario el cliente VPN.  Descargue el [cliente OpenVPN](https://openvpn.net/community-downloads/) para Windows o [Viscosity](https://www.sparklabs.com/viscosity/download/) para macOS y OS X.

1. Inicie el portal de CloudSimple y seleccione **Network** (Red).
2. Seleccione **VPN Gateway** (Puerta de enlace de VPN).
3. En la lista de puertas de enlace de VPN, haga clic en la puerta de enlace de VPN de punto a sitio.
4. Seleccione **Usuarios**.
5. Haga clic en **Download my VPN configuration** (Descargar mi configuración de VPN)

    ![Descarga de la configuración de VPN](media/download-p2s-vpn-configuration.png)

6. Importación de la configuración en el cliente VPN

    * Instrucciones para [importar la configuración en el cliente Windows](https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-windows/#openvpn-open-source-openvpn-gui-program)
    * Instrucciones para [importar la configuración en macOS u OS X](https://www.sparklabs.com/support/kb/article/getting-started-with-viscosity-mac/#creating-your-first-connection)

7. Conéctese a la puerta de enlace de VPN de CloudSimple.

En el ejemplo siguiente se muestra cómo importar la conexión mediante el **cliente Viscosity**.

#### <a name="import-connection-on-viscosity-client"></a>Importación de la conexión en el cliente Viscosity

1. Extraiga el contenido de la configuración de VPN del archivo. zip descargado.

2. Abra Viscosity en el equipo.

3. Haga clic en el icono **+** y seleccione **Import connection** > **From File** (Importar conexión > Desde archivo).

    ![Importación de la configuración de VPN desde archivo](media/import-p2s-vpn-config.png)

4. Seleccione el archivo de configuración OpenVPN (.ovpn) para el protocolo que desea usar y haga clic en **Abrir**.

    ![Captura de pantalla en la que se resaltan los archivos de configuración de OpenVPN que puede seleccionar.](media/import-p2s-vpn-config-choose-ovpn.png)

La conexión aparecerá ahora en el menú de Viscosity.

#### <a name="connect-to-the-vpn"></a>Conéctese a la VPN.

Para conectarse a una VPN mediante el cliente Viscosity OpenVPN, seleccione la conexión en el menú. El icono del menú se actualiza para indicar que se ha establecido la conexión.

![Captura de pantalla que muestra el estado de conectividad de VPN de CloudSimple.](media/vis03.png)

### <a name="connecting-to-multiple-private-clouds"></a>Conexión a varias nubes privadas

Una conexión VPN de punto a sitio resuelve los nombres DNS de la primera nube privada que cree. Si desea acceder a otras nubes privadas, debe actualizar el servidor DNS en el cliente VPN.

1. Inicie el [portal de CloudSimple](access-cloudsimple-portal.md).

2. Vaya a **Recursos** > **Nubes privadas** y seleccione la nube privada a la que desee conectarse.

3. En la página **Resumen** de la nube privada, copie la dirección IP del servidor DNS de la nube privada en **Información básica**.

    ![Servidores DNS de la nube privada](media/private-cloud-dns-server.png)

4. Haga clic con el botón derecho en el icono Viscosity de la bandeja del sistema del equipo y seleccione **Preferencias**.

    ![VPN](media/vis00.png)

5. Seleccione la conexión VPN de CloudSimple.

    ![Conexión VPN](media/viscosity-client.png)

6. Haga clic en **Editar** para cambiar las propiedades de conexión.

    ![Edición de la conexión VPN](media/viscosity-edit-connection.png)

7. Haga clic en la pestaña **Redes** y escriba las direcciones IP del servidor DNS de la nube privada separadas por una coma o un espacio, con el dominio ```cloudsimple.io```.  Seleccione **Ignore DNS settings sent by VPN server** (Omitir configuración DNS enviada por el servidor VPN).

    ![Redes VPN](media/viscosity-edit-connection-networking.png)

> [!IMPORTANT]
> Para conectarse a su primera nube privada, quite esta configuración y conéctese al servidor VPN.

## <a name="site-to-site-vpn"></a>VPN de sitio a sitio

Para crear una puerta de enlace de VPN de sitio a sitio, consulte [Creación de una puerta de enlace de VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway).  La conexión VPN de sitio a sitio desde su red local a la nube privada proporciona las siguientes ventajas.  

* Accesibilidad de su vCenter de nube privada desde cualquier estación de trabajo en la red local.
* Uso de Active Directory local como origen de identidad de vCenter.
* Transferencia cómoda de plantillas de máquina virtual, imágenes ISO y otros archivos de los recursos locales a vCenter de la nube privada.
* Accesibilidad de las cargas de trabajo que se ejecutan en la nube privada desde la red local.

Para configurar la puerta de enlace de VPN local en modo de alta disponibilidad, consulte [Configuración de una conexión VPN de alta disponibilidad](high-availability-vpn-connection.md).

> [!IMPORTANT]
>    1. Establezca la fijación de MSS de TCP en 1200 en el dispositivo VPN. O, si los dispositivos de VPN no admiten la fijación de MSS, también puede establecer el MTU en la interfaz de túnel en 1240 bytes.
> 2. Una vez configurada la VPN de sitio a sitio, reenvíe las solicitudes DNS para *.cloudsimple.io a los servidores DNS de la nube privada.  Siga las instrucciones de [Configuración de DNS local](on-premises-dns-setup.md).
