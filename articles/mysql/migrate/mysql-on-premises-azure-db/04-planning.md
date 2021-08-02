---
title: Planeamiento de la guía de migración de una instancia local de MySQL a Azure Database for MySQL
description: Una zona de aterrizaje de Azure es el entorno de destino definido como el lugar de reposo definitivo para un proyecto de migración a la nube.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: af4b64a9621f2327287a88fc9f3d00e485c424d5
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082967"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-planning"></a>Planeamiento de la guía de migración de una instancia local de MySQL a Azure Database for MySQL

## <a name="prerequisites"></a>Prerrequisitos

[Valoración](03-assessment.md)

## <a name="landing-zone"></a>Zona de aterrizaje

Una [zona de aterrizaje de Azure](/azure/cloud-adoption-framework/ready/landing-zone/) es el entorno de destino definido como el lugar de reposo definitivo para un proyecto de migración a la nube. En la mayoría de los proyectos, la zona de aterrizaje debería programarse mediante plantillas de ARM para la configuración inicial. Finalmente, debería personalizarse con PowerShell o Azure Portal para adaptarla a las necesidades de las cargas de trabajo.

Puesto que WWI se encuentra en San Francisco, todos los recursos para la zona de aterrizaje de Azure se crearon en la región `US West 2`. Se crearon los siguientes recursos para sustentar la migración:

- [Azure Database for MySQL](../../quickstart-create-mysql-server-database-using-azure-portal.md)

- [Azure Database Migration Service (DMS)](../../../dms/quickstart-create-data-migration-service-portal.md)

- [ExpressRoute](../../../expressroute/expressroute-introduction.md)

- [Azure Virtual Network](../../../virtual-network/quick-create-portal.md) con un [diseño en estrella tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) con los [emparejamientos de red virtual](../../../virtual-network/virtual-network-peering-overview.md) correspondientes

- [App Service](../../../app-service/overview.md)

- [Application Gateway](../../../load-balancer/quickstart-load-balancer-standard-internal-portal.md?tabs=option-1-create-internal-load-balancer-standard)

- [Puntos de conexión privados](../../../private-link/private-endpoint-overview.md) para App Services y la instancia de MySQL

> [!NOTE]
> Con esta guía, se proporcionaron dos plantillas de ARM (una con puntos de conexión privados, y otra, sin estos puntos de conexión) para implementar una posible zona de aterrizaje de Azure para un proyecto de migración de MySQL. La plantilla de ARM con puntos de conexión privados proporciona un escenario más seguro y apropiado para producción. En función de los requisitos, puede ser necesario configurar manualmente algunos valores más de la zona de aterrizaje de Azure.

## <a name="networking"></a>Redes

La obtención de datos del sistema de origen para Azure Database for MySQL de una manera rápida y óptima es un aspecto fundamental que debe tenerse en cuenta en un proyecto de migración. Las conexiones pequeñas no confiables pueden hacer que los administradores tengan que reiniciar la migración varias veces hasta conseguir un resultado satisfactorio. El reinicio de las migraciones por problemas de red puede suponer un esfuerzo inútil.

Dedique tiempo a conocer y evaluar la conectividad de red entre los entornos de origen, de la herramienta de migración y de destino. En algunos casos, puede ser adecuado actualizar la conectividad a Internet o configurar una conexión de ExpressRoute del entorno local a Azure. Una vez creada la conectividad entre el entorno local y Azure, el siguiente paso consiste en validar que la herramienta de migración seleccionada puede conectarse desde el origen al destino.

La ubicación de la herramienta de migración determina los requisitos de conectividad de red. Como se muestra en la tabla siguiente, la herramienta de migración seleccionada debe poder conectarse a la máquina del entorno local y a Azure. Azure debe configurarse para que solo acepte tráfico de la ubicación de la herramienta de migración.

| Herramienta de migración | Tipo | Location | Requisitos de red de entrada | Requisitos de red de salida |
|----------------|------|----------|------------------------------|-------------------------------|
| **Database Migration Service (DMS)** | Sin conexión | Azure | Permitir 3306 desde IP externa | Ruta de acceso para conectarse a la instancia de la base de datos de MySQL en Azure |
| **Import/Export (MySQL Workbench, mysqldump)** | Sin conexión | Local | Permitir 3306 desde IP interna | Ruta de acceso para conectarse a la instancia de la base de datos de MySQL en Azure |
| **Import/Export (MySQL Workbench, mysqldump)** | Sin conexión | Azure VM | Permitir 3306 desde IP externa | Ruta de acceso para conectarse a la instancia de la base de datos de MySQL en Azure |
| **mydumper/myloader** | Sin conexión | Local | Permitir 3306 desde IP interna | Ruta de acceso para conectarse a la instancia de la base de datos de MySQL en Azure |
| **mydumper/myloader** | Sin conexión | Azure VM | Permitir 3306 desde IP externa | Ruta de acceso para conectarse a la instancia de la base de datos de MySQL en Azure |
| **binlog** | Sin conexión | Local | Permitir 3306 desde IP externa o IP privada a través de puntos de conexión privados | Ruta de acceso para cada servidor de replicación al servidor maestro |

Los siguientes son otros aspectos sobre las redes que deben tenerse en cuenta:

- Al servicio DMS ubicado en una red virtual se le asigna una [IP pública dinámica](../../../dms/faq.md#setup). Cuando crea el servicio, puede colocarlo en una red virtual que tenga conectividad a través de [ExpressRoute](../../../expressroute/expressroute-introduction.md) o de una conexión [VPN de sitio a sitio](../../../vpn-gateway/tutorial-site-to-site-portal.md).

- Cuando use una máquina virtual de Azure para ejecutar las herramientas de migración, asígnele una dirección IP pública y permita que se conecte únicamente a la instancia local de MySQL.

- Los firewalls de salida deben permitir la conectividad de salida a Azure Database for MySQL. Las direcciones IP de puerta de enlace de MySQL están disponibles en la página [Arquitectura de conectividad de Azure Database for MySQL](../../concepts-connectivity-architecture.md#azure-database-for-mysql-gateway-ip-addresses).

## <a name="ssltls-connectivity"></a>Conectividad SSL/TLS

Además de las implicaciones de migrar a una comunicación basada en SSL, también deben tenerse en cuenta los tipos de conexión SSL/TLS. Después de crear la base de datos de Azure Database for MySQL, revise la configuración de SSL y lea el artículo [Conectividad SSL/TLS en Azure Database for MySQL](../../concepts-ssl-connection-security.md) para comprender cómo la configuración de TLS puede afectar a la posición de seguridad.

> [!Important]
> Preste atención a la declinación de responsabilidades de la página. El cumplimiento de la versión de TLS no está habilitado de manera predeterminada. Una vez que se ha habilitado TLS, la única manera de deshabilitarlo es volver a habilitar SSL.

## <a name="wwi-scenario"></a>Escenario de WWI

El equipo de la nube de WWI ha creado los recursos necesarios para la zona de aterrizaje de Azure en un grupo de recursos específico para Azure Database for MySQL. Para crear la zona de aterrizaje, WWI decidió programar la instalación y la implementación mediante plantillas de ARM. El uso de plantillas de ARM les permite quitar y volver a configurar el entorno rápidamente si fuera necesario.

En una plantilla de ARM, todas las conexiones entre redes virtuales se configuran con emparejamiento en una arquitectura en estrella tipo hub-and-spoke. La base de datos y la aplicación se colocan en redes virtuales aparte. Delante del servicio de aplicación, se pone una instancia de Azure App Gateway para aislarlo de Internet. Azure App Service se conecta a Azure Database for MySQL a través de un punto de conexión privado.

Al principio, WWI quiso probar una migración en línea, pero la configuración de red necesaria para que el servicio DMS se conectara a su entorno local lo hacía inviable. WWI optó entonces por llevar a cabo una migración sin conexión. Se usó la herramienta MySQL Workbench para exportar los datos del entorno local y, después, para importar los datos en la instancia de Azure Database for MySQL.

## <a name="planning-checklist"></a>Lista de comprobación de planeación

- Preparar la zona de aterrizaje de Azure. Considerar la posibilidad de usar plantillas de ARM para la implementación en el caso de que sea necesario quitar y reconstruir el entorno rápidamente.

- Comprobar la configuración de la red. Esta comprobación debe incluir: conectividad, ancho de banda, latencia y configuraciones de firewall.

- Determinar si se va a usar una estrategia de migración de datos en línea o sin conexión.

- Decidir la estrategia de certificado SSL.  


> [!div class="nextstepaction"]
> [Métodos de migración](./05-migration-methods.md)
