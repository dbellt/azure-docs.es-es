---
title: Escalado de recursos
description: En este artículo se explica cómo escalar la base de datos en Azure SQL Database e Instancia administrada de SQL mediante la adición o eliminación de recursos asignados.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: wiassaf, sstein
ms.date: 06/25/2019
ms.openlocfilehash: ca1a2edec70b13f111ffd89278aa39d1ddea7f67
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105035649"
---
# <a name="dynamically-scale-database-resources-with-minimal-downtime"></a>Escalado dinámico de recursos de base de datos con tiempo de inactividad mínimo
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database e Instancia administrada de SQL le permite agregar dinámicamente más recursos a su base de datos con un mínimo [tiempo de inactividad](https://azure.microsoft.com/support/legal/sla/sql-database); sin embargo, hay un breve período de cambio donde se pierde la conectividad a la base de datos, lo que se puede mitigar con la lógica de reintento.

## <a name="overview"></a>Información general

Cuando la demanda de una aplicación aumenta de unos cuantos dispositivos y clientes a millones, Azure SQL Database y SQL Managed Instance se escalan sobre la marcha con un tiempo de inactividad mínimo. La escalabilidad es una de las características más importantes de plataforma como servicio (PaaS) que permite agregar de forma dinámica más recursos al servicio cuando sea necesario. Azure SQL Database permite cambiar fácilmente los recursos (potencia de CPU, memoria, almacenamiento y rendimiento de E/S) asignados a las bases de datos.

Puede mitigar problemas de rendimiento debidos al aumento del uso de la aplicación que no se pueden resolver mediante métodos de indexación o reescritura de consultas. Agregar más recursos permite reaccionar rápidamente cuando la base de datos alcanza los límites de recursos actuales y se necesita más capacidad para controlar la carga de trabajo entrante. Azure SQL Database también permite reducir verticalmente los recursos cuando no se necesitan para reducir el costo.

No es necesario preocuparse de comprar hardware ni cambiar la infraestructura subyacente. El escalado de una base de datos se puede realizar fácilmente a través de Azure Portal mediante un control deslizante.

![Escalar el rendimiento de la base de datos](./media/scale-resources/scale-performance.svg)

Azure SQL Database ofrece el [modelo de compra basado en DTU](service-tiers-dtu.md) y el [modelo de compra basado en núcleo virtual](service-tiers-vcore.md), mientras que Instancia administrada de Azure SQL ofrece solo el [modelo de compra basado en núcleo virtual](service-tiers-vcore.md). 

- El [modelo de compra basado en DTU](service-tiers-dtu.md) ofrece una combinación de recursos de proceso, memoria y E/S en tres niveles de servicio para admitir cargas de trabajo de base de datos de ligeras a pesadas: Básico, Estándar y Premium. Los niveles de rendimiento de cada nivel ofrecen una mezcla diferente de estos recursos, a los que puede agregar recursos de almacenamiento adicionales.
- El [modelo de compra basado en núcleo virtual](service-tiers-vcore.md) le permite elegir el número de núcleos virtuales, la cantidad de memoria y la cantidad y velocidad del almacenamiento. Este modelo de compra ofrece tres niveles de servicio: Uso general, Crítico para la empresa e Hiperescala.

La primera aplicación se puede compilar en una base de datos pequeña con un costo muy bajo al mes en los niveles de servicio Básico, Estándar o Uso general y, después, cambiar el nivel de servicio manualmente o mediante programación en cualquier momento al nivel de servicio Premium o Crítico para la empresa para adecuarlo a las necesidades de su solución. El rendimiento se puede ajustar sin que la aplicación o los clientes sufran ningún tipo de inactividad. La escalabilidad dinámica permite que una base de datos responda transparentemente a los requisitos de recursos, que cambian con rapidez, y le permite pagar solo por los recursos que necesite cuando los necesite.

> [!NOTE]
> La escalabilidad dinámica es diferente del escalado automático. El escalado automático se produce al escalarse un servicio automáticamente en función de determinados criterios, mientras la escalabilidad dinámica permite el escalado manual con un tiempo de inactividad mínimo.

Las bases de datos sencillas de Azure SQL Database admiten la escalabilidad dinámica manual, pero no el escalado automático. Para ganar experiencia con el uso *automático*, considere los grupos elásticos, que permiten que las bases de datos compartan recursos en un grupo en función de las necesidades individuales de las bases de datos.
Pero hay scripts que pueden ayudar a automatizar la escalabilidad de una base de datos única en Azure SQL Database. En [Uso de PowerShell para supervisar y escalar una sola base de datos SQL](scripts/monitor-and-scale-database-powershell.md) encontrará un ejemplo.

Puede cambiar los [niveles de servicio de DTU](service-tiers-dtu.md) o las [características de núcleo virtual](resource-limits-vcore-single-databases.md) en cualquier momento con un tiempo de inactividad mínimo para la aplicación (normalmente una media de menos de cuatro segundos). Para muchas empresas y aplicaciones, poder crear bases de datos y aumentar o reducir el rendimiento a petición es suficiente, especialmente si los patrones de uso son relativamente predecibles. Pero si dichos patrones son impredecibles, pueden dificultar la administración de los costos y del modelo de negocio. Para este escenario, se usa un grupo elástico con un determinado número de eDTU que se comparten entre varias bases de datos del grupo.

![Introducción a SQL Database: DTU de bases de datos únicas por nivel](./media/scale-resources/single_db_dtus.png)

Azure SQL Database ofrece la posibilidad de escalar dinámicamente las bases de datos:

- Con una [base de datos única](single-database-scale.md), se pueden usar los modelos de [DTU](resource-limits-dtu-single-databases.md) o [núcleo virtual](resource-limits-vcore-single-databases.md) para definir la cantidad máxima de recursos que se asignarán a cada base de datos.
- Los [grupos elásticos](elastic-pool-scale.md) permiten definir el límite máximo de recursos por grupo de bases de datos en el grupo.

Instancia administrada de Azure SQL permite escalar también: 

- [SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) usa el modo de [núcleos virtuales](../managed-instance/sql-managed-instance-paas-overview.md#vcore-based-purchasing-model) y le permite definir el máximo de núcleos de CPU y el máximo de almacenamiento asignado a la instancia. Todas las bases de datos dentro de la instancia administrada comparten los recursos asignados a la instancia.

Si se inicia la acción de escalado o reducción vertical en cualquiera de los tipos, se reiniciará el proceso del motor de base de datos y se moverá a otra máquina virtual si es necesario. El cambio del proceso del motor de base de datos a una nueva máquina virtual es un **proceso en línea** en el que puede continuar usando el servicio de Azure SQL Database existente mientras el proceso está en curso. Una vez que el motor de base de datos de destino está completamente inicializado y listo para procesar las consultas, las conexiones [pasarán del motor de base de datos de origen al de destino](single-database-scale.md#impact).

> [!NOTE]
> No se recomienda escalar la instancia administrada si se está ejecutando una transacción de larga duración, como la importación de datos, los trabajos de procesamiento de datos o la regeneración de índices, o si tiene una conexión activa en la instancia. Para evitar que el escalado tarde en completarse más tiempo que de costumbre, debe escalar la instancia una vez finalizadas todas las operaciones de ejecución prolongada.

> [!NOTE]
> Puede esperar una pequeña interrupción de la conexión cuando el proceso de escalado o reducción vertical haya terminado. Si ha implementado la [lógica de reintentos para errores transitorios estándar](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors), no notará la conmutación por error.

## <a name="alternative-scale-methods"></a>Métodos de escala alternativos

El escalado de recursos es la manera más fácil y eficaz de mejorar el rendimiento de una base de datos sin cambiar su código ni el de la aplicación. En algunos casos, es posible que ni siquiera los más altos niveles de servicio, tamaños de proceso y optimizaciones de rendimiento puedan controlar la carga de trabajo de forma correcta y rentable. En esos casos dispone de estas otras opciones para escalar la base de datos:

- [Escalado horizontal de lectura](read-scale-out.md) es una característica disponible en la que se obtiene una réplica de solo lectura de los datos en la que se pueden ejecutar exigentes consultas de solo lectura, como por ejemplo informes. Una réplica de solo lectura controlará la carga de trabajo de solo lectura sin que el uso de los recursos en la base de datos principal se vea afectado.
- [Particionamiento de base de datos](elastic-scale-introduction.md) es un conjunto de técnicas que permite dividir los datos en varias bases de datos y escalarlas por separado.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo mejorar el rendimiento de la base de datos cambiando el código de la base de datos, vea [Búsqueda y aplicación de recomendaciones de rendimiento](database-advisor-find-recommendations-portal.md).
- Para obtener información sobre cómo permitir que la inteligencia de base de datos integrada optimice la base de datos, vea [Ajuste automático](automatic-tuning-overview.md).
- Para obtener información sobre el escalado horizontal de lectura en Azure SQL Database, vea [Uso de réplicas de solo lectura para equilibrar la carga de las cargas de trabajo de consultas de solo lectura](read-scale-out.md).
- Para obtener información sobre el particionamiento de una base de datos, vea [Escalado horizontal con Azure SQL Database](elastic-scale-introduction.md).
