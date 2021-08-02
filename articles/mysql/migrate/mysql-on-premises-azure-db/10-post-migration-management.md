---
title: 'Guía de migración de MySQL local a Azure Database for MySQL: administración posterior a la migración'
description: Una vez completada correctamente la migración, en la siguiente fase administrará los nuevos recursos de carga de trabajo de datos basados en la nube.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: 85a30571491f08adee55d2c0f19641eb838b69e8
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082910"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-post-migration-management"></a>Guía de migración de MySQL local a Azure Database for MySQL: administración posterior a la migración

## <a name="prerequisites"></a>Prerrequisitos

[Migración de datos con MySQL Workbench](09-data-migration-with-mysql-workbench.md)

## <a name="monitoring-and-alerts"></a>Supervisión y alertas

Una vez completada correctamente la migración, en la siguiente fase administrará los nuevos recursos de carga de trabajo de datos basados en la nube. Las operaciones de administración incluyen actividades del plano de control y del plano de datos. Las actividades del plano de control son las relacionadas con los recursos de Azure frente al plano de datos, que se encuentra **dentro** del recurso de Azure (en este caso, MySQL).

Azure Database for MySQL proporciona la capacidad de supervisar ambos tipos de actividades operativas mediante herramientas basadas en Azure, como [Azure Monitor,](../../../azure-monitor/overview.md) [Log Analytics](../../../azure-monitor/logs/design-logs-deployment.md) y [Azure Sentinel.](../../../sentinel/overview.md) Además de las herramientas basadas en Azure, también se pueden configurar sistemas de administración de eventos e información de seguridad (SIEM) para consumir estos registros.

Sea cual sea la herramienta que se utilice para supervisar las nuevas cargas de trabajo basadas en la nube, es necesario crear alertas para advertir a los administradores de Azure y de las bases de datos de cualquier actividad sospechosa. Si un evento de alerta determinado tiene una ruta de corrección bien definida, las alertas pueden activar [libros de ejecuciones de Azure](/azure/automation/automation-quickstart-create-runbook) para solucionar el evento.

El primer paso para crear un entorno totalmente supervisado es permitir que los datos de registro de MySQL fluyan a Azure Monitor. Consulte [Configuración de los registros de auditoría de Azure Database for MySQL y acceso a ellos en Azure Portal](../../howto-configure-audit-logs-portal.md) para más información.

Una vez que fluyan los datos de registro, use el [lenguaje de consulta de Kusto (KQL)](/azure/data-explorer/kusto/query/) para consultar la diversa información de registro. Los administradores que no están familiarizados con KQL pueden encontrar una hoja de referencia rápida [aquí](/azure/data-explorer/kusto/query/sqlcheatsheet) o en la página [Introducción a las consultas de registro en Azure Monitor](../../../azure-monitor/logs/get-started-queries.md).

Por ejemplo, para obtener el uso de memoria de Azure Database for MySQL:

```
AzureMetrics
| where TimeGenerated \> ago(15m)
| limit 10
| where ResourceProvider == "MICROSOFT.DBFORMYSQL"
| where MetricName == "memory\_percent"
| project TimeGenerated, Total, Maximum, Minimum, TimeGrain, UnitName 
| top 1 by TimeGenerated
```
Para obtener el uso de CPU:

```
AzureMetrics
| where TimeGenerated \> ago(15m)
| limit 10
| where ResourceProvider == "MICROSOFT.DBFORMYSQL"
| where MetricName == "cpu\_percent"
| project TimeGenerated, Total, Maximum, Minimum, TimeGrain, UnitName 
| top 1 by TimeGenerated
```
Una vez que haya creado la consulta KQL, cree [alertas de registro](/azure/azure-monitor/platform/alerts-unified-log) basadas en estas consultas.

## <a name="server-parameters"></a>Parámetros del servidor

Como parte de la migración, es probable que los [parámetros de servidor](/azure/mysql/concepts-server-parameters) locales se hayan modificado para admitir una salida rápida. Además, se han realizado modificaciones en los parámetros de Azure Database for MySQL para admitir una entrada rápida. Los parámetros de servidor de Azure se deben volver a establecer en sus valores originales optimizados para cargas de trabajo locales después de la migración.

Sin embargo, asegúrese de revisar y realizar cambios en los parámetros de servidor que sean adecuados para la carga de trabajo y el entorno. Algunos valores que eran buenos para un entorno local pueden no serlo para un entorno basado en la nube. Además, al planear la migración de los parámetros locales actuales a Azure, compruebe que resulta factible establecerlos.

Algunos parámetros no se pueden modificar en Azure Database for MySQL.

## <a name="powershell-module"></a>Módulo de PowerShell

Azure Portal y Windows PowerShell pueden usarse para administrar Azure Database for MySQL. Para empezar a trabajar con PowerShell, instale los cmdlets de Azure PowerShell para MySQL con el siguiente comando de PowerShell:

`Install-Module -Name Az.MySql`

Una vez instalados los módulos, consulte tutoriales como los siguientes para aprender formas de aprovechar las ventajas de crear scripts de las actividades de administración:

  - [Tutorial: Diseño de una instancia de Azure Database for MySQL mediante PowerShell](../../tutorial-design-database-using-powershell.md)

  - [Copia de seguridad y restauración de un servidor de Azure Database for MySQL mediante PowerShell](../../howto-restore-server-powershell.md)

  - [Configuración de parámetros del servidor en Azure Database for MySQL con PowerShell](../../howto-configure-server-parameters-using-powershell.md)

  - [Aumento automático del almacenamiento en el servidor de Azure Database for MySQL mediante PowerShell](../../howto-auto-grow-storage-powershell.md)

  - [Creación y administración de réplicas de lectura en Azure Database for MySQL mediante PowerShell](../../howto-read-replicas-powershell.md)

  - [Reinicio de un servidor de Azure Database for MySQL mediante PowerShell](../../howto-restart-server-powershell.md)

## <a name="azure-database-for-mysql-upgrade-process"></a>Proceso de actualización de Azure Database for MySQL

Puesto que Azure Database for MySQL es una oferta PaaS, los administradores no son responsables de la administración de las actualizaciones del sistema operativo ni del software de MySQL. Sin embargo, es importante tener en cuenta que el proceso de actualización puede ser aleatorio y, cuando se implementa, puede detener las cargas de trabajo del servidor MySQL. Planee estos tiempos de inactividad mediante cambiando la ruta de las cargas de trabajo a una réplica de lectura en caso de que la instancia concreta entre en modo de mantenimiento.

> [!NOTE]
> Este estilo de arquitectura de conmutación por error puede requerir cambios en la capa de datos de las aplicaciones para admitir este tipo de escenario de conmutación por error. Si la réplica de lectura se mantiene como tal y no se promueve, la aplicación solo puede leer datos y se puede producir un error cuando una operación intente escribir información en la base de datos.

La característica [Notificación de mantenimiento planeado](/azure/mysql/concepts-monitoring#planned-maintenance-notification) informa a los propietarios de recursos hasta 72 horas antes de la instalación de una actualización o revisión de seguridad crítica. Es posible que los administradores de bases de datos deban notificar a los usuarios de la aplicación el mantenimiento planeado y no planeado.

> [!NOTE]
> Las notificaciones de mantenimiento de Azure Database for MySQL son increíblemente importantes. El mantenimiento de la base de datos puede poner la base de datos y las aplicaciones conectadas fuera de servicio durante un período de tiempo.

## <a name="wwi-scenario"></a>Escenario de WWI

WWI ha decidido usar los registros de actividad de Azure y permitir que el registro de MySQL fluya a un [área de trabajo de Log Analytics](../../../azure-monitor/logs/design-logs-deployment.md). Este área de trabajo está configurado para formar parte de [Azure Sentinel](../../../sentinel/index.yml) de forma tal que los eventos de [Threat Analytics](/azure/mysql/concepts-data-access-and-security-threat-protection) saldrán a la superficie y se crearán incidentes.

Los administradores de base de datos de MySQL instalaron los [cmdlets de Azure PowerShell de Azure Database for MySQL](/azure/mysql/quickstart-create-mysql-server-database-using-azure-powershell) para que la administración del servidor de MySQL se automatizara en lugar de tener que iniciar sesión en Azure Portal cada vez.

## <a name="management-checklist"></a>Lista de comprobación de administración

  - Cree alertas de recursos para elementos comunes, como CPU y memoria.

  - Asegúrese de que los parámetros de servidor están configurados para la carga de trabajo de datos de destino después de la migración.

  - Cree un script para las tareas administrativas comunes.

  - Configure notificaciones para los eventos de mantenimiento, como actualizaciones y revisiones. Informe a los usuarios cuando sea necesario.  


> [!div class="nextstepaction"]
> [Optimización](./11-optimization.md)