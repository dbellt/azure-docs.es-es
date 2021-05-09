---
title: 'Versiones admitidas en Azure Database for PostgreSQL: Servidor flexible'
description: 'Se describen las versiones principales y secundarias de PostgreSQL admitidas en Azure Database for PostgreSQL: Servidor flexible.'
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/22/2021
ms.openlocfilehash: cd78d6802f6b7f71794785d9149b5cda854bf21e
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166422"
---
# <a name="supported-postgresql-major-versions-in-azure-database-for-postgresql---flexible-server"></a>Versiones principales de PostgreSQL admitidas en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

Azure Database for PostgreSQL: Servidor flexible actualmente admite las versiones principales siguientes:

## <a name="postgresql-version-12"></a>PostgreSQL, versión 12

La versión secundaria actual es la **12.6**. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/12/static/release-12-6.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria. Con esta versión secundaria, se crearán nuevos servidores. Los servidores existentes se actualizarán automáticamente a la versión secundaria admitida más reciente en la futura ventana de mantenimiento programado.

## <a name="postgresql-version-11"></a>PostgreSQL, versión 11

La versión secundaria actual es la **11.11**. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-11.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria. Con esta versión secundaria, se crearán nuevos servidores. Los servidores existentes se actualizarán automáticamente a la versión secundaria admitida más reciente en la futura ventana de mantenimiento programado.

## <a name="postgresql-version-10-and-older"></a>PostgreSQL, versión 10 y anteriores

No se admite la versión 10 de PostgreSQL ni las anteriores para Azure Database for PostgreSQL: Servidor flexible. Use la opción de implementación [Servidor único](../concepts-supported-versions.md) si necesita versiones anteriores.

## <a name="managing-upgrades"></a>Administración de actualizaciones

El proyecto de PostgreSQL publica periódicamente versiones secundarias para corregir errores detectados. Azure Database for PostgreSQL aplica automáticamente revisiones a los servidores con versiones secundarias durante las implementaciones mensuales del servicio.

Todavía no se admite la automatización para la actualización de la versión principal. Por ejemplo, actualmente no hay una actualización automática de PostgreSQL 11 a PostgreSQL 12.<!-- To upgrade to the next major version, create a [database dump and restore](howto-migrate-using-dump-and-restore.md) to a server that was created with the new engine version.-->

<!--
## Next steps

For information on supported PostgreSQL extensions, see [the extensions document](concepts-extensions.md).
-->