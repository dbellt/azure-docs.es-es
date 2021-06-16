---
title: Preguntas más frecuentes sobre Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Preguntas más frecuentes sobre el servicio Metrics Advisor.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: mbullwin
ms.openlocfilehash: e14927971b4570464991af93f5cc637c33f5cd1e
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111889461"
---
# <a name="metrics-advisor-frequently-asked-questions"></a>Preguntas más frecuentes sobre Metrics Advisor

### <a name="what-is-the-cost-of-my-instance"></a>¿Cuál es el costo de mi instancia?

Actualmente, no hay ningún costo por usar su instancia durante la versión preliminar.

### <a name="why-cant-i-create-the-resource-the-pricing-tier-is-unavailable-and-it-says-you-have-already-created-1-s0-for-this-subscription"></a>¿Por qué no puedo crear el recurso? El "plan de tarifa" no está disponible y dice "Ya ha creado 1 S0 para esta suscripción".

:::image type="content" source="media/pricing.png" alt-text="Mensaje cuando ya existe un recurso F0":::

Durante la versión preliminar pública, solo se puede crear una instancia de Metrics Advisor por región en una suscripción.

Si ya tiene una instancia creada en la misma región con la misma suscripción, puede probar una región diferente o una suscripción diferente para crear una nueva instancia. También puede eliminar una instancia existente para crear una nueva.

Si ya ha eliminado la instancia existente pero sigue apareciendo el error, espere unos veinte minutos después de eliminar el recurso antes de crear una nueva instancia.

## <a name="basic-concepts"></a>Conceptos básicos

### <a name="what-is-multi-dimensional-time-series-data"></a>¿Qué son los datos de serie temporal multidimensional?

Consulte la definición de [métrica multidimensional](glossary.md#multi-dimensional-metric) en el glosario.

### <a name="how-much-data-is-needed-for-metrics-advisor-to-start-anomaly-detection"></a>¿Cuántos datos se necesitan para que Metrics Advisor inicie la detección de anomalías?

Como mínimo, un punto de datos puede desencadenar la detección de anomalías. Sin embargo, esto no aporta la mejor precisión. Durante la creación de la fuente de distribución de datos, el servicio asumirá una ventana de puntos de datos anteriores utilizando el valor especificado como regla de "espacio de relleno".

Se recomienda tener algunos datos antes de la marca de tiempo en la que desea realizar la detección.
Según la granularidad de los datos, la cantidad de datos recomendada varía como se indica a continuación.

| Granularidad | Importe de datos recomendado para la detección |
| ----------- | ------------------------------------- |
| Menos de 5 minutos | 4 días de datos |
| 5 minutos a 1 día | 28 días de datos |
| Más de 1 día, hasta 31 días | 4 años de datos |
| Superior a 31 días | 48 años de datos |

### <a name="why-metrics-advisor-doesnt-detect-anomalies-from-historical-data"></a>¿Por qué Metrics Advisor no detecta las anomalías de los datos históricos?

Metrics Advisor está diseñado para detectar datos de streaming en vivo. Existe una limitación de la duración máxima de los datos históricos que el servicio analizará y en la que ejecutará la detección de anomalías. Esto significa que solo los puntos de datos después de una marca de tiempo más temprana determinada tendrán resultados de detección de anomalías. La marca de tiempo más temprana depende de la granularidad de los datos.

En función de la granularidad de los datos, las duraciones de los datos históricos que tendrán resultados de detección de anomalías son las siguientes.

| Granularidad | Duración máxima de los datos históricos para la detección de anomalías |
| ----------- | ------------------------------------- |
| Menos de 5 minutos | Tiempo de incorporación: 13 horas |
| De 5 minutos a menos de 1 hora | Tiempo de incorporación: 4 días  |
| De 1 hora a menos de 1 día | Tiempo de incorporación: 14 días  |
| 1 día | Tiempo de incorporación: 28 días  |
| Mayor de 1 día, menos de 31 días | Tiempo de incorporación: 2 años  |
| Superior a 31 días | Tiempo de incorporación: 24 años   |

### <a name="more-concepts-and-technical-terms"></a>Más conceptos y términos técnicos

Vea también el [Glosario](glossary.md) para obtener más información.

###  <a name="how-do-i-write-a-valid-query-for-ingesting-my-data"></a>Cómo escribir una consulta válida para ingerir los datos  

Para que Metrics Advisor ingiera los datos, debe crear una consulta que devuelva las dimensiones de los datos en una sola marca de tiempo. Metrics Advisor ejecutará esta consulta varias veces para obtener los datos de cada marca de tiempo. 

Tenga en cuenta que la consulta debe devolver como máximo un registro para cada combinación de dimensión en una marca de tiempo determinada. Todos los registros devueltos deben tener la misma marca de tiempo. La consulta no debería devolver ningún registro duplicado.

Por ejemplo, imagine que crea la consulta siguiente para una métrica diaria: 
 
`select timestamp, city, category, revenue from sampledata where Timestamp >= @StartTime and Timestamp < dateadd(DAY, 1, @StartTime)`

Asegúrese de usar la granularidad correcta para la serie temporal. Para una métrica por hora, usaría: 

`select timestamp, city, category, revenue from sampledata where Timestamp >= @StartTime and Timestamp < dateadd(hour, 1, @StartTime)`

Tenga en cuenta que estas consultas solo devuelven datos en una única marca de tiempo y contienen todas las combinaciones de dimensiones que debe ingerir Metrics Advisor. 

:::image type="content" source="media/query-result.png" alt-text="Resultado de una consulta con una marca de tiempo" lightbox="media/query-result.png":::


### <a name="how-do-i-detect-spikes--dips-as-anomalies"></a>¿Cómo detecto picos y caídas como anomalías?

Si tiene umbrales estrictos predefinidos, realmente puede establecer de forma manual el "umbral estricto" en las [configuraciones de detección de anomalías](how-tos/configure-metrics.md#anomaly-detection-methods).
Si no hay umbrales, puede usar la "detección inteligente", que se basa en IA. Consulte la [optimización de la configuración de detección](how-tos/configure-metrics.md#tune-the-detecting-configuration) para obtener detalles.

### <a name="how-do-i-detect-inconformity-with-regular-seasonal-patterns-as-anomalies"></a>¿Cómo detecto el incumplimiento con patrones normales (estacionales) como anomalías?

La "detección inteligente" es capaz de aprender el patrón de los datos, incluidos los patrones estacionales. Después, detecta los puntos de datos que no se ajustan a los patrones regulares como anomalías. Consulte la [optimización de la configuración de detección](how-tos/configure-metrics.md#tune-the-detecting-configuration) para obtener detalles.

### <a name="how-do-i-detect-flat-lines-as-anomalies"></a>¿Cómo detecto líneas planas como anomalías?

Si los datos suelen ser bastante inestables y fluctúan mucho, y desea recibir una alerta cuando pasen a ser demasiado estables o incluso se conviertan en una línea plana, el "umbral de cambio" puede configurarse para detectar dichos puntos de datos cuando el cambio sea demasiado pequeño.
Consulte las [configuraciones de detección de anomalías](how-tos/configure-metrics.md#anomaly-detection-methods) para obtener detalles.

### <a name="how-to-set-up-email-settings-and-enable-alerting-by-email"></a>¿Cómo se efectúa la configuración de correo electrónico y se habilitan las alertas por correo electrónico?

1.  Un usuario con privilegios de administrador de la suscripción o de administrador del grupo de recursos debe ir al recurso de Metrics Advisor que creó en Azure Portal y seleccionar la pestaña **Control de acceso (IAM)** . 
2.  Seleccione **Agregar asignación de roles**.
3.  Elija un rol de **administrador de Cognitive Services Metrics Advisor** y seleccione su cuenta como en la imagen siguiente.
4.  Haga clic en botón **Guardar** y, con esto, ya se le habrá agregado correctamente como administrador del recurso de Metrics Advisor. Tenga en cuenta que todas las acciones anteriores las debe realizar el administrador de la suscripción o el administrador del grupo de recursos. 

:::image type="content" source="media/access-control.png" alt-text="Página de menú Control de acceso (IAM) con la opción Agregar asignación de roles seleccionada, seguida de la casilla Assign access to selected user (Asignar acceso al usuario seleccionado) que aparece con un rol de acceso de Administrador de Cognitive Services Metrics Advisor, seguido del botón Guardar seleccionado para mostrar los pasos de búsqueda de un usuario y la incorporación de un nivel determinado de permisos de acceso." lightbox="media/access-control.png":::


5.  Los permisos pueden tardar hasta un minuto en propagarse. Después, seleccione el área de trabajo de Metrics Advisor y seleccione la opción **Configuración de correo electrónico** en el panel de navegación izquierdo. Rellene los elementos necesarios, en particular la información relacionada con SMTP. 
6.  Seleccione **Guardar** y, después, habrá completado la configuración de correo electrónico. Puede crear nuevos enlaces y suscribirse a anomalías de métricas para alertas casi en tiempo real. 

## <a name="advanced-concepts"></a>Conceptos avanzados

### <a name="how-does-metric-advisor-build-an-incident-tree-for-multi-dimensional-metrics"></a>¿Cómo crea Metric Advisor un árbol de incidentes para métricas multidimensionales?

Una métrica se puede dividir en varias series temporales mediante dimensiones. Por ejemplo, la métrica `Response latency` se supervisa en todos los servicios que posee el equipo. La categoría `Service` se podría usar como dimensión para enriquecer la métrica, por lo que se obtiene `Response latency` dividido por `Service1`, `Service2`, y así sucesivamente. Cada servicio se puede implementar en distintas máquinas de varios centros de datos, por lo que la métrica se podría dividir aún más por `Machine` y `Data center`.

|Servicio| Centro de datos| Máquina  | 
|----|------|----------------   |
| S1 |  DC1 |   M1 |
| S1 |  DC1 |   M2 |
| S1 |  DC2 |   M3 |
| S1 |  DC2 |   M4 |
| S2 |  DC1 |   M1 |
| S2 |  DC1 |   M2 |
| S2 |  DC2 |   M5 |
| S2 |  DC2 |   M6 |
| ...|      |      |

A partir del valor total de `Response latency`, se puede explorar en profundidad la métrica por `Service`, `Data center` y `Machine`. Sin embargo, puede que para los propietarios de servicios tenga más sentido usar la ruta `Service` -> `Data center` -> `Machine`, o para los ingenieros de infraestructura la ruta `Data Center` -> `Machine` -> `Service`. Todo depende de los requisitos empresariales individuales de los usuarios. 

En Metric Advisor, los usuarios pueden especificar cualquier ruta que quieran explorar en profundidad o consolidar de un nodo de la topología jerárquica. Más concretamente, la topología jerárquica es un grafo acíclico dirigido en lugar de una estructura de árbol. Hay una topología jerárquica completa que consta de todas las combinaciones posibles de dimensiones, como esta: 

:::image type="content" source="media/dimension-combinations-view.png" alt-text="Diagrama de la topología jerárquica compuesto por varios vértices y bordes interconectados con varias dimensiones etiquetadas como S, DC y M con los números correspondientes comprendidos entre 1 y 6" lightbox="media/dimension-combinations-view.png":::

En teoría, si la dimensión `Service` tiene `Ls` valores distintos, la dimensión `Data center` tiene `Ldc` valores distintos y la dimensión `Machine` tiene `Lm` valores distintos, podría haber entonces combinaciones de las dimensiones `(Ls + 1) * (Ldc + 1) * (Lm + 1)` en la topología jerárquica. 

Sin embargo, normalmente no todas las combinaciones de dimensiones son válidas, lo que puede reducir de forma considerable la complejidad. Actualmente, si los usuarios agregan la métrica por su cuenta, no se limita el número de dimensiones. Si necesita usar la funcionalidad de consolidación que ofrece Metrics Advisor, el número de dimensiones no debería ser superior a 6. Sin embargo, se limita el número de series temporales expandidas por las dimensiones de una métrica a menos de 10 000.

La herramienta de **árbol de incidentes** de la página de diagnósticos solo muestra los nodos en los que se ha detectado una anomalía, no toda la topología. De esta manera, puede centrarse en el problema actual. También puede que no se muestren todas las anomalías dentro de la métrica, sino aquellas más importantes en función de la contribución. Así, se puede averiguar rápidamente el efecto, el ámbito y la ruta de propagación de los datos anómalos. Esto reduce de forma considerable el número de anomalías en las que debemos centrarnos y ayuda a los usuarios a comprender y localizar sus problemas principales. 
 
Por ejemplo, cuando se produce una anomalía en `Service = S2 | Data Center = DC2 | Machine = M5`, la desviación de esta afecta al nodo primario `Service= S2` que también la ha detectado, pero no afecta a todo el centro de datos en `DC2` ni a todos los servicios de `M5`. El árbol de incidentes se crearía como en la siguiente captura de pantalla: la anomalía principal se captura en `Service = S2` y la causa principal podría analizarse en dos rutas que conducen a `Service = S2 | Data Center = DC2 | Machine = M5`.

 :::image type="content" source="media/root-cause-paths.png" alt-text="5 vértices etiquetados con dos rutas distintas conectadas mediante bordes con un nodo común con la etiqueta S2. La anomalía principal se captura en el servicio = S2 y la causa principal se puede analizar mediante las dos rutas que conducen al servicio = S2 | Centro de datos = DC2 | Máquina = M5" lightbox="media/root-cause-paths.png":::.

## <a name="next-steps"></a>Pasos a seguir
- [Introducción a Metrics Advisor](overview.md)
- [Uso del portal web](quickstarts/web-portal.md)