---
title: 'Tutorial: Lista de comprobación del planeamiento de red'
description: Obtenga información sobre los requisitos de red en relación con la conectividad de red y los puertos de red para Azure VMware Solution.
ms.topic: tutorial
ms.date: 06/08/2021
ms.openlocfilehash: 5719ec1a2495c6a225c35ec46cdf19506a10ba6f
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112017278"
---
# <a name="networking-planning-checklist-for-azure-vmware-solution"></a>Lista de comprobación del planeamiento de red para Azure VMware Solution 

Azure VMware Solution ofrece un entorno de nube privada de VMware accesible para usuarios y aplicaciones a partir de recursos o entornos locales y basados en Azure. La conectividad se proporciona a través de servicios de redes, como las conexiones VPN y Azure ExpressRoute. Requiere intervalos de direcciones de red y puertos de firewall específicos para habilitar los servicios. En este artículo se proporciona la información necesaria para configurar correctamente las redes para que funcionen bien con Azure VMware Solution.

En este tutorial, aprenderá:

> [!div class="checklist"]
> * Consideraciones sobre la red virtual y el circuito ExpressRoute
> * Requisitos de enrutamiento y subred
> * Puertos de red necesarios para la comunicación con los servicios
> * Consideraciones sobre DHCP y DNS en Azure VMware Solution

## <a name="prerequisite"></a>Requisito previo
Asegúrese de que todas las puertas de enlace, incluido el servicio del proveedor de ExpressRoute, admitan el número de sistema autónomo (ASN) de 4 bytes. Azure VMware Solution utiliza ASN públicos de 4 bytes para anunciar rutas.

## <a name="virtual-network-and-expressroute-circuit-considerations"></a>Consideraciones sobre la red virtual y el circuito ExpressRoute
Al crear una conexión de red virtual en la suscripción, el circuito ExpressRoute se establece por medio del emparejamiento, usa una clave de autorización y un identificador de emparejamiento que se solicita en Azure Portal. El emparejamiento es una conexión privada de uno a uno entre la nube privada y la red virtual.

> [!NOTE] 
> El circuito ExpressRoute no forma parte de una implementación de nube privada. El circuito de ExpressRoute local está fuera del ámbito de este documento. Si necesita una conectividad desde el entorno local a su nube privada, puede utilizar uno de los circuitos de ExpressRoute existentes o comprar uno en Azure Portal.

Al implementar una nube privada, se reciben direcciones IP de vCenter y NSX-T Manager. Para acceder a esas interfaces de administración, deberá crear más recursos en la red virtual de la suscripción. En los tutoriales puede encontrar los procedimientos para crear esos recursos y establecer el [emparejamiento privado de ExpressRoute](tutorial-expressroute-global-reach-private-cloud.md).

Las redes lógicas de nube privada incluyen NSX-T previamente aprovisionado. Se aprovisionan previamente de forma automática una puerta de enlace de nivel 0 y una de nivel 1. Puede crear un segmento y asociarlo a la puerta de enlace de nivel 1 existente o hacerlo a una nueva puerta de enlace de nivel 1 que puede definir. Los componentes de redes lógicas NSX-T proporcionan conectividad horizontal de derecha a izquierda entre cargas de trabajo y proporcionan conectividad vertical de arriba abajo a Internet y a los servicios de Azure.

## <a name="routing-and-subnet-considerations"></a>Consideraciones sobre el enrutamiento y la subred
La nube privada de Azure VMware Solution se conecta a la red virtual de Azure mediante una conexión de Azure ExpressRoute. Esta conexión de alto ancho de banda y baja latencia permite acceder a los servicios que se ejecutan en la suscripción de Azure desde el entorno de nube privada. El enrutamiento se basa en el Protocolo de puerta de enlace de borde (BGP), que se aprovisiona y habilita automáticamente de forma predeterminada para cada implementación de nube privada. 

Las nubes privadas de Azure VMware Solution requieren un mínimo de un bloque de direcciones de red CIDR `/22` para las subredes, que se muestran a continuación. Esta red complementa las redes locales. El bloque de direcciones no debe superponerse a los bloques de direcciones usados en otras redes virtuales ubicadas en la suscripción y las redes locales. Dentro de este bloque de direcciones, la administración, el aprovisionamiento y las redes de vMotion se aprovisionan automáticamente.

>[!NOTE]
>Los rangos permitidos para el bloque de direcciones son los espacios de direcciones privados RFC 1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16), excepto 172.17.0.0/16.

Un ejemplo de bloque de direcciones de red CIDR `/22` es `10.10.0.0/22`

Las subredes:

| Uso de red             | Subnet | Ejemplo          |
| ------------------------- | ------ | ---------------- |
| Administración de la nube privada  | `/26`  | `10.10.0.0/26`   |
| Migraciones de administración de HCX       | `/26`  | `10.10.0.64/26`  |
| Global Reach reservado     | `/26`  | `10.10.0.128/26` |
| Servicio DNS de NSX-T         | `/32`  | `10.10.0.192/32` |
| Reservado                  | `/32`  | `10.10.0.193/32` |
| Reservado                  | `/32`  | `10.10.0.194/32` |
| Reservado                  | `/32`  | `10.10.0.195/32` |
| Reservado                  | `/30`  | `10.10.0.196/30` |
| Reservado                  | `/29`  | `10.10.0.200/29` |
| Reservado                  | `/28`  | `10.10.0.208/28` |
| Emparejamiento de ExpressRoute      | `/27`  | `10.10.0.224/27` |
| Administración de ESXi           | `/25`  | `10.10.1.0/25`   |
| Red de vMotion           | `/25`  | `10.10.1.128/25` |
| Red de replicación       | `/25`  | `10.10.2.0/25`   |
| vSAN                      | `/25`  | `10.10.2.128/25` |
| Vínculo superior de HCX                | `/26`  | `10.10.3.0/26`   |
| Reservado                  | `/26`  | `10.10.3.64/26`  |
| Reservado                  | `/26`  | `10.10.3.128/26` |
| Reservado                  | `/26`  | `10.10.3.192/26` |



## <a name="required-network-ports"></a>Puertos de red necesarios

| Source | Destination | Protocolo | Port | Descripción  | 
| ------ | ----------- | :------: | :---:| ------------ | 
| Servidor DNS de la nube privada | Servidor DNS local | UDP | 53 | Cliente DNS: solicitudes de reenvío desde la instancia de vCenter de PC para cualquier consulta de DNS local (consulte la sección DNS siguiente). |  
| Servidor DNS local   | Servidor DNS de la nube privada | UDP | 53 | Cliente DNS: solicitudes de reenvío desde servicios locales a servidores DNS de la nube privada (consulte la sección DNS siguiente). |  
| Red local  | Servidor vCenter de nube privada  | TCP(HTTP)  | 80 | vCenter Server requiere el puerto 80 para conexiones HTTP directas. El puerto 80 redirige las solicitudes al puerto HTTPS 443. Este redireccionamiento ayuda si usa `http://server` en lugar de `https://server`.  <br><br>WS-Management (también requiere que se abra el puerto 443). <br><br>Si utiliza una base de datos de Microsoft SQL personalizada y no la base de datos agrupada de SQL Server 2008 en vCenter Server, SQL Reporting Services usa el puerto 80. Al instalar vCenter Server, el instalador solicita que cambie el puerto HTTP para vCenter Server. Cambie el puerto HTTP de vCenter Server a un valor personalizado para garantizar una correcta instalación. Microsoft Internet Information Services (IIS) también usa el puerto 80. Consulte el conflicto entre vCenter Server e IIS para el puerto 80. |  
| Red de administración de la nube privada | Active Directory local  | TCP  | 389 | Este puerto debe estar abierto en la instancia local y en todas las instancias remotas de vCenter Server. Se trata del número de puerto LDAP de servicios de directorio para el grupo de vCenter Server. El sistema vCenter Server debe enlazar con el puerto 389, incluso si no va a unir esta instancia de vCenter Server a un grupo en Linked Mode. Si se está ejecutando otro servicio en este puerto, podría ser preferible quitarlo o cambiarlo por otro. El servicio LDAP se puede ejecutar en cualquier puerto, del 1025 al 65535.  Si esta instancia actúa como el servicio Active Directory de Microsoft Windows, cambie el número de puerto 389 por otro disponible, del 1025 al 65535. Este puerto es opcional, para configurar AD local como un origen de identidad en el servidor vCenter de nube privada. |  
| Red local  | Servidor vCenter de nube privada  | TCP(HTTPS)  | 443 | Este puerto permite acceder a vCenter desde una red local. El puerto predeterminado que el sistema vCenter Server usa para escuchar las conexiones desde vSphere Client. Para permitir que el sistema vCenter Server reciba datos desde vSphere Cliente, abra el puerto 443 en el firewall. El sistema vCenter Server también utiliza el puerto 443 para supervisar la transferencia de datos desde clientes del SDK. Este puerto también se utiliza para los siguientes servicios: WS-Management (también requiere que se abra el puerto 80). Acceso del cliente de vSphere a vSphere Update Manager. Conexiones de cliente de administración de redes de terceros a vCenter Server. Los clientes de administración de redes de terceros acceden a los hosts. |  
| Explorador web  | Hybrid Cloud Manager  | TCP(HTTPS) | 9443 | Interfaz de administración de aplicaciones virtuales de Hybrid Cloud Manager para la configuración del sistema de Hybrid Cloud Manager. |
| Red de administración  | Hybrid Cloud Manager | SSH | 22 | Acceso SSH de administrador a Hybrid Cloud Manager. |
| HCM | Puerta de enlace en la nube | TCP(HTTPS) | 8123 | Se envían instrucciones del servicio de replicación basadas en host a Hybrid Cloud Gateway. |
| HCM | Puerta de enlace en la nube | HTTP TCP(HTTPS) | 9443 | Se envían instrucciones de administración a la instancia de Hybrid Cloud Gateway local mediante la API REST. |
| Puerta de enlace en la nube | L2C | TCP(HTTPS) | 443 | Se envían instrucciones de administración de Cloud Gateway a L2C si L2C usa la misma ruta de acceso que Hybrid Cloud Gateway. |
| Puerta de enlace en la nube | Hosts de ESXi | TCP | 80,902 | Administración e implementación de OVF. |
| Puerta de enlace en la nube (local)| Puerta de enlace en la nube (remota) | UDP | 4500 | Necesario para IPSEC<br>   Intercambio de claves por red (IKEv2) para encapsular las cargas de trabajo para el túnel bidireccional. También se admite la traducción transversal de direcciones de red (NAT-T). |
| Puerta de enlace en la nube (local) | Puerta de enlace en la nube (remota)  | UDP | 500 | Obligatorio para IPSEC<br> Intercambio de claves por red (ISAKMP) para el túnel bidireccional. |
| Red vCenter local | Red de administración de la nube privada | TCP | 8000 |  Migración de máquinas virtuales mediante vMotion, de una instancia local vCenter a una instancia en la nube privada de vCenter de nube privada.   |     

## <a name="dhcp-and-dns-resolution-considerations"></a>Consideraciones sobre DHCP y la resolución de DNS

[!INCLUDE [dhcp-dns-in-azure-vmware-solution-description](includes/dhcp-dns-in-azure-vmware-solution-description.md)]


## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido acerca de las consideraciones y los requisitos para implementar una nube privada de Azure VMware Solution. Una vez que tenga la red adecuada en su lugar, continúe con el siguiente tutorial para crear la nube privada de Azure VMware Solution.

> [!div class="nextstepaction"]
> [Creación de una nube privada de Azure VMware Solution](tutorial-create-private-cloud.md)
