---
title: Seguridad en Azure Database for PostgreSQL con un único servidor
description: Información general sobre las características de seguridad de Azure Database for PostgreSQL con un único servidor.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 11/22/2019
ms.openlocfilehash: 261f198542a7030f327dc8be055f5db0cb64f957
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952443"
---
# <a name="security-in-azure-database-for-postgresql---single-server"></a>Seguridad en Azure Database for PostgreSQL con un único servidor

Existen varios niveles de seguridad disponibles para proteger los datos en el servidor de Azure Database for PostgreSQL. En este artículo se describen esas opciones de seguridad.

## <a name="information-protection-and-encryption"></a>Protección y cifrado de información

### <a name="in-transit"></a>En tránsito
Azure Database for PostgreSQL protege los datos mediante el cifrado de datos en tránsito con Seguridad de la capa de transporte. El cifrado (SSL/TLS) se aplica de forma predeterminada.

### <a name="at-rest"></a>En reposo
El servicio Azure Database for PostgreSQL usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Los datos, incluidas las copias de seguridad, se cifran en el disco, junto con los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de almacenamiento de Azure y las claves las administra el sistema. El cifrado de almacenamiento siempre está activado y no se puede deshabilitar.


## <a name="network-security"></a>Seguridad de las redes
Las conexiones a un servidor de Azure Database for PostgreSQL se enrutan primero a través de una puerta de enlace regional. La puerta de enlace tiene una dirección IP accesible públicamente, mientras que las direcciones IP del servidor están protegidas. Para obtener más información sobre la puerta de enlace, consulte el [artículo referente a la arquitectura de conectividad](concepts-connectivity-architecture.md).  

Un servidor de Azure Database for PostgreSQL recién creado tiene un firewall que bloquea todas las conexiones externas. Aunque lleguen a la puerta de enlace, no tienen permiso para conectarse al servidor. 

### <a name="ip-firewall-rules"></a>Reglas de firewall de IP
Las reglas de firewall de IP otorgan acceso a los servidores según la dirección IP de origen de cada solicitud. Consulte la [información general sobre las reglas de firewall](concepts-firewall-rules.md) para obtener más información.

### <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual
Los puntos de conexión de servicio de red virtual amplían la conectividad de la red virtual a través de la red troncal de Azure. Cuando se usan reglas de red virtual, el servidor de Azure Database for PostgreSQL se puede habilitar para permitir conexiones desde subredes seleccionadas en una red virtual. Para obtener más información, consulte la [información general sobre los puntos de conexión de servicio de red virtual](concepts-data-access-and-security-vnet.md).

### <a name="private-ip"></a>Dirección IP privada
Private Link le permite conectarse a su servidor único de Azure Database for PostgreSQL en Azure a través de un punto de conexión privado. En esencia, Azure Private Link incorpora los servicios de Azure dentro de su red virtual privada (VNet). Se puede acceder a los recursos de PaaS mediante la dirección IP privada, al igual que cualquier otro recurso de la red virtual. Para más información, vea la [información general de Private Link](concepts-data-access-and-security-private-link.md).


## <a name="access-management"></a>Administración de acceso

Al crear el servidor de Azure Database for PostgreSQL, se deben proporcionar las credenciales de un rol de usuario administrador. Este rol de administrador se puede usar para crear más [roles de MySQL](https://www.postgresql.org/docs/current/user-manag.html).

También se puede conectar al servidor a través de la [autenticación de Azure Active Directory (AAD)](concepts-aad-authentication.md).


## <a name="threat-protection"></a>Protección contra amenazas

Puede optar por usar [Advanced Threat Protection](../security-center/defender-for-databases-introduction.md), que detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a sus servidores o de aprovechar sus vulnerabilidades.

Existe un [registro de auditoría](concepts-audit.md) disponible para realizar un seguimiento de las actividades en las bases de datos. 

## <a name="migrating-from-oracle"></a>Migración desde Oracle

Oracle admite Cifrado de datos transparente (TDE) para cifrar los datos de tabla y espacio de tabla. En Azure para PostgreSQL, los datos se cifran automáticamente en diferentes capas. Consulte la sección "En reposo" de esta página y haga también referencia a diversos temas de seguridad, incluidos [claves administradas por el cliente](./concepts-data-encryption-postgresql.md) y [cifrado doble de infraestructura](./concepts-infrastructure-double-encryption.md). También puede considerar la posibilidad de usar la extensión [pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html) que se admite en [Azure para PostgreSQL](./concepts-extensions.md).

## <a name="next-steps"></a>Pasos siguientes
- Habilite las reglas de firewall de [direcciones IP](concepts-firewall-rules.md) o de [redes virtuales](concepts-data-access-and-security-vnet.md).
- Obtenga más información sobre la [autenticación de Azure Active Directory](concepts-aad-authentication.md) en Azure Database for PostgreSQL.