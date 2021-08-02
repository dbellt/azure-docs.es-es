---
title: Exposición de detalles personalizados en las alertas de Azure Sentinel | Microsoft Docs
description: Extraiga y exponga detalles de eventos personalizados de alertas en las reglas análisis de Azure Sentinel para obtener información de incidentes mejor y más completa.
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
ms.openlocfilehash: 97cd4d51da70b9b1cca501c7f04a3b0ccfc51305
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112059692"
---
# <a name="surface-custom-event-details-in-alerts-in-azure-sentinel"></a>Exposición de detalles de eventos personalizados de alertas en Azure Sentinel 

> [!IMPORTANT]
>
> - La característica Detalles personalizados está en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="introduction"></a>Introducción

Las [reglas programadas de análisis de consultas](tutorial-detect-threats-custom.md) analizan los **eventos** de los orígenes de datos conectados a Azure Sentinel y generan **alertas** cuando el contenido de estos eventos es significativo desde una perspectiva de seguridad. Estas alertas se analizan, agrupan y filtran a través de los distintos motores de Azure Sentinel y se sintetizan en **incidentes** que garantizan la atención del analista de SOC. Sin embargo, cuando el analista ve el incidente, solo las propiedades de las alertas de componentes son inmediatamente visibles. Para obtener el contenido real (información contenida en los eventos) es necesario investigar un poco.

Con la característica **Detalles personalizados** del **Asistente para reglas de análisis**, puede exponer los datos de eventos en las alertas que se crean a partir de esos eventos, lo que hace que los datos de evento formen parte de las propiedades de la alerta. De hecho, le ofrece visibilidad inmediata del contenido del evento en sus incidentes, lo que le permite evaluar, investigar, extraer conclusiones y responder con mucha mayor velocidad y eficacia.

El procedimiento que se detalla a continuación forma parte del Asistente para crear reglas de análisis. Aquí se trata de forma independiente para abordar el escenario de agregar o cambiar los detalles personalizados en una regla de análisis existente.

## <a name="how-to-surface-custom-event-details"></a>Cómo exponer detalles de eventos personalizados

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. Seleccione una regla de consulta programada y haga clic en **Editar**. O bien, para crear una nueva regla, haga clic en **Crear > Regla de consulta programada** en la parte superior de la pantalla.

1. Haga clic en la pestaña **Establecer la lógica de la regla**.

1. En la sección **Enriquecimiento de alertas (versión preliminar)** , expanda **Detalles personalizados**.

    :::image type="content" source="media/surface-custom-details-in-alerts/alert-enrichment.png" alt-text="Buscar y seleccionar detalles personalizados":::

1. En la sección **Detalles personalizados** ahora expandida, agregue pares clave-valor correspondientes a los detalles que desea mostrar:

    1. En el campo **Clave**, escriba un nombre de su elección, que aparecerá como el nombre del campo en las alertas.

    1. En el campo **Valor**, elija el parámetro de evento que desea exponer en las alertas de la lista desplegable. Esta lista se rellenará con los valores correspondientes a los campos de las tablas que son el asunto de la consulta de regla.
    
        :::image type="content" source="media/surface-custom-details-in-alerts/custom-details.png" alt-text="Agregar detalles personalizados":::

1. Haga clic en **Agregar nuevo** para mostrar más detalles. Repita los últimos pasos para definir pares clave-valor. 

    Si cambia de opinión, o si ha cometido algún error y quiere quitar un detalle personalizado, puede hacer clic en el icono de la papelera que se encuentra junto a la lista desplegable **Valor** de ese detalle.

1. Cuando termine de definir los detalles de configuración, haga clic en la pestaña **Revisar y crear**. Cuando la validación de reglas sea correcta, haga clic en **Guardar**.

    > [!NOTE]
    > 
    > **Límites de servicio**
    > - Puede definir **hasta 20 detalles personalizados** en una sola regla de análisis.
    >
    > - El límite de tamaño de todos los detalles personalizados, colectivamente, es de **2 KB**.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a exponer detalles personalizados en alertas mediante reglas de análisis de Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Obtenga una visión completa de las [reglas de análisis de consultas programadas](tutorial-detect-threats-custom.md).
- Obtenga más información sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md).
