---
title: 'Registro de auditoría - Azure Database for PostgreSQL: Hiperescala (Citus)'
description: 'Conceptos para el registro de auditoría de pgAudit en Azure Database for PostgreSQL: Hiperescala (Citus).'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 01/29/2021
ms.openlocfilehash: 224f46f16f03f53009c3f7beb55d252c5ce5e1e7
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316162"
---
# <a name="audit-logging-in-azure-database-for-postgresql---hyperscale-citus"></a>Registro de auditoría en Azure Database for PostgreSQL: Hiperescala (Citus)

El registro de auditoría de actividades de base de datos en Azure Database for PostgreSQL: Hiperescala (Citus) está disponible a través de la extensión de auditoría de PostgreSQL: [pgAudit](https://www.pgaudit.org/). pgAudit proporciona un registro de auditoría de objeto o sesión detallado.

> [!IMPORTANT]
> pgAudit se encuentra en versión preliminar en Azure Database for PostgreSQL: Hiperescala (Citus).

Si desea obtener registros de nivel de recurso de Azure para operaciones como el escalado de procesos y de almacenamiento, consulte el [registro de actividad de Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="usage-considerations"></a>Consideraciones de uso
De forma predeterminada, las instrucciones de registro de pgAudit se emiten junto con las instrucciones de registro normales mediante el uso de la utilidad de registro estándar de Postgres. En Azure Database for PostgreSQL: Hiperescala (Citus) puede configurar todos los registros para que se envíen al almacén de registros de Azure Monitor a fin de analizarlos posteriormente en Log Analytics. Si habilita el registro de recursos de Azure Monitor, los registros se enviarán automáticamente (en formato JSON) a Azure Storage, Event Hubs o a los registros de Azure Monitor, en función de su elección.

## <a name="enabling-pgaudit"></a>Habilitación de pgAudit

La extensión pgAudit está preinstalada y habilitada en todos los nodos del grupo de servidores de Hiperescala (Citus). No se requiere ninguna acción para habilitarla.

## <a name="pgaudit-settings"></a>Configuración de pgAudit

pgAudit le permite configurar el registro de auditoría de objeto o sesión. [El registro de auditoría de sesión](https://github.com/pgaudit/pgaudit/blob/master/README.md#session-audit-logging) emite registros detallados de las instrucciones ejecutadas. [El registro de auditoría de objetos](https://github.com/pgaudit/pgaudit/blob/master/README.md#object-audit-logging) es una auditoría con un ámbito limitado a relaciones específicas. Puede optar por configurar uno o ambos tipos de registro. 

> [!NOTE]
> La configuración de pgAudit se especifica globalmente y no en el nivel de base de datos o rol.
>
> Además, la configuración de pgAudit se especifica en cada nodo de un grupo de servidores. Para hacer un cambio en todos los nodos, debe aplicarlo a cada nodo individualmente.

Debe configurar los parámetros de pgAudit para iniciar el registro. La [documentación de pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#settings) proporciona la definición de cada parámetro. Pruebe primero los parámetros y confirme que obtiene el comportamiento esperado.

> [!NOTE]
> Al establecer `pgaudit.log_client` en ON, se redirigirán los registros a un proceso de cliente (como psql), en lugar de escribirse en el archivo. Por lo general, esta configuración se debe dejar deshabilitada. <br> <br>
> `pgaudit.log_level` solo se habilita cuando `pgaudit.log_client` está activado.

> [!NOTE]
> En Azure Database for PostgreSQL: Hiperescala (Citus), `pgaudit.log` no se puede establecer con un acceso directo de signo `-` (menos), tal como se describe en la documentación de pgAudit. Todas las clases de instrucción necesarias (lectura, escritura, etc.) deben especificarse individualmente.

## <a name="audit-log-format"></a>Formato de los registros de auditoría
Cada entrada de auditoría se indica mediante `AUDIT:` cerca del principio de la línea de registro. El formato del resto de la entrada se detalla en la [documentación de pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#format).

## <a name="getting-started"></a>Introducción
Para empezar a trabajar rápidamente, establezca `pgaudit.log` en `WRITE` y abra los registros del servidor para revisar la salida. 

## <a name="viewing-audit-logs"></a>Visualización de registros de auditoría
La forma de acceder a los registros depende del punto de conexión que elija. Si se trata de Azure Storage, consulte el artículo sobre la [cuenta de almacenamiento de registros](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage). Si se trata de Event Hubs, consulte el artículo [Transmisión de los registros de Azure](../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

Si se trata de los registros de Azure Monitor, los registros se envían al área de trabajo seleccionada. Los registros de Postgres usan el modo de recopilación **AzureDiagnostics**, por lo que se pueden consultar desde la tabla AzureDiagnostics. A continuación se describen los campos de la tabla. Obtenga más información acerca de las consultas y las alertas en [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/log-query-overview.md).

Puede usar esta consulta para comenzar. Puede configurar alertas basadas en las consultas.

Buscar todas las entradas de pgAudit en los registros de Postgres de un servidor determinado del último día
```kusto
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where TimeGenerated > ago(1d) 
| where Message contains "AUDIT:"
```

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre cómo configurar el registro en Azure Database for PostgreSQL: Hiperescala (Citus) y cómo acceder a los registros](howto-hyperscale-logging.md)