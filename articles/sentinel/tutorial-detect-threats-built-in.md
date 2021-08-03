---
title: Detección de amenazas con reglas de análisis integradas en Azure Sentinel | Microsoft Docs
description: Aprenda a usar las reglas de detección de amenazas listas para usar, basadas en plantillas integradas, que le avisan cuando ocurre algo sospechoso.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/11/2021
ms.author: yelevin
ms.openlocfilehash: 95158fc5aa9d768e7174bc3c72736614efa7f57c
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111371659"
---
# <a name="tutorial-detect-threats-out-of-the-box"></a>Tutorial: Detección de amenazas integrada

Después de [conectar los orígenes de datos](quickstart-onboard.md) a Azure Sentinel, puede recibir una notificación cuando suceda algo sospechoso. Por este motivo, Azure Sentinel proporciona plantillas integradas predefinidas para ayudarle a crear reglas de detección de amenazas. Estas plantillas fueron diseñadas por un equipo de expertos y analistas en seguridad de Microsoft basándose en amenazas conocidas, vectores de ataque habituales y cadenas de escalado de actividades sospechosas. Las reglas creadas a partir de estas plantillas buscarán automáticamente en el entorno las actividades que parezcan sospechosas. Muchas de las plantillas se pueden personalizar para buscar o filtrar actividades, según sus necesidades. Las alertas generadas por estas reglas crearán incidentes que puede asignar e investigar en su entorno.

Este tutorial ayuda a detectar amenazas con Azure Sentinel:

> [!div class="checklist"]
> * Uso de detecciones de amenazas predefinidas
> * Automatizar las respuestas frente a amenazas

## <a name="about-out-of-the-box-detections"></a>Acerca de las detecciones integradas

Para ver todas las detecciones estándar, vaya a **Análisis** y, después, a **Rule templates** (Plantillas de reglas). Esta pestaña contiene todas las reglas integradas de Azure Sentinel.

   :::image type="content" source="media/tutorial-detect-built-in/view-oob-detections.png" alt-text="Usar las detecciones integradas para encontrar amenazas con Azure Sentinel":::

En las secciones siguientes se describen los tipos de plantillas estándar disponibles:

### <a name="microsoft-security"></a>Seguridad de Microsoft

Las plantillas de seguridad de Microsoft crean automáticamente incidentes de Azure Sentinel a partir de las alertas generadas en otras soluciones de seguridad de Microsoft, en tiempo real. Puede usar las reglas de seguridad de Microsoft como plantilla para crear nuevas reglas con una lógica parecida.

Para más información sobre las reglas de seguridad, consulte [Creación automática de incidentes a partir de alertas de seguridad de Microsoft](create-incidents-from-alerts.md).

### <a name="fusion"></a>Fusión

Basada en la tecnología de fusión, la detección avanzada de ataques de varias fases de Azure Sentinel emplea algoritmos de aprendizaje automático escalables que pueden correlacionar muchas alertas y eventos de baja fidelidad de varios productos con incidentes útiles y de alta fidelidad. La fusión está habilitada de manera predeterminada. Dado que la lógica está oculta y, por lo tanto, no se puede personalizar, solo puede crear una regla con esta plantilla.

> [!IMPORTANT]
> Algunas de las detecciones de la plantilla de reglas de fusión se encuentran actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
>
> Para ver qué detecciones se encuentran en versión preliminar, consulte [Detección avanzada de ataques de varias fases en Azure Sentinel](fusion.md).

Además, el motor de Fusion ahora puede correlacionar las alertas generadas por las [reglas de análisis programadas](#scheduled) con las de otros sistemas, lo da lugar a incidentes de alta fidelidad.

### <a name="machine-learning-ml-behavioral-analytics"></a>Análisis de comportamiento de aprendizaje automático (ML)

Estas plantillas se basan en algoritmos de aprendizaje automático de Microsoft, por lo que no se puede ver la lógica interna de cómo funcionan y cuándo se ejecutan. Dado que la lógica está oculta y, por lo tanto, no se puede personalizar, solo puede crear una regla con cada plantilla de este tipo.

> [!IMPORTANT]
> - Las plantillas de reglas de análisis de comportamiento de aprendizaje automático se encuentran actualmente en **versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
>
> - Mediante la creación y la habilitación de reglas basadas en las plantillas de análisis de comportamiento de ML, **se concede permiso a Microsoft para copiar los datos ingeridos fuera de la geografía del área de trabajo de Azure Sentinel** según sea necesario para el procesamiento por parte de los modelos y motores de aprendizaje automático.

### <a name="anomaly"></a>Anomalía

Las plantillas de reglas de anomalías usan SOC-ML (aprendizaje automático) para detectar tipos específicos de comportamiento anómalo. Cada regla tiene sus propios parámetros y umbrales únicos, adecuados para el comportamiento que se está analizando y, aunque su configuración no se puede cambiar ni ajustar, puede duplicar la regla, cambiar y ajustar el duplicado, ejecutar el duplicado en modo **Vuelo** y el original simultáneamente en modo de **Producción**, comparar los resultados y cambiar el duplicado a **Producción** si y cuando su ajuste sea de su agrado. Obtenga más información sobre [SOC-ML y](soc-ml-anomalies.md) [cómo trabajar con reglas de anomalías](work-with-anomaly-rules.md).

> [!IMPORTANT]
> Las plantillas de regla de anomalías se encuentran actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

### <a name="scheduled"></a>Programado

Las reglas de análisis programado se basan en consultas integradas escritas por expertos de seguridad de Microsoft. Puede ver la lógica de consulta y realizar cambios en ella. Puede usar la plantilla de reglas programadas y personalizar la lógica de consulta y la configuración de programación para crear otras reglas.

Varias plantillas de reglas de análisis programadas nuevas generan alertas correlacionadas por el motor de Fusion con alertas de otros sistemas para generar incidentes de alta fidelidad. Consulte [Detección avanzada de ataques de varias fases en Azure Sentinel](fusion.md#configure-scheduled-analytics-rules-for-fusion-detections).

> [!TIP]
> Las opciones de programación de reglas incluyen configurar la regla para que se ejecute cada número especificado de minutos, horas o días, y el tiempo empezará a correr al habilitar la regla.
>
> Recomendamos tener en cuenta cuándo se activa una regla de análisis nueva o editada para garantizar que las reglas reciban la nueva pila de incidentes a tiempo. Por ejemplo, es posible que desee ejecutar una regla en sincronización con el momento en que sus analistas SOC comienzan su jornada laboral, y activar las reglas en ese momento.
>

## <a name="use-out-of-the-box-detections"></a>Usar detecciones integradas

1. Para usar una plantilla integrada, haga clic en el nombre de la plantilla y, luego, haga clic en el botón **Crear regla** en el panel de detalles para crear una regla activa basada en esa plantilla. Cada plantilla tiene una lista de orígenes de datos necesarios. Al abrir la plantilla, se comprueba automáticamente la disponibilidad de los orígenes de datos. Si hay un problema de disponibilidad, es posible que el botón **Crear regla** esté deshabilitado, o puede que vea una advertencia a tal efecto.

    :::image type="content" source="media/tutorial-detect-built-in/use-built-in-template.png" alt-text="Panel de vista previa de reglas de detección":::

1. Al hacer clic en el botón **Crear regla**, se abre el Asistente para la creación de reglas basado en la plantilla seleccionada. Todos los detalles se rellenan automáticamente y, con las plantillas **Programado** o **Microsoft security** (Seguridad de Microsoft), puede personalizar la lógica y otras configuraciones de reglas para satisfacer mejor sus necesidades específicas. Este proceso puede repetirse para crear reglas adicionales en función de la plantilla integrada. Después de seguir los pasos del Asistente para la creación de reglas hasta el final, habrá terminado de crear una regla basada en la plantilla. Las nuevas reglas aparecerán en la pestaña **Active rules** (Reglas activas).

    Para más información sobre cómo personalizar las reglas en el Asistente para la creación de reglas, consulte [Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas](tutorial-detect-threats-custom.md).

## <a name="export-rules-to-an-arm-template"></a>Exportación de reglas a una plantilla de ARM

Puede [exportar fácilmente la regla a una plantilla de Azure Resource Manager (ARM)](import-export-analytics-rules.md) si desea administrar e implementar las reglas como código. También puede importar reglas de archivos de plantilla para verlos y editarlos en la interfaz de usuario.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a detectar amenazas casos mediante Azure Sentinel.

Para aprender a automatizar las respuestas a las amenazas, consulte [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](tutorial-respond-threats-playbook.md).
