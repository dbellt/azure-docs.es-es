---
title: Estado del escenario de migración de bases de datos
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre el estado de los escenarios de migración que admite Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: troubleshooting
ms.date: 07/08/2020
ms.openlocfilehash: 35ef2ea3f03ca86aea34f736de4317d75a44ae59
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108330519"
---
# <a name="status-of-migration-scenarios-supported-by-azure-database-migration-service"></a>Estado de los escenarios de migración que admite Azure Database Migration Service

Azure Database Migration Service está diseñado para admitir una serie de escenarios de migración (pares de origen/destino) para las migraciones sin conexión (de un solo uso) y en línea (sincronización continua). Azure Database Migration Service proporciona una cobertura de escenarios que se amplía constantemente. Nos encargamos de agregar nuevos escenarios de forma regular. En este artículo se identifican los escenarios de migración que actualmente son compatibles con Azure Database Migration Service y el estado (versión preliminar privada, versión preliminar pública o disponibilidad general) de cada escenario.

## <a name="offline-versus-online-migrations"></a>Migraciones sin conexión o en línea

Con Azure Database Migration Service, puede realizar una migración sin conexión o en línea. Si usa las migraciones *sin conexión*, el tiempo de inactividad de la aplicación comienza al mismo tiempo que la migración. Para limitar el tiempo de inactividad al tiempo necesario para la transición al nuevo entorno cuando finalice la migración, use una migración *en línea*. Es recomendable probar una migración sin conexión para determinar si el tiempo de inactividad es aceptable; si no fuera así, realice una migración en línea.

## <a name="migration-scenario-status"></a>Estado del escenario de migración

El estado de los escenarios de migración que admite Azure Database Migration Service varía con el tiempo. Por lo general, los escenarios se lanzan primero en **versión preliminar privada**. Después de la versión preliminar privada, el estado del escenario cambia a **versión preliminar pública**. Los usuarios de Azure Database Migration Service pueden probar escenarios de migración en la versión preliminar pública directamente desde la interfaz de usuario. No es preciso registrarse.  Sin embargo, los escenarios de migración en versión preliminar pública podrían no estar disponibles en todas las regiones y podrían sufrir cambios adicionales antes del lanzamiento de la versión final. Después de la versión preliminar pública, el estado del escenario cambia a **disponibilidad general**. Disponibilidad general (GA) es el estado de la versión final, y la funcionalidad es completa y accesible para todos los usuarios.

## <a name="migration-scenario-support"></a>Compatibilidad del escenario de migración

En las siguientes tablas se muestran qué escenarios de migración se admiten cuando se usa Azure Database Migration Service.

> [!NOTE]
> Si un escenario que se indica como admitido no aparece en la interfaz de usuario, póngase en contacto con el alias para [preguntar a Azure Database Migration Service](mailto:AskAzureDatabaseMigrations@service.microsoft.com) para obtener información adicional.

### <a name="offline-one-time-migration-support"></a>Compatibilidad de la migración sin conexión (de un solo uso)

En la siguiente tabla se muestra la compatibilidad de Azure Database Migration Service con las migraciones sin conexión.

| Destino  | Source | Soporte técnico | Estado |
| ------------- | ------------- |:-------------:|:-------------:|
| **Azure SQL DB** | SQL Server | ✔ | Disponibilidad general |
|   | SQL de RDS | X |  |
|   | Oracle | X |  |
| **Azure SQL DB MI** | SQL Server | ✔ | Disponibilidad general |
|   | SQL de RDS | X |  |
|   | Oracle | X |   |
| **Máquina virtual de Azure SQL** | SQL Server | ✔ | Disponibilidad general |
|   | Oracle | X |   |
| **Azure Cosmos DB** | MongoDB | ✔ | Disponibilidad general |
| **Azure DB para MySQL** | MySQL | ✔ |   |
|   | MySQL de RDS | X |   |
| **Azure DB for PostgreSQL: servidor único** | PostgreSQL | X |
|  | PostgreSQL de RDS | X |   |
| **Azure DB for PostgreSQL: Hiperescala (Citus)** | PostgreSQL | X |
|  | PostgreSQL de RDS | X |   |

### <a name="online-continuous-sync-migration-support"></a>Compatibilidad con la migración en línea (sincronización continua)

En la siguiente tabla se muestra la compatibilidad de Azure Database Migration Service con las migraciones en línea.

| Destino  | Source | Soporte técnico | Estado |
| ------------- | ------------- |:-------------:|:-------------:|
| **Azure SQL DB** | SQL Server | X |  |
|   | SQL de RDS | X |  |
|   | Oracle | X |  |
| **Azure SQL DB MI** | SQL Server | ✔ | Disponibilidad general |
|   | SQL de RDS | X |  |
|   | Oracle | X |  |
| **Máquina virtual de Azure SQL** | SQL Server | X |   |
|   | Oracle  | X |  |
| **Azure Cosmos DB** | MongoDB | ✔ | Disponibilidad general |
| **Azure DB para MySQL** | MySQL | ✔ | Disponibilidad general |
|   | MySQL de RDS | ✔ | Disponibilidad general |
| **Azure DB for PostgreSQL: servidor único** | PostgreSQL | ✔ | Disponibilidad general |
|   | Azure DB for PostgreSQL: servidor único | ✔ | Disponibilidad general |
|   | PostgreSQL de RDS | ✔ | Disponibilidad general |
|   | Oracle | ✔ | Versión preliminar pública (quedará en desuso después del 1 de mayo de 2021) |
| **Azure DB for PostgreSQL: Hiperescala (Citus)** | PostgreSQL | ✔ | Disponibilidad general |
|   | PostgreSQL de RDS | ✔ | Disponibilidad general |

> [!IMPORTANT]
> El escenario de migración "Oracle a Azure Database for PostgreSQL" (actualmente en versión preliminar) dejará de estar disponible después del 1 de mayo de 2021. Seguiremos brindando soporte a través de herramientas alternativas (como Ora2pg) y proporcionaremos la mejor experiencia para las migraciones de Oracle a PostgreSQL. Para ver los procedimientos recomendados de migración, consulte la [Guía de migración de Oracle a Azure Database for PostgreSQL](https://aka.ms/OracletoPGguide).


## <a name="next-steps"></a>Pasos siguientes

Para información general sobre Azure Database Migration Service y la disponibilidad regional, consulte el artículo [¿Qué es Azure Database Migration Service?](dms-overview.md)
