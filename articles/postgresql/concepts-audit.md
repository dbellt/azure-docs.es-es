---
title: 'Registro de auditoría en Azure Database for PostgreSQL: servidor único'
description: Conceptos para el registro de auditoría de pgAudit en Azure Database for PostgreSQL con un único servidor.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 01/28/2020
ms.openlocfilehash: 519105374a2c12807bdac92bfb2b60feaeeb89c2
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108324804"
---
# <a name="audit-logging-in-azure-database-for-postgresql---single-server"></a>Registro de auditoría en Azure Database for PostgreSQL con un único servidor

El registro de auditoría de actividades de base de datos en Azure Database for PostgreSQL con un único servidor está disponible a través de la extensión de auditoría de PostgreSQL: [pgAudit](https://www.pgaudit.org/). pgAudit proporciona un registro de auditoría de objeto o sesión detallado.

> [!NOTE]
> pgAudit se encuentra en versión preliminar en Azure Database for PostgreSQL.
> Esta extensión se puede habilitar en servidores optimizados para memoria y de uso general.

Si desea obtener registros de nivel de recurso de Azure para operaciones como el escalado de procesos y de almacenamiento, consulte el [registro de actividad de Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="usage-considerations"></a>Consideraciones de uso
De forma predeterminada, las instrucciones de registro de pgAudit se emiten junto con las instrucciones de registro normales mediante el uso de la utilidad de registro estándar de Postgres. En Azure Database for PostgreSQL, estos archivos. log se pueden descargar mediante Azure Portal o la CLI. El almacenamiento máximo de la colección de archivos es de 1 GB y cada archivo está disponible durante un máximo de siete días (el valor predeterminado es tres días). Este servicio es una opción de almacenamiento a corto plazo.

Como alternativa, puede configurar todos los registros para que se envíen al almacén de registros de Azure Monitor a fin de analizarlos posteriormente en Log Analytics. Si habilita el registro de recursos de Azure Monitor, los registros se enviarán automáticamente (en formato JSON) a Azure Storage, Event Hubs o a los registros de Azure Monitor, en función de su elección.

Al habilitar pgAudit, se genera un gran volumen de registro en un servidor, lo que afecta al rendimiento y al almacenamiento de registros. Es recomendable usar los registros de Azure Monitor, lo que ofrece opciones de almacenamiento a más largo plazo, así como características de análisis y alertas. Se recomienda desactivar el registro estándar para reducir el impacto sobre el rendimiento del registro adicional:

   1. Establezca el parámetro `logging_collector` en OFF. 
   2. Reinicie el servidor para aplicar este cambio.

Para aprender a configurar el registro en Azure Storage, Event Hubs o los registros de Azure Monitor, consulte la sección de registros de recursos del [artículo de registros de servidor](concepts-server-logs.md).

## <a name="installing-pgaudit"></a>Instalación de pgAudit

Para instalar pgAudit, tendrá que incluirlo en las bibliotecas de carga previa compartidas del servidor. Si cambia el parámetro `shared_preload_libraries` de Postgres deberá reiniciar el servidor para que tenga efecto. Puede cambiar los parámetros mediante [Azure Portal](howto-configure-server-parameters-using-portal.md), la [CLI de Azure](howto-configure-server-parameters-using-cli.md) o [API REST](/rest/api/postgresql/singleserver/configurations/createorupdate).

Mediante [Azure Portal](https://portal.azure.com):

   1. Seleccione su servidor de Azure Database for PostgreSQL.
   2. En la barra lateral, seleccione **Parámetros del servidor**.
   3. Busque el parámetro `shared_preload_libraries`.
   4. Seleccione **pgaudit**.
   5. Reinicie el servidor para aplicar el cambio.

   6. Conéctese al servidor mediante un cliente (como psql) y habilite la extensión pgAudit.
      ```SQL
      CREATE EXTENSION pgaudit;
      ```

> [!TIP]
> Si ve un error, confirme que ha reiniciado el servidor después de guardar `shared_preload_libraries`.

## <a name="pgaudit-settings"></a>Configuración de pgAudit

pgAudit le permite configurar el registro de auditoría de objeto o sesión. [El registro de auditoría de sesión](https://github.com/pgaudit/pgaudit/blob/master/README.md#session-audit-logging) emite registros detallados de las instrucciones ejecutadas. [El registro de auditoría de objetos](https://github.com/pgaudit/pgaudit/blob/master/README.md#object-audit-logging) es una auditoría con un ámbito limitado a relaciones específicas. Puede optar por configurar uno o ambos tipos de registro. 

> [!NOTE]
> La configuración de pgAudit se especifica globalmente y no se puede en el nivel de base de datos o rol.

Una vez que haya [instalado pgAudit](#installing-pgaudit), puede configurar sus parámetros para iniciar el registro. La [documentación de pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#settings) proporciona la definición de cada parámetro. Pruebe primero los parámetros y confirme que obtiene el comportamiento esperado.

> [!NOTE]
> Al establecer `pgaudit.log_client` en ON, se redirigirán los registros a un proceso de cliente (como psql), en lugar de escribirse en el archivo. Por lo general, esta configuración se debe dejar deshabilitada. <br> <br>
> `pgaudit.log_level` solo se habilita cuando `pgaudit.log_client` está activado.

> [!NOTE]
> En Azure Database for PostgreSQL, `pgaudit.log` no se puede establecer con un acceso directo de signo `-` (menos) como se describe en la documentación de pgAudit. Todas las clases de instrucción necesarias (lectura, escritura, etc.) deben especificarse individualmente.

### <a name="audit-log-format"></a>Formato de los registros de auditoría
Cada entrada de auditoría se indica mediante `AUDIT:` cerca del principio de la línea de registro. El formato del resto de la entrada se detalla en la [documentación de pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#format).

Si necesita algún otro campo para satisfacer sus requisitos de auditoría, use el parámetro `log_line_prefix` de Postgres. `log_line_prefix` es una cadena que se genera al principio de cada línea de registro de Postgres. Por ejemplo, la siguiente opción de `log_line_prefix` proporciona la marca de tiempo, el nombre de usuario, el nombre de la base de datos y el identificador de proceso siguiente:

```
t=%m u=%u db=%d pid=[%p]:
```

Para obtener más información acerca de `log_line_prefix`, visite la [documentación de PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-LINE-PREFIX).

### <a name="getting-started"></a>Introducción
Para empezar a trabajar rápidamente, establezca `pgaudit.log` en `WRITE` y abra los registros para revisar la salida. 

## <a name="viewing-audit-logs"></a>Visualización de registros de auditoría
Si usa archivos .log, los registros de auditoría se incluirán en el mismo archivo que los registros de errores de PostgreSQL. Puede descargar archivos de registro de [Azure Portal](howto-configure-server-logs-in-portal.md) o de la [CLI](howto-configure-server-logs-using-cli.md). 

Si usa el registro de recursos de Azure, la forma de acceder a los registros depende del punto de conexión que elija. Si se trata de Azure Storage, consulte el artículo sobre la [cuenta de almacenamiento de registros](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage). Si se trata de Event Hubs, consulte el artículo [Transmisión de los registros de Azure](../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

Si se trata de los registros de Azure Monitor, los registros se envían al área de trabajo seleccionada. Los registros de Postgres usan el modo de recopilación **AzureDiagnostics**, por lo que se pueden consultar desde la tabla AzureDiagnostics. A continuación se describen los campos de la tabla. Obtenga más información acerca de las consultas y las alertas en [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/log-query-overview.md).

Puede usar esta consulta para comenzar. Puede configurar alertas basadas en las consultas.

Buscar todos los registros de Postgres de un servidor determinado del último día
```
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where TimeGenerated > ago(1d) 
| where Message contains "AUDIT:"
```

## <a name="next-steps"></a>Pasos siguientes
- [Obtener información sobre el registro en Azure Database for PostgreSQL](concepts-server-logs.md)
- Aprenda cómo establecer parámetros mediante [Azure Portal](howto-configure-server-parameters-using-portal.md), la [CLI de Azure](howto-configure-server-parameters-using-cli.md) o [API REST](/rest/api/postgresql/singleserver/configurations/createorupdate).
