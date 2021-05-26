---
title: Procedimientos para elegir entre la escalabilidad automática y manual en Azure Cosmos DB
description: Obtenga información sobre cómo elegir entre el rendimiento aprovisionado estándar (manual) y el de escalabilidad automática para la carga de trabajo.
author: deborahc
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: dech
ms.openlocfilehash: 1cf5d2cd191b787b6981c8c54950ad341e9d3dad
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110073673"
---
# <a name="how-to-choose-between-standard-manual-and-autoscale-provisioned-throughput"></a>Cómo elegir entre el rendimiento aprovisionado estándar (manual) y el de escalabilidad automática 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB admite dos tipos u ofertas de rendimiento aprovisionado: estándar (manual) y de escalabilidad automática. Ambos tipos de rendimiento son adecuados para cargas de trabajo críticas que requieren un alto rendimiento y escalado, y están respaldados por los mismos SLA de Azure Cosmos DB en términos de rendimiento, disponibilidad, latencia y coherencia.

En este artículo se describe cómo elegir entre el rendimiento aprovisionado estándar (manual) y el de escalabilidad automática para la carga de trabajo. 

## <a name="overview-of-provisioned-throughput-types"></a>Información general de los tipos de rendimiento aprovisionado
Antes de profundizar en la diferencia entre el tipo estándar (manual) y el de escalabilidad automática, es importante comprender cómo funciona el rendimiento aprovisionado en Azure Cosmos DB. 

Cuando se usa el rendimiento aprovisionado, se establece el rendimiento, medido en unidades de solicitud por segundo (RU/s), necesario para la carga de trabajo. El servicio aprovisiona la capacidad necesaria para los requisitos de rendimiento. Las operaciones de base de datos en el servicio, como lecturas, escrituras y consultas, consumen cierta cantidad de unidades de solicitud (RU). Más información sobre las [unidades de solicitud](request-units.md).

En la tabla siguiente se muestra una comparación de alto nivel entre el rendimiento estándar (manual) y de escalabilidad automática.

|Descripción|Estándar (manual)|Escalado automático|
|-------------|------|-------|
|Idónea para|Cargas de trabajo con un tráfico estable o predecible|Cargas de trabajo con un tráfico variable o impredecible. Consulte [casos de uso de escalabilidad automática](provision-throughput-autoscale.md#use-cases-of-autoscale).|
|Funcionamiento|Se aprovisiona una cantidad establecida de RU/s `T` que es estática a lo largo del tiempo, a menos que la cambie manualmente. Cada segundo, puede usar hasta `T` RU/s de rendimiento. <br/><br/>Por ejemplo, si establece la opción estándar (manual) en 400 RU/s, el rendimiento permanecerá en 400 RU/s.|También puede establecer el valor máximo de RU/s `Tmax` que no quiere que el sistema supere. El sistema escala automáticamente el rendimiento `T` de modo que `0.1* Tmax <= T <= Tmax`. <br/><br/>Por ejemplo, si configura el valor máximo de RU/s de escalabilidad automática en 4000 RU/s, el sistema escalará entre 400 y 4000 RU/s.|
|Cuándo se debe usar|Es recomendable administrar manualmente la capacidad de rendimiento (RU/s) y escalarse.<br/><br/>Tiene un uso elevado y uniforme de RU/s aprovisionadas. De todas las horas de un mes, si establece un valor aprovisionado de RU/s `T` y usa la cantidad completa durante el 66 % de las horas o más, se calcula que ahorrará con la opción de RU/s aprovisionadas estándar (manual).<br/><br/>Esto se basa en una comparación entre la configuración de `T` en la opción estándar (manual) y la misma cantidad `Tmax` en la de escalabilidad automática. |Es recomendable que Azure Cosmos DB administre la capacidad de rendimiento (RU/s) y la escala en función del uso.<br/><br/>Tiene un uso de RU/s variable o difícil de predecir. De todas las horas del mes, si establece el valor de RU/s máximo de escalabilidad automática `Tmax` y usa toda la cantidad `Tmax` durante el 66 % de las horas o menos, se calcula que ahorrará con la escalabilidad automática.<br/><br/>Esto se basa en una comparación al establecer el valor de `Tmax` de escalabilidad automática y la misma cantidad `T` en el rendimiento estándar (manual).|
|Modelo de facturación|La facturación se realiza por hora para las RU/s aprovisionadas, independientemente de las que se consuman.<br/><br/>Ejemplo: <li>Aprovisionamiento de 400 RU/s</li><li>Hora 1: sin solicitudes</li><li>Hora 2: 400 RU/s de solicitudes</li><br/><br/>Para las horas 1 y 2, se le facturarán 400 RU/s con las [tarifas de la opción estándar (manual)](https://azure.microsoft.com/pricing/details/cosmos-db/).|La facturación se realiza por hora con el valor más alto de RU/s al que escaló el sistema durante esa hora. <br/><br/>Ejemplo: <li>Aprovisionar una escalabilidad automática de RU/s máxima de 4000 RU/s (escala entre 400 y 4000 RU/s)</li><li>Hora 1: el sistema se escaló verticalmente hasta el valor de 3500 RU/s</li><li>Hora 2: el sistema se redujo verticalmente al mínimo de 400 RU/s (siempre el 10 % de `Tmax`), debido a la inactividad</li><br/><br/>Se le facturará por 3500 RU/s en la hora 1 y 400 RU/s en la hora 2 con las [tarifas de rendimiento aprovisionado de escalabilidad automática](https://azure.microsoft.com/pricing/details/cosmos-db/). La tasa de escalabilidad automática por RU/s equivale a 1,5 veces la tarifa estándar (manual).
|¿Qué ocurre si se superan las RU/s aprovisionadas?|Las RU/s permanecen estáticas en el valor aprovisionado. Cualquier solicitud que consuma por encima del valor de RU aprovisionado en un segundo tendrá una frecuencia limitada, con una respuesta que recomendará esperar un tiempo antes de volver a intentarlo. Si es necesario, puede aumentar o disminuir manualmente el valor de RU/s.| El sistema escalará las RU/s hasta el valor máximo de escalabilidad automática. Cualquier solicitud que consuma un valor por encima del de RU/s de escalabilidad automática aprovisionado en un segundo tendrá una frecuencia limitada, con una respuesta que recomendará esperar un tiempo antes de volver a intentarlo.|

## <a name="understand-your-traffic-patterns"></a>Comprensión de los patrones de tráfico

### <a name="new-applications"></a>Aplicaciones nuevas ###

Si va a crear una nueva aplicación y no conoce el patrón de tráfico todavía, es recomendable empezar con el valor de punto de entrada de RU/s (o el mínimo) para evitar el aprovisionamiento excesivo al principio. O bien, si tiene una pequeña aplicación que no necesita una gran escala, puede que quiera aprovisionar solo el valor de RU/s del punto de entrada mínimo para optimizar el costo. En el caso de las aplicaciones pequeñas con poco tráfico previsto, también puede tener en cuenta el modo de capacidad [sin servidor](throughput-serverless.md).

Si tiene previsto usar el valor estándar (manual) o la escalabilidad automática, esto es lo que debe tener en cuenta:

Si aprovisiona el valor de RU/s estándar (manual) del punto de entrada de 400 RU/s, no podrá consumir más de 400 RU/s, a menos que cambie manualmente el rendimiento. Se le facturará por 400 RU/s con la tarifa de rendimiento aprovisionado estándar (manual), por hora.

Si aprovisiona el rendimiento de escalabilidad automática en el punto de entrada del máximo de RU/s de 4000 RU/s, el recurso se escalará entre 400 y 4000 RU/s. Dado que la tarifa de facturación de rendimiento de escalabilidad automática por RU/s equivale a 1,5 veces la tarifa estándar (manual), para horas en las que el sistema se haya reducido al mínimo de 400 RU/s, la factura será mayor que si hubiera aprovisionado 400 RU/s manualmente. Sin embargo, con la escalabilidad automática, en cualquier momento, si el tráfico de la aplicación alcanza un pico, puede consumir hasta 4000 RU/s sin intervención del usuario. En general, debe sopesar la ventaja de poder consumir hasta el valor máximo de RU/s en cualquier momento con una tarifa equivalente a 1,5 veces la de escalabilidad automática.

Use la [calculadora de capacidad](estimate-ru-with-capacity-planner.md) de Azure Cosmos DB para calcular los requisitos de rendimiento. 

### <a name="existing-applications"></a>Aplicaciones existentes ###

Si tiene una aplicación existente que usa el rendimiento aprovisionado estándar (manual), puede usar [métricas de Azure Monitor](../azure-monitor/insights/cosmosdb-insights-overview.md) para determinar si el patrón de tráfico es adecuado para la escalabilidad automática. 

En primer lugar, busque la [métrica de consumo de unidad de solicitud normalizada](monitor-normalized-request-units.md#view-the-normalized-request-unit-consumption-metric) de la base de datos o el contenedor. El uso normalizado es una medida de cuánto está usando el rendimiento aprovisionado estándar (manual). Cuanto más se acerque el número al 100 %, más se usarán totalmente las RU/s aprovisionadas. [Más información](monitor-normalized-request-units.md#view-the-normalized-request-unit-consumption-metric) sobre la métrica.

A continuación, determine cómo varía el uso normalizado a lo largo del tiempo. Busque el uso normalizado más alto para cada hora. A continuación, calcule el uso normalizado promedio durante todas las horas. Si observa que el uso promedio es menor que el 66 %, considere la posibilidad de habilitar la escalabilidad automática en la base de datos o el contenedor. Por el contrario, si el uso promedio es mayor que el 66 %, se recomienda permanecer con el rendimiento aprovisionado estándar (manual).

> [!TIP]
> Si su cuenta está configurada para usar escrituras en varias regiones y tiene más de una región, la tarifa por 100 RU/s es la misma para la escalabilidad automática que para la manual. Esto significa que al habilitar la escalabilidad automática no se incurre en ningún costo adicional, independientemente de su uso. Como resultado, siempre se recomienda usar la escalabilidad automática con las escrituras en varias regiones cuando se tiene más de una región, ya que podrá ahorrar al pagar solo por las RU/s a las que la aplicación escala. Si realiza escrituras en varias regiones y tiene una región, utilice el uso promedio para determinar si la escalabilidad automática permitirá ahorrar costos. 

#### <a name="examples"></a>Ejemplos

Echemos un vistazo a dos cargas de trabajo de ejemplo diferentes y analicemos si son adecuadas para el rendimiento de escalabilidad automática o manual. Para ilustrar el enfoque general, analizaremos tres horas de historial a fin de determinar la diferencia de costos entre el uso de la escalabilidad automática y manual. En el caso de las cargas de trabajo de producción, se recomienda usar un historial de entre 7 y 30 días (o más tiempo si está disponible) para establecer un patrón de uso de RU/s.

> [!NOTE]
> Todos los ejemplos que se muestran en este documento se basan en el precio de una cuenta de Azure Cosmos implementada en una región no gubernamental de Estados Unidos. Los precios y el cálculo varían en función de la región que use; consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) de Azure Cosmos DB para obtener más información.

Se supone que:
- Supongamos que actualmente tenemos un rendimiento manual de 30 000 RU/s. 
- Nuestra región está configurada con escrituras en una sola región, con una región. Si tenemos varias regiones, multiplicaremos el costo por hora por el número de regiones.
- Use tarifas de precios públicas para el rendimiento manual (0,008 USD por 100 RU/s por hora) y de escalabilidad automática (0,012 USD por 100 RU/s por hora) en cuentas de escritura en una sola región. Vea la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener más detalles. 

#### <a name="example-1-variable-workload-autoscale-recommended"></a>Ejemplo 1: Carga de trabajo de variable (se recomienda la escalabilidad automática)

En primer lugar, miramos el consumo normalizado de RU. Esta carga de trabajo tiene tráfico variable, con un consumo normalizado de RU que oscila entre el 6 % y el 100 %. Se producen picos ocasionales hasta el 100 % que son difíciles de predecir, pero durante muchas horas el uso es bajo. 

:::image type="content" source="media/how-to-choose-offer/variable-workload_use_autoscale.png" alt-text="Carga de trabajo con tráfico variable: consumo de RU normalizado entre el 6 % y el 100 % durante todo el tiempo":::

Comparemos el costo de aprovisionar un rendimiento manual de 30 000 RU/s frente a establecer un máximo de escalabilidad automática de RU/s en 30 000 (escala entre 3000 y 30 000 RU/s). 

Ahora, analicemos el historial. Supongamos que el uso es el descrito en la tabla siguiente. El uso promedio durante estas tres horas es del 39 %. Dado que el promedio de consumo normalizado de RU es menor que el 66 %, ahorramos si usamos la escalabilidad automática. 

Tenga en cuenta que durante la hora 1, cuando el uso es de un 6 %, la escalabilidad automática facturará RU/s para el 10 % del máximo de RU/s, que es el mínimo por hora. Aunque el costo de la escalabilidad automática puede ser mayor que el del rendimiento manual en determinadas horas, siempre y cuando el uso promedio sea inferior al 66 % durante todas las horas, la escalabilidad automática será más barata por lo general.

|Período de tiempo  | Utilización |RU/s de escalabilidad automática facturadas  |Opción 1: manual con 30 000 RU/s  | Opción 2: escalabilidad automática entre 3000 y 30 000 RU/s |
|---------|---------|---------|---------|---------|
|Hora 1  | 6 %  |     3000  |  30 000 * 0,008/100 = 2,40 USD        |   3000 * 0,012/100 = 0,36 USD      |
|Hora 2  | 100%  |     30,000    |  30 000 * 0,008/100 = 2,40 USD       |  30 000 * 0,012/100 = 3,60 USD      |
|Hora 3 |  11 %  |     3300    |  30 000 * 0,008/100 = 2,40 USD       |    3300 * 0,012/100 = 0,40 USD     |
|**Total**   |  |        |  7,20 USD       |    4,36 USD (39 % de ahorro)    |

#### <a name="example-2-steady-workload-manual-throughput-recommended"></a>Ejemplo 2: Carga de trabajo estable (se recomienda el rendimiento manual)

Esta carga de trabajo tiene tráfico estable, con un consumo normalizado de RU que oscila entre el 72 % y el 100 %. Con 30 000 RU/s aprovisionadas, significa que estamos consumiendo entre 21 600 y 30 000 RU/s.

:::image type="content" source="media/how-to-choose-offer/steady_workload_use_manual_throughput.png" alt-text="Carga de trabajo con tráfico estable: consumo de RU normalizado entre el 72 % y el 100 % durante todo el tiempo":::

Comparemos el costo de aprovisionar un rendimiento manual de 30 000 RU/s frente a establecer un máximo de escalabilidad automática de RU/s en 30 000 (escala entre 3000 y 30 000 RU/s).

Supongamos que el historial de uso es el descrito en la tabla. Nuestro uso promedio durante estas tres horas es del 88 %. Dado que el promedio de consumo normalizado de RU es mayor que el 66 %, ahorramos si usamos el rendimiento manual.

En general, si el uso promedio durante las 730 horas de un mes es superior al 66 %, ahorraremos si usamos el rendimiento manual. 

| Período de tiempo | Utilización |RU/s de escalabilidad automática facturadas  |Opción 1: manual con 30 000 RU/s  | Opción 2: escalabilidad automática entre 3000 y 30 000 RU/s |
|---------|---------|---------|---------|---------|
|Hora 1  | 72 %  |     21 600   |  30 000 * 0,008/100 = 2,40 USD        |   21 600 * 0,012/100 = 2,59 USD      |
|Hora 2  | 93 %  |     28 000    |  30 000 * 0,008/100 = 2,40 USD       |  28 000 * 0,012/100 = 3,36 USD       |
|Hora 3 |  100%  |     30,000    |  30 000 * 0,008/100 = 2,40 USD       |    30 000 * 0,012/100 = 3,60 USD     |
|**Total**   |  |        |  7,20 USD       |    9,55 USD     |

> [!TIP]
> Con el rendimiento estándar (manual), puede usar la métrica de utilización normalizada para calcular las RU/s reales que puede usar si cambia a la escalabilidad automática. Multiplique la utilización normalizada de un momento dado por las RU/s estándar aprovisionadas actualmente (manual). Por ejemplo, si ha aprovisionado 5000 RU/s y la utilización normalizada es del 90 %, el uso de RU/s es 0,9 * 5000 = 4500 RU/s. Si observa que el patrón de tráfico es variable, pero está por encima o por debajo del aprovisionamiento, puede habilitar la escalabilidad automática y, a continuación, cambiar la configuración de máximo de RU/s de escalabilidad automática según corresponda.

#### <a name="how-to-calculate-average-utilization"></a>Procedimientos para calcular el uso promedio
En la escalabilidad automática se factura por la RU/s más alta a la que se ha escalado en una hora. Cuando se analiza el consumo normalizado de RU a lo largo del tiempo, es importante utilizar el uso más alto por hora al calcular el promedio. 

Para calcular el promedio del uso normalizado más alto durante todas las horas:
1. Establezca **Agregación** en la métrica de consumo de normalizado de RU en **Máx.**
1. Seleccione 1 hora para la **Granularidad de tiempo**.
1. Vaya a **Opciones de gráfico**.
1. Seleccione la opción de gráfico de barras. 
1. En **Compartir**, seleccione la opción **Download to Excel** (Descargar en Excel). En la hoja de cálculo generada, calcule el uso promedio durante todas las horas. 

:::image type="content" source="media/how-to-choose-offer/variable-workload-highest-util-by-hour.png" alt-text="Para ver el consumo normalizado de RU por hora, 1) Seleccione una granularidad de tiempo de 1 hora; 2) Edite la configuración del gráfico; 3) Seleccione la opción de gráfico de barras; 4) En Compartir, seleccione la opción descargar en Excel para calcular el promedio durante todas las horas. ":::

## <a name="measure-and-monitor-your-usage"></a>Medición y supervisión del uso
Con el tiempo, después de elegir el tipo de rendimiento, debe supervisar la aplicación y realizar los ajustes necesarios. 

Si usa la escalabilidad automática, use Azure Monitor para ver el máximo de RU/s de escalabilidad automática aprovisionado (**rendimiento máximo de escalabilidad automática**) y el valor de RU/s al que el sistema está escalado actualmente (**rendimiento aprovisionado**). A continuación se muestra un ejemplo de una carga de trabajo variable o imprevisible que usa la escalabilidad automática. Tenga en cuenta que, cuando no hay tráfico, el sistema escala las RU/s al mínimo del 10 % de la cantidad máxima de RU/s, que, en este caso, es de 5000 RU/s y 50 000 RU/s, respectivamente. 

:::image type="content" source="media/how-to-choose-offer/autoscale-metrics-azure-monitor.png" alt-text="Ejemplo de carga de trabajo con escalabilidad automática, con un máximo de escalabilidad automática de RU/s de 50 000 RU/s y un rendimiento que va desde 5000 a 50 000 RU/s":::

> [!NOTE]
> Cuando se usa el rendimiento aprovisionado estándar (manual), la métrica de **rendimiento aprovisionado** hace referencia al valor que ha definido como usuario. Cuando se usa el rendimiento de escalabilidad automática, esta métrica se refiere a las RU/s a que se escala el sistema actualmente.

## <a name="next-steps"></a>Pasos siguientes
* Use la [calculadora de RU](https://cosmos.azure.com/capacitycalculator/) para calcular el rendimiento de las cargas de trabajo nuevas.
* Use [Azure Monitor](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) para supervisar las cargas de trabajo existentes.
* Obtenga más información sobre el [aprovisionamiento del rendimiento de escalabilidad automática en una base de datos o contenedor de Azure Cosmos](how-to-provision-autoscale-throughput.md).
* Revise las [preguntas más frecuentes sobre Escalabilidad automática](autoscale-faq.yml).