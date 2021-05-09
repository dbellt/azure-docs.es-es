---
title: 'Azure VMware Solution by CloudSimple: configuración de una puerta de enlace de VPN'
description: Describe cómo configurar la puerta de enlace de VPN de punto a sitio y la puerta de enlace de VPN de sitio a sitio y cómo crear conexiones entre la red local y la nube privada de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 5d67958b05bd07e97b46a8ad3812b7ff0ca41f03
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108178149"
---
# <a name="set-up-vpn-gateways-on-cloudsimple-network"></a>Configuración de puertas de enlace de VPN en la red de CloudSimple

Las puertas de enlace de VPN le permiten conectarse a la red de CloudSimple desde la red local y desde un equipo cliente de forma remota. Una conexión VPN entre la red local y la red de CloudSimple proporciona acceso a vCenter y a cargas de trabajo en la nube privada. CloudSimple admite puertas de enlace de VPN de sitio a sitio y de punto a sitio.

## <a name="vpn-gateway-types"></a>Tipos de puertas de enlace de VPN

* La conexión **VPN de sitio a sitio** le permite configurar las cargas de trabajo de la nube privada para acceder a los servicios locales. También puede usar Active Directory local como origen de identidad para autenticarse en el servidor vCenter de la nube privada.  Actualmente, solo se admite el tipo de **VPN basada en directivas**.
* La conexión **VPN de punto a sitio** es la manera más sencilla de conectarse a la nube privada desde su equipo. Use la conectividad VPN de punto a sitio si se conecta a la nube privada de forma remota. Para obtener información sobre cómo instalar un cliente para una conexión VPN de punto a sitio, consulte [Configuración de una conexión VPN a la nube privada](set-up-vpn.md).

En una región, puede crear una puerta de enlace de VPN de punto a sitio y una puerta de enlace de VPN de sitio a sitio.

## <a name="automatic-addition-of-vlansubnets"></a>Adición automática de VLAN/subredes

Las puertas de enlace de VPN de CloudSimple proporcionan directivas para agregar VLAN/subredes a las puertas de enlace de VPN.  Las directivas permiten especificar diferentes reglas para las VLAN/subredes de administración y las VLAN/subredes definidas por el usuario.  Las reglas para las VLAN/subredes de administración se aplican a todas las nubes privadas nuevas que cree.  Las reglas para las VLAN/subredes definidas por el usuario permiten agregar automáticamente nuevas VLAN/subredes a nubes privadas nuevas o existentes para una puerta de enlace de VPN de sitio a sitio. Deberá definir la directiva para cada conexión.

Las directivas sobre la adición de VLAN/subredes a puertas de enlace de VPN se aplican tanto a puertas de enlace de VPN de sitio a sitio como a puertas de enlace de VPN de punto a sitio.

## <a name="automatic-addition-of-users"></a>Adición automática de usuarios

Una puerta de enlace de VPN de punto a sitio permite definir una directiva de adición automática para los nuevos usuarios. De forma predeterminada, todos los propietarios y colaboradores de la suscripción tienen acceso al portal de CloudSimple.  Los usuarios solo se crean cuando se inicia el portal de CloudSimple por primera vez.  La selección de reglas de **Automatically add** (Agregar automáticamente) permite a cualquier usuario nuevo acceder a la red de CloudSimple mediante una conexión VPN de punto a sitio.

## <a name="set-up-a-site-to-site-vpn-gateway"></a>Configuración de una puerta de enlace de VPN de sitio a sitio

1. [Acceda](access-cloudsimple-portal.md) al portal de CloudSimple y seleccione **Network** (Red).
2. Seleccione **VPN Gateway** (Puerta de enlace de VPN).
3. Haga clic en **New VPN Gateway** (Nueva puerta de enlace de VPN).

    ![Creación de una puerta de enlace de VPN](media/create-vpn-gateway.png)

4. En **Gateway Configuration** (Configuración de puerta de enlace), especifique la configuración que se indica a continuación y haga clic en **Next** (Siguiente).

    * Seleccione **Site-to-Site VPN** (VPN de sitio a sitio) como el tipo de puerta de enlace.
    * Escriba un nombre para identificar la puerta de enlace.
    * Seleccione la ubicación de Azure en que se va a implementar el servicio de CloudSimple.
    * Opcionalmente, habilite la alta disponibilidad.

    ![Creación de una puerta de enlace de VPN de sitio a sitio](media/create-vpn-gateway-s2s.png)

    > [!WARNING]
    > Para habilitar la alta disponibilidad, es necesario que el dispositivo VPN local admita la conexión a dos direcciones IP. Esta opción no se puede deshabilitar una vez implementada la puerta de enlace de VPN.

5. Cree la primera conexión desde la red local y haga clic en **Next** (Siguiente).

    * Escriba un nombre para identificar la conexión.
    * Para la IP del mismo nivel, escriba la dirección IP pública de la puerta de enlace de VPN local.
    * Escriba el identificador del mismo nivel de la puerta de enlace de VPN local.  El identificador del mismo nivel suele ser la dirección IP pública de la puerta de enlace de VPN local.  Si ha configurado un identificador específico en la puerta de enlace, escriba el identificador.
    * Copie la clave compartida que se usará para la conexión desde la puerta de enlace de VPN local.  Para cambiar la clave compartida predeterminada y especificar una nueva, haga clic en el icono de edición.
    * En **On-Premises Prefixes** (Prefijos locales), escriba los prefijos CIDR locales que tendrán acceso a la red de CloudSimple.  Puede agregar varios prefijos CIDR al crear la conexión.

    ![Creación de una conexión de puerta de enlace de VPN de sitio a sitio](media/create-vpn-gateway-s2s-connection.png)

6. Habilite las VLAN/subredes de la red de nube privada a las que se tendrá acceso desde la red local y haga clic en **Siguiente**.

    * Para agregar una VLAN/subred de administración, habilite **Add management VLANs/Subnets of Private Clouds** (Agregar VLAN y subredes de administración de nubes privadas).  La subred de administración es necesaria para subredes de vMotion y vSAN.
    * Para agregar subredes de vMotion, habilite **Add vMotion network of Private Clouds** (Agregar red de vMotion de nubes privadas).
    * Para agregar subredes de vSAN, habilite **Add vSAN subnet of Private Clouds** (Agregar subred de vSAN de nubes privadas).
    * Seleccione o anule la selección de VLAN específicas.

    ![Crear conexión](media/create-vpn-gateway-s2s-connection-vlans.png)

7. Revise la configuración y haga clic en **Enviar**.

    ![Revisión y creación de puerta de enlace de VPN de sitio a sitio](media/create-vpn-gateway-s2s-review.png)

## <a name="create-point-to-site-vpn-gateway"></a>Creación de puerta de enlace de VPN de punto a sitio

1. [Acceda](access-cloudsimple-portal.md) al portal de CloudSimple y seleccione **Network** (Red).
2. Seleccione **VPN Gateway** (Puerta de enlace de VPN).
3. Haga clic en **New VPN Gateway** (Nueva puerta de enlace de VPN).

    ![Creación de una puerta de enlace de VPN](media/create-vpn-gateway.png)

4. En **Gateway Configuration** (Configuración de puerta de enlace), especifique la configuración que se indica a continuación y haga clic en **Next** (Siguiente).

    * Seleccione **Point-to-Site VPN** (VPN de punto a sitio) como el tipo de puerta de enlace.
    * Escriba un nombre para identificar la puerta de enlace.
    * Seleccione la ubicación de Azure en que se va a implementar el servicio de CloudSimple.
    * Especifique la subred de cliente para la puerta de enlace de punto a sitio.  Cuando se conecte, se proporcionarán direcciones DHCP desde la subred de cliente.

5. En **Connection/User** (Conexión/Usuario), especifique la configuración que se indica a continuación y haga clic en **Next** (Siguiente).

    * Para permitir automáticamente que todos los usuarios actuales y futuros tengan acceso a la nube privada a través de la puerta de enlace de punto a sitio, seleccione **Automatically add all users** (Agregar automáticamente a todos los usuarios). Al seleccionar esta opción, se seleccionan automáticamente todos los usuarios de la lista de usuarios. Para invalidar la opción automática, anule la selección de usuarios individuales en la lista.
    * Para seleccionar usuarios individuales, haga clic en las casillas de verificación de la lista de usuarios.

6. En la sección VLANs/Subnets (VLAN/Subredes) se pueden especificar las VLAN y las subredes de administración y de usuario para la puerta de enlace y las conexiones.

    * Las opciones de **Automatically add** (Agregar automáticamente) establecen la directiva global para la puerta de enlace. La configuración se aplica a la puerta de enlace actual. Se puede invalidar la configuración en el área **Select** (Seleccionar).
    * Seleccione **Add management VLANs/Subnets of Private Clouds** (Agregar VLAN y subredes de administración de nubes privadas). 
    * Para agregar todas las VLAN y subredes definidas por el usuario, haga clic en **Add user-defined VLANs/Subnets** (Agregar VLAN y subredes definidas por el usuario).
    * La configuración de **Select** (seleccionar) invalida la configuración global en **Automatically add** (Agregar automáticamente).

7. Haga clic en **Next** (Siguiente) para revisar la configuración. Haga clic en los iconos de edición para realizar cambios.
8. Haga clic en **Crete** (Crear) para crear la puerta de enlace VPN.

### <a name="client-subnet-and-protocols-for-point-to-site-vpn-gateway"></a>Subred de cliente y protocolos de puerta de enlace de VPN de punto a sitio

La puerta de enlace de VPN de punto a sitio permite conexiones TCP y UDP.  Elija el protocolo que se usará al conectarse desde el equipo seleccionando la configuración TCP o UDP.

La subred de cliente configurada se usa para los clientes TCP y UDP.  El prefijo CIDR se divide en dos subredes, una para TCP y otra para clientes UDP. Elija la máscara de prefijo en función del número de usuarios de VPN que se conectarán simultáneamente.  

En la tabla siguiente se muestra el número de conexiones de cliente simultáneas para la máscara de prefijo.

| Máscara de prefijo | /24 | /25 | /26 | /27 | /28 |
|-------------|-----|-----|-----|-----|-----|
| Número de conexiones TCP simultáneas | 124 | 60 | 28 | 12 | 4 |
| Número de conexiones UDP simultáneas | 124 | 60 | 28 | 12 | 4 |

Para conectarse mediante VPN de punto a sitio, consulte [Conexión a CloudSimple mediante VPN de punto a sitio](set-up-vpn.md#connect-to-cloudsimple-using-point-to-site-vpn).
