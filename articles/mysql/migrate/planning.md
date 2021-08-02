---
title: Planeamiento de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: una zona de aterrizaje de Azure es el entorno de destino definido como el lugar de reposo final de un proyecto de migración a la nube.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 05/25/2021
ms.openlocfilehash: 1cd9d78fab41305fc6cf4fc814a39a999502c795
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958593"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-planning"></a>Planeamiento de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

### <a name="landing-zone"></a>Zona de aterrizaje

Una [zona de aterrizaje de Azure](/azure/cloud-adoption-framework/ready/landing-zone/) es el entorno de destino definido como el lugar de reposo final de un proyecto de migración a la nube. En la mayoría de proyectos, la zona de aterrizaje debe programarse a través de plantillas de ARM para su configuración inicial. Por último, debe personalizarse con PowerShell o Azure Portal para ajustarse a las necesidades de las cargas de trabajo.

Puesto que la sede de WWI está en San Francisco, todos los recursos para la zona de aterrizaje de Azure se crearon en la región `US West 2`. Se crearon los siguientes recursos para dar asistencia a la migración:

  - [Azure Database for MySQL](../quickstart-create-mysql-server-database-using-azure-portal.md)

  - [Azure Database Migration Service (DMS) ](../../dms/quickstart-create-data-migration-service-portal.md)

  - [Express Route ](../../expressroute/expressroute-introduction.md)

  - [Azure Virtual Network](../../virtual-network/quick-create-portal.md) con [diseño de centro de conectividad y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) con los correspondientes [emparejamientos de red virtual ](../../virtual-network/virtual-network-peering-overview.md) establecidos.

  - [App Service](../../app-service/overview.md)

  - [Application Gateway](../../load-balancer/quickstart-load-balancer-standard-internal-portal.md?tabs=option-1-create-internal-load-balancer-standard)

  - [Puntos de conexión privados](../../private-link/private-endpoint-overview.md) para las instancias de App Services y MySQL

> [!NOTE]
> Como parte de esta guía, se proporcionaron dos plantillas de ARM (una con puntos de conexión privados, otra sin) para implementar una posible zona de aterrizaje de Azure para un proyecto de migración de MySQL. La plantilla de ARM de puntos de conexión privados proporciona un escenario más seguro y de producción. Puede ser necesaria una configuración manual adicional de la zona de aterrizaje de Azure, en función de los requisitos.

### <a name="networking"></a>Redes

La obtención de datos del sistema de origen para Azure Database for MySQL de una manera rápida y óptima es un componente fundamental que se debe tener en cuenta en un proyecto de migración. Las conexiones pequeñas no fiables pueden requerir que los administradores reinicien la migración varias veces hasta que se consiga un resultado satisfactorio. Reiniciar las migraciones debido a problemas de red puede llevar a realizar esfuerzos inútiles.

Dedique tiempo a comprender y evaluar la conectividad de red entre los entornos de origen, herramientas y destino. En algunos casos, puede ser adecuado actualizar la conectividad a Internet o configurar una conexión ExpressRoute desde el entorno local a Azure. Una vez creada la conectividad local a Azure, el paso siguiente consiste en validar que la herramienta de migración seleccionada puede conectarse desde el origen al destino.

La ubicación de la herramienta de migración determina los requisitos de conectividad de red. Como se muestra en la tabla siguiente, la herramienta de migración seleccionada debe poder conectarse a la máquina local y a Azure. Azure debe configurarse para que solo acepte tráfico de la ubicación de la herramienta de migración.

| Herramienta de migración                             | Tipo              | Location        | Requisitos de red de entrada                                    | Requisitos de red de salida                          |
|--------------------------------------------|-------------------|-----------------|-----------------------------------------------------------------|--------------------------------------------------------|
| **Database Migration Service (DMS)**           | Sin conexión           | Azure           | Permitir 3306 desde IP externa                                     | Ruta de acceso para conectarse a la instancia de base de datos de MySQL de Azure |
| **Import/Export (MySQL Workbench, mysqldump)** | Sin conexión           | Local     | Permitir 3306 desde IP interna                                     | Ruta de acceso para conectarse a la instancia de base de datos de MySQL de Azure |
| **Import/Export (MySQL Workbench, mysqldump)** | Sin conexión           | Azure VM        | Permitir 3306 desde IP externa                                     | Ruta de acceso para conectarse a la instancia de base de datos de MySQL de Azure |
| **mydumper/myloader**                          | Sin conexión           | Local     | Permitir 3306 desde IP interna                                     | Ruta de acceso para conectarse a la instancia de base de datos de MySQL de Azure |
| **mydumper/myloader**                          | Sin conexión           | Azure VM        | Permitir 3306 desde IP externa                                     | Ruta de acceso para conectarse a la instancia de base de datos de MySQL de Azure |
| **binlog**                                     | Sin conexión           | Local     | Permitir 3306 desde IP externa o IP privada a través de puntos de conexión privados | Ruta de acceso de cada servidor de replicación al servidor maestro       |

Existen otras consideraciones de redes:

  - A DMS ubicado en una red virtual se le asigna una [IP pública dinámica](../../dms/faq.md#setup) al servicio. En el momento de la creación, puede colocar el servicio dentro de una red virtual que tenga conectividad a través de [ExpressRoute](../../expressroute/expressroute-introduction.md) o a través de una [VPN de sitio a sitio. ](../../vpn-gateway/tutorial-site-to-site-portal.md)

  - Cuando use una máquina virtual de Azure para ejecutar las herramientas de migración, asígnele una IP pública y, a continuación, solo permita que se conecte a la instancia de MySQL local.

  - Los firewalls de salida deben garantizar la conectividad de salida a Azure Database for MySQL. Las direcciones IP de puerta de enlace de MySQL están disponibles en la página [Arquitectura de conectividad de Azure Database for MySQL](../concepts-connectivity-architecture.md#azure-database-for-mysql-gateway-ip-addresses).

### <a name="ssltls-connectivity"></a>Conectividad SSL/TLS

Además de las implicaciones de la aplicación de migrar a la comunicación basada en SSL, los tipos de conexión SSL/TLS también deben tenerse en cuenta. Después de crear la base de datos Azure Database for MySQL, revise la configuración de SSL y lea el artículo [Conectividad SSL/TLS en Azure Database for MySQL](../concepts-ssl-connection-security.md) para comprender cómo la configuración de TLS puede afectar a la posición de seguridad.

> [!Important]
> Preste atención a la declinación de responsabilidades de la página. El cumplimiento de la versión de TLS no está habilitado de manera predeterminada. Una vez TLS se ha habilitado, la única manera de deshabilitarlo es volver a habilitar SSL.

### <a name="wwi-scenario"></a>Escenario de WWI

El equipo en la nube de WWI ha creado los recursos necesarios de la zona de aterrizaje de Azure en un grupo de recursos específico para la Azure Database for MySQL. Para crear la zona de aterrizaje, WWI decidió programar la instalación y la implementación mediante plantillas de ARM. Mediante el uso de plantillas de ARM, podrían demoler y volver a configurar rápidamente el entorno, si fuera necesario.

Como parte de la plantilla de ARM, todas las conexiones entre redes virtuales se configuran con emparejamiento en una arquitectura de centro de conectividad y radio. La base de datos y la aplicación se colocan en redes virtuales independientes. Se coloca una Azure Application Gateway delante del servicio de aplicaciones para permitir que se aísle de Internet. El Azure App Service se conecta a la Azure Database for MySQL mediante un punto de conexión privado.

Originalmente, WWI quería probar una migración en línea, pero la configuración de red necesaria para que DMS se conectara a su entorno local lo hacía inviable. WWI eligió realizar una migración sin conexión. La herramienta MySQL Workbench se usó para exportar los datos locales y, a continuación, se usó para importar los datos a la instancia de Azure Database for MySQL.

### <a name="planning-checklist"></a>Lista de comprobación en el diseño de una aplicación

  - Prepare la zona de aterrizaje de Azure. Considere la posibilidad de usar la implementación de plantillas de ARM en caso de que el entorno se deba demoler y reconstruir rápidamente.

  - Verifique la configuración de la red. La comprobación debe incluir: conectividad, ancho de banda, latencia y configuraciones de firewall.

  - Determine si va a usar la estrategia de migración de datos en línea o sin conexión.

  - Decida qué estrategia de certificado SSL quiere seguir.  


> [!div class="nextstepaction"]
> [Métodos de migración](./migration-methods.md)