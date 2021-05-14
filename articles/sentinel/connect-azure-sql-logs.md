---
title: Conexión de todos los registros de auditoría y diagnóstico de bases de datos de Azure SQL a Azure Sentinel
description: Aprenda a usar Azure Policy para exigir la conexión de los registros de auditoría de seguridad y diagnóstico de bases de datos de Azure SQL a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 04/21/2021
ms.author: yelevin
ms.openlocfilehash: ba4cefaca7225f25076efa5cdcb81de46aa5cd60
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891327"
---
# <a name="connect-azure-sql-database-diagnostics-and-auditing-logs"></a>Conexión de los registros de auditoría y diagnóstico de bases de datos de Azure SQL

Azure SQL es un motor de base de datos de plataforma como servicio (PaaS) totalmente administrado que se encarga de la mayoría de las funciones de administración de bases de datos, como actualización, aplicación de revisiones, creación de copias de seguridad y supervisión, sin la intervención del usuario. 

El conector de Azure SQL Database permite transmitir registros de auditoría y diagnóstico de bases de datos a Azure Sentinel, para poder supervisar continuamente la actividad en todas las instancias.

- La conexión de registros de diagnóstico permite enviar registros de diagnóstico de base de datos de diferentes tipos de datos al área de trabajo de Azure Sentinel.

- La conexión de registros de auditoría permite transmitir los registros de auditoría de seguridad de todas las bases de datos de Azure SQL en el nivel de servidor.

Más información sobre la [telemetría de diagnóstico de Azure SQL Database](../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md) y sobre la [auditoría de servidores de Azure SQL](../azure-sql/database/auditing-overview.md).

## <a name="prerequisites"></a>Requisitos previos

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

- Para conectar los registros de auditoría, debe tener permisos de lectura y escritura para la configuración de auditoría de Azure SQL Server.

- Para usar Azure Policy con el fin de aplicar una directiva de streaming de registro a recursos de servidor y base de datos de Azure SQL, debe tener el rol de propietario para el ámbito de asignación de directivas.

## <a name="connect-to-azure-sql-database"></a>Conexión a una base de datos de Azure SQL

Este conector usa Azure Policy para aplicar una única configuración de streaming de registro de Azure SQL a una colección de instancias, definida como un ámbito. El conector de Azure SQL Database envía dos tipos de registros a Azure Sentinel: registros de diagnóstico (de bases de datos SQL) y registros de auditoría (en el nivel de SQL Server). Puede ver los tipos de registro ingeridos de las bases de datos y los servidores de Azure SQL en el lado izquierdo de la página del conector, en **Data types** (Tipos de datos).

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. Seleccione **Azure SQL Database** en la galería de conectores de datos y, luego, elija **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. En la sección **Configuración** de la página del conector, tenga en cuenta las dos categorías de registros que puede conectar.

### <a name="connect-diagnostics-logs"></a>Conexión de registros de diagnóstico

1. Expanda **Stream diagnostics logs from your Azure SQL databases at scale** (Transmitir registros de diagnóstico desde la base de datos de Azure SQL a gran escala).

1. Seleccione el botón **Launch Azure Policy Assignment wizard** (Iniciar el asistente para asignación de Azure Policy).

    Se abre el asistente para asignación de directivas, preparado para crear una directiva denominada **Deploy - Configure diagnostic settings for Azure Key Vault to Log Analytics workspace** (Implementar: configurar valores de diagnóstico de Azure Key Vault para el área de trabajo de Log Analytics).

    1. En la pestaña **Básico**, haga clic en el botón con los tres puntos debajo de **Ámbito** para seleccionar la suscripción (y, opcionalmente, un grupo de recursos). También puede agregar una descripción.

    1. En la pestaña **Parámetros**, deje las dos primeras opciones como están. Elija el área de trabajo de Azure Sentinel en la lista desplegable **Área de trabajo de Log Analytics**. Los campos desplegables restantes representan los tipos de registro de diagnóstico disponibles. Deje marcado como "True" todos los tipos de registro que quiera ingerir.

    1. La directiva se aplicará a los recursos agregados en el futuro. Para aplicar la directiva también a los recursos existentes, seleccione la pestaña **Corrección** y marque la casilla **Crear una tarea de corrección**.

    1. En la pestaña **Revisar y crear**, haga clic en **Crear**. La directiva se asigna ahora al ámbito elegido.

### <a name="connect-audit-logs"></a>Conexión de los registros de auditoría

1. De nuevo en la página del conector, expanda **Stream auditing logs from your Azure SQL databases at the server level at scale** (Transmitir registros de auditoría desde las bases de datos de Azure SQL en el nivel de servidor a gran escala).

1. Seleccione el botón **Launch Azure Policy Assignment wizard** (Iniciar el asistente para asignación de Azure Policy).

    Se abre el asistente para asignación de directivas, preparado para crear una directiva denominada **Deploy - Configure auditing settings for Azure Key Vault to Log Analytics workspace** (Implementar: configurar valores de auditoría de Azure Key Vault para el área de trabajo de Log Analytics).

    1. En la pestaña **Básico**, haga clic en el botón con los tres puntos debajo de **Ámbito** para seleccionar la suscripción (y, opcionalmente, un grupo de recursos). También puede agregar una descripción.

    1. En la pestaña **Parámetros**, elija el área de trabajo de Azure Sentinel en la lista desplegable **Área de trabajo de Log Analytics**. Deje la opción **Efecto**.

    1. La directiva se aplicará a los recursos agregados en el futuro. Para aplicar la directiva también a los recursos existentes, seleccione la pestaña **Corrección** y marque la casilla **Crear una tarea de corrección**.

    1. En la pestaña **Revisar y crear**, haga clic en **Crear**. La directiva se asigna ahora al ámbito elegido.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) mostrarán el estado *Conectado* (verde) solo si los datos se han ingerido en algún momento en los últimos 14 días. Transcurrido este período sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a usar Azure Policy para conectar los registros diagnóstico y auditoría de bases de datos de Azure SQL a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
