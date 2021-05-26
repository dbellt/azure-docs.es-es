---
title: Consultas de registro en Azure Monitor
description: Información de referencia del lenguaje de consulta de Kusto que utiliza Azure Monitor. Incluye elementos adicionales específicos de Azure Monitor y elementos no admitidos en las consultas del registro de Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/09/2020
ms.openlocfilehash: cb4f074cb9487c22fc340b2e7b2941cd7b69ca98
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459059"
---
# <a name="log-queries-in-azure-monitor"></a>Consultas de registro en Azure Monitor
Los registros de Azure Monitor se basan en Azure Data Explorer y las consultas de registros se escriben con el mismo lenguaje de consulta de Kusto (KQL). Se trata de un lenguaje enriquecido que se ha diseñado para ser fácil de leer y de crear, por lo que debería poder empezar a escribir consultas con las instrucciones básicas.

Las áreas de Azure Monitor donde usará las consultas incluyen lo siguiente:

- [Log Analytics](../logs/log-analytics-overview.md). Esta es la herramienta principal en Azure Portal para escribir consultas de registros y analizar sus resultados de forma interactiva. Incluso si piensa usar una consulta de registro en otro lugar que no sea Azure Monitor, normalmente la escribirá y probará en Log Analytics antes de copiarla a su ubicación final.
- [Reglas de alertas de registro](../alerts/alerts-overview.md). Estas reglas identifican de manera proactiva los problemas de datos del área de trabajo.  Cada regla de alertas se basa en una consulta de registros que se ejecuta automáticamente a intervalos regulares.  Los resultados se inspeccionan para determinar si se debe crear una alerta.
- [Libros](../visualize/workbooks-overview.md). Incluyen los resultados de las consultas de registros mediante diferentes visualizaciones de informes visuales interactivos en Azure Portal.
- [Paneles de Azure](../visualize/tutorial-logs-dashboards.md). Con ellos puede anclar los resultados de cualquier consulta en un panel de Azure, que le permitirá visualizar los datos de registros y métricas en conjunto y, opcionalmente, compartirlos con otros usuarios de Azure.
- [Logic Apps](../logs/logicapp-flow-connector.md).  Puede usar los resultados de una consulta de registro en un flujo de trabajo automatizado mediante Logic Apps.
- [PowerShell](/powershell/module/az.operationalinsights/get-azoperationalinsightssearchresult). Use los resultados de una consulta de registro en un script de PowerShell desde una línea de comandos o un runbook de Azure Automation que use Get-AzOperationalInsightsSearchResults.
- [API de registros de Azure Monitor](https://dev.loganalytics.io). Recupere los datos de registro del área de trabajo desde cualquier cliente de API de REST.  La solicitud de API incluye una consulta que se ejecuta en Azure Monitor para determinar los datos que se van a recuperar.

## <a name="getting-started"></a>Introducción
La mejor manera de empezar a aprender a escribir consultas de registros mediante KQL es aprovechar los tutoriales y ejemplos disponibles.

- [Tutorial de Log Analytics](./log-analytics-tutorial.md): tutorial sobre el uso de las características de Log Analytics, que es la herramienta que utilizará en Azure Portal para editar y ejecutar consultas. También permite escribir consultas simples sin tener que trabajar directamente con el lenguaje de la consulta. Si no ha usado Log Analytics antes, empiece aquí para aprender a usar esta herramienta con otros tutoriales y ejemplos.
- [Tutorial de KQL](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor): guía de los conceptos básicos de KQL y los operadores comunes. Este es el mejor lugar para comenzar a usar el lenguaje en si, y la estructura de las consultas de registro. 
- [Consultas de ejemplo](../logs/queries.md): descripción de las consultas de ejemplo disponibles en Log Analytics. Puede usar las consultas sin modificaciones o utilizarlas como ejemplos para aprender a usar KQL.
- [Ejemplos de consultas](/azure/data-explorer/kusto/query/samples?pivots=azuremonitor): consultas de ejemplo que ilustran una variedad de conceptos diferentes.



## <a name="reference-documentation"></a>Documentación de referencia
La [documentación de KQL](/azure/data-explorer/kusto/query/), incluidas las referencias a todos los comandos y operadores, está disponible en la documentación de Azure Data Explorer. A medida que vaya dominando KQL, también seguirá usando regularmente la referencia para investigar nuevos comandos y escenarios que no haya usado antes.


## <a name="language-differences"></a>Diferencias entre los lenguajes
Aunque Azure Monitor usa la misma instancia de KQL que Azure Data Explorer, existen algunas diferencias. La documentación de KQL especificará los operadores que no sean compatibles con Azure Monitor o que tengan una funcionalidad diferente. Los operadores específicos de Azure Monitor se detallan en el contenido de Azure Monitor. Asimismo, en las secciones siguientes se ofrece una lista de las diferencias entre las versiones del lenguaje, para obtener una referencia rápida.

### <a name="statements-not-supported-in-azure-monitor"></a>Instrucciones que no se admiten en Azure Monitor

* [Alias](/azure/kusto/query/aliasstatement)
* [Parámetros de consulta](/azure/kusto/query/queryparametersstatement)

### <a name="functions-not-supported-in-azure-monitor"></a>Funciones que no se admiten en Azure Monitor

* [cluster()](/azure/kusto/query/clusterfunction)
* [cursor_after()](/azure/kusto/query/cursorafterfunction)
* [cursor_before_or_at()](/azure/kusto/query/cursorbeforeoratfunction)
* [cursor_current(), current_cursor()](/azure/kusto/query/cursorcurrent)
* [database()](/azure/kusto/query/databasefunction)
* [current_principal()](/azure/kusto/query/current-principalfunction)
* [extent_id()](/azure/kusto/query/extentidfunction)
* [extent_tags()](/azure/kusto/query/extenttagsfunction)

### <a name="operators-not-supported-in-azure-monitor"></a>Operadores que no se admiten en Azure Monitor

* [Cross-Cluster Join](/azure/kusto/query/joincrosscluster)

### <a name="plugins-not-supported-in-azure-monitor"></a>Complementos que no se admiten en Azure Monitor

* [Complemento de Python](/azure/kusto/query/pythonplugin)
* [sql_request plugin](/azure/kusto/query/sqlrequestplugin)


### <a name="additional-operators-in-azure-monitor"></a>Operadores adicionales de Azure Monitor
Los operadores siguientes admiten características específicas de Azure Monitor que no están disponibles fuera de Azure Monitor.

* [app()](../logs/app-expression.md)
* [resource()](./resource-expression.md)
* [workspace()](../logs/workspace-expression.md)

## <a name="next-steps"></a>Pasos siguientes
- Realice un [tutorial sobre cómo escribir consultas](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor).
- Acceda a la [documentación de referencia completa del lenguaje de consulta de Kusto](/azure/kusto/query/).