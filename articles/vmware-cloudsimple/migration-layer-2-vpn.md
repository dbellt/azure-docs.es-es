---
title: 'Azure VMware Solution by CloudSimple: extensión de una red de Capa 2 en el entorno local a la nube privada'
description: Se describe cómo configurar una VPN de Capa 2 entre NSX-T en una nube privada de CloudSimple y un cliente de NSX Edge independiente local.
author: shortpatti
ms.author: v-patsho
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 1b4b3e85323e2023afb67b2175377cb9b2d13a4f
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177519"
---
# <a name="migrate-workloads-using-layer-2-stretched-networks"></a>Migración de cargas de trabajo mediante redes extendidas de Capa 2

En esta guía, aprenderá a usar la VPN de Capa 2 (L2VPN) para extender una red de Capa 2 desde el entorno local a la nube privada de CloudSimple. Esta solución permite la migración de cargas de trabajo que se ejecutan en el entorno de VMware local a la nube privada de Azure en el mismo espacio de direcciones de subred sin tener que volver a crear las direcciones IP de las cargas de trabajo.

La extensión basada en L2VPN de redes de Capa 2 puede funcionar con o sin redes basadas en NSX en el entorno de VMware local. Si no tiene redes basadas en NSX para cargas de trabajo locales, puede utilizar una puerta de enlace de servicios de NSX Edge independiente.

> [!NOTE]
> En esta guía se describe el escenario en el que los centros de datos de nube privada y local se conectan a través de VPN de sitio a sitio.

## <a name="deployment-scenario"></a>Escenario de implementación

Para extender la red local mediante L2VPN, debe configurar un servidor de L2VPN (enrutador de nivel 0 de NSX-T de destino) y un cliente L2VPN (cliente independiente de origen).  

En este escenario de implementación, la nube privada se conecta al entorno local mediante un túnel VPN de sitio a sitio que permite a las subredes de vMotion y de administración locales comunicarse con las subredes de vMotion y de administración de la nube privada. Esta disposición es necesaria para Cross vCenter vMotion (xVC-vMotion). Un enrutador de nivel 0 de NSX-T se implementa como un servidor de L2VPN en la nube privada.

NSX Edge independiente se implementa en el entorno local como un cliente L2VPN y, posteriormente, se empareja con el servidor de L2VPN. Se crea un punto de conexión de túnel GRE en cada lado y se configura para "extender" la red de Capa 2 local a la nube privada. Esta configuración se describe en la siguiente ilustración.

![Escenario de implementación](media/l2vpn-deployment-scenario.png)

Para obtener más información sobre la migración mediante la VPN de Capa 2, consulte [Redes privadas virtuales](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) en la documentación de VMware.

## <a name="prerequisites-for-deploying-the-solution"></a>Requisitos previos para la implementación de la solución

Compruebe que se ha implementado lo siguiente antes de implementar y configurar la solución:

* La versión local de vSphere es 6.7U1+ o 6.5P03+.
* La licencia de vSphere local se encuentra en el nivel Enterprise Plus (del conmutador distribuido de vSphere).
* Identifique la red de Capa 2 de la carga de trabajo que se va a extender a la nube privada.
* Identifique una red de Capa 2 en el entorno local para implementar el dispositivo cliente L2VPN.
* [Ya se ha creado una nube privada](create-private-cloud.md).
* La versión del dispositivo de Edge de NSX-T independiente es compatible con la versión del administrador de NSX-T (NSX-T 2.3.0) que se usa en el entorno de nube privada.
* Se ha creado un grupo de puertos de tronco en el entorno local de vCenter con las transmisiones falsificadas habilitadas.
* Se ha reservado una dirección IP pública para su uso en la dirección IP de vínculo superior del cliente independiente de NSX-T, y NAT 1:1 está lista para la traducción entre las dos direcciones.
* El reenvío de DNS está configurado en los servidores DNS locales para que el dominio az.cloudsimple.io apunte a los servidores DNS de la nube privada.
* La latencia de RTT es menor o igual que 150 ms, según sea necesario para que vMotion funcione en los dos sitios.

## <a name="limitations-and-considerations"></a>Limitaciones y consideraciones

En la tabla siguiente se enumeran las versiones de vSphere compatibles y los tipos de adaptadores de red.  

| Versión de vSphere | Tipo de vSwitch de origen | Controlador de NIC virtual | Tipo de vSwitch de destino | Se admite? |
------------ | ------------- | ------------ | ------------- | ------------- 
| All | DVS | All | DVS | Sí |
| vSphere 6.7UI o versiones posteriores, 6.5P03 o versiones posteriores | DVS | VMXNET3 | N-VDS | Sí |
| vSphere 6.7UI o versiones posteriores, 6.5P03 o versiones posteriores | DVS | E1000 | N-VDS | [No se admite por VWware](https://kb.vmware.com/s/article/56991) |
| vSphere 6.7UI o 6.5P03, NSX-V o versiones inferiores a NSX-T2.2, 6.5P03 o versiones superiores | All | All | N-VDS | [No se admite por VWware](https://kb.vmware.com/s/article/56991) |

A partir de la versión de VMware NSX-T 2.3:

* El conmutador lógico en el lado de la nube privada que se extiende al entorno local a través de L2VPN no se puede enrutar al mismo tiempo. El conmutador lógico extendido no puede estar conectado a un enrutador lógico.
* Las VPN de IPSEC basadas en enrutamiento y L2VPN solo se pueden configurar mediante llamadas API.

Para obtener más información, consulte [Redes privadas virtuales](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) en la documentación de VMware.

### <a name="sample-l2-vpn-deployment-addressing"></a>Ejemplo de direccionamiento de la implementación de VPN de Capa 2

### <a name="on-premises-network-where-the-standalone-esg-l2-vpn-client-is-deployed"></a>Red local en la que se implementa el ESG independiente (cliente VPN de Capa 2)

| **Elemento** | **Valor** |
|------------|-----------------|
| Nombre de red | MGMT_NET_VLAN469 |
| VLAN | 469 |
| CIDR| 10.250.0.0/24 |
| Dirección IP del dispositivo de Edge independiente | 10.250.0.111 |
| Dirección IP de NAT del dispositivo de Edge independiente | 192.227.85.167 |

### <a name="on-premises-network-to-be-stretched"></a>Red local que se va a expandir

| **Elemento** | **Valor** |
|------------|-----------------|
| VLAN | 472 |
| CIDR| 10.250.3.0/24 |

### <a name="private-cloud-ip-schema-for-nsx-t-tier0-router-l2-vpn-serve"></a>Esquema IP de la nube privada para el enrutador de nivel 0 de NSX-T (servicio de VPN de Capa 2)

| **Elemento** | **Valor** |
|------------|-----------------|
| Interfaz de bucle invertido | 192.168.254.254/32 |
| Interfaz de túnel | 5.5.5.1/29 |
| Conmutador lógico (extendido) | Stretch_LS |
| Interfaz de bucle invertido (dirección IP de NAT) | 104.40.21.81 |

### <a name="private-cloud-network-to-be-mapped-to-the-stretched-network"></a>Red en la nube privada que se va a asignar a la red extendida

| **Elemento** | **Valor** |
|------------|-----------------|
| VLAN | 712 |
| CIDR| 10.200.15.0/24 |

## <a name="fetch-the-logical-router-id-needed-for-l2vpn"></a>Captura del identificador del enrutador lógico necesario para L2VPN

En los pasos siguientes se muestra cómo capturar el identificador del enrutador lógico de la instancia de enrutador lógico de DR de nivel 0 para los servicios IPsec y L2VPN. El identificador del enrutador lógico se necesitará más adelante al implementar L2VPN.

1. Inicie sesión en el administrador de NSX, `https://*nsx-t-manager-ip-address*`, y seleccione **Networking** > **Router** > **Provider-LR**  > **Overview** (Redes > Enrutador > Proveedor de enrutador lógico > Información general). En **Modo de alta disponibilidad**, seleccione **Espera activo**. Esta acción abre una ventana emergente que muestra la VM de Edge en la que el enrutador de nivel 0 está activo actualmente.

    ![Selección de espera activo](media/l2vpn-fetch01.png)

2. Seleccione **Fabric** > **Nodos** > **Edge**. Tome nota de la dirección IP de administración de la VM de Edge activa (VM1 de Edge) identificada en el paso anterior.

    ![IP de administración de notas](media/l2vpn-fetch02.png)

3. Abra una sesión de SSH en la dirección IP de administración de la VM de Edge. Ejecute el comando ```get logical-router``` con el nombre de usuario de **administrador** y la contraseña **CloudSimple 123!** .

    ![Captura de pantalla que muestra una sesión abierta de SSH.](media/l2vpn-fetch03.png)

4. Si no ve la entrada "DR-Provider-L" (Proveedor de DR del enrutador lógico), complete los pasos siguientes.

5. Cree dos conmutadores lógicos de superposición con copia de seguridad. Un conmutador lógico se extiende al entorno local en el que residen las cargas de trabajo migradas. El otro, es un conmutador ficticio. Para obtener instrucciones, consulte [Crear un conmutador lógico](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-23194F9A-416A-40EA-B9F7-346B391C3EF8.html) en la documentación de VMware.

    ![Creación de un servidor lógico](media/l2vpn-fetch04.png)

6. Adjunte el modificador ficticio al enrutador de nivel 1 con una dirección IP local de vínculo o cualquier subred no superpuesta desde el entorno local o la nube privada. Consulte [Agregar un puerto de vínculo inferior en un enrutador lógico de nivel 1](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-E7EA867C-604C-4224-B61D-2A8EF41CB7A6.html) en la documentación de VMware.

    ![Asociación de un conmutador ficticio](media/l2vpn-fetch05.png)

7. Vuelva a ejecutar el comando `get logical-router` en la sesión SSH de la VM de Edge. Se muestra el UUID del enrutador lógico "DR-Provider-L" (Proveedor de DR del enrutador lógico). Anote el UUID, que es necesario al configurar L2VPN.

    ![Captura de pantalla que muestra el UUID del enrutador lógico.](media/l2vpn-fetch06.png)

## <a name="fetch-the-logical-switch-id-needed-for-l2vpn"></a>Captura del identificador del conmutador lógico necesario para L2VPN

1. Inicie sesión en el administrador de NSX-T (`https://nsx-t-manager-ip-address`).
2. Seleccione **Networking** > **Switching** > **Switches** >  **<\Logical switch\>**  > **Overview** (Redes > Cambiar > Modificadores >\Modificador lógico > Información general).
3. Anote el UUID del conmutador lógico de extensión, que es necesario al configurar L2VPN.

    ![obtención de la salida del enrutador lógico](media/l2vpn-fetch-switch01.png)

## <a name="routing-and-security-considerations-for-l2vpn"></a>Consideraciones sobre enrutamiento y seguridad para L2VPN

Para establecer una instancia de VPN basada en rutas de IPsec entre el enrutador de nivel 0 de NSX-T y el cliente de NSX Edge independiente, la interfaz de bucle invertido del enrutador de nivel 0 de NSX-T debe ser capaz de comunicarse con la dirección IP pública del cliente independiente de NSX local a través de UDP 500/4500.

### <a name="allow-udp-5004500-for-ipsec"></a>Permitir UDP 500/4500 para IPsec

1. [Cree una dirección IP pública](public-ips.md) para la interfaz de bucle invertido de NSX-T de nivel 0 en el portal de CloudSimple.

2. [Cree una tabla de firewall](firewall.md) con reglas con estado que permitan el tráfico de entrada UDP 500/4500 y adjunten la tabla de firewall a la subred HostTransport de NSX-T.

### <a name="advertise-the-loopback-interface-ip-to-the-underlay-network"></a>Anunciar la dirección IP de la interfaz de bucle invertido a la red subyacente

1. Cree una ruta nula para la red de la interfaz de bucle invertido. Inicie sesión en el administrador de NSX y seleccione **Redes** > **Enrutamiento** > **Enrutadores** > **Provider-LR (Proveedor del enrutador lógico)**  > **Enrutamiento** > **Rutas estáticas**. Haga clic en **Agregar**. En **Red**, escriba la dirección IP de la interfaz de bucle invertido. En **Próximos saltos**, haga clic en **Agregar**, especifique "Null" para el próximo salto y mantenga el valor predeterminado de 1 para la distancia de administrador.

    ![Agregar ruta estática](media/l2vpn-routing-security01.png)

2. Cree una lista de prefijos IP. Inicie sesión en el administrador de NSX y seleccione **Redes** > **Enrutamiento** > **Enrutadores** > **Provider-LR (Proveedor del enrutador lógico)**  > **Enrutamiento** > **IP Prefix Lists** (Listas de prefijos IP). Haga clic en **Agregar**. Escriba un nombre para identificar la lista. En **Prefijos**, haga clic en **Agregar** dos veces. En la primera línea, escriba "0.0.0.0/0" para **Red** y "Denegar" para **Acción**. En la segunda línea, seleccione **Cualquiera** para **Red** y **Permitir** para **Acción**.
3. Adjunte la lista de prefijos IP a ambos vecinos de BGP (TOR). Al adjuntar la lista de prefijos IP al vecino de BGP, se evita que la ruta predeterminada se anuncie en BGP para los conmutadores TOR. Sin embargo, cualquier otra ruta que incluya la ruta nula anunciará la dirección IP de la interfaz de bucle invertido a los conmutadores TOR.

    ![Creación de una lista de prefijos IP](media/l2vpn-routing-security02.png)

4. Inicie sesión en el administrador de NSX y seleccione **Redes** > **Enrutamiento** > **Enrutadores** > **Provider-LR (Proveedor del enrutador lógico)**  > **Enrutamiento** > **BGP** > **Vecinos**. Seleccione el primer vecino. Haga clic en **Editar** > **Familias de direcciones**. En la familia IPv4, edite la columna **Out Filter** y seleccione la lista de prefijos IP que creó. Haga clic en **Save**(Guardar). Repita este paso para el segundo vecino.

    ![Adjuntar la lista de prefijos IP 1](media/l2vpn-routing-security03.png) ![Adjuntar la lista de prefijos IP 2](media/l2vpn-routing-security04.png)

5. Redistribuya la ruta estática nula en BGP. Para anunciar la ruta de la interfaz de bucle invertido a la subyacente, debe redistribuir la ruta estática nula en BGP. Inicie sesión en el administrador de NSX y seleccione **Redes** > **Enrutamiento** > **Enrutadores** > **Provider-LR (Proveedor del enrutador lógico)**  > **Enrutamiento** > **Route Redistribution (Redistribución de rutas)**  > **Vecinos**. Seleccione **Provider-LR-Route_Redistribution** y haga clic en **Editar**. Active la casilla **Estático** y haga clic en **Guardar**.

    ![Redistribución de la ruta estática null en BGP](media/l2vpn-routing-security05.png)

## <a name="configure-a-route-based-vpn-on-the-nsx-t-tier0-router"></a>Configuración de una VPN basada en rutas en el enrutador de nivel 0 de NSX-T

Use la plantilla siguiente para rellenar todos los detalles de configuración de una VPN basada en rutas en el enrutador de nivel 0 de NSX-T. El UUID de cada llamada POST es necesario en las llamadas POST posteriores. Las direcciones IP de las interfaces de bucle invertido y de túnel para L2VPN deben ser únicas y no superponerse con las redes locales o en la nube privada.

Las direcciones IP elegidas para las interfaces de bucle invertido y de túnel para L2VPN deben ser únicas y no superponerse con las redes locales o de la nube privada. La red de la interfaz de bucle invertido siempre debe ser /32.

```
Loopback interface ip : 192.168.254.254/32
Tunnel interface subnet : 5.5.5.0/29
Logical-router ID : UUID of Tier0 DR logical router obtained in section "Steps to fetch Logical-Router ID needed for L2VPN"
Logical-switch ID(Stretch) : UUID of Stretch Logical Switch obtained earlier
IPSec Service ID :
IKE profile ID :
DPD profile ID :
Tunnel Profile ID :
Local-endpoint ID :
Peer end-point ID :
IPSec VPN session ID (route-based) :
L2VPN service ID :
L2VPN session ID :
Logical-Port ID :
Peer Code :
```

Para todas las siguientes llamadas API, reemplace la dirección IP por la dirección IP del administrador de NSX-T. Puede ejecutar todas estas llamadas API desde el cliente POSTMAN o mediante comandos `curl`.

### <a name="enable-the-ipsec-vpn-service-on-the-logical-router"></a>Habilitación del servicio VPN de IPsec en el enrutador lógico

```
POST   https://192.168.110.201/api/v1/vpn/ipsec/services/
{
"resource_type": "IPSecVPNService",
"description": "Manage VPN service",
"display_name": "IPSec VPN service",
"logical_router_id": "Logical-router ID",
"ike_log_level": "INFO",
"enabled": true
}
```

### <a name="create-profiles-ike"></a>Creación de perfiles: IKE

```
POST https://192.168.110.201/api/v1/vpn/ipsec/ike-profiles
 
{
"resource_type": "IPSecVPNIKEProfile",
"description": "IKEProfile for siteA",
"display_name": "IKEProfile siteA",
"encryption_algorithms": ["AES_128"],
"ike_version": "IKE_V2",
"digest_algorithms": ["SHA2_256"],
"sa_life_time":21600,
"dh_groups": ["GROUP14"]
}
```

### <a name="create-profiles-dpd"></a>Creación de perfiles: DPD

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/dpd-profiles  

{
"resource_type": "IPSecVPNDPDProfile",
"display_name": "nsx-default-dpd-profile",
"enabled": true
}
```

### <a name="create-profiles-tunnel"></a>Creación de perfiles: Túnel

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/tunnel-profiles
 
{
"resource_type": "IPSecVPNTunnelProfile",
"display_name": "nsx-default-tunnel-profile",
"enable_perfect_forward_secrecy": true,
"encryption_algorithms": ["AES_GCM_128"],
"digest_algorithms": [],
"sa_life_time":3600,
"dh_groups": ["GROUP14"],
"encapsulation_mode": "TUNNEL_MODE",
"transform_protocol": "ESP",
"df_policy": "COPY"
}
```

### <a name="create-a-local-endpoint"></a>Creación de un punto de conexión local

``` 
POST https://192.168.110.201/api/v1/vpn/ipsec/local-endpoints
 
{
"resource_type": "IPSecVPNLocalEndpoint",
"description": "Local endpoint",
"display_name": "Local endpoint",
"local_id": "<Public IP of Loopback interface>",
"ipsec_vpn_service_id": {
"target_id": "IPSec VPN service ID"},
"local_address": "<IP of Loopback interface>",
"trust_ca_ids": [],
"trust_crl_ids": []
}
```

### <a name="create-a-peer-endpoint"></a>Creación de un punto de conexión del mismo nivel

```
POST https://192.168.110.201/api/v1/vpn/ipsec/peer-endpoints

{
"resource_type": "IPSecVPNPeerEndpoint",
"description": "Peer endpoint for site B",
"display_name": "Peer endpoint for site B",
"connection_initiation_mode": "INITIATOR",
"authentication_mode": "PSK",
"ipsec_tunnel_profile_id": "IPSec Tunnel profile ID",
"dpd_profile_id": "DPD profile ID",
"psk":"nsx",
"ike_profile_id": "IKE profile ID",
"peer_address": "<Public IP of Standalone client",
"peer_id": "<Public IP of Standalone client>"
}
```

### <a name="create-a-route-based-vpn-session"></a>Creación de una sesión de VPN basada en rutas

```
POST :  https://192.168.110.201/api/v1/vpn/ipsec/sessions
 
{
"resource_type": "RouteBasedIPSecVPNSession",
"peer_endpoint_id": "Peer Endpoint ID",
"ipsec_vpn_service_id": "IPSec VPN service ID",
"local_endpoint_id": "Local Endpoint ID",
"enabled": true,
"tunnel_ports": [
{
"ip_subnets": [
{
"ip_addresses": [
 "5.5.5.1"
],
"prefix_length": 24
}
  ]
}
]
}
```

## <a name="configure-l2vpn-on-nsx-t-tier0-router"></a>Configuración de L2VPN en el enrutador de nivel 0 de NSX-T

Rellene la información siguiente tras cada llamada POST. Los identificadores son necesarios en las llamadas POST posteriores.

```
L2VPN Service ID:
L2VPN Session ID:
Logical Port ID:
```

### <a name="create-the-l2vpn-service"></a>Creación del servicio de L2VPN

La salida del comando GET siguiente estará en blanco porque la configuración aún no se ha completado.

```
GET : https://192.168.110.201/api/v1/vpn/l2vpn/services
```

Para el siguiente comando POST, el identificador del enrutador lógico es el UUID del enrutador lógico de nivel 0 de DR obtenido anteriormente.

```
POST : https://192.168.110.201/api/v1/vpn/l2vpn/services

{
"logical_router_id": "Logical Router ID",
"enable_full_mesh" : true
}
```

### <a name="create-the-l2vpn-session"></a>Creación de la sesión de L2VPN

Para el siguiente comando POST, el identificador de servicio de L2VPN es el identificador que acaba de obtener y el identificador de sesión de VPN de IPsec es el identificador obtenido en la sección anterior.

```    
POST: https://192.168.110.201/api/v1/vpn/l2vpn/sessions

{
"l2vpn_service_id" : "L2VPN service ID",
"transport_tunnels" : [
    {
    "target_id" : "IPSec VPN session ID"
    }]
}
```

Estas llamadas crean un punto de conexión de túnel GRE. Para comprobar el estado, ejecute el comando siguiente.

```
edge-2> get tunnel-port
Tunnel      : 44648dae-8566-5bc9-a065-b1c4e5c3e03f
IFUID       : 328
LOCAL       : 169.254.64.1
REMOTE      : 169.254.64.2
ENCAP       : GRE

Tunnel      : cf950ca1-5cf8-5438-9b1a-d2c8c8e7229b
IFUID       : 318
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.152
ENCAP       : GENEVE

Tunnel      : 63639321-87c5-529e-8a61-92c1939799b2
IFUID       : 304
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.156
ENCAP       : GENEVE
```

### <a name="create-logical-port-with-the-tunnel-id-specified"></a>Creación de un puerto lógico con el identificador de túnel especificado

```
    POST https://192.168.110.201/api/v1/logical-ports/

{
"resource_type": "LogicalPort",
"display_name": "Extend logicalSwitch, port for service",
"logical_switch_id": "Logical switch ID",
"admin_state" : "UP",
"attachment": {
"attachment_type":"L2VPN_SESSION",
"id":"L2VPN session ID",
"context" : {
"resource_type" : "L2VpnAttachmentContext",
    "tunnel_id" : 10
}
    }
        }
```

## <a name="obtain-the-peer-code-for-l2vpn-on-the-nsx-t-side"></a>Obtención del código del mismo nivel para L2VPN en el lado de NSX-T

Obtenga el código del mismo nivel del punto de conexión de NSX-T. El código del mismo nivel es necesario al configurar el punto de conexión remoto. El <session-id> de L2VPN se puede obtener de la sección anterior. Para más información, consulte la [Guía de API de NSX-T 2.3](https://www.vmware.com/support/nsxt/doc/nsxt_23_api.html).

```
GET https://192.168.110.201/api/v1/vpn/l2vpn/sessions/<session-id>/peer-codes
```

## <a name="deploy-the-nsx-t-standalone-client-on-premises"></a>Implementación del cliente independiente de NSX-T (local)

Antes de la implementación, compruebe que las reglas de firewall local permiten el tráfico entrante y saliente de UDP 500/4500 desde y hacia la dirección IP pública de CloudSimple que se ha reservado anteriormente para la interfaz de bucle invertido del enrutador de nivel 0 de NSX-T. 

1. [Descargue OVF del cliente de NSX Edge independiente](https://my.vmware.com/group/vmware/details?productId=673&rPId=33945&downloadGroup=NSX-T-230) y extraiga los archivos del conjunto descargado en una carpeta.

    ![Descarga del cliente de NSX Edge independiente](media/l2vpn-deploy-client01.png)

2. Vaya a la carpeta con todos los archivos extraídos. Seleccione todos los archivos VMDK (NSX-l2t-client-large.mf y NSX-l2t-client-large.ovf para tamaños de dispositivos grandes o NSX-l2t-client-Xlarge.mf y NSX-l2t-client-Xlarge.ovf para tamaños de dispositivos muy grandes). Haga clic en **Next**.

    ![Seleccionar plantilla](media/l2vpn-deploy-client02.png) ![Captura de pantalla que muestra los archivos vmdk seleccionados.](media/l2vpn-deploy-client03.png)

3. Escriba un nombre para el cliente independiente de NSX-T y haga clic en **Siguiente**.

    ![Escribir el nombre de la plantilla](media/l2vpn-deploy-client04.png)

4. Haga clic en **Siguiente** cuando sea necesario para llegar a la configuración del almacén de datos. Seleccione el almacén de datos adecuado para el cliente independiente de NSX-T y haga clic en **Siguiente**.

    ![Selección del almacén de datos](media/l2vpn-deploy-client06.png)

5. Seleccione los grupos de puertos correctos para Tronco (Tronco PG), Público (Vínculo superior PG) e interfaz de alta disponibilidad (Vínculo superior PG) para el cliente independiente de NSX-T. Haga clic en **Next**.

    ![Selección de los grupos de puertos](media/l2vpn-deploy-client07.png)

6. Rellene los siguientes detalles en la pantalla **Personalizar plantilla** y haga clic en **Siguiente**:

    Expanda L2T:

    * **Dirección del mismo nivel**. Escriba la dirección IP reservada en el portal de Azure CloudSimple para la interfaz de bucle invertido de nivel 0 de NSX-T.
    * **Código del mismo nivel**. Pegue el código del mismo nivel obtenido en el último paso de la implementación del servidor de L2VPN.
    * **Subinterfaces de VLAN (identificador de túnel)** . Escriba el identificador de VLAN que se va a expandir. Entre paréntesis (), escriba el identificador de túnel que se configuró previamente.

    Expanda la interfaz de vínculo superior:

    * **Dirección IP de DNS**. Escriba la dirección IP del DNS local.
    * **Puerta de enlace predeterminada**.  Escriba la puerta de enlace predeterminada de la VLAN que actuará como puerta de enlace predeterminada para este cliente.
    * **Dirección IP**. Escriba la dirección IP de vínculo superior del cliente independiente.
    * **Longitud del prefijo**. Escriba la longitud del prefijo de la VLAN o subred de vínculo superior.
    * **Contraseña del usuario raíz, administrador de CLI o de habilitación**. Establezca la contraseña de la cuenta raíz, del administrador o de habilitación.

      ![Personalizar plantilla](media/l2vpn-deploy-client08.png)
      ![Personalizar plantilla: más](media/l2vpn-deploy-client09.png)

7. Revise la configuración y haga clic en **Finish** (Finalizar).

    ![Conclusión de la configuración](media/l2vpn-deploy-client10.png)

## <a name="configure-an-on-premises-sink-port"></a>Configuración de un puerto de recepción local

Si uno de los sitios de VPN no tiene NSX implementado, puede configurar una VPN de Capa 2 mediante la implementación de una instancia de NSX Edge independiente en ese sitio. Una instancia de NSX Edge independiente se implementa mediante un archivo OVF en un host que no está administrado por NSX. De esta forma, se implementa un dispositivo de puerta de enlace de servicios de NSX Edge para que funcione como un cliente VPN de Capa 2.

Si un vNIC de tronco de Edge independiente se conecta a un conmutador distribuido de vSphere, se requiere el modo promiscuo o un puerto de recepción para la función de VPN de Capa 2. El uso del modo promiscuo puede producir pings duplicados y respuestas duplicadas. Por esta razón, use el modo de puerto de recepción en la configuración de NSX Edge independiente de VPN de Capa 2. Consulte [Configurar un puerto de recepción](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.admin.doc/GUID-3CDA4346-E692-4592-8796-ACBEEC87C161.html) en la documentación de VMware.

## <a name="ipsec-vpn-and-l2vpn-verification"></a>Comprobación de L2VPN y VPN de IPsec

Use los siguientes comandos para comprobar las sesiones de IPsec y L2VPN desde la instancia de NSX-T Edge independiente.

```
nsx-l2t-edge> show service ipsec
-----------------------------------------------------------------------
vShield Edge IPSec Service Status:
IPSec Server is running.
AESNI is enabled.
Total Sites: 1, 1 UP, 0 Down
Total Tunnels: 1, 1 UP, 0 Down
----------------------------------
Site:  10.250.0.111_0.0.0.0/0-104.40.21.81_0.0.0.0/0
Channel: PeerIp: 104.40.21.81    LocalIP: 10.250.0.111  Version: IKEv2  Status: UP
Tunnel: PeerSubnet: 0.0.0.0/0    LocalSubnet: 0.0.0.0/0   Status: UP
----------------------------------
```

```
nsx-l2t-edge> show service l2vpn
L2 VPN is running
----------------------------------------
L2 VPN type: Client/Spoke

SITENAME                       IPSECSTATUS          VTI                  GRE
1ecb00fb-a538-4740-b788-c9049e8cb6c6 UP                   vti-100              l2t-1
```

Use los siguientes comandos para comprobar las sesiones de IPsec y L2VPN desde el enrutador de nivel 0 de NSX-T.

```
edge-2> get ipsecvpn session
Total Number of Sessions: 1

IKE Session ID : 3
UUID           : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Type           : Route

Local IP       : 192.168.254.254      Peer IP        : 192.227.85.167
Local ID       : 104.40.21.81         Peer ID        : 192.227.85.167
Session Status : Up

Policy Rules
    VTI UUID       : 4bf96e3b-e50b-49cc-a16e-43a6390e3d53
    ToRule ID      : 560874406            FromRule ID    : 2708358054
    Local Subnet   : 0.0.0.0/0            Peer Subnet    : 0.0.0.0/0
    Tunnel Status  : Up
```

```
edge-2> get l2vpn session
Session       : f7147137-5dd0-47fe-9e53-fdc2b687b160
Tunnel        : b026095b-98c8-5932-96ff-dda922ffe316
IPSEC Session : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Status        : UP
```

Use los siguientes comandos para comprobar el puerto de recepción en el host ESXi en el que se encuentra la VM de cliente independiente de NSX-T en el entorno local.

```
 [root@esxi02:~] esxcfg-vswitch -l |grep NSX
  53                  1           NSXT-client-large.eth2
  225                1           NSXT-client-large.eth1
  52                  1           NSXT-client-large.eth0
```

```
[root@esxi02:~] net-dvs -l | grep "port\ [0-9]\|SINK\|com.vmware.common.alias"
                com.vmware.common.alias = csmlab2DS ,   propType = CONFIG
        port 24:
        port 25:
        port 26:
        port 27:
        port 13:
        port 19:
        port 169:
        port 54:
        port 110:
        port 6:
        port 107:
        port 4:
        port 199:
        port 168:
        port 201:
        port 0:
        port 49:
        port 53:
        port 225:
                com.vmware.etherswitch.port.extraEthFRP =   SINK
        port 52:
```
