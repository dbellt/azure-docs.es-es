---
title: Asignación de campos de datos a entidades de Azure Sentinel | Microsoft Docs
description: Asigne campos de datos de tablas a entidades de Azure Sentinel en reglas de análisis para obtener una mejor información sobre los incidentes.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/10/2021
ms.author: yelevin
ms.openlocfilehash: 247ae926dde9b07b1d627621c456333282f00a42
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060082"
---
# <a name="map-data-fields-to-entities-in-azure-sentinel"></a>Asignación de campos de datos a entidades en Azure Sentinel 

> [!IMPORTANT]
>
> - La nueva versión de la característica de asignación de entidades está en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

> [!IMPORTANT]
>
> - Consulte [Notas sobre la nueva versión](#notes-on-the-new-version) al final de este documento para obtener información importante sobre la compatibilidad con versiones anteriores y las diferencias entre las versiones nuevas y antiguas de la asignación de entidades.

## <a name="introduction"></a>Introducción

La asignación de entidades es una parte integral de la configuración de las [reglas de análisis de consultas programadas](tutorial-detect-threats-custom.md). Enriquece la salida de las reglas (alertas e incidentes) con datos esenciales que actúan como bloques de creación de cualquier proceso de investigación y de las acciones de corrección posteriores.

El procedimiento que se detalla a continuación forma parte del asistente para crear reglas de análisis. Aquí se trata de forma independiente para abordar el escenario de agregar o cambiar las asignaciones de entidades de una regla de análisis existente.

## <a name="how-to-map-entities"></a>Asignación de entidades

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. Seleccione una regla de consulta programada y haga clic en **Editar**. O bien, para crear una nueva regla, haga clic en **Crear > Regla de consulta programada** en la parte superior de la pantalla.

1. Haga clic en la pestaña **Establecer la lógica de la regla**. 

1. En la sección **Enriquecimiento de alertas (versión preliminar)** , expanda **Asignación de entidades**.

    :::image type="content" source="media/map-data-fields-to-entities/alert-enrichment.png" alt-text="Expansión de la asignación de entidades":::

1. En la sección **Asignación de entidades**, seleccione un tipo de entidad en la lista desplegable **Tipo de entidad**.

    :::image type="content" source="media/map-data-fields-to-entities/choose-entity-type.png" alt-text="Elegir un tipo de entidad":::

1. Seleccione un **identificador** para la entidad. Los identificadores son atributos de una entidad que puede identificarla de manera suficiente. Elija uno en la lista desplegable **Identificador** y, a continuación, elija un campo de datos en la lista desplegable **Valor** que se corresponda con el identificador. Con algunas excepciones, la lista **Valor** se rellena con los campos de datos de la tabla definida como asunto de la regla de consulta.

    Puede definir **hasta tres identificadores** para una entidad determinada. Algunos identificadores son obligatorios y otros son opcionales. Debe elegir al menos un identificador obligatorio. Si no lo hace, un mensaje de advertencia le indicará qué identificadores son necesarios. Para obtener los mejores resultados para una identificación única máxima, debe usar **identificadores seguros** siempre que sea posible y el uso de varios identificadores seguros permitirá una mayor correlación entre los orígenes de datos. Consulte la lista completa de [entidades e identificadores](entities-reference.md) disponibles.

    :::image type="content" source="media/map-data-fields-to-entities/map-entities.png" alt-text="Asignación de campos a entidades":::

1. Haga clic en **Agregar nueva entidad** para asignar más entidades. Puede asignar **hasta cinco entidades** en una sola regla de análisis. También puede asignar más de una del mismo tipo. Por ejemplo, puede asignar dos entidades de **dirección IP**, una a partir de un campo de *dirección IP de origen* y otra a partir de un campo de *dirección IP de destino*. De este modo, puede realizar el seguimiento de ambas.

    Si cambia de opinión, o si ha cometido algún error, puede quitar una asignación de entidad; para ello, haga clic en el icono de la papelera que se encuentra junto a la lista desplegable Entidad.

1. Cuando termine la asignación de entidades, haga clic en la pestaña **Revisar y crear**. Cuando la validación de la regla sea correcta, haga clic en **Guardar**.

## <a name="notes-on-the-new-version"></a>Notas sobre la nueva versión

- Si previamente definió asignaciones de entidades para esta regla de análisis con la versión anterior, esas asignaciones aparecen en el código de la consulta. Las asignaciones de entidades definidas con la nueva versión **no aparecen en el código de la consulta**. Las reglas de análisis solo admiten una versión de las asignaciones de entidades cada vez y la nueva versión tiene prioridad. Por lo tanto, cualquier asignación que defina aquí hará que se **descarten** **todas las asignaciones definidas** en el código de la consulta cuando se ejecute la consulta. 

- Si todavía necesita usar la **versión anterior** de la asignación de entidades (siempre que la nueva versión esté aún en versión preliminar), todavía puede acceder a ella mediante una marca de características en la dirección URL. Coloque el cursor entre `https://portal.azure.com/` y `#blade` e inserte el texto `?feature.EntityMapping=false`.

  - Se seguirán aplicando los límites de la versión anterior. Solo puede asignar las entidades usuario, host, dirección IP, dirección URL y hash de archivo, y solo una de cada.

  - Debe **quitar** todas las asignaciones de entidades creadas con la nueva versión **antes** de volver a la versión anterior; de lo contrario, las asignaciones de entidades que usen la versión anterior **no funcionarán**.

- Una vez que la nueva versión de la asignación de entidades esté disponible con carácter general, ya no será posible usar la versión anterior. Se recomienda que migre las asignaciones de entidades antiguas a la nueva versión.


## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido cómo asignar campos de datos a entidades en las reglas de análisis de Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Obtenga una visión completa de las [reglas de análisis de consultas programadas](tutorial-detect-threats-custom.md).
- Obtenga más información sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md).
