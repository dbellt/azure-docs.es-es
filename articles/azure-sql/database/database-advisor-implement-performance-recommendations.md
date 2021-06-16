---
title: Recomendaciones de rendimiento de Database Advisor para Azure SQL Database
description: Azure SQL Database ofrece recomendaciones para bases de datos que pueden mejorar el rendimiento de las consultas en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: fasttrack-edit, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: NikaKinska
ms.author: nnikolic
ms.reviewer: mathoma, wiassaf
ms.date: 03/10/2020
ms.openlocfilehash: b4959d2f7f65d4a9896a2e293f47a00ed84b1acd
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111590943"
---
# <a name="database-advisor-performance-recommendations-for-azure-sql-database"></a>Recomendaciones de rendimiento de Database Advisor para Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database comprende su aplicación y se adapta a ella. Azure SQL Database tiene varias instancias de Database Advisor que proporcionan recomendaciones personalizadas que permiten maximizar el rendimiento. Estos asesores de bases de datos evalúan y analizan continuamente el historial de uso, y proporcionan recomendaciones basadas en patrones de carga de trabajo que ayudan a mejorar el rendimiento.

## <a name="performance-overview"></a>Información general sobre rendimiento

La información general sobre el rendimiento ofrece un resumen del rendimiento de la base de datos y le ayuda a ajustar su rendimiento y a solucionar problemas.

![Información general sobre rendimiento de Azure SQL Database](./media/database-advisor-implement-performance-recommendations/performance-overview-annotated.png)

- El icono de **Recomendaciones** proporciona un desglose de recomendaciones de ajuste para la base de datos (se muestran las tres primeras recomendaciones si hay más). Al hacer clic en este icono, irá a **[Performance recommendation options](database-advisor-find-recommendations-portal.md#viewing-recommendations)** (Opciones de recomendaciones de rendimiento).
- El icono de **Tuning activity** (Actividad de optimización) proporciona un resumen de las acciones de optimización en curso y finalizadas para la base de datos, lo que le brinda una vista rápida del historial de la actividad de optimización. Si hace clic en este icono irá a la vista completa del historial de optimización de la basa de datos.
- El icono **Ajuste automático** muestra la **[configuración de ajuste automático](automatic-tuning-enable.md)** de la base de datos (las opciones de ajuste que se aplican automáticamente a la base de datos). Al hacer clic en este icono se abre el cuadro de diálogo de configuración de automatización.
- El icono de **Consultas de bases de datos** muestra el resumen del rendimiento de consultas de la base de datos (uso general de DTU y consultas que consumen más recursos). Haga clic en este icono para acceder a **[Información de rendimiento de consultas](query-performance-insight-use.md)** .

## <a name="performance-recommendation-options"></a>Opciones de recomendaciones de rendimiento

Las opciones de recomendaciones de rendimiento disponibles en Azure SQL Database son:

| Recomendación de rendimiento | Compatibilidad con bases de datos únicas y bases de datos agrupadas | Compatibilidad de base de datos de instancia |
| :----------------------------- | ----- | ----- |
| **Recomendaciones Crear índice**: recomienda la creación de índices que podrían mejorar el rendimiento de la carga de trabajo. | Sí | No |
| **Recomendaciones Quitar índice**: recomienda la eliminación diaria de los índices duplicados y redundantes, excepto los índices únicos y los que no se han usado durante mucho tiempo (más de 90 días). Tenga en cuenta que esta opción no es compatible con las aplicaciones que usan sugerencias de índice y la conmutación de particiones. No se admite la eliminación de índices sin usar para los niveles de servicio Prémium y Crítico para la empresa. | Sí | No |
| **Recomendaciones Parametrizar consultas (versión preliminar)** : recomienda la parametrización forzada en aquellos casos en que hay una o varias consultas que se recompilan constantemente, pero acaban con el mismo plan de ejecución de consulta. | Sí | No |
| **Recomendaciones Solucionar problemas de esquema (versión preliminar)** : las recomendaciones de corrección de esquema aparecen cuando Azure SQL Database detecta alguna anomalía en el número de errores de SQL relacionados con el esquema que se producen en la base de datos. Microsoft está dejando de usar las recomendaciones de corrección de problemas de esquema. | Sí | No |

![Recomendaciones de rendimiento para Azure SQL Database](./media/database-advisor-implement-performance-recommendations/performance-recommendations-annotated.png)

Para aplicar las recomendaciones de rendimiento, consulte la sección [Aplicación de las recomendaciones](database-advisor-find-recommendations-portal.md#applying-recommendations). Para ver el estado de las recomendaciones, consulte [Supervisión de operaciones](database-advisor-find-recommendations-portal.md#monitoring-operations).

También puede encontrar el historial completo de las acciones de ajuste que se aplicaron en el pasado.

## <a name="create-index-recommendations"></a>Recomendaciones Crear índice

Azure SQL Database supervisa continuamente las consultas que se ejecutan e identifica los índices que podrían mejorar el rendimiento. Después de que se sabe con bastante confianza que falta un índice, se crea una nueva recomendación **Crear índice**.

Para generar confianza, Azure SQL Database calcula la ganancia de rendimiento que el índice aportaría a lo largo del tiempo. Según la ganancia de rendimiento estimada, las recomendaciones se clasifican como alta, media o baja.

Los índices creados mediante recomendaciones se marcan siempre como índices auto_created. Para saber cuáles son los índices que se crean automáticamente, vaya a la [vista sys.indexes](/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql). Los índices creados de forma automática no bloquean los comandos ALTER/RENAME.

Si intenta descartar la columna que tiene un índice creado automáticamente por encima, el comando se pasa. El índice creado automáticamente también se descarta con el comando. Los índices normales bloquean el comando ALTER/RENAME en las columnas que están indexadas.

Después de aplicar la recomendación Crear índice, Azure SQL Database compara el rendimiento de las consultas con el de línea de base. Si el nuevo índice mejoró el rendimiento, la recomendación está marcada como correcta y el informe del impacto está disponible. Sin embargo, si no lo mejoró, se revierte de forma automática. Azure SQL Database emplea este proceso para asegurarse de que las recomendaciones mejoran el rendimiento de la base de datos.

Cualquier recomendación **Crear índice** tiene una directiva de rechazo que no permite aplicar la recomendación si la utilización de recursos de una base de datos o de un grupo es elevada. La directiva de rechazo tiene en cuenta la CPU, la E/S de datos, la E/S de registros y el almacenamiento disponible.

Si los valores de estos elementos son superiores al 80 % en los 30 minutos anteriores, la recomendación Crear índice se pospone. Si el almacenamiento disponible va a ser inferior al 10 % una vez creado el índice, la recomendación entra en un estado de error. Si al cabo de un par de días el ajuste automático considera que el índice resultaría beneficioso, el proceso comienza de nuevo.

Este proceso se repite hasta que haya suficiente almacenamiento disponible para crear un índice o hasta que el índice ya no se considere beneficioso.

## <a name="drop-index-recommendations"></a>Recomendaciones Quitar índice

Además de detectar índices que faltan, Azure SQL Database analiza continuamente el rendimiento de los índices existentes. Si no se usa un índice, Azure SQL Database recomienza descartarlo. Se recomienda quitar un índice en dos casos:

- El índice es un duplicado de otro (misma columna indexada e incluida, esquema de partición y filtros).
- El índice no se ha usado durante un período prolongado (93 días).

Las recomendaciones Quitar índice también llevan a cabo la comprobación después de la implementación. Si mejora el rendimiento, está disponible el informe del impacto. Si se degrada el rendimiento, se revierte la recomendación.

## <a name="parameterize-queries-recommendations-preview"></a>Recomendaciones Parametrización de consultas (versión preliminar)

Las recomendaciones de *parametrización de consultas* aparecen cuando tiene una o varias consultas que se vuelven a compilar continuamente, pero que terminan con el mismo plan de ejecución de consultas. Esta condición crea una oportunidad de aplicar la parametrización forzada. A su vez, la parametrización forzada permite que se almacenen en caché planes de consulta y se vuelvan a usar en el futuro, lo que mejora el rendimiento y reduce el uso de recursos.

Inicialmente, se deben compilar consultas de evento para generar un plan de ejecución. Cada plan generado se agrega a la caché de planes. Las sucesivas ejecuciones de la misma consulta pueden volver a usar este plan desde la caché, lo que elimina la necesidad de compilación adicional.

Las consultas con valores sin parámetros pueden llevar a una sobrecarga en el rendimiento porque el plan de ejecución se vuelve a compilar cada vez que los valores sin parámetros son diferentes. En muchos casos, las mismas consultas con distintos valores de parámetro generan los mismos planes de ejecución. Estos planes, sin embargo, se agregan aún por separado a la caché de planes.

El proceso de volver a compilar planes de ejecución usa recursos de base de datos, aumenta el tiempo de duración de la consulta y desborda la caché de planes. Estos eventos, a su vez, hacen que los planes se expulsen de la caché. Este comportamiento se puede modificar estableciendo la opción de parametrización forzada en la base de datos.

Para ayudarlo a estimar el impacto de esta recomendación, se le proporciona una comparación entre el uso real y el previsto de la CPU (como si se aplicase la recomendación). Esta recomendación puede ayudarle a obtener ahorros de CPU. También puede ayudarle a reducir la duración de la consulta y la sobrecarga de la caché de planes, lo que significa que varios de los planes pueden permanecer en la caché y volverse a utilizar. Para aplicar esta recomendación rápidamente, seleccione el comando **Aplicar**.

Después de aplicar esta recomendación, se habilita la parametrización forzada en cuestión de minutos en la base de datos. Se inicia el proceso de supervisión, que tiene una validez de aproximadamente 24 horas. Después de este período, puede ver el informe de validación. Este informe muestra el uso de CPU de la base de datos 24 horas antes y después de haber aplicado la recomendación. Azure SQL Database Advisor cuenta con un mecanismo de seguridad que revierte automáticamente la recomendación aplicada en caso de detectarse una regresión del rendimiento.

## <a name="fix-schema-issues-recommendations-preview"></a>Recomendaciones para solucionar problemas del esquema (vista previa)

> [!IMPORTANT]
> Microsoft está dejando de usar las recomendaciones de corrección de problemas de esquema. Se recomienda usar [Intelligent Insights](intelligent-insights-overview.md) para supervisar los problemas de rendimiento de la base de datos, incluidos los problemas de esquema que anteriormente trataban las recomendaciones de corrección de problemas de esquema.

Las recomendaciones **Solucionar problemas de esquema** aparecen cuando Azure SQL Database detecta alguna anomalía en el número de errores de SQL relacionados con el esquema que se producen en la base de datos. Esta recomendación suele aparecer cuando la base de datos encuentra varios errores relacionados con el esquema (nombre de columna no válido, nombre de objeto no válido, etc.) en el curso de una hora.

Los "problemas de esquema" son un tipo de errores de sintaxis. Se producen cuando la definición de la consulta SQL y la definición del esquema de base de datos no concuerdan. Por ejemplo, puede que en la tabla de destino falte una de las columnas que espera la consulta o viceversa.

Las recomendaciones "Solucionar problemas de esquema" aparecen cuando Azure SQL Database detecta alguna anomalía en el número de errores de SQL relacionados con el esquema que se producen en la base de datos. En la tabla siguiente se muestran los errores relacionados con los problemas del esquema:

| Código de error SQL | Message |
| --- | --- |
| 201 |El procedimiento o la función ' *' espera parámetros '* ', que no se han proporcionado. |
| 207 |Nombre de columna '*' no válido. |
| 208 |Nombre de objeto '*' no válido. |
| 213 |El nombre de columna o los valores especificados no corresponden a la definición de la tabla. |
| 2812 |No se pudo encontrar el procedimiento almacenado '*'. |
| 8144 |La función o el procedimiento * tiene demasiados argumentos. |

## <a name="custom-applications"></a>Aplicaciones personalizadas

Los desarrolladores pueden considerar la posibilidad de desarrollar aplicaciones personalizadas con las recomendaciones de rendimiento de Azure SQL Database. Todas las recomendaciones que se muestran en el portal para una base de datos están disponibles a través de la API [Get-AzSqlDatabaseRecommendedAction](/powershell/module/az.sql/get-azsqldatabaserecommendedaction).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo realizar un ajuste automático de los índices de base de datos y consultar los planes de ejecución, consulte [Ajuste automático de Azure SQL Database](automatic-tuning-overview.md).
- Para más información sobre cómo supervisar de forma automática el rendimiento de la base de datos con diagnósticos automatizados y análisis de causa principal de problemas de rendimiento, consulte [Azure SQL Intelligent Insights](intelligent-insights-overview.md).
- Consulte [Query Performance Insight](query-performance-insight-use.md) para más información sobre el impacto en el rendimiento de las principales consultas.