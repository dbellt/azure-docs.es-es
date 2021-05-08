---
title: 'Azure VMware Solution by CloudSimple: configuración de alta disponibilidad de una instancia local a una puerta de enlace VPN de CloudSimple'
description: Se describe cómo configurar una conexión de alta disponibilidad desde el entorno local a una instancia de puerta de enlace VPN de CloudSimple habilitada para alta disponibilidad.
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 44bbee4e179cd77159cd269e69dba4cd0812624e
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176799"
---
# <a name="configure-a-high-availability-connection-from-on-premises-to-cloudsimple-vpn-gateway"></a>Configuración de una conexión de alta disponibilidad desde la ubicación local a la puerta de enlace VPN de CloudSimple

Los administradores de red pueden configurar una conexión VPN de sitio a sitio de alta disponibilidad por IPsec desde su entorno local a una puerta de enlace VPN de CloudSimple.

En esta guía se presentan los pasos para configurar el firewall local para una conexión VPN de sitio a sitio de alta disponibilidad por IPsec. Los pasos detallados son específicos para cada tipo de firewall local. Como ejemplo, en esta guía se presentan los pasos para dos tipos de firewall: Cisco ASA y Palo Alto Networks.

## <a name="before-you-begin"></a>Antes de empezar

Complete las tareas siguientes antes de configurar el firewall local.

1. Compruebe que la organización ha [aprovisionado](create-nodes.md) los nodos necesarios y ha creado al menos una nube privada de CloudSimple.
2. [Configure una puerta de enlace VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre la red local y la nube privada de CloudSimple.

Consulte [Información general sobre puertas de enlace VPN](cloudsimple-vpn-gateways.md) para conocer las propuestas admitidas de fase 1 y fase 2.

## <a name="configure-on-premises-cisco-asa-firewall"></a>Configuración del firewall de Cisco ASA local

Las instrucciones de esta sección se aplican a Cisco ASA versión 8.4 y posteriores. En el ejemplo de configuración, se implementa y configura Cisco Adaptive Security Appliance Software versión 9.10 en modo IKEv1.

Para que la VPN de sitio a sitio funcione, debe permitir UDP 500/4500 y ESP (protocolo IP 50) desde la IP pública principal y secundaria de CloudSimple (IP del mismo nivel) en la interfaz externa de la puerta de enlace VPN de Cisco ASA local.

### <a name="1-configure-phase-1-ikev1"></a>1. Configuración de la fase 1 (IKEv1)

Para habilitar la fase 1 (IKEv1) en la interfaz externa, escriba el siguiente comando de la CLI en el firewall de Cisco ASA.

```crypto ikev1 enable outside```

### <a name="2-create-an-ikev1-policy"></a>2. Creación de una directiva de IKEv1

Cree una directiva de IKEv1 que defina los algoritmos y métodos que se usarán para el hash, la autenticación, el grupo Diffie-Hellman, la duración y el cifrado.

```
crypto ikev1 policy 1
authentication pre-share
encryption aes-256
hash sha
group 2
lifetime 28800
```

### <a name="3-create-a-tunnel-group"></a>3. Creación de un grupo de túneles

Cree un grupo de túneles en los atributos de IPsec. Configure la dirección IP del mismo nivel y la clave previamente compartida del túnel, que estableció [al configurar la puerta de enlace VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway).

```
tunnel-group <primary peer ip> type ipsec-l2l
tunnel-group <primary peer ip> ipsec-attributes
ikev1 pre-shared-key *****

tunnel-group <secondary peer ip> type ipsec-l2l
tunnel-group <secondary peer ip> ipsec-attributes
ikev1 pre-shared-key *****
```

### <a name="4-configure-phase-2-ipsec"></a>4. Configuración de la fase 2 (IPsec)

Para configurar la fase 2 (IPsec), cree una lista de control de acceso (ACL) que defina el tráfico que se va a cifrar y tunelizar. En el ejemplo siguiente, el tráfico de interés pertenece al túnel que procede de la subred local (10.16.1.0/24) y se envía a la subred remota de la nube privada (192.168.0.0/24). La ACL puede contener varias entradas si hay varias subredes entre los sitios.

En las versiones 8.4 y posteriores de Cisco ASA, se pueden crear objetos o grupos de objetos que actúan como contenedores para las redes, subredes, direcciones IP de host o varios objetos. Cree un objeto para las subredes locales y un objeto para las remotas y úselos para la ACL de cifrado y las instrucciones NAT.

#### <a name="define-an-on-premises-local-subnet-as-an-object"></a>Definición de una subred local como un objeto

```
object network AZ_inside
subnet 10.16.1.0 255.255.255.0
```

#### <a name="define-the-cloudsimple-remote-subnet-as-an-object"></a>Definición de la subred remota de CloudSimple como un objeto

```
object network CS_inside
subnet 192.168.0.0 255.255.255.0
```

#### <a name="configure-an-access-list-for-the-traffic-of-interest"></a>Configuración de una lista de acceso para el tráfico de interés

```
access-list ipsec-acl extended permit ip object AZ_inside object CS_inside
```

### <a name="5-configure-the-transform-set"></a>5. Configuración del conjunto de transformaciones

Configure el conjunto de transformaciones (TS), que debe incluir la palabra clave ```ikev1```. Los atributos hash y de cifrado especificados en el conjunto de transformaciones deben coincidir con los parámetros que se muestran en la [Configuración predeterminada de las puertas de enlace VPN de CloudSimple](cloudsimple-vpn-gateways.md).

```
crypto ipsec ikev1 transform-set devtest39 esp-aes-256 esp-sha-hmac 
```

### <a name="6-configure-the-crypto-map"></a>6. Configuración del mapa de cifrado

Configure el mapa de cifrado, que contiene estos componentes:

* Dirección IP del mismo nivel
* ACL definida que contiene el tráfico pertinente
* Conjunto de transformación

```
crypto map mymap 1 set peer <primary peer ip> <secondary peer ip>
crypto map mymap 1 match address ipsec-acl
crypto map mymap 1 set ikev1 transform-set devtest39
```

### <a name="7-apply-the-crypto-map"></a>7. Aplicación del mapa de cifrado

Aplique el mapa de cifrado en la interfaz externa:

```crypto map mymap interface outside```

### <a name="8-confirm-applicable-nat-rules"></a>8. Confirmación de las reglas NAT aplicables

A continuación se encuentra la regla NAT que se usó. Asegúrese de que el tráfico de VPN no está sujeto a ninguna otra regla NAT.

```nat (inside,outside) source static AZ_inside AZ_inside destination static CS_inside CS_inside```

### <a name="sample-ipsec-site-to-site-vpn-established-output-from-cisco-asa"></a>Salida de ejemplo de VPN de sitio a sitio por IPsec de Cisco ASA

Salida de la fase 1:

![Salida de la fase 1 del firewall de Cisco ASA](media/ha-vpn-connection-cisco-phase1.png)

Salida de la fase 2:

![Salida de la fase 2 del firewall de Cisco ASA](media/ha-vpn-connection-cisco-phase2.png)

## <a name="configure-on-premises-palo-alto-networks-firewall"></a>Configuración del firewall de Palo Alto Networks local

Las instrucciones de esta sección se aplican a Palo Alto Networks versión 7.1 y posteriores. En el ejemplo de configuración, se implementa y configura Palo Alto Networks VM-Series Software versión 8.1.0 en modo IKEv1.

Para que la VPN de sitio a sitio funcione, debe permitir UDP 500/4500 y ESP (protocolo IP 50) desde la IP pública principal y secundaria de CloudSimple (IP del mismo nivel) en la interfaz externa de la puerta de enlace de Palo Alto Networks local.

### <a name="1-create-primary-and-secondary-tunnel-interfaces"></a>1. Creación de las interfaces de túnel principal y secundaria

Inicie sesión en el firewall de Palo Alto y seleccione **Network** > **Interfaces** > **Tunnel** > **Add** (Red > Interfaces > Túnel > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

* Interface Name (Nombre de la interfaz). El primer campo se rellena automáticamente con la palabra clave "tunnel". En el campo adyacente, escriba cualquier número entre 1 y 9999. Esta interfaz se usará como una interfaz de túnel principal para transportar el tráfico de sitio a sitio entre el centro de recursos local y la nube privada.
* Comment (Comentario). Escriba comentarios para facilitar la identificación de la finalidad del túnel.
* Perfil de NetFlow. Deje el valor predeterminado.
* Config (Configuración). Asignar interfaz a: enrutador virtual: seleccione el **valor predeterminado**. 
        Zona de seguridad: seleccione la zona para el tráfico de LAN de confianza. En este ejemplo, el nombre de la zona para el tráfico LAN es "Trust".
* IPv4. Haga clic en **Add** (Agregar) y agregue cualquier dirección IP sin usar /32 que no se superponga en su entorno. Esta se asignará a la interfaz de túnel principal y se usará para supervisar los túneles (se explica más adelante).

Dado que esta configuración es para una VPN de alta disponibilidad, se necesitan dos interfaces de túnel: una primaria y una secundaria. Repita los pasos anteriores para crear la interfaz de túnel secundaria. Seleccione otro id. de túnel y otra dirección IP /32 sin usar.

### <a name="2-set-up-static-routes-for-private-cloud-subnets-to-be-reached-over-the-site-to-site-vpn"></a>2. Configuración de rutas estáticas para comunicarse con las subredes de nube privada a través de la VPN de sitio a sitio

Las rutas son necesarias para que las subredes locales puedan comunicarse con las subredes de nube privada de CloudSimple.

Seleccione **Network** > **Virtual Routers** > *default* > **Static Routes** > **Add** (Red > Enrutadores virtuales > predeterminado > rutas estáticas > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

* Name (Nombre). Escriba cualquier nombre para facilitar la identificación de la finalidad de la ruta.
* Destination (Destino). Especifique las subredes de nube privada de CloudSimple con las que se comunicará a través de interfaces de túnel S2S desde el entorno local.
* . Seleccione la interfaz de túnel principal creada en el paso 1 (sección 2) de la lista desplegable. En este ejemplo, es tunnel.20.
* Next Hop (Próximo salto). Seleccione **Ninguno**.
* Admin Distance (Distancia de administrador). Deje el valor predeterminado.
* Metric (Métrica). Escriba cualquier valor entre 1 y 65535. La clave consiste en especificar una métrica inferior para la ruta correspondiente a la interfaz de túnel principal en comparación con la ruta correspondiente a la interfaz de túnel secundaria, de modo que la primera sea la ruta preferida. Si tunnel.20 tiene un valor de métrica de 20 y tunnel.30 tiene un valor de métrica de 30, se prefiere tunnel.20.
* Route Table (Tabla de rutas). Deje el valor predeterminado.
* BFD Profile (Perfil BFD). Deje el valor predeterminado.
* Path monitoring (Supervisión de ruta de acceso). Deje la opción desactivada.

Repita los pasos anteriores para crear otra ruta para subredes de nube privada que se usará como ruta secundaria o ruta de copia de seguridad a través de la interfaz de túnel secundario. Esta vez, seleccione un id. de túnel diferente y una métrica superior a la de la ruta principal.

### <a name="3-define-the-cryptographic-profile"></a>3. Definición del perfil criptográfico

Defina un perfil criptográfico que especifique los protocolos y los algoritmos de identificación, autenticación y cifrado que se usarán para configurar túneles VPN en la fase 1 de IKEv1.

Seleccione **Network** > **Expand Network Profiles** > **IKE Crypto** > **Add** (Red > Expandir perfiles de red > Cifrado de IKE > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

* Name (Nombre). Escriba cualquier nombre de perfil de cifrado de IKE.
* DH Group (Grupo DH). Haga clic en **Agregar** y seleccione el grupo DH adecuado.
* Cifrado. Haga clic en **Agregar** y seleccione el método de cifrado adecuado.
* Autenticación. Haga clic en **Agregar** y seleccione el método de autenticación adecuado.
* Key lifetime (Vigencia de clave). Deje el valor predeterminado.
* IKEv2 Authentication Multiple (Múltiplo de autenticación IKEv2). Deje el valor predeterminado.

### <a name="4-define-ike-gateways"></a>4. Definición de puertas de enlace IKE

Defina puertas de enlace IKE para establecer la comunicación entre los equipos del mismo nivel en cada extremo del túnel VPN.

Seleccione **Network** > **Expand Network Profiles** > **IKE Gateways** > **Add** (Red > Expandir perfiles de red > Puertas de enlace de IKE > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

Pestaña General:

* Name (Nombre). Escriba el nombre de la puerta de enlace de IKE que se va a emparejar con la VPN de CloudSimple principal del mismo nivel.
* Se trata de la versión. Seleccione **IKEv1 only mode** (Modo solo IKEv1).
* Address Type (Tipo de dirección). Seleccione **IPv4**.
* . Seleccione la interfaz pública o externa.
* Local IP Address (Dirección IP local). Deje el valor predeterminado.
* Peer IP Address Type (Tipo de dirección IP del mismo nivel). Seleccione **IP**.
* Peer Address (Dirección del mismo nivel). Escriba la dirección IP del mismo nivel de la VPN de CloudSimple principal.
* Autenticación. Seleccione **Clave previamente compartida** (Clave previamente compartida).
* Pre-shared Key/Confirm Pre-shared Key (Clave previamente compartida/Confirmar clave precompartida). Escriba la clave previamente compartida para que coincida con la clave de puerta de enlace de la VPN de CloudSimple.
* Local Identification (Identificación local). Escriba la dirección IP pública del firewall de Palo Alto local.
* Peer Identification (Identificación del mismo nivel). Escriba la dirección IP del mismo nivel de la VPN de CloudSimple principal.

Pestaña Opciones avanzadas:

* Enable Passive Mode (Habilitar el modo pasivo). Deje la opción desactivada.
* Enable NAT Traversal (Habilitar NAT transversal). Deje la opción desactivada si el firewall de Palo Alto local no está detrás de ningún dispositivo NAT. De lo contrario, active la casilla.

IKEv1:

* Exchange Mode (Modo de intercambio). Seleccione **main** (principal).
* IKE Crypto Profile (Perfil criptográfico de IKE). Seleccione el perfil criptográfico de IKE que creó anteriormente. Deje desactivada la casilla Enable Fragmentation (Habilitar fragmentación).
* Dead Peer Detection (Detección de elementos del mismo nivel inactivos). Deje la casilla desactivada.

Repita los pasos anteriores para crear la puerta de enlace de IKE secundaria.

### <a name="5-define-ipsec-crypto-profiles"></a>5. Definición de los perfiles criptográficos de IPSEC

Seleccione **Network** > **Expand Network Profiles** > **IPSEC Crypto** > **Add** (Red > Expandir perfiles de red > Cifrado de IPSEC > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

* Name (Nombre). Escriba un nombre para el perfil de cifrado de IPsec.
* IPsec Protocol (Protocolo IPsec). Seleccione **ESP**.
* Cifrado. Haga clic en **Agregar** y seleccione el método de cifrado adecuado.
* Autenticación. Haga clic en **Agregar** y seleccione el método de autenticación adecuado.
* DH Group (Grupo DH). Seleccione **no-pfs**.
* Lifetime (Duración). Establezca un valor de 30 minutos.
* Enable (Habilitar). Deje la casilla desactivada.

Repita los pasos anteriores para crear otro perfil de cifrado de IPsec, que se usará para la VPN de CloudSimple secundaria del mismo nivel. También se puede usar el mismo perfil de cifrado IPSEC para los túneles de IPsec principales y secundarios (consulte el procedimiento siguiente).

### <a name="6-define-monitor-profiles-for-tunnel-monitoring"></a>6. Definición de los perfiles de monitor para la supervisión del túnel

Seleccione **Network** > **Expand Network Profiles** > **Monitor** > **Add** (Red > Expandir perfiles de red > Monitor > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

* Name (Nombre). Escriba el nombre del perfil de monitor que se va a usar para supervisar el túnel para encontrar reacciones proactivas al error.
* Action (Acción). Seleccione **Fail Over** (conmutar por error).
* Interval (Intervalo). Escriba el valor **3**.
* Threshold (Umbral). Escriba el valor **7**.

### <a name="7-set-up-primary-and-secondary-ipsec-tunnels"></a>7. Configuración de túneles de IPsec principales y secundarios

Seleccione **Network** > **IPsec Tunnels** > **Add** (Red > Túneles de IPsec > Agregar), configure los siguientes campos y haga clic en **Aceptar**.

Pestaña General:

* Name (Nombre). Escriba el nombre del túnel de IPsec principal que se va a emparejar con la VPN de CloudSimple principal del mismo nivel.
* Tunnel Interface (Interfaz de túnel). Seleccione la interfaz de túnel principal.
* Type (Tipo). Deje el valor predeterminado.
* Address Type (Tipo de dirección). Seleccione **IPv4**.
* IKE Gateway (Puerta de enlace de IKE). Seleccione la puerta de enlace de IKE principal.
* IPsec Crypto Profile (Perfil criptográfico de IPsec). Seleccione el perfil de IPsec principal. Seleccione **Show Advanced options** (Mostrar opciones avanzadas).
* Enable Replay Protection (Habilitar la protección de reproducción). Deje el valor predeterminado.
* Copy TOS Header (Copiar encabezado TOS). Deje la casilla desactivada.
* Tunnel Monitor (Monitor de túnel). Active la casilla.
* Destination IP (IP de destino). Escriba cualquier dirección IP perteneciente a la subred de la nube privada de CloudSimple que esté permitida a través de la conexión de sitio a sitio. Asegúrese de que las interfaces de túnel (como tunnel.20 - 10.64.5.2/32 y tunnel.30 - 10.64.6.2/32) en Palo Alto puedan comunicarse con la dirección IP de la nube privada de CloudSimple a través de la VPN de sitio a sitio. Consulte la configuración siguiente para obtener los id. de proxy.
* Profile (Perfil). Seleccione el perfil del monitor.

Pestaña Proxy IDs (Id. de proxy): haga clic en **IPv4** > **Agregar** y configure los siguientes valores:

* Proxy ID (Id. de proxy). Escriba cualquier nombre para el tráfico interesante. Podría haber varios Id. de proxy transportados dentro de un túnel de IPsec.
* Local. Especifique las subredes locales que pueden comunicarse con subredes de nube privada a través de la VPN de sitio a sitio.
* Remote (Remotas). Especifique las subredes remotas de la nube privada que pueden comunicarse con las subredes locales.
* Protocol (Protocolo). Seleccione **any** (cualquiera).

Repita los pasos anteriores para crear otro túnel de IPsec que se usará para la VPN de CloudSimple secundaria del mismo nivel.

## <a name="references"></a>Referencias

Configuración de NAT en Cisco ASA:

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa84/configuration/guide/asa_84_cli_config/nat_objects.html" target="_blank">Cisco ASA 5500 Series Configuration Guide</a> (Guía de configuración de la serie 5500 de Cisco ASA)

Atributos IKEv1 y IKEv2 admitidos en Cisco ASA:

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa90/configuration/guide/asa_90_cli_config/vpn_ike.html#21661" target="_blank">Cisco ASA Series CLI Configuration Guide</a> (Guía de configuración de la serie Cisco ASA mediante CLI)

Configuración de VPN sitio a sitio por IPsec en Cisco ASA versión 8.4 y posteriores:

<a href="https://www.cisco.com/c/en/us/support/docs/security/asa-5500-x-series-next-generation-firewalls/119141-configure-asa-00.html#anc8" target="_blank">Configure IKEv1 IPsec Site-to-Site Tunnels with the ASDM or CLI on the ASA</a> (Configuración de los túneles de sitio a sitio de IKEv1 IPsec con ASDM o CLI en ASA)

Configuración de Adaptive Security Virtual Appliance (ASAv) en Azure:

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa96/asav/quick-start-book/asav-96-qsg/asav-azure.html" target="_blank">Guía de inicio rápido de Cisco Adaptive Security Virtual Appliance [ASAv]</a>

Configuración de una VPN de sitio a sitio con id. de proxy en Palo Alto:

[Set Up Site-to-Site VPN](https://docs.paloaltonetworks.com/pan-os/9-0/pan-os-admin/vpns/set-up-site-to-site-vpn#) (Configuración de VPN de sitio a sitio)

Configuración del monitor de túnel:

[Set Up Tunnel Monitoring](https://docs.paloaltonetworks.com/pan-os/7-1/pan-os-admin/vpns/set-up-tunnel-monitoring.html) (Configuración de la supervisión de túneles)

Operaciones de puerta de enlace de IKE o túnel de IPsec:

<a href="https://docs.paloaltonetworks.com/pan-os/9-0/pan-os-admin/vpns/set-up-site-to-site-vpn/enabledisable-refresh-or-restart-an-ike-gateway-or-ipsec-tunnel#" target="_blank">Enable/Disable, Refresh or Restart an IKE Gateway or IPsec Tunnel</a> (Habilitación, deshabilitación, actualización o reinicio de una puerta de enlace de IKE o túnel de IPsec)
