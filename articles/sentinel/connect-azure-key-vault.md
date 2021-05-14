---
title: Conexión de los registros de diagnóstico de Azure Key Vault a Azure Sentinel
description: Obtenga información sobre cómo usar Azure Policy para conectar los registros de diagnóstico de Azure Key Vault a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 04/22/2021
ms.author: yelevin
ms.openlocfilehash: fe86c963ab0ec009b527d2d6138a8af2af9ba68a
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107890841"
---
# <a name="connect-azure-key-vault-diagnostics-logs"></a>Conexión de los registros de diagnóstico de Azure Key Vault

Azure Key Vault es un servicio en la nube para el almacenamiento de los secretos y el acceso a estos de forma segura. Un secreto es todo aquello cuyo acceso desea controlar de forma estricta, como las claves API, las contraseñas, los certificados o las claves criptográficas.

Este conector permite transmitir los registros de diagnóstico de Azure Key Vault a Azure Sentinel, para poder supervisar continuamente la actividad en todas las instancias.

Más información sobre la [supervisión de Azure Key Vault](../azure-monitor/insights/key-vault-insights-overview.md) y sobre la [telemetría de diagnósticos de Azure Key Vault](../key-vault/general/logging.md).

## <a name="prerequisites"></a>Requisitos previos

Para ingerir los registros de Azure Key Vault en Azure Sentinel:

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

- Para usar Azure Policy con el fin de aplicar una directiva de streaming de registro a recursos de Azure Key Vault, debe tener el rol de propietario para el ámbito de asignación de directivas.

## <a name="connect-to-azure-key-vault"></a>Conexión a Azure Key Vault

Este conector usa Azure Policy para aplicar una única configuración de streaming de registro de Azure Key Vault a una colección de instancias, que se define como un ámbito. Puede ver los tipos de registro ingeridos desde Azure Key Vault en el lado izquierdo de la página del conector, en **Tipos de datos**.

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. Seleccione **Azure Key Vault** en la galería de conectores de datos y, luego, seleccione **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. En la sección **Configuración** de la página del conector, expanda **Stream diagnostics logs from your Azure Key Vault at scale** (Transmitir registros de diagnósticos desde Azure Key Vault a escala).

1. Seleccione el botón **Launch Azure Policy Assignment wizard** (Iniciar el asistente para asignación de directiva de Azure).

    Se abre el asistente para asignación de directiva, preparado para crear una nueva directiva denominada **Deploy - Configure diagnostic settings for Azure Key Vault to Log Analytics workspace** (Implementar: configurar valores de diagnóstico de Azure Key Vault para el área de trabajo de Log Analytics).

    1. En la pestaña **Aspectos básicos**, haga clic en el botón con los tres puntos debajo de **Ámbito** para seleccionar la suscripción (y, opcionalmente, un grupo de recursos). También puede agregar una descripción.

    1. En la pestaña **Parámetros,** deje los campos **Efecto** y **Nombre de configuración** tal y como están. Elija el área de trabajo de Azure Sentinel en la lista desplegable **Área de trabajo de Log Analytics**. Los campos desplegables restantes representan los tipos de registro de diagnóstico disponibles. Deje marcado como "True" todos los tipos de registro que quiera ingerir.

    1. La directiva se aplicará a los recursos agregados en el futuro. Para aplicar la directiva también a los recursos existentes, seleccione la pestaña **Corrección** y marque la casilla **Crear una tarea de corrección**.

    1. En la pestaña **Revisar y crear**, haga clic en **Crear**. La directiva se asigna ahora al ámbito elegido.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) se mostrarán como *Conectado* (verde) solo si los datos se han ingerido en algún momento en los últimos catorce días. Transcurridos catorce días sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha obtenido información sobre cómo usar Azure Policy pata conectar Azure Key Vault a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
