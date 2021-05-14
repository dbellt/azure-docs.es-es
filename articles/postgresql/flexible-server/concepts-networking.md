---
title: 'Información general sobre redes: Servidor flexible de Azure Database for PostgreSQL'
description: Obtenga información sobre las opciones de conectividad y redes en la opción de implementación Servidor flexible para Azure Database for PostgreSQL
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/22/2021
ms.openlocfilehash: 5b832ca7f1b5fb8a6b0044ca299c75f01a2d0f32
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987060"
---
# <a name="networking-overview---azure-database-for-postgresql---flexible-server"></a>Información general sobre redes: Servidor flexible de Azure Database for PostgreSQL

> [!IMPORTANT]
> Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

En este artículo se describen los conceptos de conectividad y redes para Servidor flexible de Azure Database for PostgreSQL. 

## <a name="choosing-a-networking-option"></a>Elección de una opción de red
Tiene dos opciones de red para Servidor flexible de Azure Database for PostgreSQL. Las opciones son **acceso privado (integración con red virtual)** y **acceso público (direcciones IP permitidas)** . Al crear el servidor, debe elegir una opción. 

> [!NOTE]
> La opción de red no se puede cambiar una vez que se haya creado el servidor. 

* **Acceso privado (integración con red virtual)** : puede implementar el servidor flexible en la instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes virtuales de Azure proporcionan una comunicación de red privada y segura. Los recursos de una red virtual se pueden comunicar mediante direcciones IP privadas.

   Elija la opción Integración con red virtual si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure en la misma red virtual a un servidor flexible mediante direcciones IP privadas
   * Uso de VPN o ExpressRoute para conectarse desde recursos que no son de Azure al servidor flexible
   * El servidor flexible no tiene un punto de conexión público

* **Acceso público (direcciones IP permitidas)** : se accede al servidor flexible a través de un punto de conexión público. El punto de conexión público es una dirección DNS que se puede resolver públicamente. La frase "direcciones IP permitidas" hace referencia a un intervalo de direcciones IP a las que decida conceder permiso para acceder al servidor. Estos permisos se denominan **reglas de firewall**. 

   Elija el método de acceso público si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure que no admiten redes virtuales
   * Conexión desde recursos externos de Azure que no están conectados mediante VPN o ExpressRoute 
   * El servidor flexible tiene un punto de conexión público

Las características siguientes se aplican tanto si decide usar la opción de acceso privado como la de acceso público:
* Las conexiones desde direcciones IP permitidas se deben autenticar en el servidor PostgreSQL con credenciales válidas
* Hay [cifrado de conexión](#tls-and-ssl) disponible para el tráfico de red
* El servidor tiene un nombre de dominio completo (FQDN). En el caso de la propiedad hostname en las cadenas de conexión, se recomienda usar el FQDN en lugar de una dirección IP.
* Las dos opciones controlan el acceso en el nivel de servidor, no en el nivel de base de datos o de tabla. Tendría que usar las propiedades de los roles de PostgreSQL para controlar el acceso de bases de datos, tablas y otros objetos.


## <a name="private-access-vnet-integration"></a>Acceso privado (integración con red virtual)
El acceso privado con la integración de red virtual (vnet) proporciona una comunicación segura y privada para el servidor flexible de PostgreSQL.

:::image type="content" source="./media/how-to-manage-virtual-network-portal/flexible-pg-vnet-diagram.png" alt-text="Red virtual Postgres de servidor flexible":::

En el diagrama anterior:
1. Los servidores flexibles se insertan en una subred delegada: 10.0.1.0/24 de la VNET **VNet-1**.
2. Las aplicaciones que se implementan en subredes diferentes dentro de la misma red virtual pueden acceder directamente a los servidores flexibles.
3. Las aplicaciones que se implementan en otra red virtual **VNet-2** no tienen acceso directo a servidores flexibles. Debe realizar el [emparejamiento de red virtual de zona DNS privada](#private-dns-zone-and-vnet-peering) para poder acceder al servidor flexible.
   
### <a name="virtual-network-concepts"></a>Conceptos de redes virtuales
Estos son algunos conceptos que debe conocer al usar redes virtuales con servidores flexibles de PostgreSQL.

* **Red virtual**: una instancia de Azure Virtual Network (VNet) contiene un espacio de direcciones IP privadas que se configura para su uso. Para obtener más información sobre las redes virtuales de Azure, visite la [Información general sobre Azure Virtual Network](../../virtual-network/virtual-networks-overview.md).

    La red virtual debe estar en la misma región de Azure que el servidor flexible.

* **Subred delegada**: una red virtual contiene subredes. Las subredes permiten segmentar la red virtual en espacios de direcciones más pequeños. Los recursos de Azure se implementan en subredes específicas dentro de una red virtual. 

   El servidor flexible de PostgreSQL debe estar en una subred **delegada** para uso exclusivo del servidor flexible de PostgreSQL. Esta delegación significa que solo los servidores flexibles de Azure Database for PostgreSQL pueden usar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada. Puede delegar una subred si asigna su propiedad de delegación como Microsoft.DBforPostgreSQL/flexibleServers.

* **Grupos de seguridad de red (NSG)** : las reglas de seguridad de grupos de seguridad de red permiten filtrar el tipo de tráfico de red que puede fluir dentro y fuera de las interfaces de red y las subredes de redes virtuales. Para más información, consulte [Introducción a los grupos de seguridad de red](../../virtual-network/network-security-groups-overview.md).

* **Integración de DNS privado**: la integración de la zona DNS privada de Azure permite resolver el DNS privado dentro de la red virtual actual o en cualquier red virtual emparejada en la región en la que esté vinculada la zona DNS privada. Consulte la [documentación de la zona DNS privada](https://docs.microsoft.com/azure/dns/private-dns-overview) para obtener más detalles.

Obtenga información sobre cómo crear un servidor flexible con acceso privado (integración con red virtual) en [Azure Portal](how-to-manage-virtual-network-portal.md) o [la CLI de Azure](how-to-manage-virtual-network-cli.md).

> [!NOTE]
> Si usa el servidor DNS personalizado, debe usar un reenviador DNS para resolver el FQDN de Azure Database for PostgreSQL: servidor flexible. Para obtener más información, consulte [Resolución de nombres con su propio servidor DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

### <a name="private-dns-zone-and-vnet-peering"></a>Zona DNS privada y emparejamiento de la red virtual

La configuración de la zona DNS privada y el emparejamiento de la red virtual son independientes entre sí.

* De forma predeterminada, una nueva zona DNS privada se aprovisiona automáticamente por servidor con el nombre de servidor proporcionado. Sin embargo, si desea configurar su propia zona DNS privada para usarla con el servidor flexible, consulte la documentación de [ información general de DNS privado](https://docs.microsoft.com/azure/dns/private-dns-overview).
* Si desea conectarse al servidor flexible desde un cliente que se aprovisiona en otra red virtual, debe vincular la zona DNS privada con la red virtual. Vea la documentación de [cómo vincular la red virtual](https://docs.microsoft.com/azure/dns/private-dns-getstarted-portal#link-the-virtual-network).


### <a name="unsupported-virtual-network-scenarios"></a>Escenarios de red virtual no admitidos
* Punto de conexión público (o dirección IP pública o DNS): un servidor flexible implementado en una red virtual no puede tener un punto de conexión público
* Una vez que se haya implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a otra red virtual o subred. No se puede trasladar la red virtual a otro grupo de recursos o suscripción.
* No se puede aumentar el tamaño de la subred (espacios de direcciones) cuando existen recursos en la subred
* No se admite el emparejamiento de redes virtuales entre regiones


## <a name="public-access-allowed-ip-addresses"></a>Acceso público (direcciones IP permitidas)
Entre las características del método de acceso público se incluyen las siguientes:
* Solo las direcciones IP que permita tienen autorización para acceder al servidor flexible de PostgreSQL. De forma predeterminada, no se permite ninguna dirección IP. Puede agregar direcciones IP durante la creación del servidor o después.
* El servidor PostgreSQL tiene un nombre DNS que se puede resolver de forma pública
* El servidor flexible no está en una de las redes virtuales de Azure
* El tráfico de red hacia y desde el servidor no se realiza a través de una red privada. El tráfico usa las rutas de Internet generales.

### <a name="firewall-rules"></a>Reglas de firewall
La concesión de permisos a una dirección IP se denomina regla de firewall. Si un intento de conexión procede de una dirección IP no permitida, el cliente de origen verá un error.

Obtenga información sobre cómo crear un servidor flexible con acceso público (direcciones IP permitidas) en [Azure Portal](how-to-manage-firewall-portal.md) o [la CLI de Azure](how-to-manage-firewall-cli.md).


### <a name="allowing-all-azure-ip-addresses"></a>Permitir todas las direcciones IP de Azure
Si no hay ninguna dirección IP saliente fija disponible para su servicio de Azure, puede habilitar las conexiones de todas las direcciones IP del centro de datos de Azure.

> [!IMPORTANT]
> La opción **Permitir acceso público desde los recursos y servicios de Azure dentro de Azure** configura el firewall para permitir todas las conexiones desde Azure, incluidas las que provienen de suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.

### <a name="troubleshooting-public-access-issues"></a>Solución de problemas de acceso público
Tenga en cuenta los siguientes puntos cuando el acceso al servicio del servidor de Microsoft Azure Database for PostgreSQL no se comporte de la manera prevista:

* **Los cambios realizados en la lista de permitidos no han surtido efecto todavía:** puede haber un retraso de hasta cinco minutos hasta que los cambios en la configuración del firewall del servidor de Azure Database for PostgreSQL surtan efecto.

* **Error de autenticación:** si un usuario no tiene permisos en el servidor de Azure Database for PostgreSQL o la contraseña usada es incorrecta, se denegará la conexión al servidor de Azure Database for PostgreSQL. La creación de una configuración de firewall solo ofrece a los clientes una oportunidad de intentar conectarse al servidor. Cada cliente todavía tiene que proporcionar las credenciales de seguridad necesarias.

* **Dirección IP del cliente dinámica:** Si tiene una conexión a Internet con un direccionamiento IP dinámico y tiene problemas al acceder al firewall, pruebe una de las soluciones siguientes:

   * Pida a su proveedor de acceso a Internet (ISP) el intervalo de direcciones IP asignado a los equipos cliente que acceden al servidor de Azure Database for PostgreSQL y agréguelo como regla de firewall.
   * Obtenga el direccionamiento IP estático en su lugar para los equipos cliente y luego agregue la dirección IP estática como regla de firewall.

* **La regla de firewall no está disponible en formato IPv6:** las reglas de firewall deben estar en formato IPv4. Si especifica reglas de firewall en formato IPv6, aparece el error de validación.

## <a name="hostname"></a>Nombre de host
Con independencia de la opción de red que elija, se recomienda usar siempre un nombre de dominio completo (FQDN) como nombre de host al conectarse al servidor flexible. No se garantiza que la dirección IP del servidor permanezca estática. El uso del FQDN le ayudará a evitar realizar cambios en la cadena de conexión. 

Ejemplo
* Recomendado `hostname = servername.postgres.database.azure.com`
* Siempre que sea posible, evite usar `hostname = 10.0.0.4` (una dirección privada) o `hostname = 40.2.45.67` (una dirección pública)


## <a name="tls-and-ssl"></a>TLS y SSL
Servidor flexible de Azure Database for PostgreSQL admite la conexión de las aplicaciones cliente al servicio PostgreSQL mediante Seguridad de la capa de transporte (TLS). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente. TLS es un protocolo actualizado de SSL (Capa de sockets seguros).

Servidor flexible de Azure Database for PostgreSQL solo admite conexiones cifradas con Seguridad de la capa de transporte. Se denegarán todas las conexiones entrantes con TLS 1.0 y TLS 1.1. 

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre cómo crear un servidor flexible con **acceso privado (integración con red virtual)** en [Azure Portal](how-to-manage-virtual-network-portal.md) o [la CLI de Azure](how-to-manage-virtual-network-cli.md).
* Obtenga información sobre cómo crear un servidor flexible con **acceso público (direcciones IP permitidas)** en [Azure Portal](how-to-manage-firewall-portal.md) o [la CLI de Azure](how-to-manage-firewall-cli.md).
