---
title: 'Azure Defender para SQL: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para SQL.
author: memildin
ms.author: memildin
ms.date: 05/27/2021
ms.topic: overview
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: 7f15ce79ddb8a952b66acd155423749554d4c520
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113002529"
---
# <a name="introduction-to-azure-defender-for-sql"></a>Introducción a Azure Defender para SQL

Azure Defender para SQL incluye dos planes de Azure Defender que amplían el [paquete de seguridad de datos](../azure-sql/database/azure-defender-for-sql.md) de Azure Security Center para proteger las bases de datos y sus datos dondequiera que se encuentren. Azure Defender para SQL incluye funcionalidades para detectar y mitigar las posibles vulnerabilidades de las bases de datos, así como para detectar actividades anómalas que puedan indicar una amenaza para ellas.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|**Azure Defender para servidores de base de datos de Azure SQL**: disponible con carácter general (GA)<br>**Azure Defender para servidores SQL Server en máquinas**: disponible con carácter general (GA) |
|Precios:|Los dos planes que forman **Azure Defender para SQL** se facturan como se muestra en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).|
|Versiones de SQL protegidas:|[SQL en máquinas virtuales de Azure](../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)<br>[Servidores SQL Server habilitados para Azure Arc](/sql/sql-server/azure-arc/overview)<br>Servidores SQL Server locales en máquinas Windows sin Azure Arc<br>[Bases de datos únicas](../azure-sql/database/single-database-overview.md) y [grupos elásticos](../azure-sql/database/elastic-pool-overview.md) de Azure SQL<br>[Instancia administrada de Azure SQL](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)<br>[Grupo de SQL dedicado de Azure Synapse Analytics (anteriormente SQL DW)](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![Sí](./media/icons/yes-icon.png) Azure China (**Parcial**: subconjunto de alertas y evaluación de vulnerabilidades en servidores SQL. No hay disponible protección contra amenazas de comportamiento).|
|||

## <a name="what-does-azure-defender-for-sql-protect"></a>¿Qué protege Azure Defender para SQL?

**Azure Defender para SQL** consta de dos planes de Azure Defender independientes:

- **Azure Defender para servidores de base de datos de Azure SQL** protege:
    - [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md)
    - [Instancia administrada de Azure SQL](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)
    - [Grupo de SQL dedicado en Azure Synapse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

- **Azure Defender para servidores SQL Server en máquinas** amplía las protecciones de los servidores SQL Server nativos de Azure para admitir totalmente los entornos híbridos y proteger los servidores SQL Server (todas las versiones compatibles) hospedados en Azure, otros entornos en la nube e incluso en máquinas locales:
    - [SQL Server en Virtual Machines](https://azure.microsoft.com/services/virtual-machines/sql-server/)
    - Servidores SQL Server locales:
        - [SQL Server habilitado para Azure Arc (versión preliminar)](/sql/sql-server/azure-arc/overview)
        - [Servidores SQL Server que se ejecutan en máquinas Windows sin Azure Arc](../azure-monitor/agents/agent-windows.md)


## <a name="what-are-the-benefits-of-azure-defender-for-sql"></a>¿Cuáles son las ventajas de Azure Defender para SQL?

Estos dos planes incluyen funcionalidad para detectar y mitigar posibles vulnerabilidades de las bases de datos, además de detectar actividades anómalas que puedan suponer alguna amenaza para ellas:

Un servicio de evaluación de vulnerabilidades detecta posibles vulnerabilidades de la base de datos, las sigue y ayuda a corregirlas. Los exámenes de evaluación proporcionan información general sobre el estado de seguridad de las máquinas SQL y detalles de las conclusiones de seguridad.

- Obtenga más información sobre la [evaluación de vulnerabilidades para Azure SQL Database](../azure-sql/database/sql-vulnerability-assessment.md).
- Obtenga más información sobre la [evaluación de vulnerabilidades para servidores de Azure SQL en máquinas](defender-for-sql-on-machines-vulnerability-assessment.md).

Un servicio de protección contra amenazas avanzada supervisa continuamente los servidores SQL Server para detectar amenazas tales como inyección de código SQL, ataques por fuerza bruta y abuso de privilegios. Este servicio proporciona alertas de seguridad orientadas a acciones en Azure Security Center con detalles de la actividad sospechosa, instrucciones sobre cómo mitigar las amenazas y opciones para continuar las investigaciones con Azure Sentinel. Más información sobre la [protección contra amenazas avanzada](../azure-sql/database/threat-detection-overview.md).

 > [!TIP]
 > Consulte la lista de alertas de seguridad de los servidores de SQL Server [en la página de referencia de alertas](alerts-reference.md#alerts-sql-db-and-warehouse).


## <a name="is-there-a-performance-impact-from-deploying-azure-defender-for-sql-on-machines"></a>La implementación de Azure Defender para SQL en las máquinas, ¿tiene un impacto en el rendimiento?

Evidentemente, el foco de **Azure Defender para SQL en máquinas** es la seguridad. Pero también nos interesa su actividad comercial y, por tanto, hemos priorizado el rendimiento para garantizar el impacto mínimo en los servidores SQL. 

El servicio tiene una arquitectura dividida para equilibrar la carga y la velocidad de los datos con el rendimiento: 

- Algunos de nuestros detectores se ejecutan en la máquina para obtener ventajas de velocidad en tiempo real.
- Otros se ejecutan en la nube para evitar cargas de cálculo pesadas en la máquina.

Las pruebas de laboratorio de nuestra solución, que la comparaban con las cargas de referencia, mostraron un promedio de uso de CPU del 3 % para los segmentos máximos. Un análisis de la telemetría de nuestros usuarios actuales muestra un impacto insignificante en el uso de CPU y memoria.

Por supuesto, el rendimiento siempre varía en función de los entornos, máquinas y cargas. Las declaraciones y números anteriores se brindan como guía general, no como garantía para ninguna implementación individual.


## <a name="what-kind-of-alerts-does-azure-defender-for-sql-provide"></a>¿Qué tipos de alertas ofrece Azure Defender para SQL?

Las alertas de seguridad enriquecidas con inteligencia sobre amenazas se desencadenan en estos casos:

- **Posibles ataques por inyección de código SQL**, incluidas las vulnerabilidades detectadas cuando las aplicaciones generan una instrucción SQL defectuosa en la base de datos.
- **Acceso a bases de datos y patrones de consulta anómalos**: por ejemplo, un número anormalmente alto de intentos de inicio de sesión incorrectos con credenciales diferentes (un intento por fuerza bruta).
- **Actividad sospechosa en las bases de datos**: por ejemplo, un usuario legítimo que accede a un servidor SQL Server desde un equipo vulnerado que se comunica con un servidor de comando y control de minería de datos de cifrado.

Las alertas incluyen detalles acerca del incidente que las desencadenó, así como recomendaciones acerca de cómo investigar y corregir las amenazas.



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para SQL. Para usar los servicios que se han descrito:

- Use servidores de Azure Defender para SQL en las máquinas para [examinar las vulnerabilidades de los servidores SQL Server](defender-for-sql-usage.md).
- Para una presentación de Azure Defender para SQL, vea cómo [Azure Defender para SQL puede proteger servidores SQL en cualquier lugar](https://www.youtube.com/watch?v=V7RdB6RSVpc).
