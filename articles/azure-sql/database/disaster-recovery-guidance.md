---
title: Recuperación ante desastres
description: Obtenga información sobre cómo recuperar una base de datos tras un error o una interrupción de un centro de datos regional con las funciones de replicación geográfica activa y restauración geográfica de Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 06/21/2019
ms.openlocfilehash: e79d6f5a07fc7b796319ed4ce3d3530f1b1ca2da
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705520"
---
# <a name="restore-your-azure-sql-database-or-failover-to-a-secondary"></a>Restauración de un instancia de Azure SQL Database o conmutación por error a una base de datos secundaria
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database ofrece las siguientes funcionalidades para recuperarse de un corte en el suministro eléctrico:

- [Replicación geográfica activa](active-geo-replication-overview.md)
- [Grupos de conmutación por error automática](auto-failover-group-overview.md)
- [Restauración geográfica](recovery-using-backups.md#point-in-time-restore)
- [Bases de datos con redundancia de zona](high-availability-sla.md)

Para obtener información sobre los escenarios de continuidad empresarial y sus características, consulte el artículo sobre [continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).

> [!NOTE]
> Si usa bases de datos o grupos de nivel Premium o Crítico para la empresa con redundancia de zona, el proceso de recuperación se automatiza y el resto de este material no se aplica.
>
> Es necesario que tanto la base de datos principal como las secundarias tengan el mismo nivel de servicio. También se recomienda encarecidamente que la base de datos secundaria se cree con el mismo tamaño de proceso (unidades de transacción de base de datos o núcleos virtuales) que la principal. Para obtener más información, consulte [Actualización o degradación como base de datos principal](active-geo-replication-overview.md#upgrading-or-downgrading-primary-database).
>
> Use uno o varios grupos de conmutación por error para administrar la conmutación por error de varias bases de datos.
> Si agrega una relación de replicación geográfica existente al grupo de conmutación por error, asegúrese de que la base de datos geográfica secundaria esté configurada con el mismo nivel de servicio y tamaño de proceso que la principal. Para obtener más información, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](auto-failover-group-overview.md).

## <a name="prepare-for-the-event-of-an-outage"></a>Preparación ante interrupciones

Para que el proceso de recuperación pueda realizarse sin problemas en otra región de datos mediante los grupos de conmutación por error o las copias de seguridad con redundancia geográfica, debe preparar un servidor en otra interrupción de un centro de datos para convertirlo en el nuevo servidor principal si lo considera necesario. También hay que contar con pasos bien definidos que se hayan documentado y probado para garantizar que la recuperación se lleve a cabo correctamente. Estos son algunos de los pasos correspondientes a la fase de preparación:

- Identifique el servidor lógico en otra región que se convertirá en el nuevo servidor principal. Para la restauración geográfica, normalmente suele ser un servidor de la [región asociada](../../best-practices-availability-paired-regions.md) a aquella en donde se encuentre la base de datos. Esto elimina el costo de tráfico adicional durante las operaciones de restauración geográfica.
- Identificar y, de manera opcional, definir las reglas de firewall de IP de nivel de servidor que deben estar activas para que los usuarios accedan a la nueva base de datos principal.
- Determinar cómo va a redirigir a los usuarios al nuevo servidor principal; por ejemplo, cambiar las cadenas de conexión o las entradas DNS.
- Identificar y, de manera opcional, crear los inicios de sesión que deben estar presentes en la base de datos maestra del nuevo servidor principal, así como asegurarse de que tienen los permisos adecuados en la base de datos maestra, si procede. Para obtener más información, consulte [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md)
- Identificar las reglas de alerta que deben actualizarse para que se asignen a la nueva base de datos principal.
- Documentar la configuración de auditoría de la base de datos principal actual.
- Realice [maniobras de recuperación ante desastres](disaster-recovery-drills.md). Para simular una interrupción con la restauración geográfica, puede eliminar o cambiar el nombre de la base de datos de origen para provocar el error de conectividad de la aplicación. Para simular una interrupción mediante los grupos de conmutación por error, puede deshabilitar la aplicación web o la máquina virtual conectada a la base de datos, o bien puede llevar a cabo una conmutación por error en la base de datos para provocar errores de conectividad en la aplicación.

## <a name="when-to-initiate-recovery"></a>Cuándo iniciar la recuperación

La operación de recuperación repercute en la aplicación. Este proceso requiere cambiar el redireccionamiento o la cadena de conexión SQL mediante DNS, y podría provocar la pérdida de datos permanente. Por lo tanto, debe realizarse solo cuando la duración de la interrupción pueda durar más que el objetivo de tiempo de recuperación de la aplicación. Cuando se implementa la aplicación en producción, deberá supervisar con frecuencia el estado de la aplicación. Para ello, use los siguientes puntos de datos para garantizar la recuperación:

1. Error de conectividad permanente de la capa de aplicación en la base de datos.
2. El Portal de Azure muestra una alerta sobre una incidencia en una región con un gran impacto.

> [!NOTE]
> Si utiliza grupos de conmutación por error y realiza conmutaciones automáticas por error, el proceso de recuperación es automático y transparente para la aplicación.

En función de la tolerancia de la aplicación al tiempo de inactividad y de la posible responsabilidad civil, puede considerar las siguientes opciones de recuperación.

Use la opción [Get Recoverable Database](/previous-versions/azure/reference/dn800985(v=azure.100)) (Obtener base de datos recuperable) (*LastAvailableBackupDate*) para obtener el último punto de restauración de replicación geográfica.

## <a name="wait-for-service-recovery"></a>Espera para la recuperación del servicio

Los equipos de Azure trabajan diligentemente para restaurar la disponibilidad del servicio lo antes posible, pero dependiendo de la causa principal pueden tardar horas, o incluso días.  Si la aplicación puede tolerar un tiempo de inactividad considerable, puede esperar hasta que se complete la recuperación. En este caso, no se requieren acciones por su parte. El estado actual del servicio se puede ver en el [panel de estado de los servicios de Azure](https://azure.microsoft.com/status/). Después de la recuperación de la región, se restaura la disponibilidad de la aplicación.

## <a name="fail-over-to-geo-replicated-secondary-server-in-the-failover-group"></a>Conmutación por error en el servidor secundario de una replicación geográfica en el grupo de conmutación por error

Si el tiempo de inactividad de la aplicación da lugar a responsabilidades civiles, le recomendamos que use grupos de conmutación por error. Esto permite que la aplicación restaure rápidamente la disponibilidad en una región diferente en caso de interrupción. Para obtener un tutorial, consulte [Implementación de una base de datos distribuida geográficamente](geo-distributed-application-configure-tutorial.md).

Para restaurar la disponibilidad de las bases de datos, es preciso iniciar la conmutación por error en el servidor secundario mediante uno de los métodos admitidos.

Utilice una de las siguientes guías para realizar la conmutación por error en una base de datos secundaria con replicación geográfica:

- [Conmutar por error un servidor secundario replicado geográficamente mediante Azure Portal](active-geo-replication-configure-portal.md)
- [Conmutar por error el servidor secundario mediante PowerShell](scripts/setup-geodr-and-failover-database-powershell.md)
- [Conmutación por error a un servidor secundario con Transact-SQL (T-SQL)](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true#e-failover-to-a-geo-replication-secondary)

## <a name="recover-using-geo-restore"></a>Recuperación mediante la restauración geográfica

Si el tiempo de inactividad de la aplicación no da lugar a responsabilidades civiles, puede usar la [restauración geográfica](recovery-using-backups.md) como método para recuperar las bases de datos de la aplicación. Dicho método crea una copia de la base de datos a partir de la última copia de seguridad con redundancia geográfica.

## <a name="configure-your-database-after-recovery"></a>Configuración de la base de datos después de realizar la recuperación

Si se usa la restauración geográfica para recuperarse tras una interrupción, hay que asegurarse de que la conectividad con las bases de datos nuevas está configurada correctamente para que se pueda reanudar el funcionamiento normal de la aplicación. Esta es una lista de comprobación de las tareas necesarias para que la producción de la base de datos recuperada esté lista.

### <a name="update-connection-strings"></a>Actualización de cadenas de conexión

Dado que la base de datos recuperada reside en otro servidor, es preciso que actualice la cadena de conexión de la aplicación para que apunte a dicho servidor.

Para obtener más información sobre cómo cambiar las cadenas de conexión, consulte el lenguaje de desarrollo adecuado para la [biblioteca de conexiones](connect-query-content-reference-guide.md#libraries).

### <a name="configure-firewall-rules"></a>Configurar reglas de firewall

Es preciso que se asegure de que las reglas del firewall configuradas tanto en el servidor como en la base de datos coinciden con las configuradas en el servidor principal y en la base de datos principal. Para más información, vea: [Cómo: configurar el firewall (Azure SQL Database)](firewall-configure.md).

### <a name="configure-logins-and-database-users"></a>Configuración de inicios de sesión de y de usuarios de la base de datos

Es preciso asegurarse de que todos los inicios de sesión que usa la aplicación existen en el servidor que hospeda la base de datos recuperada. Para obtener más información, consulte [Configuración de seguridad para Replicación geográfica activa o estándar](active-geo-replication-security-configure.md).

> [!NOTE]
> Debe configurar y probar las reglas de firewall del servidor y los inicios de sesión (y sus permisos) durante una maniobra de recuperación ante desastres. Es posible que estos objetos de nivel de servidor y su configuración no estén disponibles durante la interrupción.

### <a name="setup-telemetry-alerts"></a>Configuración de alertas de telemetría

Debe asegurarse de que la configuración actual de la regla de alertas está actualizada para asignarla a la base de datos recuperada y al otro servidor.

Para obtener más información sobre las reglas de alerta de las bases de datos, consulte [Recibir notificaciones de alerta](../../azure-monitor/alerts/alerts-overview.md) y [Realización de seguimiento del estado](../../service-health/service-notifications.md).

### <a name="enable-auditing"></a>Habilitar auditoría

Si se requiere una auditoría para tener acceso a una base de datos, será preciso habilitar Auditoría tras la recuperación de la base de datos. Para obtener más información, consulte [este artículo sobre la realización de auditorías en las bases de datos](../../azure-sql/database/auditing-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre las copias de seguridad automatizadas de Azure SQL Database, consulte [Copias de seguridad automatizadas de SQL Database](automated-backups-overview.md)
- Para obtener información sobre los escenarios de recuperación y diseño de la continuidad empresarial, consulte el artículo sobre [escenarios de continuidad](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- Si quiere saber cómo utilizar las copias de seguridad automatizadas para procesos de recuperación, consulte [Recover an Azure SQL database using automated database backups](recovery-using-backups.md)