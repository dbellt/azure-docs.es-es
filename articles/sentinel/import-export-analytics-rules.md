---
title: Importación y exportación de reglas de análisis de Azure Sentinel | Microsoft Docs
description: Exportación de reglas de análisis a plantillas de ARM y su importación desde ellas para facilitar la implementación
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
ms.date: 05/30/2021
ms.author: yelevin
ms.openlocfilehash: b5c9b2d6a586d440218cf4740155bc9fa3bdd3d7
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111372767"
---
# <a name="export-and-import-analytics-rules-to-and-from-arm-templates"></a>Exportación de reglas de análisis a plantillas de ARM y su importación desde

> [!IMPORTANT]
>
> - La exportación e importación de reglas se encuentra en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="introduction"></a>Introducción

Ahora puede exportar reglas de análisis a archivos de plantilla de Azure Resource Manager (ARM) e importar reglas de estos archivos, como parte de la administración y el control de las implementaciones de Azure Sentinel como código. La acción de exportación creará un archivo JSON (denominado *Azure_Sentinel_analytic_rule.json*) en la ubicación de descargas del explorador, que luego puede cambiar el nombre, mover y controlar de otra manera, como cualquier otro archivo.

El archivo JSON exportado es independiente del área de trabajo, por lo que se puede importar a otras áreas de trabajo e incluso a otros inquilinos. Como código, también se puede controlar, actualizar e implementar en un marco de CI/CD administrado.

El archivo incluye todos los parámetros definidos en la regla de análisis, por lo que para las reglas **Programadas** incluye la consulta subyacente y la configuración de programación que lo acompaña, la gravedad, la creación de incidentes, la configuración de la agrupación de eventos y alertas, las tácticas de MITRE ATT&CK asignadas, etc. Cualquier tipo de regla de análisis, no solo **Programada** se puede exportar a un archivo JSON.

## <a name="export-rules"></a>Exportación de reglas

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. Seleccione la regla que desea exportar y haga clic en **Exportar** en la barra de la parte superior de la pantalla.

    :::image type="content" source="./media/import-export-analytics-rules/export-rule.png" alt-text="Exportación de regla de análisis" lightbox="./media/import-export-analytics-rules/export-rule.png":::

    > [!NOTE]
    > - Para seleccionar varias reglas de análisis a la vez para la exportación, marque las casillas situadas junto a las reglas y haga clic en **Exportar** al final.
    >
    > - Puede exportar todas las reglas de una sola página de la cuadrícula de presentación a la vez, marcando la casilla de la fila de encabezado (junto a **GRAVEDAD**) antes de hacer clic en **Exportar**. Sin embargo, no se pueden exportar más reglas que las de una página a la vez.
    >
    > - Tenga en cuenta que, en este escenario, se creará un único archivo (denominado *Azure_Sentinel_analytic_ **rules**.json)* y contendrá código JSON para todas las reglas exportadas.

## <a name="import-rules"></a>Importación de reglas

1. Tenga listo un archivo JSON de plantilla de ARM de reglas de análisis.

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. Haga clic en **Importar** en la barra de la parte superior de la pantalla. En el cuadro de diálogo resultante, vaya al archivo JSON que representa la regla que quiere importar y selecciónelo; a continuación, seleccione **Abrir**.

    :::image type="content" source="./media/import-export-analytics-rules/import-rule.png" alt-text="Importación de regla de análisis" lightbox="./media/import-export-analytics-rules/import-rule.png":::

    > [!NOTE]
    > Puede importar **hasta 50** reglas de análisis desde un único archivo de plantilla de ARM.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a exportar reglas de análisis a plantillas de ARM e importarlas desde ellas.
- Obtenga más información sobre las [reglas de análisis](tutorial-detect-threats-built-in.md), incluidas las [reglas programadas personalizadas](tutorial-detect-threats-custom.md).
- Más información sobre [plantillas de ARM](../azure-resource-manager/templates/overview.md).
