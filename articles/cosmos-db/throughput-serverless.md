---
title: Elección entre el rendimiento aprovisionado y sin servidor en Azure Cosmos DB
description: Más información acerca de cómo elegir entre el rendimiento aprovisionado y sin servidor para la carga de trabajo.
author: ThomasWeiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/25/2021
ms.openlocfilehash: 057004a7a322505b137c04b76c3407e4c36f6f89
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378204"
---
# <a name="how-to-choose-between-provisioned-throughput-and-serverless"></a>Elección entre el rendimiento aprovisionado y sin servidor
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB admite dos tipos de modos de capacidad: [rendimiento aprovisionado](set-throughput.md) y [sin servidor](serverless.md). Puede realizar las mismas operaciones de base de datos exactas en ambos modos, pero la forma de facturar estas operaciones es radicalmente diferente. En el vídeo siguiente se explican las diferencias principales entre estos modos y cómo se ajustan a los diferentes tipos de cargas de trabajo:

> [!VIDEO https://www.youtube.com/embed/CgYQo6uHyt0]

## <a name="detailed-comparison"></a>Comparación detallada

| Criterios | Rendimiento aprovisionado | Sin servidor |
| --- | --- | --- |
| Idónea para | Cargas de trabajo con tráfico sostenido que requieren un rendimiento predecible | Cargas de trabajo con tráfico intermitente o imprevisible y una baja relación de tráfico de promedio a pico |
| Funcionamiento | Para cada uno de los contenedores, aprovisiona una cantidad de rendimiento expresada en [unidades de solicitud](request-units.md) por segundo. Cada segundo, esta cantidad de unidades de solicitud está disponible para las operaciones de base de datos. El rendimiento aprovisionado se puede actualizar manualmente o ajustar automáticamente con [escalado automático](provision-throughput-autoscale.md). | Las operaciones de base de datos se ejecutan en los contenedores sin tener que aprovisionar ninguna capacidad. |
| Distribución geográfica | Disponible (regiones de Azure ilimitadas) | No disponible (una cuenta sin servidor solo puede ejecutarse en una única región de Azure) |
| Almacenamiento máximo por contenedor | Sin límite | 50 GB |
| Rendimiento | < 10 ms de latencia para las lecturas de punto y escrituras que se incluyen en el Acuerdo de Nivel de Servicio | < 10 ms de latencia para las lecturas de punto y < 30 ms para las escrituras que se incluyen en el Objetivo de nivel de servicio |
| Modelo de facturación | La facturación se realiza por hora para las RU/s aprovisionadas, independientemente de las que se consuman. | La facturación se realiza por horas para la cantidad de RU consumida por las operaciones de base de datos. |

## <a name="estimating-your-expected-consumption"></a>Estimación del consumo esperado

En algunas situaciones, puede que no esté claro si se debe elegir el rendimiento aprovisionado o sin servidor para una carga de trabajo determinada. Para ayudarle a tomar esta decisión, puede calcular el consumo **esperado en general**, que es el número total de RU que puede consumir durante un mes (puede calcular esto con ayuda de la tabla que se muestra [aquí](plan-manage-costs.md#estimating-serverless-costs)).

**Ejemplo 1**: se espera que una carga de trabajo llegue a una ráfaga con un máximo de 500 RU/s y consuma un total de 20 millones de RU durante un mes.

- En el modo de rendimiento aprovisionado, aprovisionaría un contenedor con 500 RU/s por un costo mensual de: 0,008 USD * 5 * 730 = **29,20 USD**
- En el modo sin servidor, pagaría por los RU consumidos: 0,25 USD * 20 = **5,00 USD**

**Ejemplo 2**: se espera que una carga de trabajo llegue a una ráfaga con un máximo de 500 RU/s y consuma un total de 250 millones de RU durante un mes.

- En el modo de rendimiento aprovisionado, aprovisionaría un contenedor con 500 RU/s por un costo mensual de: 0,008 USD * 5 * 730 = **29,20 USD**
- En el modo sin servidor, pagaría por los RU consumidos: 0,25 USD * 250 = **62,50 USD**

(estos ejemplos no tienen en cuenta el costo de almacenamiento, que es el mismo para los dos modos)

> [!NOTE]
> Los costos que se muestran en el ejemplo anterior son solo para fines de demostración. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener la información más reciente sobre los precios.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [rendimiento aprovisionado en Azure Cosmos DB](set-throughput.md)
- Más información sobre [Azure Cosmos DB sin servidor](serverless.md)
- Familiarícese con el concepto de [unidades de solicitud](request-units.md)
