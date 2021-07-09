---
title: Consideraciones de redes para Azure Files | Microsoft Docs
description: Una introducción a las opciones de red para Azure Files.
author: roygara
ms.service: storage
ms.topic: overview
ms.date: 02/22/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 3ba86c8f0d28e48e0c93834b30afe0ad77bfa87d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110477420"
---
# <a name="azure-files-networking-considerations"></a>Consideraciones de redes para Azure Files 
Puede conectarse a un recurso compartido de archivos de Azure de dos maneras:

- Acceder al recurso compartido directamente mediante los protocolos Bloque de mensajes del servidor (SMB), Network File System (NFS) (versión preliminar) o FileREST. Este patrón de acceso se emplea principalmente cuando se quieren eliminar tantos servidores locales como sea posible.
- Cree una memoria caché del recurso compartido de archivos de Azure en un servidor local (o en una VM de Azure) con Azure File Sync y acceda a los datos del recurso compartido de archivos desde el servidor local con el protocolo que elija (SMB, NFS, FTPS, etc.) para su caso de uso. Este patrón de acceso es útil porque combina lo mejor de la escala en la nube y el rendimiento local junto con los servicios conectables sin servidor, como Azure Backup.

Este artículo se centra en cómo configurar las funciones de red cuando el caso de uso necesita acceder al recurso compartido de archivos de Azure directamente, en lugar de usar Azure File Sync. Para obtener más información acerca de las consideraciones de red para una implementación de Azure File Sync, consulte [Consideraciones de redes para Azure File Sync](../file-sync/file-sync-networking-overview.md).

La configuración de red de los recursos compartidos de archivos de Azure se realiza en la cuenta de Azure Storage. Una cuenta de almacenamiento es una construcción de administración que representa un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas. Las cuentas de almacenamiento exponen varias configuraciones que le ayudan a proteger el acceso de red a los recursos compartidos de archivos: puntos de conexión de red, configuración de firewall de la cuenta de almacenamiento y cifrado en tránsito. 

> [!Important]  
> Azure Files admite varias opciones de enrutamiento de red. La opción predeterminada, el enrutamiento de Microsoft, funciona con todas las configuraciones de Azure Files. La opción de enrutamiento de Internet no admite escenarios de unión a un dominio de AD ni Azure File Sync.

Se recomienda leer [Planeamiento de una implementación de Azure Files](storage-files-planning.md) antes de pasar a esta guía conceptual.

:::row:::
    :::column:::
        <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/jd49W33DxkQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    :::column-end:::
    :::column:::
        Este vídeo es una guía y demostración sobre cómo exponer de forma segura recursos compartidos de archivos de Azure directamente para las aplicaciones y trabajadores de la información en cinco sencillos pasos. En las secciones siguientes se proporcionan vínculos y contexto adicional de la documentación a la que se hace referencia en el vídeo.
   :::column-end:::
:::row-end:::

## <a name="accessing-your-azure-file-shares"></a>Acceso a los recursos compartidos de archivos de Azure
Cuando se implementa un recurso compartido de archivos de Azure en una cuenta de almacenamiento, este es accesible inmediatamente mediante el punto de conexión público de la cuenta de almacenamiento. Esto significa que las solicitudes autenticadas, como las solicitudes autorizadas por la identidad de inicio de sesión de un usuario, pueden originarse de forma segura dentro o fuera de Azure. 

En muchos entornos de cliente, se producirá un error en el montaje inicial del recurso compartido de archivos de Azure en la estación de trabajo local, aunque los montajes desde las máquinas virtuales de Azure se realicen correctamente. El motivo es que muchas organizaciones y proveedores de servicios de Internet (ISP) bloquean el puerto que usa SMB para comunicarse, el puerto 445. Esta práctica procede de las instrucciones de seguridad sobre las versiones heredadas y en desuso del protocolo SMB. Aunque SMB 3.x es un protocolo seguro para Internet, las versiones anteriores, especialmente SMB 1.0, no lo son. Solo se puede acceder externamente a recursos compartidos de archivos de Azure por medio de SMB 3.x y el protocolo FileREST (que también es un protocolo seguro para Internet) a través del punto de conexión público.

Dado que la manera más fácil de acceder al recurso compartido de archivos SMB de Azure desde el entorno local es abrir la red local al puerto 445, Microsoft recomienda los siguientes pasos para eliminar SMB 1.0 de su entorno:

1. Asegúrese de que se ha quitado o deshabilitado SMB 1.0 en los dispositivos de la organización. Todas las versiones de Windows y Windows Server compatibles actualmente admiten la eliminación o desactivación de SMB 1.0 y, a partir de Windows 10, versión 1709, SMB 1.0 no se instala en Windows de forma predeterminada. Para obtener más información sobre cómo deshabilitar SMB 1.0, consulte nuestras páginas específicas para cada sistema operativo:
    - [Protección de Windows y Windows Server](storage-how-to-use-files-windows.md#securing-windowswindows-server)
    - [Protección de Linux](storage-how-to-use-files-linux.md#securing-linux)
1. Asegúrese de que ningún producto de la organización requiere SMB 1.0 y quite los que sí lo requieran. Mantenemos un [Centro de enrutamiento de productos de SMB1](https://aka.ms/stillneedssmb1), que contiene todos los productos propios y de terceros que Microsoft sabe que requieren de SMB 1.0. 
1. (Opcional) Use un firewall de terceros con la red local de su organización para impedir que el tráfico de SMB 1.0 abandone los límites de la organización.

Si la organización requiere el bloqueo del puerto 445 según las directivas o normativas, o si la organización requiere que el tráfico a Azure siga una ruta de acceso determinista, puede usar Azure VPN Gateway o ExpressRoute para tunelizar el tráfico a los recursos compartidos de archivos de Azure. Los recursos compartidos de NFS no tienen estos requisitos, ya que no necesitan el puerto 445.

> [!Important]  
> Incluso si decide usar un método alternativo para acceder a los recursos compartidos de archivos de Azure, Microsoft aún recomienda quitar SMB 1.0 del entorno.

### <a name="tunneling-traffic-over-a-virtual-private-network-or-expressroute"></a>Tunelización del tráfico a través de una red privada virtual o de ExpressRoute
Cuando se establece un túnel de red entre la red local y Azure, se empareja la red local con una o más redes virtuales de Azure. Una [red virtual](../../virtual-network/virtual-networks-overview.md) es similar a una red tradicional que puede usar en el entorno local. Al igual que una cuenta de almacenamiento de Azure o una máquina virtual de Azure, una red virtual es un recurso de Azure que se implementa en un grupo de recursos. 

Azure Files admite los siguientes mecanismos para tunelizar el tráfico entre los servidores y las estaciones de trabajo locales y los recursos compartidos de archivos de SMB y NFS de Azure:

- Una puerta de enlace de VPN es un tipo específico de puerta de enlace de red virtual que se usa para enviar tráfico cifrado entre una red virtual de Azure y una ubicación alternativa (por ejemplo, en un entorno local) a través de Internet. Una instancia de Azure VPN Gateway es un recurso de Azure que se puede implementar en un grupo de recursos junto con una cuenta de almacenamiento u otros recursos de Azure. Las puertas de enlace de VPN exponen dos tipos de conexión distintos:
    - Las conexiones de puerta de enlace de [VPN de punto a sitio](../../vpn-gateway/point-to-site-about.md), que son conexiones VPN entre Azure y un cliente individual. Esta solución es especialmente útil para los dispositivos que no forman parte de la red local de su organización, como los trabajadores remotos que quieren montar su recurso compartido de archivos de Azure desde casa, una cafetería o un hotel mientras viajan. Para usar una conexión VPN de punto a sitio con Azure Files, se debe configurar una conexión VPN de punto a sitio para cada cliente que quiera conectarse. Para simplificar la implementación de una conexión VPN P2S, consulte [Configuración de una VPN de punto a sitio (P2S) en Windows para su uso con Azure Files](storage-files-configure-p2s-vpn-windows.md) y [Configuración de una VPN de punto a sitio (P2S) en Linux para su uso con Azure Files](storage-files-configure-p2s-vpn-linux.md).
    - Las [VPN de sitio a sitio](../../vpn-gateway/design.md#s2smulti), que son conexiones VPN entre Azure y la red de su organización. Una conexión de VPN de sitio a sitio le permite configurar una conexión VPN una vez, para un servidor VPN o un dispositivo hospedado en la red de su organización, en lugar de hacerlo para cada dispositivo cliente que necesite tener acceso al recurso compartido de archivos de Azure. Para simplificar la implementación de una conexión VPN S2S, consulte [Configuración de una VPN de sitio a sitio (S2S) para su uso con Azure Files](storage-files-configure-s2s-vpn.md).
- [ExpressRoute](../../expressroute/expressroute-introduction.md), que permite crear una ruta definida entre Azure y la red local que no atraviesa Internet. Como ExpressRoute proporciona una ruta de acceso dedicada entre el centro de recursos local y Azure, ExpressRoute puede resultar útil cuando se tiene en cuenta el rendimiento de la red. ExpressRoute también es una buena opción cuando la directiva o los requisitos normativos de la organización requieren una ruta de acceso determinista a los recursos en la nube.

Con independencia del método de tunelización que use para acceder a los recursos compartidos de archivos de Azure, necesita un mecanismo para asegurarse de que el tráfico a su cuenta de almacenamiento atraviese el túnel en lugar de la conexión a Internet normal. Técnicamente es posible enrutarlo al punto de conexión público de la cuenta de almacenamiento; sin embargo, para ello es necesario codificar de forma rígida todas las direcciones IP de los clústeres de almacenamiento de Azure en una región, ya que las cuentas de almacenamiento pueden moverse entre los clústeres de almacenamiento en cualquier momento. También es necesario actualizar en todo momento las asignaciones de direcciones IP, ya que continuamente se agregan nuevos clústeres.

En lugar de codificar de forma rígida la dirección IP de las cuentas de almacenamiento en las reglas de enrutamiento de VPN, se recomienda usar puntos de conexión privados, que proporcionan a la cuenta de almacenamiento una dirección IP del espacio de direcciones de una red virtual de Azure. Dado que la creación de un túnel a Azure establece un emparejamiento entre la red local y una o más redes virtuales, esto permite el enrutamiento correcto de forma duradera.

### <a name="private-endpoints"></a>Puntos de conexión privados
Además del punto de conexión público predeterminado para una cuenta de almacenamiento, Azure Files ofrece la opción de tener uno o más puntos de conexión privados. Un punto de conexión privado es un punto de conexión que solo es accesible dentro de una red virtual de Azure. Cuando se crea un punto de conexión privado para la cuenta de almacenamiento, esta obtiene una dirección IP privada del espacio de direcciones de la red virtual, de forma muy parecida a cómo un servidor de archivos local o un dispositivo NAS reciben una dirección IP dentro del espacio de direcciones dedicado de la red local. 

Un punto de conexión privado está asociado a una subred de red virtual de Azure específica. Una cuenta de almacenamiento puede tener puntos de conexión privados en más de una red virtual.

El uso de puntos de conexión privados con Azure Files le permite:
- Conectarse de forma segura a los recursos compartidos de archivos de Azure desde redes locales mediante una conexión VPN o ExpressRoute con emparejamiento privado.
- Proteger los recursos compartidos de archivos mediante la configuración del firewall de la cuenta de almacenamiento para bloquear todas las conexiones el punto de conexión público. De forma predeterminada, crear un punto de conexión privado no bloquea las conexiones al punto de conexión público.
- Aumentar la seguridad de la red virtual, al permitirle bloquear la filtración de datos desde la red virtual (y los límites de emparejamiento).

Para crear un punto de conexión privado, consulte [Configuración de puntos de conexión privados para Azure Files](storage-files-networking-endpoints.md).

### <a name="private-endpoints-and-dns"></a>Puntos de conexión privados y DNS
Cuando se crea un punto de conexión privado, de forma predeterminada también se crea una zona DNS privada (o se actualiza una existente) correspondiente al subdominio `privatelink`. En realidad, no es necesario crear una zona DNS privada para usar un punto de conexión privado para la cuenta de almacenamiento, pero se recomienda encarecidamente en general y se requiere de forma explícita al montar el recurso compartido de archivos de Azure con una entidad de seguridad de usuario de Active Directory o al acceder desde la API de FileREST.

> [!Note]  
> En este artículo se usa el sufijo DNS de la cuenta de almacenamiento para las regiones públicas de Azure `core.windows.net`. Este comentario también se aplica a nubes soberanas de Azure, como la nube de Azure US Government y la nube de Azure China; simplemente sustituya los sufijos adecuados en su entorno. 

En la zona DNS privada, se va a crea un registro D para `storageaccount.privatelink.file.core.windows.net` y un registro CNAME para el nombre normal de la cuenta de almacenamiento, que sigue el patrón `storageaccount.file.core.windows.net`. Como la zona DNS privada de Azure está conectada a la red virtual que contiene el punto de conexión privado, puede observar la configuración de DNS cuando se llama al cmdlet `Resolve-DnsName` desde PowerShell en una máquina virtual de Azure (como alternativa `nslookup` en Windows y Linux):

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

Esto refleja el hecho de que la cuenta de almacenamiento puede exponer el punto de conexión público y uno o varios puntos de conexión privados. Para asegurarse de que el nombre de la cuenta de almacenamiento se resuelve en la dirección IP privada del punto de conexión privado, debe cambiar la configuración en los servidores DNS locales. Esto puede realizarse de varias maneras:

- Modifique el archivo de hosts de los clientes para que `storageaccount.file.core.windows.net` se resuelva en la dirección IP privada del punto de conexión privado deseado. Este método no se recomienda en entornos de producción, ya que estos cambios se deberán realizar en todos los clientes que quieran montar los recursos compartidos de archivos de Azure y los cambios en la cuenta de almacenamiento o en el punto de conexión privado no se administrarán automáticamente.
- Cree un registro D para `storageaccount.file.core.windows.net` en los servidores DNS locales. Este método tiene la ventaja de que los clientes de su entorno local podrán resolver automáticamente la cuenta de almacenamiento sin necesidad de configurar cada cliente; sin embargo, esta solución es igualmente frágil para modificar el archivo de hosts, ya que los cambios no se reflejan. Aunque esta solución es frágil, puede ser la mejor opción para algunos entornos.
- Reenvíe la zona `core.windows.net` de los servidores DNS locales a la zona DNS privada de Azure. Se puede acceder al host DNS privado de Azure a través de una dirección IP especial (`168.63.129.16`) que solo es accesible dentro de las redes virtuales que están vinculadas a la zona DNS privada de Azure. Para solucionar esta limitación, puede ejecutar servidores DNS adicionales dentro de la red virtual que reenviarán `core.windows.net` a la zona DNS privada de Azure. Para simplificar esta configuración, se han proporcionado cmdlets de PowerShell que implementarán automáticamente los servidores DNS en la red virtual de Azure y los configurarán según sea necesario. Para aprender a configurar el reenvío de DNS, consulte [Configuración de DNS con Azure Files](storage-files-networking-dns.md).

## <a name="storage-account-firewall-settings"></a>Configuración del firewall de la cuenta de almacenamiento
Un firewall es una directiva de red que controla qué solicitudes pueden acceder al punto de conexión público de una cuenta de almacenamiento. Con el firewall de la cuenta de almacenamiento, puede restringir el acceso al punto de conexión público de la cuenta de almacenamiento a determinadas direcciones IP o intervalos de direcciones IP o a una red virtual. En general, la mayoría de las directivas de firewall de una cuenta de almacenamiento restringirán el acceso de red a una o varias redes virtuales. 

Existen dos enfoques para restringir el acceso de una cuenta de almacenamiento a una red virtual:
- Cree uno o varios puntos de conexión privados para la cuenta de almacenamiento y restrinja todo el acceso al punto de conexión público. De esta forma se garantiza que solo el tráfico que se origina en las redes virtuales deseadas pueda acceder a los recursos compartidos de archivos de Azure dentro de la cuenta de almacenamiento.
- Restrinja el punto de conexión público a una o más redes virtuales. Para ello, se usa una funcionalidad de la red virtual llamada *puntos de conexión de servicio*. Al restringir el tráfico a una cuenta de almacenamiento a través de un punto de conexión de servicio, sigue teniendo acceso a la cuenta de almacenamiento a través de la dirección IP pública.

> [!NOTE]
> Los recursos compartidos de NFS no pueden acceder al punto de conexión público de la cuenta de almacenamiento mediante la dirección IP pública, solo pueden acceder al punto de conexión público de la cuenta de almacenamiento mediante redes virtuales. Los recursos compartidos de NFS también pueden acceder a la cuenta de almacenamiento mediante puntos de conexión privados.

Para más información sobre cómo configurar el firewall de la cuenta de almacenamiento, consulte [Configuración de los firewalls y las redes virtuales de Azure Storage](../common/storage-network-security.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="encryption-in-transit"></a>Cifrado en tránsito

> [!IMPORTANT]
> En esta sección se tratan los detalles del cifrado en tránsito para recursos compartidos SMB. Para más información sobre el cifrado en tránsito con recursos compartidos NFS, consulte [Seguridad](storage-files-compare-protocols.md#security).

De forma predeterminada, todas las cuentas de Azure Storage tienen habilitado el cifrado en tránsito. Esto significa que al montar un recurso compartido de archivos a través de SMB o acceder a él a través del protocolo de FileREST (por ejemplo, a través de Azure Portal, la CLI o PowerShell, o los SDK de Azure), Azure Files solo permitirá la conexión si se realiza con una versión posterior a SMB 3.x con cifrado o HTTPS. Los clientes que no admiten SMB 3.x, o los clientes que admiten SMB 3.x, pero no al cifrado SMB no podrán montar el recurso compartido de archivos de Azure si está habilitado el cifrado en tránsito. Para obtener más información sobre qué sistemas operativos admiten SMB 3.x con cifrado, consulte nuestra documentación detallada para [Windows](storage-how-to-use-files-windows.md), [macOS](storage-how-to-use-files-mac.md) y [Linux](storage-how-to-use-files-linux.md). Todas las versiones actuales de PowerShell, la CLI y los SDK admiten HTTPS.  

Puede deshabilitar el cifrado en tránsito para una cuenta de almacenamiento de Azure. Cuando el cifrado está deshabilitado, Azure Files también permite el uso de SMB 2.1, SMB 3.x sin cifrado y las llamadas API de FileREST sin cifrar a través de HTTP. La razón principal para deshabilitar el cifrado en tránsito es admitir una aplicación heredada que debe ejecutarse en un sistema operativo anterior, como Windows Server 2008 R2 o una distribución de Linux anterior. Azure Files solo permite conexiones SMB 2.1 dentro de la misma región de Azure del recurso compartido de archivos de Azure. Un cliente SMB 2.1 fuera de la región de Azure del recurso compartido de archivos de Azure (por ejemplo, en un entorno local o en una región de Azure diferente) no podrá acceder al recurso compartido de archivos.

Para obtener más información sobre el cifrado en tránsito, consulte [Requerir transferencia segura en Azure Storage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="see-also"></a>Consulte también
- [Introducción a Azure Files](storage-files-introduction.md)
- [Planeamiento de una implementación de Azure Files](storage-files-planning.md)