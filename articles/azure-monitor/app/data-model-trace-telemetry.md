---
title: 'Modelo de datos de Azure Application Insights: telemetría de seguimiento'
description: Modelo de datos de Application Insights para la telemetría de seguimientos
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.openlocfilehash: cb298812b9970882c4bb3c7d2562bb5cbbf69b2d
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110098909"
---
# <a name="trace-telemetry-application-insights-data-model"></a>Telemetría de seguimientos: modelo de datos de Application Insights

La telemetría de seguimientos (en [Application Insights](./app-insights-overview.md)) representa instrucciones de seguimiento de estilo `printf` en las que se pueden realizar búsquedas de texto. `Log4Net`, `NLog` y las demás entradas de archivo de registro basadas en texto se convierten a instancias de este tipo. El seguimiento no tiene medidas como una extensibilidad.

## <a name="message"></a>Message

Mensaje de seguimiento.

Longitud máxima: 32 768 caracteres

## <a name="severity-level"></a>Nivel de gravedad

El nivel de gravedad del seguimiento. El valor puede ser `Verbose`, `Information`, `Warning`, `Error`, `Critical`.

## <a name="custom-properties"></a>Propiedades personalizadas

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Exploración de los registros de seguimiento de .NET en Application Insights](./asp-net-trace-logs.md).
- [Exploración de los registros de seguimiento de Java en Application Insights](java-2x-trace-logs.md).
- Consulte [modelo de datos](data-model.md) para los tipos y el modelo de datos de Application Insights.
- [Escritura de telemetría de seguimiento personalizada](./api-custom-events-metrics.md#tracktrace)
- Consulte las [plataformas](./platforms.md) compatibles con Application Insights.

