---
title: Conexión de los registros de diagnóstico de Azure Kubernetes Service (AKS) a Azure Sentinel
description: Obtenga información sobre cómo usar Azure Policy para conectar los registros de diagnóstico de Azure Kubernetes Service a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 04/22/2021
ms.author: yelevin
ms.openlocfilehash: f1ef860f1b84de84c42996a7523af8ce174d5981
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107890805"
---
# <a name="connect-azure-kubernetes-service-diagnostics-logs"></a>Conexión de los registros de diagnóstico de Azure Kubernetes Service

Azure Kubernetes Service (AKS) es un servicio de orquestación de contenedores de código abierto y totalmente administrado que le permite implementar, escalar y administrar contenedores de Docker y aplicaciones basadas en contenedores en un entorno de clústeres.

Este conector permite transmitir los registros de diagnóstico de Azure Kubernetes Service (AKS) a Azure Sentinel, para poder supervisar continuamente la actividad en todas las instancias. 

Más información sobre la [supervisión de Azure Kubernetes Service](../azure-monitor/containers/container-insights-overview.md) y sobre la [telemetría de diagnóstico de AKS](../aks/view-control-plane-logs.md).

## <a name="prerequisites"></a>Requisitos previos

Para ingerir los registros de AKS en Azure Sentinel:

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

- Para usar Azure Policy con el objeto de aplicar una directiva de streaming de registro a recursos de AKS, debe tener asignado el rol de propietario para el ámbito de asignación de directivas.

## <a name="connect-to-azure-kubernetes-service"></a>Conexión a Azure Kubernetes Service

Este conector usa Azure Policy para aplicar una única configuración de streaming de registro de Azure Kubernetes Service a una colección de recursos, definida como un ámbito. Puede ver los tipos de registro ingeridos desde Azure Kubernetes Service en el lado izquierdo de la página del conector, en **Data types** (Tipos de datos).

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. Seleccione **Azure Kubernetes Service (AKS)** en la galería de conectores de datos y, a continuación, seleccione **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. En la sección **Configuración** de la página del conector, expanda **Stream diagnostics logs from your Azure Kubernetes Service (AKS) at scale** (Transmitir registros de diagnóstico desde Azure Kubernetes Service [AKS] a gran escala).

1. Seleccione el botón **Launch Azure Policy Assignment wizard** (Iniciar el asistente para asignación de Azure Policy).

    Se abre el asistente para asignación de directiva, preparado para crear una nueva directiva llamada **Deploy - Configure diagnostic settings for Azure Kubernetes Service to Log Analytics workspace** (Implementar: configurar opciones de diagnóstico para Azure Kubernetes Service en el área de trabajo de Log Analytics).

    1. En la pestaña **Básico**, haga clic en el botón con los tres puntos debajo de **Ámbito** para seleccionar la suscripción (y, opcionalmente, un grupo de recursos). También puede agregar una descripción.

    1. En la pestaña **Parámetros**, deje los campos **Efecto** y **Nombre de configuración** tal y como están. Elija el área de trabajo de Azure Sentinel en la lista desplegable **Área de trabajo de Log Analytics**. Los campos desplegables restantes representan los tipos de registro de diagnóstico disponibles. Deje marcado como "True" todos los tipos de registro que quiera ingerir.

    1. La directiva se aplicará a los recursos agregados en el futuro. Para aplicar la directiva también a los recursos existentes, seleccione la pestaña **Corrección** y marque la casilla **Crear una tarea de corrección**.

    1. En la pestaña **Revisar y crear**, haga clic en **Crear**. La directiva se asigna ahora al ámbito elegido.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) mostrarán el estado *Conectado* (verde) solo si los datos se han ingerido en algún momento en los últimos 14 días. Transcurrido este período sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a usar Azure Policy para conectar Azure Kubernetes Service a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
