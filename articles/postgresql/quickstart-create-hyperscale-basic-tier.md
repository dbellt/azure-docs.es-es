---
title: 'Inicio rápido: Creación de un grupo de servidores Hiperescala (Citus) de nivel Básico: Azure Database for PostgreSQL'
description: Introducción al nivel Básico de Hiperescala (Citus) de Azure Database for PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: quickstart
ms.date: 04/07/2021
ms.openlocfilehash: 07ace217e5299662a1c3145a225abc25f4f1f337
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023914"
---
# <a name="create-a-hyperscale-citus-basic-tier-server-group-in-the-azure-portal"></a>Creación de un grupo de servidores Hiperescala (Citus) de nivel Básico en Azure Portal

Hiperescala (Citus) de Azure Database for PostgreSQL es un servicio administrado que se usa para ejecutar, administrar y escalar bases de datos PostgreSQL de alta disponibilidad en la nube. Su [nivel Básico](concepts-hyperscale-tiers.md) es una opción de implementación práctica para el desarrollo y las pruebas iniciales.

Este inicio rápido muestra cómo crear un grupo de servidores Hiperescala (Citus) de nivel Básico mediante Azure Portal. Aprovisionará el grupo de servidores y comprobará que puede conectarse a él para ejecutar consultas.

> [!IMPORTANT]
> El nivel Básico de Hiperescala (Citus) está actualmente en versión preliminar.  Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar de Hiperescala (Citus) de PostgreSQL](hyperscale-preview-features.md).

[!INCLUDE [azure-postgresql-hyperscale-create-basic-tier](../../includes/azure-postgresql-hyperscale-create-basic-tier.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a aprovisionar un grupo de servidores de Hiperescala (Citus). Se conectó a él con psql, creó un esquema y distribuyó datos.

- Siga un tutorial para [crear aplicaciones escalables multiinquilino](./tutorial-design-database-hyperscale-multi-tenant.md).
- Determine el mejor [tamaño inicial](howto-hyperscale-scale-initial.md) para el grupo de servidores.
