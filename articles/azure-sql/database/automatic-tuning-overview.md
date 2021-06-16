---
title: Introducción al ajuste automático
description: Azure SQL Database y la instancia administrada de Azure SQL analizan una consulta SQL y se adaptan automáticamente a la carga de trabajo del usuario.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: NikaKinska
ms.author: nnikolic
ms.reviewer: mathoma, wiassaf
ms.date: 03/23/2021
ms.openlocfilehash: ae950029e0e20fb0233cb1ca7905309f9c3e7b2a
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591285"
---
# <a name="automatic-tuning-in-azure-sql-database-and-azure-sql-managed-instance"></a>Ajuste automático en Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

El ajuste automático de Azure SQL Database e Instancia administrada de Azure SQL proporciona un alto rendimiento y cargas de trabajo estables gracias al ajuste continuo del rendimiento basado en la inteligencia artificial y el aprendizaje automático.

El ajuste automático es un servicio de rendimiento inteligente completamente administrado que usa la inteligencia integrada para supervisar continuamente las consultas ejecutadas en una base de datos y mejora automáticamente su rendimiento. Esto se logra mediante la adaptación dinámica de la base de datos a las cargas de trabajo cambiantes y la aplicación de las recomendaciones de ajuste. El ajuste automático aprende horizontalmente de todas las bases de datos de Azure mediante la inteligencia artificial y mejora dinámicamente sus acciones de ajuste. Cuanto más tiempo se ejecute una base de datos con el ajuste automático activado, mejor rendimiento tendrá.

El ajuste automático de Azure SQL Database e Instancia administrada de Azure SQL puede ser una de las características más importantes que puede habilitar para proporcionar cargas de trabajo de base de datos estables y con un alto rendimiento.

## <a name="what-can-automatic-tuning-do-for-you"></a>¿Cómo le puede ayudar el ajuste automático?

- Ajuste del rendimiento automatizado de bases de datos
- Comprobación automática de mejoras de rendimiento
- Corrección y reversión automatizadas
- Historial de ajuste
- Ajuste de scripts de Transact-SQL (T-SQL) de acción para implementaciones manuales
- Supervisión proactiva del rendimiento de las cargas de trabajo
- Funcionalidad de escalabilidad horizontal de cientos de miles de bases de datos
- Impacto positivo para los recursos de DevOps y el costo total de propiedad

## <a name="safe-reliable-and-proven"></a>Seguro, confiable y probado

Las operaciones de ajuste aplicadas a las bases de datos de Azure SQL son totalmente seguras para el rendimiento de las cargas de trabajo más intensas. El sistema se ha diseñado cuidadosamente para que no interfiera con las cargas de trabajo del usuario. Las recomendaciones de ajuste automático se aplican solo durante momentos de uso escaso. El sistema también puede deshabilitar temporalmente las operaciones de ajuste automático para proteger el rendimiento de la carga de trabajo. En este caso, se mostrará el mensaje "Disabled by the system" (Deshabilitado por el sistema) en Azure Portal. El ajuste automático tiene en cuenta las cargas de trabajo con la máxima prioridad de recursos.

Los mecanismos de ajuste automático están muy desarrollados y se han perfeccionado en varios millones de bases de datos que se ejecutan en Azure. Las operaciones de ajuste automatizado aplicadas se comprueban automáticamente para asegurarse de que hay una mejora en el rendimiento de la carga de trabajo. Las recomendaciones de rendimiento con regresión se detectan de forma dinámica y se revierten rápidamente. A través del historial de ajustes existe un claro seguimiento de las mejoras de los ajustes realizados en cada base de datos de Azure SQL Database e Instancia administrada de Azure SQL.

![¿Cómo funciona el ajuste automático?](./media/automatic-tuning-overview/how-does-automatic-tuning-work.png)

El ajuste automático de Azure SQL Database comparte su lógica básica con el motor de ajuste automático de SQL Server en el motor de base de datos. Para obtener información técnica adicional sobre el mecanismo de inteligencia integrada, vea el artículo sobre el [ajuste automático de SQL Server](/sql/relational-databases/automatic-tuning/automatic-tuning).

Para obtener información general sobre cómo funciona el ajuste automático y los escenarios de uso habituales, vea el vídeo insertado:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Improve-Azure-SQL-Database-Performance-with-automatic-tuning/player]

## <a name="enable-automatic-tuning"></a>Habilitación del ajuste automático

- [Habilite el ajuste automático para Azure SQL Database en Azure Portal](automatic-tuning-enable.md) o mediante la instrucción [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true) de T-SQL.
- Habilite el ajuste automático para Instancia administrada de Azure SQL mediante la instrucción [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-mi-current&preserve-view=true) de T-SQL.

## <a name="automatic-tuning-options"></a>Opciones de ajuste automático

Las siguientes son las opciones de ajuste automático disponibles en Azure SQL Database e Instancia administrada de Azure SQL:

| Opción de ajuste automático | Compatibilidad con bases de datos únicas y bases de datos agrupadas | Compatibilidad de base de datos de instancia |
| :----------------------------- | ----- | ----- |
| **CREATE INDEX**: identifica los índices que pueden mejorar el rendimiento de la carga de trabajo, crea índices y comprueba automáticamente que el rendimiento de las consultas ha mejorado. | Sí | No |
| **DROP INDEX**: quita los índices duplicados y sin usar (en los últimos 90 días). Los índices únicos, incluidos los índices que admiten restricciones únicas y de clave principal, nunca se quitan. Esta opción se puede deshabilitar automáticamente cuando hay consultas con sugerencias de índice presentes en la carga de trabajo, o cuando la carga de trabajo realiza una modificación de particiones. En los niveles de servicio Premium y Crítico para la empresa, esta opción nunca quitará los índices no usados, pero sí quitará los índices duplicados, si los hay. | Sí | No |
| **FORCE LAST GOOD PLAN** (corrección automática del plan): identifica consultas de Azure SQL que usan un plan de ejecución más lento que el plan correcto anterior, y consultas que usan el último plan correcto conocido, en lugar del plan revertido. | Sí | Sí |

### <a name="automatic-tuning-for-sql-database"></a>Ajuste automático para SQL Database

En el ajuste automático de Azure SQL Database se emplean las recomendaciones del asesor de base de datos **CREATE INDEX**, **DROP INDEX** y **FORCE LAST GOOD PLAN** para optimizar el rendimiento de la base de datos. Para más información, consulte [Recomendaciones del asesor de base de datos en Azure Portal](database-advisor-find-recommendations-portal.md), en [PowerShell](/powershell/module/az.sql/get-azsqldatabaserecommendedaction) y en la [API REST](/rest/api/sql/serverautomatictuning).

Las recomendaciones de ajuste se pueden aplicar manualmente mediante Azure Portal, o bien puede dejar que se apliquen de forma autónoma. Las ventajas de dejar que el sistema aplique las recomendaciones de ajuste de forma autónoma es que valida automáticamente y se produce una ganancia positiva en el rendimiento de la carga de trabajo y, si se detecta que no hay una mejora significativa, revertirá automáticamente la recomendación de ajuste. Tenga en cuenta de que si las consultas implicadas en las recomendaciones de ajuste no se ejecutan con frecuencia, la fase de validación puede durar hasta 72 horas intencionadamente.

En caso de que aplique las recomendaciones de ajuste mediante T-SQL, ni la validación automática del rendimiento ni los mecanismos de inversión estarán disponibles. Las recomendaciones aplicadas de este modo permanecerán activas y se mostrarán en la lista de recomendaciones de optimización durante 24-48 horas. antes de que el sistema las retire automáticamente. Si quiere quitar una recomendación antes, puede descartarla en Azure Portal.

Las opciones de ajuste automático se pueden habilitar o deshabilitar en cada base de datos de forma independiente o pueden configurarse en el nivel de servidor y aplicarse en todas las bases de datos que hereden la configuración de este. Los servidores pueden heredar valores predeterminados de Azure para la configuración de optimización automática. Los valores predeterminados de Azure en este momento son FORCE_LAST_GOOD_PLAN (habilitado), CREATE_INDEX (habilitado) y DROP_INDEX (deshabilitado).

> [!IMPORTANT]
> A partir de marzo de 2020, los cambios en los valores predeterminados de Azure para el ajuste automático tendrán efecto de la manera siguiente:
>
> - Los nuevos valores predeterminados de Azure serán FORCE_LAST_GOOD_PLAN = habilitado, CREATE_INDEX = deshabilitado y DROP_INDEX = deshabilitado.
> - Los servidores existentes que no tengan preferencias de ajuste automático se configurarán automáticamente para HEREDAR los nuevos valores predeterminados de Azure. Esto se aplica a todos los clientes que tienen actualmente la configuración del servidor para el ajuste automático en un estado indefinido.
> - Los servidores creados se configurarán automáticamente para HEREDAR con los nuevos valores predeterminados de Azure (a diferencia de antes en que la configuración de ajuste automático se encontraba en un estado indefinido tras la creación de un servidor).

El método recomendado para configurar el ajuste automático es configurar las opciones de ajuste automático en un servidor y heredar la configuración de las bases de datos que pertenecen al servidor primario. De ese modo, resulta más fácil administrar las opciones de ajuste automático en un gran número de bases de datos.

Para aprender a crear notificaciones por correo electrónico para las recomendaciones de ajuste automático, consulte [Notificaciones por correo electrónico para el ajuste automático](automatic-tuning-email-notifications-configure.md).

### <a name="automatic-tuning-for-azure-sql-managed-instance"></a>Ajuste automático con Instancia administrada de Azure SQL

El ajuste automático con Instancia administrada de SQL solo admite **FORCE LAST GOOD PLAN**. Para más información sobre la configuración de las opciones de ajuste automático mediante T-SQL, consulte [El ajuste automático introduce la corrección automática del plan](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management/) y [Corrección automática del plan](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la inteligencia integrada que se usa en el ajuste automático, consulte [La inteligencia artificial ajusta Azure SQL Database](https://azure.microsoft.com/blog/artificial-intelligence-tunes-azure-sql-databases/).
- Para obtener más información sobre cómo funciona el ajuste automático de forma interna, consulte [Indexación automática de millones de bases de datos en Microsoft Azure SQL Database](https://www.microsoft.com/research/uploads/prod/2019/02/autoindexing_azuredb.pdf).