---
title: Guardado de una consulta en Azure Monitor Log Analytics (versión preliminar)
description: Describe cómo guardar una consulta en Log Analytics.
ms.subservice: logs
ms.topic: article
author: bwren
ms.author: bwren
ms.date: 05/20/2021
ms.openlocfilehash: 0334fa67849acdf628ec77a55fcf7d7f5435cbf6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110482956"
---
# <a name="save-a-query-in-azure-monitor-log-analytics-preview"></a>Guardado de una consulta en Azure Monitor Log Analytics (versión preliminar)
Las [consultas de registro](log-query-overview.md) son solicitudes en Azure Monitor que le permiten procesar y recuperar datos en un área de trabajo de Log Analytics. Guardar una consulta de registro le permite:

- usar la consulta en todo el contexto de Log Analytics, incluidos el área de trabajo y el contexto basado en el recurso.
- autorizar que otros usuarios ejecuten la misma consulta.
- crear una biblioteca de consultas comunes para su organización.

## <a name="save-options"></a>Opciones de almacenamiento
Al guardar una consulta, se almacena en un paquete de consultas que tiene las siguientes ventajas con respecto al método anterior de almacenamiento de la consulta en un área de trabajo. Guardar en un paquete de consultas es el método preferido y proporciona las siguientes ventajas:

- Detectabilidad más sencilla con la capacidad de filtrar y agrupar consultas por diferentes propiedades.
- Consultas disponibles cuando se usa un ámbito de recursos en Log Analytics.
- Las consultas están disponibles entre suscripciones.
- Más datos disponibles para describir y clasificar la consulta.


## <a name="save-a-query"></a>Almacenamiento de una consulta
Para guardar una consulta en un paquete de consultas, seleccione **Save as Log Analytics Query** (guardar como consulta de Log Analytics) en la lista desplegable **Guardar** de Log Analytics.

[![Menú Guardar consulta](media/save-query/save-query.png)](media/save-query/save-query.png#lightbox)

Cuando guarde una consulta en un paquete de consultas, se mostrará el siguiente cuadro de diálogo, donde podrá proporcionar valores para las propiedades de consulta. Las propiedades de consulta se usan para filtrar y agrupar consultas similares a gin de ayudarle a encontrar la consulta que busca. Consulte [Propiedades de consulta](queries.md#query-properties) para obtener una descripción detallada de cada propiedad.

La mayoría de los usuarios deben elegir la opción **Save to the default query pack** (guardar en el paquete de consultas predeterminado) que guardará la consulta en el [paquete de consultas predeterminado](query-packs.md#default-query-pack). Desactive esta casilla si hay otros paquetes de consulta en su suscripción. Consulte [Paquetes de consultas en registros de Azure Monitor (versión preliminar)](query-packs.md) para obtener más información sobre cómo crear un nuevo paquete de consultas.

[![Cuadro de diálogo Guardar consulta](media/save-query/save-query-dialog.png)](media/save-query/save-query-dialog.png#lightbox)

## <a name="edit-a-query"></a>Edición de una consulta
Quizá quiera editar una consulta que ya guardó. Esto puede ser con el fin de cambiar la consulta o modificar cualquiera de sus propiedades. Después de abrir una consulta existente en Log Analytics, puede editarla al seleccionar **Edit query details** (editar detalles de la consulta) en la lista desplegable **Guardar**. Esto le permitirá guardar la consulta editada con las mismas propiedades, o bien modificar las propiedades antes de guardarlas.

Si quiere guardar la consulta con un nombre diferente, seleccione **Save as Log Analytics Query** (guardar como consulta de Log Analytics) igual que si estuviera creando una nueva consulta. 


## <a name="save-as-a-legacy-query"></a>Guardado como una consulta heredada
No se recomienda guardar como una consulta heredada debido a las ventajas de los paquetes de consultas que se describieron anteriormente. Sin embargo, puede guardar una consulta en el área de trabajo para combinarla con otras consultas que se hayan guardado ahí mismo antes del lanzamiento de los paquetes de consultas. 

Para guardar una consulta heredada, seleccione **Save as Log Analytics Query** (guardar como consulta de Log Analytics) en la lista desplegable **Guardar** de Log Analytics. Elija la opción **Save as Legacy query** (guardar como consulta heredada). La única opción disponible será la de categoría heredada.


## <a name="next-steps"></a>Pasos siguientes

[Introducción a las consultas KQL](get-started-queries.md)
