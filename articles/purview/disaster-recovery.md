---
title: Recuperación ante desastres para Azure Purview
description: Obtenga información acerca de cómo configurar un entorno de recuperación ante desastres para Azure Purview.
author: sudheerreddykoppula
ms.author: sukoppul
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 04/23/2021
ms.openlocfilehash: 170f72c0076f6614e59c37ba0f06071480726b09
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108021189"
---
# <a name="disaster-recovery-for-purview"></a>Recuperación ante desastres para Purview

En este artículo se explica cómo configurar un entorno de recuperación ante desastres para Azure Purview. Las interrupciones del centro de datos de Azure son poco frecuentes, pero pueden durar desde unos minutos hasta algunas horas. Las interrupciones del centro de datos pueden provocar interrupciones en los entornos en los que se confía para la gobernanza de datos. Al seguir los pasos que se detallan en este artículo, puede seguir administrando los datos en caso de que se produzca una interrupción del centro de datos en la región primaria de la cuenta de Purview.

## <a name="achieve-business-continuity-for-azure-purview"></a>Continuidad empresarial para Azure Purview

La continuidad empresarial y la recuperación ante desastres (BCDR) en una instancia de Azure Purview hacen referencia a los mecanismos, directivas y procedimientos que permiten una su empresa protegerse ante la pérdida de datos y seguir funcionando en caso de interrupción, especialmente en sus niveles de análisis, catálogo e información. En esta página se explica cómo configurar un entorno de recuperación ante desastres para Azure Purview.

En la actualidad, Azure Purview no admite BCDR automatizada. Hasta que se agregue esa compatibilidad, el usuario será responsable de las actividades de copia de seguridad y restauración. Puede crear manualmente una cuenta de Purview secundaria como una instancia de espera semiactiva en otra región.

Los siguientes pasos muestran cómo puede realizar la recuperación ante desastres manualmente:

1. Una vez creada la cuenta de Purview principal en una región determinada, debe aprovisionar una o varias cuentas de Purview secundarias en regiones independientes de Azure Portal. 

2. Todas las actividades realizadas en la cuenta de Purview principal también se deben llevar a cabo en las cuentas de Purview secundarias. Esto incluye: 

    - Mantenimiento de la información de cuenta
    - Creación y mantenimiento de conjuntos de reglas de análisis, clasificaciones y reglas de clasificación personalizadas
    - Registro y análisis de orígenes
    - Creación y mantenimiento de colecciones, junto con la asociación de orígenes con las colecciones
    - Creación y mantenimiento de las credenciales usadas durante el análisis.
    - Mantenimiento de recursos de datos
    - Creación y mantenimiento de términos del glosario


A medida que cree el plan de BCDR manual, tenga en cuenta los siguientes puntos: 

- Se le cobrará por las cuentas de Purview principales y secundarias. 

- Las cuentas de Purview principal y secundaria no se pueden configurar para las mismas cuentas de Azure Data Factory, Azure Data Share y Synapse Analytics, si procede.  Como resultado, el linaje de datos de Azure Data Factory y Azure Data Share no se puede ver en las cuentas de Purview secundarias. Además, el área de trabajo de Synapse Analytics asociada a la cuenta de Purview principal no se puede asociar a las cuentas de Purview secundarias. Esta es una limitación actualmente y se solucionará cuando se admita BCDR automatizada. 

- Los entornos de ejecución de integración son específicos de una cuenta de Purview. Por lo tanto, si los análisis deben ejecutarse en cuentas de Purview principales y secundarias en paralelo, se deben mantener varios entornos de ejecución de integración auto-hospedados. Esta limitación también se solucionará cuando se admita BCDR automatizada. 

- La ejecución en paralelo de análisis de las cuentas de Purview principal y secundaria en el mismo origen puede afectar el rendimiento del origen. Esto puede dar lugar a duraciones de análisis que varían entre las cuentas de Purview.   

## <a name="related-information"></a>Información relacionada

- [Continuidad empresarial y recuperación ante desastres](../best-practices-availability-paired-regions.md)
- [Creación de alta disponibilidad a su estrategia de continuidad empresarial y recuperación ante desastres](/azure/architecture/solution-ideas/articles/build-high-availability-into-your-bcdr-strategy)
- [Estado de Azure](https://status.azure.com/status)

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con Azure Purview, consulte [Creación de una cuenta de Azure Purview](create-catalog-portal.md).
