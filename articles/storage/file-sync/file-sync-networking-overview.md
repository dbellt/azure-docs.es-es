---
title: Consideraciones de redes para Azure File Sync | Microsoft Docs
description: Obtenga información sobre cómo configurar redes para usar Azure File Sync para almacenar archivos en caché de forma local.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 04/13/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 6c761edec571f404a538025c868750bc5712eced
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796458"
---
# <a name="azure-file-sync-networking-considerations"></a>Consideraciones de redes para Azure File Sync
Puede conectarse a un recurso compartido de archivos de Azure de dos maneras:

- Acceda de forma directa al recurso compartido a través de los protocolos SMB o FileREST. Este patrón de acceso se emplea principalmente cuando se quieren eliminar tantos servidores locales como sea posible.
- Cree una memoria caché del recurso compartido de archivos de Azure en un servidor local (o en una VM de Azure) con Azure File Sync y acceda a los datos del recurso compartido de archivos desde el servidor local con el protocolo que elija (SMB, NFS, FTPS, etc.) para su caso de uso. Este patrón de acceso es útil porque combina lo mejor de la escala en la nube y el rendimiento local junto con los servicios conectables sin servidor, como Azure Backup.

Este artículo se centra en cómo configurar las redes cuando el caso de uso necesita acceder a Azure File Sync para almacenar en caché archivos localmente, en lugar de montar el recurso compartido de archivos de Azure directamente a través de SMB. Para obtener más información acerca de las consideraciones de red para una implementación de Azure Files, consulte [Consideraciones de redes para Azure Files](../files/storage-files-networking-overview.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json).

La configuración de red para Azure File Sync abarca dos objetos de Azure diferentes: un servicio de sincronización de almacenamiento y una cuenta de almacenamiento de Azure. Una cuenta de almacenamiento es una construcción de administración que representa un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas. Un servicio de sincronización de almacenamiento es una construcción de administración que representa servidores registrados, que son servidores de archivos de Windows con una relación de confianza establecida con Azure File Sync, y grupos de sincronización, que definen la topología de la relación de sincronización. 

## <a name="connecting-windows-file-server-to-azure-with-azure-file-sync"></a>Conexión de un servidor de archivos de Windows a Azure con Azure File Sync 
Para configurar y usar Azure Files y Azure File Sync con un servidor de archivos de Windows local, no se requiere ninguna conexión de red especial a Azure más allá de una conexión a Internet básica. Para implementar Azure File Sync, instale el agente de Azure File Sync en el servidor de archivos de Windows que desea sincronizar con Azure. El agente de Azure File Sync logra la sincronización con un recurso compartido de archivos de Azure a través de dos canales:

- El protocolo FileREST, que es un protocolo basado en HTTPS para acceder al recurso compartido de archivos de Azure. Dado que el protocolo FileREST usa HTTPS estándar para la transferencia de datos, solo el puerto 443 debe ser de salida accesible. Azure File Sync no usa el protocolo SMB para transferir datos de los servidores de Windows locales al recurso compartido de archivos de Azure.
- El protocolo de sincronización de Azure File Sync, que es un protocolo basado en HTTPS para intercambiar el conocimiento de la sincronización, es decir, la información de la versión de los archivos y las carpetas de su entorno, entre los puntos de conexión de su entorno. Este protocolo también se usa para intercambiar metadatos acerca de los archivos y las carpetas de su entorno, como las marcas de tiempo y las listas de control de acceso (ACL). 

Dado que Azure Files ofrece acceso directo al protocolo SMB en recursos compartidos de archivos de Azure, los clientes suelen preguntarse si necesitan configurar redes especiales para montar los recursos compartidos de archivos de Azure con SMB para el acceso del agente de Azure File Sync. Esto no solo no es necesario, sino que se desaconseja, excepto en el caso de los escenarios de administrador, debido a la falta de detección rápida de los cambios realizados directamente en el recurso compartido de archivos de Azure (es posible que los cambios no se detecten durante más de 24 horas, según el tamaño y el número de los elementos del recurso compartido de archivos de Azure). Si desea usar el recurso compartido de archivos de Azure directamente, es decir, no usar Azure File Sync para el almacenamiento en caché local, puede obtener más información sobre las consideraciones de red para el acceso directo mediante en [Información general sobre redes de Azure Files](../files/storage-files-networking-overview.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json).

Aunque Azure File Sync no requiere ninguna configuración de red especial, es posible que algunos clientes deseen configurar opciones de red avanzadas para habilitar los siguientes escenarios:

- Interoperar con la configuración del servidor proxy de la organización.
- Abrir el firewall local de la organización en los servicios Azure Files y Azure File Sync.
- Tunelizar Azure Files y Azure File Sync a través de ExpressRoute o una conexión VPN.

### <a name="configuring-proxy-servers"></a>Configuración de los servidores proxy
Muchas organizaciones usan un servidor proxy como intermediario entre los recursos dentro de la red local y los recursos fuera de la red, como en Azure. Los servidores proxy son útiles para muchas aplicaciones, como el aislamiento y la seguridad de la red, y la supervisión y el registro. Azure File Sync puede interoperar completamente con un servidor proxy, aunque debe configurar manualmente los valores del punto de conexión de proxy para su entorno con Azure File Sync. Esto se debe hacer a través de PowerShell con los cmdlets del servidor de Azure File Sync. 

Para obtener más información sobre cómo configurar Azure File Sync con un servidor proxy, consulte [Configuración de Azure File Sync con un servidor proxy](file-sync-firewall-and-proxy.md).

### <a name="configuring-firewalls-and-service-tags"></a>Configuración de firewalls y etiquetas de servicio
Puede aislar los servidores de archivos de la mayoría de las ubicaciones de Internet con fines de seguridad. Para usar Azure File Sync en su entorno, debe ajustar el firewall para abrirlo y seleccionar servicios de Azure. Para ello, puede recuperar los intervalos de direcciones IP para los servicios que necesita a través de un mecanismo denominado [etiquetas de servicio](../../virtual-network/service-tags-overview.md).

Azure File Sync requiere los intervalos de direcciones IP para los siguientes servicios, identificados por sus etiquetas de servicio:

| Servicio | Descripción | Etiqueta de servicio |
|---------|-------------|-------------|
| Azure File Sync | El servicio Azure File Sync, como se representa mediante el objeto del servicio de sincronización de almacenamiento, es responsable de la actividad principal de sincronización de datos entre un recurso compartido de archivos de Azure y un servidor de archivos de Windows. | `StorageSyncService` |
| Azure Files | Todos los datos sincronizados a través de Azure File Sync se almacenan en el recurso compartido de archivos de Azure. Los archivos modificados en los servidores de archivos de Windows se replican en el recurso compartido de archivos de Azure, y los archivos en capas en el servidor de archivos local se descargan sin problemas cuando un usuario los solicita. | `Storage` |
| Azure Resource Manager | Azure Resource Manager es la interfaz de administración de Azure. Todas las llamadas de administración, incluidas las tareas de registro del servidor de Azure File Sync y del servidor de sincronización en curso, se realizan a través de Azure Resource Manager. | `AzureResourceManager` |
| Azure Active Directory | Azure Active Directory, o Azure AD, contiene las entidades de seguridad de usuario necesarias para autorizar el registro del servidor en un servicio de sincronización de almacenamiento y las entidades de servicio necesarias para la autorización de Azure File Sync para el acceso a sus recursos de la nube. | `AzureActiveDirectory` |

Si usa Azure File Sync en Azure, aunque sea una región diferente, puede usar el nombre de la etiqueta de servicio directamente en el grupo de seguridad de red para permitir el tráfico a ese servicio. Para saber cómo, vea [Grupos de seguridad de red](../../virtual-network/network-security-groups-overview.md). 

Si usa Azure File Sync de forma local, puede usar la API de etiquetas de servicio para obtener intervalos de direcciones IP específicos para la lista de elementos permitidos del firewall. Existen dos formas de obtener esta información:

- La lista actual de intervalos de direcciones IP de todos los servicios de Azure que admiten etiquetas de servicio se publica semanalmente en el Centro de descarga de Microsoft en forma de documento JSON. Cada nube de Azure tiene su propio documento JSON con los intervalos de direcciones IP pertinentes para esa nube:
    - [Azure público](https://www.microsoft.com/download/details.aspx?id=56519)
    - [Azure US Government](https://www.microsoft.com/download/details.aspx?id=57063)
    - [Azure en China](https://www.microsoft.com/download/details.aspx?id=57062)
    - [Azure Alemania](https://www.microsoft.com/download/details.aspx?id=57064)
- La API de detección de etiquetas de servicio (versión preliminar) permite recuperar mediante programación la lista actual de etiquetas de servicio. En la versión preliminar, es posible que la API de detección de etiquetas de servicio devuelva información menos actual que la información incluida en los documentos JSON publicados en el Centro de descarga de Microsoft. Puede usar la superficie de API en función de su preferencia de automatización:
    - [REST API](/rest/api/virtualnetwork/servicetags/list)
    - [Azure PowerShell](/powershell/module/az.network/Get-AzNetworkServiceTag)
    - [CLI de Azure](/cli/azure/network#az_network_list_service_tags)

Para obtener más información sobre cómo usar la API de etiquetas de servicio para recuperar las direcciones de sus servicios, consulte [Lista de direcciones IP de Azure File Sync permitidas](file-sync-firewall-and-proxy.md#allow-list-for-azure-file-sync-ip-addresses).

### <a name="tunneling-traffic-over-a-virtual-private-network-or-expressroute"></a>Tunelización del tráfico a través de una red privada virtual o de ExpressRoute
Algunas organizaciones requieren la comunicación con Azure para pasar por un túnel de red, como una red privada virtual (VPN) o ExpressRoute, para una capa de seguridad adicional o para garantizar que la comunicación con Azure siga una ruta determinista. 

Cuando se establece un túnel de red entre la red local y Azure, se empareja la red local con una o más redes virtuales de Azure. Una [red virtual](../../virtual-network/virtual-networks-overview.md) es similar a una red tradicional que puede usar en el entorno local. Al igual que una cuenta de almacenamiento de Azure o una máquina virtual de Azure, una red virtual es un recurso de Azure que se implementa en un grupo de recursos. 

Azure Files y File Sync admiten los siguientes mecanismos para tunelizar el tráfico entre los servidores locales y Azure:

- [Azure VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md): Una puerta de enlace de VPN es un tipo específico de puerta de enlace de red virtual que se usa para enviar tráfico cifrado entre una red virtual de Azure y una ubicación alternativa (por ejemplo, en un entorno local) a través de Internet. Una instancia de Azure VPN Gateway es un recurso de Azure que se puede implementar en un grupo de recursos junto con una cuenta de almacenamiento u otros recursos de Azure. Dado que Azure File Sync está diseñado para usarse con un servidor de archivos de Windows local, normalmente usaría una [VPN de sitio a sitio (S2S)](../../vpn-gateway/design.md#s2smulti), aunque es técnicamente posible usar una [VPN de punto a sitio (P2S)](../../vpn-gateway/point-to-site-about.md). 

    Las conexiones VPN de sitio a sitio (S2S) conectan Azure Virtual Network y la red local de su organización. Una conexión de VPN de sitio a sitio le permite configurar una conexión VPN una vez, para un servidor VPN o un dispositivo hospedado en la red de su organización, en lugar de hacerlo para cada dispositivo cliente que necesite tener acceso al recurso compartido de archivos de Azure. Para simplificar la implementación de una conexión VPN S2S, consulte [Configuración de una VPN de sitio a sitio (S2S) para su uso con Azure Files](../files/storage-files-configure-s2s-vpn.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json).

- [ExpressRoute](../../expressroute/expressroute-introduction.md), que permite crear una ruta definida entre Azure y la red local que no atraviesa Internet. Como ExpressRoute proporciona una ruta de acceso dedicada entre el centro de recursos local y Azure, ExpressRoute puede resultar útil cuando se tiene en cuenta el rendimiento de la red. ExpressRoute también es una buena opción cuando la directiva o los requisitos normativos de la organización requieren una ruta de acceso determinista a los recursos en la nube.

### <a name="private-endpoints"></a>Puntos de conexión privados
Además de los puntos de conexión públicos predeterminados que Azure Files y File Sync proporcionan a través de la cuenta de almacenamiento y el servicio de sincronización de almacenamiento, Azure Files y File Sync ofrecen la opción de tener uno o más puntos de conexión privados por recurso. Cuando se crea un punto de conexión privado para un recurso de Azure, este obtiene una dirección IP privada del espacio de direcciones de su red virtual, de forma muy parecida a cómo un servidor de archivos de Windows local tiene una dirección IP dentro del espacio de direcciones dedicado de su red local. 

> [!Important]  
> Para usar puntos de conexión privados en el recurso del servicio de sincronización de almacenamiento, debe usar la versión del agente Azure File Sync versión 10.1 o superior. Las versiones del agente anteriores a 10.1 no admiten puntos de conexión privados en el servicio de sincronización de almacenamiento. Todas las versiones anteriores del agente admiten puntos de conexión privados en el recurso de la cuenta de almacenamiento.

Un punto de conexión privado está asociado a una subred de red virtual de Azure específica. Las cuentas de almacenamiento y los servicios de sincronización de almacenamiento pueden tener puntos de conexión privados en más de una red virtual.

El uso de puntos de conexión privados le permite:
- Conectarse de forma segura a sus recursos de Azure desde redes locales mediante una conexión VPN o ExpressRoute con emparejamiento privado.
- Proteger sus recursos de Azure mediante la deshabilitación de los puntos de conexión públicos para Azure Files y File Sync. De forma predeterminada, crear un punto de conexión privado no bloquea las conexiones al punto de conexión público.
- Aumentar la seguridad de la red virtual, al permitirle bloquear la filtración de datos desde la red virtual (y los límites de emparejamiento).

Para crear un punto de conexión privado, consulte [Configuración de puntos de conexión privados para Azure File Sync](file-sync-networking-endpoints.md).

### <a name="private-endpoints-and-dns"></a>Puntos de conexión privados y DNS
Cuando se crea un punto de conexión privado, de forma predeterminada también se crea una zona DNS privada (o se actualiza una existente) correspondiente al subdominio `privatelink`. Para las regiones de la nube pública, estas zonas DNS son `privatelink.file.core.windows.net` para Azure Files y `privatelink.afs.azure.net` para Azure File Sync.

> [!Note]  
> En este artículo se usa el sufijo DNS de la cuenta de almacenamiento para las regiones públicas de Azure `core.windows.net`. Este comentario también se aplica a nubes soberanas de Azure, como la nube de Azure US Government y la nube de Azure China; simplemente sustituya los sufijos adecuados en su entorno. 

Cuando crea puntos de conexión privados para una cuenta de almacenamiento y un servicio de sincronización de almacenamiento, se crean registros D para estos en sus respectivas zonas DNS privadas. También se actualiza la entrada DNS pública de modo que los nombres de dominio completos normales son CNAME para el nombre privatelink pertinente. Esto permite que los nombres de dominio completos apunten a las direcciones IP del punto de conexión privado cuando el solicitante se encuentre dentro de la red virtual y apunten a las direcciones IP del punto de conexión público cuando el solicitante esté fuera de la red virtual. 

Para Azure Files, cada punto de conexión privado tiene un nombre de dominio completo único, que sigue el patrón `storageaccount.privatelink.file.core.windows.net`, asignado a una dirección IP privada para el punto de conexión privado. Para Azure File Sync, cada punto de conexión privado tiene cuatro nombres de dominio completos, para los cuatro puntos de conexión diferentes que Azure File Sync expone: administración, sincronización (principal), sincronización (secundaria) y supervisión. Los nombres de dominio completos de estos puntos de conexión normalmente seguirán el nombre del servicio de sincronización de almacenamiento, a menos que el nombre contenga caracteres que no sean ASCII. Por ejemplo, si el nombre del servicio de sincronización de almacenamiento es `mysyncservice` en la región Oeste de EE. UU. 2, los puntos de conexión equivalentes serían `mysyncservicemanagement.westus2.afs.azure.net`, `mysyncservicesyncp.westus2.afs.azure.net`, `mysyncservicesyncs.westus2.afs.azure.net` y `mysyncservicemonitoring.westus2.afs.azure.net`. Cada punto de conexión privado para un servicio de sincronización de almacenamiento contendrá 4 direcciones IP distintas. 

Como la zona DNS privada de Azure está conectada a la red virtual que contiene el punto de conexión privado, puede observar la configuración de DNS cuando se llama al cmdlet `Resolve-DnsName` desde PowerShell en una máquina virtual de Azure (como alternativa `nslookup` en Windows y Linux):

```powershell
Resolve-DnsName -Name "storageaccount.file.core.windows.net"
```

En este ejemplo, la cuenta de almacenamiento `storageaccount.file.core.windows.net` se resuelve en la dirección IP privada del punto de conexión privado, que resulta ser `192.168.0.4`.

```Output
Name                              Type   TTL   Section    NameHost
----                              ----   ---   -------    --------
storageaccount.file.core.windows. CNAME  29    Answer     csostoracct.privatelink.file.core.windows.net
net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 1769
Section    : Answer
IP4Address : 192.168.0.4


Name                   : privatelink.file.core.windows.net
QueryType              : SOA
TTL                    : 269
Section                : Authority
NameAdministrator      : azureprivatedns-host.microsoft.com
SerialNumber           : 1
TimeToZoneRefresh      : 3600
TimeToZoneFailureRetry : 300
TimeToExpiration       : 2419200
DefaultTTL             : 300
```

Si ejecuta el mismo comando en el entorno local, verá que el mismo nombre de cuenta de almacenamiento se resuelve en la dirección IP pública de la cuenta de almacenamiento; `storageaccount.file.core.windows.net` es un registro CNAME para `storageaccount.privatelink.file.core.windows.net`, que, a su vez, es un registro CNAME para el clúster de almacenamiento de Azure que hospeda la cuenta de almacenamiento:

```Output
Name                              Type   TTL   Section    NameHost
----                              ----   ---   -------    --------
storageaccount.file.core.windows. CNAME  60    Answer     storageaccount.privatelink.file.core.windows.net
net
storageaccount.privatelink.file.c CNAME  60    Answer     file.par20prdstr01a.store.core.windows.net
ore.windows.net

Name       : file.par20prdstr01a.store.core.windows.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 52.239.194.40
```

Esto refleja el hecho de que Azure Files y Azure File Sync pueden exponer puntos de conexión públicos y uno o más puntos de conexión privados por recurso. Para asegurarse de que los nombres de dominio completos de sus recursos se resuelven en las direcciones IP privadas de los puntos de conexión privados, debe cambiar la configuración de sus servidores DNS locales. Esto puede realizarse de varias maneras:

- Modificando el archivo de hosts en sus clientes para que los nombres de dominio completos de sus cuentas de almacenamiento y servicios de sincronización de almacenamiento se resuelvan en las direcciones IP privadas deseadas. Esto no se recomienda en absoluto para entornos de producción, ya que necesitará realizar estos cambios en cada cliente que necesite acceso a sus puntos de conexión privados. Los cambios en los puntos de conexión o recursos privados (eliminaciones, modificaciones, etc.) no se controlarán automáticamente.
- Creación de zonas DNS en los servidores locales para `privatelink.file.core.windows.net` y `privatelink.afs.azure.net` con registros D para los recursos de Azure. Este método tiene la ventaja de que los clientes de su entorno local podrán resolver automáticamente los recursos de Azure sin necesidad de configurar cada cliente; sin embargo, esta solución es igualmente frágil para modificar el archivo de hosts, ya que los cambios no se reflejan. Aunque esta solución es frágil, puede ser la mejor opción para algunos entornos.
- Reenvíe las zonas `core.windows.net` y `afs.azure.net` de los servidores DNS locales a la zona DNS privada de Azure. Se puede acceder al host DNS privado de Azure a través de una dirección IP especial (`168.63.129.16`) que solo es accesible dentro de las redes virtuales que están vinculadas a la zona DNS privada de Azure. Para solucionar esta limitación, puede ejecutar servidores DNS adicionales dentro de la red virtual que reenviarán `core.windows.net` y `afs.azure.net` a las zonas DNS privadas de Azure equivalentes. Para simplificar esta configuración, se han proporcionado cmdlets de PowerShell que implementarán automáticamente los servidores DNS en la red virtual de Azure y los configurarán según sea necesario. Para aprender a configurar el reenvío de DNS, consulte [Configuración de DNS con Azure Files](../files/storage-files-networking-dns.md?toc=%2fazure%2fstorage%2ffilesync%2ftoc.json).

## <a name="encryption-in-transit"></a>Cifrado en tránsito
Las conexiones realizadas desde el agente de Azure File Sync en el recurso compartido de archivos de Azure o el servicio de sincronización de almacenamiento siempre están cifradas. Aunque las cuentas de almacenamiento de Azure tienen una opción para deshabilitar el requisito del cifrado en tránsito para las comunicaciones a Azure Files (y los otros servicios de almacenamiento de Azure que se administran fuera de la cuenta de almacenamiento), la deshabilitación de esta opción no afectará al cifrado de Azure File Sync cuando se comunique con Azure Files. De forma predeterminada, todas las cuentas de Azure Storage tienen habilitado el cifrado en tránsito. 

Para obtener más información sobre el cifrado en tránsito, consulte [Requerir transferencia segura en Azure Storage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="see-also"></a>Consulte también
- [Planeamiento de una implementación de Azure File Sync](file-sync-planning.md)
- [Implementación de Azure File Sync](file-sync-deployment-guide.md)