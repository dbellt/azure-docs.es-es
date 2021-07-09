---
title: Notas de la versión del servidor flexible de Azure Database for PostgreSQL
description: Notas de la versión del servidor flexible de Azure Database for PostgreSQL
author: sr-msft
ms.author: srranga
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 05/25/2021
ms.openlocfilehash: 4ed84ab94173961412a7769da89136c39413a92e
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983085"
---
# <a name="release-notes---azure-database-for-postgresql---flexible-server"></a>Notas de la versión del servidor flexible de Azure Database for PostgreSQL

En esta página se proporcionan las últimas noticias y actualizaciones relacionadas con las adiciones de características, la compatibilidad con las versiones del motor, las extensiones y cualquier otro anuncio importante para la opción de servidor flexible de Azure Database for PostgreSQL.

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

## <a name="release-may-25-2021"></a>Versión: 25 de mayo de 2021

* Compatibilidad con la [versión principal 13 de PostgreSQL](./concepts-supported-versions.md).
* Compatibilidad con extensiones, como pg_partman, pg_cron y pgaudit. Consulte la [página de extensiones](./concepts-extensions.md) para ver las versiones compatibles con cada versión principal.
* Varias correcciones de errores y mejoras en la estabilidad y el rendimiento.

## <a name="release-april-26-2021"></a>Versión: 26 de abril de 2021

* Compatibilidad con las [versiones secundarias más recientes de PostgreSQL](./concepts-supported-versions.md) (12.6 y 11.11) en la creación de nuevos servidores.
* Compatibilidad con la [zona DNS privada](./concepts-networking.md#private-access-vnet-integration) de red virtual (VNET).
* Se admite elegir la zona de disponibilidad durante la operación de recuperación a un momento dado.
* Se admiten nuevas [regiones](./overview.md#azure-regions), entre ellas, Este de Australia, Centro de Canadá y Centro de Francia.
* Compatibilidad con el agrupador de conexiones [PgBouncer integrado](./concepts-pgbouncer.md). 
<!--- * Support for [pglogical](https://github.com/2ndQuadrant/pglogical) extension version 2.3.2. -->
* [Rendimiento inteligente](concepts-query-store.md) en versión preliminar pública.
* Varias correcciones de errores y mejoras en la estabilidad y el rendimiento.

## <a name="contacts"></a>Contactos

Para cualquier pregunta o sugerencia que pueda tener sobre el servidor flexible de Azure Database for PostgreSQL, envíe un correo electrónico al equipo de Azure Database for PostgreSQL ([@Ask Azure DB for PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com)). Tenga en cuenta que esta dirección de correo electrónico no es un alias de soporte técnico.

Además, tenga en cuenta los siguientes puntos de contacto según corresponda:

- Para ponerse en contacto con el servicio de soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
- Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).
  

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha leído la introducción al modo de implementación Servidor flexible de Azure Database for PostgreSQL, ya está listo para crear su primer servidor: [Creación de una instancia de Azure Database for PostgreSQL con la opción Servidor flexible mediante Azure Portal](./quickstart-create-server-portal.md)