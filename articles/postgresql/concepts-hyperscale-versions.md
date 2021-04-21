---
title: 'Versiones compatibles: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: Versiones de PostgreSQL disponibles en Azure Database for PostgreSQL para Hiperescala (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: d8a584b6ba752e8f9220defa575f519828ba07e6
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023905"
---
# <a name="supported-database-versions-in-azure-database-for-postgresql--hyperscale-citus"></a>Versiones de base de datos compatibles en Azure Database for PostgreSQL para Hiperescala (Citus)

## <a name="postgresql-versions"></a>Versiones de PostgreSQL

> [!IMPORTANT]
> Las versiones personalizables de PostgreSQL en Hiperescala (Citus) están actualmente en versión preliminar.  Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar de Hiperescala (Citus) de PostgreSQL](hyperscale-preview-features.md).

La versión de PostgreSQL que se ejecuta en un grupo de servidores de Hiperescala (Citus) se puede personalizar durante la creación. Cualquier versión que no sea la 11 es actualmente una característica en versión preliminar.

Actualmente Hiperescala (Citus) admite las siguientes versiones principales:

### <a name="postgresql-version-13-preview"></a>PostgreSQL versión 13 (versión preliminar)

La versión secundaria actual es la 13.2. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/13/static/release-13-2.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria.

### <a name="postgresql-version-12-preview"></a>PostgreSQL versión 12 (versión preliminar)

La versión secundaria actual es la 12.6. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/12/static/release-12-6.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria.

### <a name="postgresql-version-11"></a>PostgreSQL, versión 11

La versión secundaria actual es la 11.11. Consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-11.html) para obtener más información sobre las mejoras y correcciones de esta versión secundaria.

### <a name="postgresql-version-10-and-older"></a>PostgreSQL, versión 10 y anteriores

No se admite la versión 10 de PostgreSQL ni las anteriores para Azure Database for PostgreSQL: Hiperescala (Citus).

## <a name="citus-and-other-extension-versions"></a>Citus y otras versiones de extensión

En función de la versión de PostgreSQL que se ejecute en un grupo de servidores, también se instalarán diferentes [versiones de las extensiones de Postgres](concepts-hyperscale-extensions.md).  En concreto, Postgres 13 viene con Citus 10, y las versiones anteriores de Postgres vienen con Citus 9.5.

## <a name="next-steps"></a>Pasos siguientes

* Consulte qué [extensiones](concepts-hyperscale-extensions.md) se instalan con cada versión.
* Aprenda a [crear un grupo de servidores de Hiperescala (Citus)](quickstart-create-hyperscale-portal.md).
