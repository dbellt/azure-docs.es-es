---
title: 'Información general sobre redes: Servidor flexible de Azure Database for MySQL'
description: Obtenga información sobre las opciones de conectividad y redes en la opción de implementación Servidor flexible para Azure Database for MySQL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 9/23/2020
ms.openlocfilehash: c83f36216e7488df94c372234d0541a4ee9f99b5
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492229"
---
# <a name="connectivity-and-networking-concepts-for-azure-database-for-mysql---flexible-server-preview"></a>Conceptos de conectividad y redes para Servidor flexible (versión preliminar) de Azure Database for MySQL

En este artículo se describen las opciones de conectividad pública y privada para el servidor. Aprenderá en detalle los conceptos de red para Servidor flexible de Azure Database for MySQL a fin de crear un servidor de forma segura en Azure.

> [!IMPORTANT]
> Servidor flexible de Azure Database for MySQL está en versión preliminar.

## <a name="choosing-a-networking-option"></a>Elección de una opción de red
Tiene dos opciones de red para el Servidor flexible de Azure Database for MySQL. Las opciones son **acceso privado (integración con red virtual)** y **acceso público (direcciones IP permitidas)** . Al crear el servidor, debe elegir una opción. 

> [!NOTE]
> La opción de red no se puede cambiar una vez que se haya creado el servidor. 

* **Acceso privado (Integración con red virtual)** : puede implementar el servidor flexible en la instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes virtuales de Azure proporcionan una comunicación de red privada y segura. Los recursos de una red virtual se pueden comunicar mediante direcciones IP privadas.

   Elija la opción Integración con red virtual si quiere las funcionalidades siguientes:
   * Conexión desde los recursos de Azure en la misma red virtual o [red virtual emparejada](../../virtual-network/virtual-network-peering-overview.md) a su servidor flexible
   * Uso de VPN o ExpressRoute para conectarse desde recursos que no son de Azure al servidor flexible
   * Ningún punto de conexión público

* **Acceso público (direcciones IP permitidas)** : se accede al servidor flexible a través de un punto de conexión público. El punto de conexión público es una dirección DNS que se puede resolver públicamente. La frase "direcciones IP permitidas" hace referencia a un intervalo de direcciones IP a las que decida conceder permiso para acceder al servidor. Estos permisos se denominan **reglas de firewall**. 

   Elija el método de acceso público si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure que no admiten redes virtuales
   * Conexión desde recursos externos de Azure que no están conectados mediante VPN o ExpressRoute 
   * El servidor flexible tiene un punto de conexión público

Las características siguientes se aplican tanto si decide usar la opción de acceso privado como la de acceso público:
* Las conexiones de direcciones IP permitidas se deben autenticar en el servidor MySQL con credenciales válidas
* Hay [cifrado de conexión](#tls-and-ssl) disponible para el tráfico de red
* El servidor tiene un nombre de dominio completo (FQDN). En el caso de la propiedad hostname en las cadenas de conexión, se recomienda usar el FQDN en lugar de una dirección IP.
* Las dos opciones controlan el acceso en el nivel de servidor, no en el nivel de base de datos o de tabla. Tendría que usar las propiedades de los roles de MySQL para controlar el acceso de bases de datos, tablas y otros objetos.


## <a name="private-access-vnet-integration"></a>Acceso privado (integración con red virtual)
El acceso privado con la integración de red virtual (vnet) proporciona una comunicación segura y privada para el servidor flexible de MySQL.

### <a name="virtual-network-concepts"></a>Conceptos de redes virtuales
Estos son algunos conceptos que debe conocer al usar redes virtuales con servidores flexibles de MySQL.

* **Red virtual**: una instancia de Azure Virtual Network (VNet) contiene un espacio de direcciones IP privadas que se configura para su uso. Para obtener más información sobre las redes virtuales de Azure, visite la [Información general sobre Azure Virtual Network](../../virtual-network/virtual-networks-overview.md).

    La red virtual debe estar en la misma región de Azure que el servidor flexible.

* **Subred delegada**: una red virtual contiene subredes. Las subredes permiten segmentar la red virtual en espacios de direcciones más pequeños. Los recursos de Azure se implementan en subredes específicas dentro de una red virtual. 

   El servidor flexible de MySQL debe estar en una subred **delegada** para uso exclusivo del servidor flexible de MySQL. Esta delegación significa que solo los servidores flexibles de Azure Database for MySQL pueden utilizar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada. Puede delegar una subred si asigna su propiedad de delegación como Microsoft.DBforMySQL/flexibleServers.

* **Grupos de seguridad de red (NSG)** Las reglas de seguridad de grupos de seguridad de red permiten filtrar el tipo de tráfico de red que puede fluir dentro y fuera de las interfaces de red y las subredes de redes virtuales. Revise la [Introducción a los grupos de seguridad de red](../../virtual-network/network-security-groups-overview.md) para obtener más información.

* **Emparejamiento de red virtual:** el emparejamiento de red virtual permite conectar sin problemas dos o más redes virtuales en Azure. A efectos de conectividad las redes virtuales emparejadas aparecen como una sola. El tráfico entre las máquinas virtuales de la red virtual emparejada usa la infraestructura de la red troncal de Microsoft. El tráfico entre la aplicación cliente y el servidor flexible en redes virtuales emparejadas se enruta a través de la red privada de Microsoft únicamente y está aislado a esa red únicamente.

El servidor flexible admite el emparejamiento de redes virtuales dentro de la misma región de Azure. **No se admite** el emparejamiento de redes virtuales entre regiones. Revise los [conceptos de emparejamiento de redes virtuales](../../virtual-network/virtual-network-peering-overview.md) para obtener más información.

### <a name="connecting-from-peered-vnets-in-same-azure-region"></a>Conexión desde redes virtuales emparejadas en la misma región de Azure
Si la aplicación cliente que intenta conectarse al servidor flexible está en la red virtual emparejada, es posible que no pueda conectarse mediante el nombre de servidor del servidor flexible, ya que no puede resolver el nombre DNS para el servidor flexible de la red virtual emparejada. Hay dos opciones de autoservicio para resolver esto:
* Usar dirección IP privada (recomendado para el escenario de desarrollo/pruebas): esta opción se puede usar con fines de desarrollo o prueba. Puede usar nslookup para deshacer la búsqueda inversa de la dirección IP privada para el nombre de servidor flexible (nombre de dominio completo) y usar la dirección IP privada para conectarse desde la aplicación cliente. No se recomienda usar la dirección IP privada para la conexión con el servidor flexible para su uso en producción, ya que puede cambiar durante un evento planeado o no planeado.
* Usar zona DNS privada (recomendado para producción): esta opción es adecuada para fines de producción. Debe aprovisionar una [zona DNS privada](../../dns/private-dns-getstarted-portal.md) y vincularla a la red virtual del cliente. En la zona DNS privada, se agrega un [registro A](../../dns/dns-zones-records.md#record-types) para el servidor flexible mediante su dirección IP privada. Después, puede usar el registro A para conectarse desde la aplicación cliente en una red virtual emparejada a un servidor flexible.

### <a name="connecting-from-on-premises-to-flexible-server-in-virtual-network-using-expressroute-or-vpn"></a>Conexión de un servidor local a un servidor flexible en Virtual Network mediante ExpressRoute o VPN
En el caso de cargas de trabajo que requieran acceso a un servidor flexible en la red virtual desde la red local, necesitará [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute/) o [VPN](/azure/architecture/reference-architectures/hybrid-networking/vpn/) y la red virtual [conectados al entorno local](/azure/architecture/reference-architectures/hybrid-networking/). Con esta configuración en su lugar, necesitará un reenviador DNS para resolver el nombre de servidor flexible si desea conectarse desde la aplicación cliente (como MySQL Workbench) que se ejecuta en una red virtual local. Este reenviador DNS es responsable de resolver todas las consultas de DNS a través de un reenviador de nivel de servidor en el servicio DNS proporcionado por Azure [168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md).

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red local
- Servidor flexible de MySQL aprovisionado con acceso privado (integración con red virtual)
- Red virtual [conectada al entorno local](/azure/architecture/reference-architectures/hybrid-networking/)
- Uso del reenviador de DNS [168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md) implementado en Azure

Después, puede usar el nombre de servidor flexible (FQDN) para conectarse desde la aplicación cliente en una red virtual emparejada o en una red local a un servidor flexible.

### <a name="unsupported-virtual-network-scenarios"></a>Escenarios de red virtual no admitidos
* Punto de conexión público (o dirección IP pública o DNS): un servidor flexible implementado en una red virtual no puede tener un punto de conexión público
* Una vez que se haya implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a otra red virtual o subred. No se puede trasladar la red virtual a otro grupo de recursos o suscripción.
* No se puede aumentar el tamaño de la subred (espacios de direcciones) cuando existen recursos en la subred
* No se admite el emparejamiento de redes virtuales entre regiones

Obtenga información sobre cómo habilitar el acceso privado (integración con red virtual) con [Azure Portal](how-to-manage-virtual-network-portal.md) o la [CLI de Azure](how-to-manage-virtual-network-cli.md).

> [!NOTE]
> Si usa el servidor DNS personalizado, debe usar un reenviador DNS para resolver el FQDN de Azure Database for MySQL: servidor flexible. Para obtener más información, consulte [Resolución de nombres con su propio servidor DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

## <a name="public-access-allowed-ip-addresses"></a>Acceso público (direcciones IP permitidas)
Entre las características del método de acceso público se incluyen las siguientes:
* Solo las direcciones IP que permita pueden acceder al servidor flexible de MySQL. De forma predeterminada, no se permite ninguna dirección IP. Puede agregar direcciones IP durante la creación del servidor o después.
* El servidor MySQL tiene un nombre DNS que se pueda resolver de forma pública
* El servidor flexible no está en una de las redes virtuales de Azure
* El tráfico de red hacia y desde el servidor no se realiza a través de una red privada. El tráfico usa las rutas de Internet generales.

### <a name="firewall-rules"></a>Reglas de firewall
La concesión de permisos a una dirección IP se denomina regla de firewall. Si un intento de conexión procede de una dirección IP no permitida, el cliente de origen verá un error.


### <a name="allowing-all-azure-ip-addresses"></a>Permitir todas las direcciones IP de Azure
Si no hay ninguna dirección IP saliente fija disponible para su servicio de Azure, puede habilitar las conexiones de todas las direcciones IP del centro de datos de Azure.

> [!IMPORTANT]
> La opción **Permitir acceso público desde los recursos y servicios de Azure dentro de Azure** configura el firewall para permitir todas las conexiones desde Azure, incluidas las que provienen de suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.

Obtenga información sobre cómo habilitar y administrar el acceso público (direcciones IP permitidas) mediante [Azure Portal](how-to-manage-firewall-portal.md) o la [CLI de Azure](how-to-manage-firewall-cli.md).


### <a name="troubleshooting-public-access-issues"></a>Solución de problemas de acceso público
Tenga en cuenta los aspectos siguientes cuando el acceso al servicio del servidor de Microsoft Azure Database for MySQL no se comporte de la manera prevista:

* **Los cambios realizados en la lista de permitidos no han surtido efecto todavía:** puede haber un retraso de hasta cinco minutos hasta que los cambios en la configuración del firewall del servidor de Azure Database for MySQL surtan efecto.

* **Error de autenticación:** si un usuario no tiene los permisos en el servidor de Azure Database for MySQL o la contraseña usada es incorrecta, se denegará la conexión al servidor de Azure Database for MySQL. La creación de una configuración de firewall solo ofrece a los clientes una oportunidad de intentar conectarse al servidor. Cada cliente todavía tiene que proporcionar las credenciales de seguridad necesarias.

* **Dirección IP del cliente dinámica:** Si tiene una conexión a Internet con un direccionamiento IP dinámico y tiene problemas al acceder al firewall, pruebe una de las soluciones siguientes:

   * Pida al proveedor de acceso a Internet (ISP) el intervalo de direcciones IP asignado a los equipos cliente que acceden al servidor de Azure Database for MySQL y agréguelo como regla de firewall.
   * Obtenga el direccionamiento IP estático en su lugar para los equipos cliente y luego agregue la dirección IP estática como regla de firewall.
  
* **La regla de firewall no está disponible en formato IPv6:** las reglas de firewall deben estar en formato IPv4. Si especifica reglas de firewall en formato IPv6, aparece el error de validación.


## <a name="hostname"></a>Nombre de host
Con independencia de la opción de red que elija, se recomienda usar siempre un nombre de dominio completo (FQDN) como nombre de host al conectarse al servidor flexible. No se garantiza que la dirección IP del servidor permanezca estática. El uso del FQDN le ayudará a evitar realizar cambios en la cadena de conexión. 

Ejemplo
* Recomendado `hostname = servername.mysql.database.azure.com`
* Siempre que sea posible, evite usar `hostname = 10.0.0.4` (una dirección privada) o `hostname = 40.2.45.67` (una dirección IP pública)


## <a name="tls-and-ssl"></a>TLS y SSL
El servidor flexible de Azure Database for MySQL admite la conexión de las aplicaciones cliente al servidor MySQL mediante el cifrado de la Capa de sockets seguros (SSL) con la Seguridad de la capa de transporte (TLS). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente, lo que le permite ajustarse a los requisitos de cumplimiento.

De manera predeterminada, el servidor flexible de Azure Database for MySQL solo admite conexiones cifradas mediante la Seguridad de la capa de transporte (TLS 1.2), y todas las conexiones entrantes con TLS 1.0 y TLS 1.1 se denegarán de manera predeterminada. La obligatoriedad de conexiones cifradas o la configuración de versiones de TLS en el servidor flexible se pueden configurar y cambiar. 

A continuación, se muestran las distintas configuraciones de SSL y TLS que puede tener para el servidor flexible:

| Escenario   | Configuración de parámetros del servidor      | Descripción                                    |
|------------|--------------------------------|------------------------------------------------|
|Deshabilitar SSL (conexiones cifradas) | require_secure_transport = OFF |Si la aplicación heredada no admite conexiones cifradas al servidor MySQL, puede deshabilitar la obligatoriedad de conexiones cifradas en el servidor flexible al establecer require_secure_transport=OFF.|
|Exigir SSL con la versión de TLS < 1.2 | require_secure_transport = ON y tls_version = TLSV1 o TLSV1.1| Si la aplicación heredada admite conexiones cifradas, pero requiere una versión de TLS anterior a la 1.2, puede habilitar las conexiones cifradas, pero debe configurar el servidor flexible para permitir conexiones con la versión de TLS (v1.0 o v1.1) que admite la aplicación.|
|Exigir SSL con la versión de TLS = 1.2 (configuración predeterminada)|require_secure_transport = ON y tls_version = TLSV1.2| Esta es la configuración predeterminada y recomendada para el servidor flexible.|
|Exigir SSL con la versión de TLS = 1.3 (compatible con MySQL v8.0 y versiones posteriores)| require_secure_transport = ON y tls_version = TLSV1.3| Esta configuración es útil y recomendada para el desarrollo de nuevas aplicaciones.|

> [!Note]
> No se admiten cambios en el cifrado SSL del servidor flexible. Los conjuntos de cifrado FIPS se aplican de manera predeterminada cuando tls_version está establecido en la versión de TLS 1.2. En el caso de las versiones de TLS distintas de la versión 1.2, el cifrado SSL se establece en la configuración predeterminada que se incluye con la instalación de la comunidad de MySQL.

Consulte cómo [conectarse mediante SSL/TLS](how-to-connect-tls-ssl.md) para obtener más información. 


## <a name="next-steps"></a>Pasos siguientes
* Información sobre cómo habilitar el acceso privado (integración con red virtual) con [Azure Portal](how-to-manage-virtual-network-portal.md) o la [CLI de Azure](how-to-manage-virtual-network-cli.md)
* Información sobre cómo habilitar el acceso público (direcciones IP permitidas) mediante [Azure Portal](how-to-manage-firewall-portal.md) o la [CLI de Azure](how-to-manage-firewall-cli.md)
* Información sobre cómo [usar TLS](how-to-connect-tls-ssl.md)
