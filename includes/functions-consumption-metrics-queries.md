---
ms.openlocfilehash: 0bae8946b7d81fbf45698cefe7c8b00bbc94d00c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92168124"
---
#### <a name="determine-memory-usage"></a>Determinación del uso de memoria 

En **Supervisión**, seleccione **Registros (Analytics)** , copie la siguiente consulta de telemetría y péguela en la ventana de consulta y seleccione **Ejecutar**. Esta consulta devuelve el uso de memoria total en cada tiempo muestreado.

```
performanceCounters
| where name == "Private Bytes"
| project timestamp, name, value
```

Los resultados deben tener un aspecto similar al ejemplo siguiente:

| marca de tiempo \[UTC\]          | name          | value       |
|----------------------------|---------------|-------------|
| 9/12/2019, 1:05:14\.947 AM | Bytes privados | 209,932,288 |
| 9/12/2019, 1:06:14\.994 AM | Bytes privados | 212 189 184 |
| 9/12/2019, 1:06:30\.010 AM | Bytes privados | 231,714,816 |
| 9/12/2019, 1:07:15\.040 AM | Bytes privados | 210,591,744 |
| 9/12/2019, 1:12:16\.285 AM | Bytes privados | 216,285,184 |
| 9/12/2019, 1:12:31\.376 AM | Bytes privados | 235,806,720 |

#### <a name="determine-duration"></a>Determinación de la duración 

Azure Monitor realiza un seguimiento de las métricas en el nivel de recursos, que para en el caso de Functions es la aplicación de funciones. La integración de Application Insights emite métricas por función. Esta es una consulta de análisis de ejemplo para obtener la duración media de una función:

```
customMetrics
| where name contains "Duration"
| extend averageDuration = valueSum / valueCount
| summarize averageDurationMilliseconds=avg(averageDuration) by name
```

| name                       | averageDurationMilliseconds |
|----------------------------|-----------------------------|
| QueueTrigger AvgDurationMs | 16\.087                     |
| QueueTrigger MaxDurationMs | 90\.249                     |
| QueueTrigger MinDurationMs | 8\.522                      |
