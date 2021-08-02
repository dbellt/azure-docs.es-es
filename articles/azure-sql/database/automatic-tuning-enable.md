---
title: Habilitación del ajuste automático
description: Puede habilitar fácilmente el ajuste automático en su base de datos mediante Azure Portal.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: NikaKinska
ms.author: nnikolic
ms.reviewer: mathoma, wiassaf
ms.date: 03/03/2021
ms.openlocfilehash: 77dfee73496dda4bf1f6f3616fe9ac5846e6766c
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591753"
---
# <a name="enable-automatic-tuning-in-the-azure-portal-to-monitor-queries-and-improve-workload-performance"></a>Habilitación del ajuste automático en Azure Portal para supervisar las consultas y mejorar el rendimiento de las cargas de trabajo
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database administra automáticamente los servicios de datos que supervisan constantemente las consultas, e identifica la acción que puede realizar para mejorar el rendimiento de la carga de trabajo. Puede revisar las recomendaciones y aplicarlas manualmente o dejar que Azure SQL Database aplique automáticamente acciones correctoras, lo que se conoce como **modo de ajuste automático**.

El ajuste automático se puede habilitar en el nivel de servidor o de base de datos mediante:

- [Azure Portal](automatic-tuning-enable.md#azure-portal)
- Llamadas a la [API de REST](automatic-tuning-enable.md#rest-api)
- Comandos de [T-SQL](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true)

> [!NOTE]
> Para Azure SQL Managed Instance, la opción admitida FORCE_LAST_GOOD_PLAN se puede configurar mediante [T-SQL](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management). La configuración basada en Azure Portal y las opciones de ajuste automático de índices que se describen en este artículo no se aplican a Instancia administrada de Azure SQL.

> [!NOTE]
> En este momento no se admite la configuración de opciones de ajuste automático a través de la plantilla de ARM (Azure Resource Manager).

## <a name="enable-automatic-tuning-on-server"></a>Habilitación del ajuste automático en servidor

En el nivel de servidor, puede optar por heredar la configuración de ajuste automático de "Valores predeterminados de Azure" o no heredarla. Los valores predeterminados de Azure son FORCE_LAST_GOOD_PLAN (habilitado), CREATE_INDEX (deshabilitado) y DROP_INDEX (deshabilitado).

> [!IMPORTANT]
> A partir de marzo de 2020, los nuevos valores predeterminados de Azure para el ajuste automático son los siguientes:
>
> - FORCE_LAST_GOOD_PLAN = habilitado, CREATE_INDEX = deshabilitado y DROP_INDEX = deshabilitado.
> - Los servidores existentes que no tengan configuradas preferencias de ajuste automático se configurarán automáticamente para HEREDAR los valores predeterminados de Azure. Esto se aplica a todos los clientes que tienen actualmente la configuración del servidor para el ajuste automático en un estado indefinido.
> - Los nuevos servidores creados se configurarán automáticamente para HEREDAR los valores predeterminados de Azure (a diferencia de antes, cuando la configuración de ajuste automático se encontraba en un estado indefinido tras la creación de un servidor).

### <a name="azure-portal"></a>Azure portal

Para habilitar el ajuste automático en un [servidor](logical-servers.md) en Azure SQL Database, vaya al servidor en Azure Portal y seleccione **Ajuste automático** en el menú.

![Captura de pantalla que muestra el ajuste automático en Azure Portal, donde puede aplicar opciones para un servidor.](./media/automatic-tuning-enable/server.png)

> [!NOTE]
> Tenga en cuenta que, en este momento, la opción **DROP_INDEX** no es compatible con las aplicaciones que usan sugerencias de índice y la conmutación de particiones, y no debe estar habilitada en estos casos. No se admite la eliminación de índices sin usar para los niveles de servicio Prémium y Crítico para la empresa.

Seleccione las opciones de ajuste automático que quiere habilitar y haga clic en **Aplicar**.

Las opciones de ajuste automático de un servidor se aplican a todas las bases de datos que contiene. De forma predeterminada, todas las bases de datos heredan la configuración de su servidor primario; sin embargo, puede invalidar esta opción y configurar cada base de datos individualmente.

### <a name="rest-api"></a>API DE REST

Para obtener más información sobre el uso de la API de REST para habilitar el ajuste automático en un **servidor**, consulte los [métodos HTTP UPDATE y GET de Ajuste automático de servidores](/rest/api/sql/serverautomatictuning).

## <a name="enable-automatic-tuning-on-an-individual-database"></a>Habilitar el ajuste automático en una base de datos individual

Azure SQL Database le permite especificar individualmente la configuración de ajuste automático en cada base de datos. En el nivel de base de datos, puede decidir si desea heredar la configuración de ajuste automático del servidor primario, si desea heredarla de los valores predeterminados de Azure o si prefiere no heredarla. Los valores predeterminados de Azure son FORCE_LAST_GOOD_PLAN (habilitado), CREATE_INDEX (deshabilitado) y DROP_INDEX (deshabilitado).

> [!TIP]
> La recomendación general es administrar la configuración de ajuste automático en el **nivel de servidor**, de forma que se pueda aplicar la misma configuración en todas las bases de datos automáticamente. Solo debe configurar el ajuste automático en una base de datos específica si necesita que esa base de datos tenga una configuración distinta a la configuración heredada del mismo servidor.

### <a name="azure-portal"></a>Azure portal

Para habilitar el ajuste automático en una **base de datos única**, vaya a la base de datos en Azure Portal y seleccione **Ajuste automático**.

El ajuste automático se puede configurar por separado en cada base de datos. Puede configurar manualmente la opción de ajuste automático o especificar una opción que herede la configuración del servidor.

![Captura de pantalla que muestra el ajuste automático en Azure Portal, donde puede aplicar opciones para una sola base de datos.](./media/automatic-tuning-enable/database.png)

Tenga en cuenta que, en este momento, la opción DROP_INDEX no es compatible con las aplicaciones que usan sugerencias de índice y la conmutación de particiones, y no debe estar habilitada en estos casos.

Cuando haya seleccionado la configuración que desee, haga clic en **Aplicar**.

### <a name="rest-api"></a>API de REST

Para obtener más información sobre el uso de la API de REST para habilitar el ajuste automático en una base de datos única, consulte los [métodos HTTP UPDATE y GET de Ajuste automático de Azure SQL Database](/rest/api/sql/databaseautomatictuning).

### <a name="t-sql"></a>T-SQL

Para habilitar el ajuste automático en una sola base de datos mediante T-SQL, conéctese a la base de datos y ejecute la consulta siguiente:

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM
```

Si establece el ajuste automático en AUTO, se aplicarán los valores predeterminados de Azure. Si se establece en INHERIT, se heredará la configuración de ajuste automático del servidor primario. Si elige CUSTOM, deberá configurar manualmente el ajuste automático.

Para configurar las opciones individuales de ajuste automático mediante T-SQL, conéctese a la base de datos y ejecute una consulta como esta:

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON, CREATE_INDEX = ON, DROP_INDEX = OFF)
```

Si establece la opción de ajuste individual en ACTIVADO, invalidará todas las configuraciones que la base de datos haya heredado y habilitará la opción de ajuste. Si la establece en DESACTIVADO, también invalidará todas las configuraciones que la base de datos haya heredado y deshabilitará la opción de ajuste. Las opciones de ajuste automático para las que se especifique DEFAULT heredarán la configuración de ajuste automático de la opción de nivel de servidor.  

> [!IMPORTANT]
> En caso de la [replicación geográfica activa](auto-failover-group-overview.md), el ajuste automático solo debe configurarse en la base de datos principal. Las acciones de ajuste aplicadas automáticamente, como la creación o la eliminación de un índice, se replicarán de forma automática en la base de datos secundaria de solo lectura. Al intentar habilitar el ajuste automático mediante T-SQL en la base de datos secundaria de solo lectura, se producirá un error ya que no se puede tener una configuración de ajuste diferente en la base de datos secundaria de solo lectura.
>

Para obtener más información sobre las opciones de T-SQL para configurar el ajuste automático, consulte [Opciones de ALTER DATABASE SET (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="automated-recommendation-management-is-disabled"></a>La administración automatizada de recomendaciones está deshabilitada

En el caso de los mensajes de error de que se ha deshabilitado la administración automatizada de recomendaciones, o simplemente de que se ha deshabilitado por el sistema, las causas más comunes son:
- El Almacén de consultas no está habilitado o
- El Almacén de consultas está en modo de solo lectura para una base de datos especificada, o
- El Almacén de consultas dejó de ejecutarse porque usaba el espacio de almacenamiento asignado.

Se pueden tener en cuenta los siguientes pasos para corregir este problema:
- Limpie el Almacén de consultas o modifique el período de retención de datos a "auto" mediante T-SQL. Consulte cómo [configurar la directiva de captura y retención recomendadas para el Almacén de consultas](./query-performance-insight-use.md#recommended-retention-and-capture-policy).
- Use SQL Server Management Studio (SSMS) y siga estos pasos:
  - Conéctese a Azure SQL Database.
  - Haga clic con el botón secundario en la base de datos.
  - Vaya a Propiedades y haga clic en Almacén de consultas.
  - Cambie el modo de operación a lectura y escritura.
  - Cambie el modo de captura del almacén a automático.
  - Cambie el modo de limpieza basada en el tamaño a automático.

### <a name="permissions"></a>Permisos

Dado que el ajuste automático es una característica de Azure, para poder utilizarla, necesitará emplear los roles integrados en Azure. Usar solo la autenticación de SQL no será suficiente para usar la característica de Azure Portal.

Para usar el ajuste automático, el permiso mínimo necesario que se debe conceder al usuario es el rol integrado de Azure de [Colaborador de SQL Database](../../role-based-access-control/built-in-roles.md#sql-db-contributor). También puede considerar el uso de roles con privilegios más elevados, como Colaborador de SQL Server, Colaborador de Instancia administrada de SQL, Colaborador y Propietario.

## <a name="configure-automatic-tuning-e-mail-notifications"></a>Configuración de notificaciones por correo electrónico para el ajuste automático

Para recibir notificaciones automatizadas por correo electrónico sobre las recomendaciones realizadas por el ajuste automático, consulte la guía sobre [notificaciones por correo electrónico del ajuste automático](automatic-tuning-email-notifications-configure.md).

## <a name="next-steps"></a>Pasos siguientes

- Para aprender más sobre el ajuste automático y cómo puede ayudarle a mejorar su rendimiento, lea el [artículo sobre este tema](automatic-tuning-overview.md).
- Consulte [Recomendaciones de rendimiento](database-advisor-implement-performance-recommendations.md) para ver información general sobre las recomendaciones de rendimiento de Azure SQL Database.
- Consulte [Query Performance Insight](query-performance-insight-use.md) para más información sobre el impacto en el rendimiento de las principales consultas.
