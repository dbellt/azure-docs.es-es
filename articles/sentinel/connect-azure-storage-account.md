---
title: Conexión de registros de diagnóstico de la cuenta de Azure Storage a Azure Sentinel
description: Aprenda a usar Azure Policy para conectar los registros de diagnóstico de la cuenta de Azure Storage a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 04/21/2021
ms.author: yelevin
ms.openlocfilehash: e9f27b5cc3c4cb526dbcf6e6511fcd53e3c2fe67
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891727"
---
# <a name="connect-azure-storage-account-diagnostics-logs"></a>Conexión de registros de diagnóstico de la cuenta de Azure Storage

> [!IMPORTANT]
> El conector de la cuenta de Azure Storage está actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

La cuenta de Azure Storage es una solución de nube para los escenarios modernos de almacenamiento de datos. Contiene todos los objetos de datos: blobs, archivos, colas, tablas y discos.

Este conector le permite transmitir los registros de diagnóstico de las cuentas de Azure Storage a Azure Sentinel, de forma que puede supervisar continuamente la actividad y detectar amenazas de seguridad en los recursos de almacenamiento de Azure de toda la organización.

Más información sobre la [supervisión de la cuenta de Azure Storage](../storage/common/storage-analytics-logging.md).

## <a name="prerequisites"></a>Prerequisites

Para ingerir registros de diagnóstico de la cuenta de Azure Storage en Azure Sentinel:

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

- Para usar Azure Policy con el fin de aplicar una directiva de streaming de registro a recursos de Azure Storage, debe tener el rol de propietario para el ámbito de asignación de directivas.

## <a name="connect-to-azure-storage-account"></a>Conexión a la cuenta de Azure Storage

Este conector usa Azure Policy para aplicar una única configuración de streaming de registro a una colección de recursos de la cuenta de Azure Storage, definida como un ámbito. Puede ver los tipos de registros y métricas de Azure Storage disponibles en el lado izquierdo de la página del conector, en **Tipos de datos**.

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. Seleccione **Cuenta de Azure Storage** en la galería de conectores de datos y, luego, elija **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. En la sección **Configuración** de la página del conector, expanda **Stream diagnostics logs from your Azure Key Vault at scale** (Transmitir registros de diagnósticos desde Azure Key Vault a gran escala).

1. Seleccione el botón **Launch Azure Policy Assignment wizard** (Iniciar el asistente para asignación de Azure Policy).

    Se abre el asistente para asignación de directivas, preparado para crear una directiva denominada **Configure diagnostic settings for Azure Key Vault to Log Analytics workspace** (Configurar valores de diagnóstico de Azure Key Vault para el área de trabajo de Log Analytics).

    1. En la pestaña **Aspectos básicos**, haga clic en el botón con los tres puntos debajo de **Ámbito** para seleccionar la suscripción (y, opcionalmente, un grupo de recursos). También puede agregar una descripción.

    1. En la pestaña **Parámetros**:
        - Elija el área de trabajo de Azure Sentinel en la lista desplegable **Área de trabajo de Log Analytics**.
        - En la lista desplegable **Storage services to deploy** (Servicios de almacenamiento para implementar), seleccione los tipos de recursos de almacenamiento (archivo, tabla, cola, etc.) en los que quiere implementar la configuración de diagnóstico.
        - Deje los campos **Nombre de configuración** y **Efecto** tal y como están.
        - El resto de campos desplegables representan los tipos de métricas y registros de diagnóstico disponibles. Deje marcado como "True" todos los tipos que quiera ingerir.

    1. Los pasos anteriores aplicarán la directiva a todos los recursos de almacenamiento futuros. Para aplicar la directiva a los recursos existentes, seleccione la pestaña **Corrección** y marque la casilla **Crear una tarea de corrección**.

    1. En la pestaña **Revisar y crear**, haga clic en **Crear**. La directiva se asigna ahora al ámbito elegido.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) mostrarán el estado *Conectado* (verde) solo si los datos se han ingerido en algún momento en los últimos 14 días. Transcurrido este período sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a conectar la cuenta de Azure Storage a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
