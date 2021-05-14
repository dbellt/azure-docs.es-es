---
title: Varias ramas en un flujo de datos de asignación
description: Replicación de flujos de datos en el flujo de datos de asignación con varias ramas
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 04/16/2021
ms.openlocfilehash: f9f2bf2e2204e6b74bb8a31ac856dbe276a6e983
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588758"
---
# <a name="creating-a-new-branch-in-mapping-data-flow"></a>Creación de una nueva rama en el flujo de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Agregue una nueva rama para realizar varios conjuntos de operaciones y transformaciones en la misma secuencia de datos. La incorporación de una nueva rama resulta útil si quiere utilizar el mismo origen para varios receptores o para autocombinar datos.

Se puede agregar una nueva rana desde la lista de transformación del mismo modo en que se agregan otras transformaciones. La opción **Nueva rama** solo estará disponible como acción cuando exista una transformación después de la transformación que está intentando ramificar.

![Captura de pantalla que muestra la opción New branch (Nueva rama) en el menú Multiple inputs / outputs (Varias entradas y salidas).](media/data-flow/new-branch2.png "Adición de una nueva rama")

En el ejemplo siguiente, el flujo de datos está leyendo los datos de carreras de taxi. Es obligatorio agregar los resultados por día y por proveedor. En lugar de crear dos flujos de datos independientes que leen del mismo origen, se puede agregar una nueva rama. De este modo, ambas agregaciones se pueden ejecutar como parte del mismo flujo de datos. 

![Captura de pantalla que muestra el flujo de datos con dos ramas desde el origen.](media/data-flow/new-branch.png "Adición de una nueva rama")

> [!NOTE]
> Al hacer clic en el signo más (+) para agregar transformaciones al gráfico, solo verá la opción Nueva rama cuando haya bloques de transformación posteriores. Esto se debe a que Nueva rama crea una referencia a la secuencia existente y requiere un procesamiento ascendente adicional para funcionar. Si no ve la opción Nueva rama, agregue primero una columna derivada u otra transformación y, a continuación, vuelva al bloque anterior y verá Nueva rama como opción.

## <a name="next-steps"></a>Pasos siguientes

Después de la bifurcación, es posible que desee utilizar las [transformaciones de flujos de datos](data-flow-transformation-overview.md).
