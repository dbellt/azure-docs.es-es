---
title: ¿Qué es una base de datos única?
description: Obtenga información sobre el tipo de recurso de base de datos única en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: service-overview
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 04/08/2019
ms.openlocfilehash: 334ca6aadf43756985c7d187b460b625c5588525
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693037"
---
# <a name="what-is-a-single-database-in-azure-sql-database"></a>¿Qué es una base de datos única en Azure SQL Database?
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

El tipo de recurso de base de datos única crea una base de datos en Azure SQL Database con su propio conjunto de recursos y se administra a través de un [servidor](logical-servers.md). Con una sola base de datos, cada base de datos está aislada y se puede transportar. Cada una tiene su propio nivel de servicio en el [modelo de compra basado en DTU](service-tiers-dtu.md) o en el [modelo de compra basado en núcleo virtual](service-tiers-vcore.md) y un tamaño de proceso garantizado.

> [!IMPORTANT]
> Una base de datos única es un tipo de recurso en Azure SQL Database. El otro son los [grupos elásticos](elastic-pool-overview.md).

## <a name="dynamic-scalability"></a>Escalabilidad dinámica

Puede compilar su primera aplicación en una base de datos pequeña a bajo coste en el nivel de proceso sin servidor o en un tamaño de proceso pequeño en el nivel de proceso aprovisionado. Puede cambiar el [nivel de servicio o de proceso](single-database-scale.md) de forma manual o mediante programación en cualquier momento para adecuarlo a las necesidades de su solución. El rendimiento se puede ajustar sin que la aplicación o los clientes sufran ningún tipo de inactividad. La escalabilidad dinámica permite que una base de datos responda transparentemente a los requisitos de recursos, que cambian con rapidez, y le permite pagar solo por los recursos que necesite cuando los necesite.

## <a name="single-databases-and-elastic-pools"></a>Bases de datos únicas y grupos elásticos

Una base de datos única se puede mover dentro o fuera de un [grupo elástico](elastic-pool-overview.md) para que compartan recursos. Para muchas empresas y aplicaciones, poder crear bases de datos individuales y aumentar o reducir el rendimiento a petición es suficiente, especialmente si los patrones de uso son relativamente predecibles. Pero si dichos patrones son impredecibles, pueden dificultar la administración de los costos y del modelo de negocio. Los grupos elásticos están diseñados para solucionar este problema. El concepto es sencillo. Se asignan los recursos de rendimiento a un grupo, en lugar a una base de datos individual y se paga por los recursos de rendimiento colectivos del grupo, no por el rendimiento de la base de datos única.

## <a name="monitoring-and-alerting"></a>Supervisión y alertas

Utilice las herramientas integradas de [supervisión de rendimiento](performance-guidance.md) y de [alertas](alerts-insights-configure-portal.md) en combinación con las clasificaciones del rendimiento. Uso de estas herramientas, puede evaluar rápidamente el impacto de escalar verticalmente en función de su suscripción actual o se proyecta necesidades de rendimiento. Además, SQL Database puede [emitir métricas y registros de recurso](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) para facilitar la supervisión.

## <a name="availability-capabilities"></a>Funcionalidades de disponibilidad

Las bases de datos únicas y los grupos elásticos proporcionan muchas características de disponibilidad. Para obtener más información al respecto, consulte las [características de disponibilidad](sql-database-paas-overview.md#availability-capabilities).

## <a name="transact-sql-differences"></a>Diferencias de Transact-SQL

La mayoría de las características de Transact-SQL que usan las aplicaciones se admiten en Microsoft SQL Server y Azure SQL Database. Por ejemplo, los componentes principales de SQL, como tipos de datos, operadores, funciones de cadena, aritméticas, lógicas y de cursor, funcionan de la misma forma en SQL Server y SQL Database. Pero hay algunas diferencias de T-SQL en los elementos DDL (lenguaje de definición de datos) y DML (lenguaje de manipulación de datos) que generan instrucciones y consultas de T-SQL que solo se admiten parcialmente (como se describe más adelante en este artículo).

Además, hay algunas características y sintaxis que no se admiten, porque Azure SQL Database se ha diseñado para aislar las características de cualquier dependencia de la base de datos maestra y el sistema operativo. Por eso, muchas actividades de nivel de servidor no son apropiadas para SQL Database. Las instrucciones y opciones de T-SQL no están disponibles si configuran opciones de nivel de servidor y componentes del sistema operativo, o especifican la configuración del sistema de archivos. Cuando se necesitan estas capacidades, suele estar disponible una alternativa adecuada en alguna otra manera de SQL Database o de otra función o servicio de Azure.

Para obtener más información, consulte [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](transact-sql-tsql-differences-sql-server.md).

## <a name="security"></a>Seguridad

SQL Database proporciona varias [características integradas de seguridad y cumplimiento](security-overview.md) que facilitan que su aplicación cumpla los distintos requisitos de seguridad y cumplimiento normativo.

> [!IMPORTANT]
> Azure SQL Database ha obtenido la certificación de diversas normas de cumplimiento. Para más información, visite el [Centro de confianza de Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942), donde encontrará la lista más reciente de certificaciones de cumplimiento de SQL Database.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una introducción rápida a las bases de datos únicas, comience por la [guía de inicio rápido](quickstart-content-reference-guide.md).
- Para obtener información sobre cómo migrar una base de datos de SQL Server a Azure, vea [Migración a Azure SQL Database](migrate-to-database-from-sql-server.md).
- Para obtener información sobre las características admitidas, consulte el [artículo que trata sobre dicho tema](features-comparison.md).
