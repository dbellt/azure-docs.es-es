---
title: 'Seguridad: Azure Database for MariaDB'
description: Información general sobre las características de seguridad de Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 02202255f82af33190e35060196a09e1f5457492
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955950"
---
# <a name="security-in-azure-database-for-mariadb"></a>Seguridad en Azure Database for MariaDB

Existen varios niveles de seguridad disponibles para proteger los datos en el servidor de Azure Database for MariaDB. En este artículo se describen esas opciones de seguridad.

## <a name="information-protection-and-encryption"></a>Protección y cifrado de información

### <a name="in-transit"></a>En tránsito
Azure Database for MariaDB protege los datos mediante el cifrado de datos en tránsito mediante la Seguridad de la capa de transporte. El cifrado (SSL/TLS) se aplica de forma predeterminada.

### <a name="at-rest"></a>En reposo
El servicio Azure Database for MariaDB usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Los datos, incluidas las copias de seguridad, se cifran en el disco, a excepción de los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de almacenamiento de Azure y las claves las administra el sistema. El cifrado de almacenamiento siempre está activado y no se puede deshabilitar.


## <a name="network-security"></a>Seguridad de las redes
Las conexiones a un servidor de Azure Database for MariaDB se enrutan por primera vez a través de una puerta de enlace regional. La puerta de enlace tiene una dirección IP accesible públicamente, mientras que las direcciones IP del servidor están protegidas. Para obtener más información sobre la puerta de enlace, consulte el [artículo referente a la arquitectura de conectividad](concepts-connectivity-architecture.md).  

Un servidor de Azure Database for MariaDB recién creado tiene un firewall que bloquea todas las conexiones externas. Aunque lleguen a la puerta de enlace, no tienen permiso para conectarse al servidor. 

### <a name="ip-firewall-rules"></a>Reglas de firewall de IP
Las reglas de firewall de IP otorgan acceso a los servidores según la dirección IP de origen de cada solicitud. Consulte la [información general sobre las reglas de firewall](concepts-firewall-rules.md) para obtener más información.

### <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual
Los puntos de conexión de servicio de red virtual amplían la conectividad de la red virtual a través de la red troncal de Azure. Cuando se usan reglas de red virtual, el servidor de Azure Database for MariaDB se puede habilitar para permitir conexiones desde subredes seleccionadas en una red virtual. Para obtener más información, consulte la [información general sobre los puntos de conexión de servicio de red virtual](concepts-data-access-security-vnet.md).


## <a name="access-management"></a>Administración de acceso

Al crear el servidor de Azure Database for MariaDB, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de MariaDB.


## <a name="threat-protection"></a>Protección contra amenazas

Puede optar por usar [Advanced Threat Protection](../security-center/defender-for-databases-introduction.md), que detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a sus servidores o de aprovechar sus vulnerabilidades.

Existe un [registro de auditoría](concepts-audit-logs.md) disponible para realizar un seguimiento de las actividades en las bases de datos. 


## <a name="next-steps"></a>Pasos siguientes
- Habilite las reglas de firewall de [direcciones IP](concepts-firewall-rules.md) o de [redes virtuales](concepts-data-access-security-vnet.md).