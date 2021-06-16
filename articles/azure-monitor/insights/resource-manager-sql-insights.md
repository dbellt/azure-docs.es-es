---
title: Ejemplos de plantilla de Resource Manager para SQL Insights
description: Plantillas de ejemplo de Azure Resource Manager para implementar y configurar SQL Insights
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 03/25/2021
ms.openlocfilehash: c82a46fa249bd55207e3fc20fbe991c0d7d24bf9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958150"
---
# <a name="resource-manager-template-samples-for-sql-insights"></a>Ejemplos de plantilla de Resource Manager para SQL Insights
En este artículo se incluyen [plantillas de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md) de ejemplo para permitir que SQL Insights supervise SQL en Azure.  Consulte la [documentación de SQL Insights](sql-insights-overview.md) para obtener más información sobre la oferta y las versiones admitidas de SQL. Cada ejemplo incluye un archivo de plantilla y un archivo de parámetros con valores de ejemplo para la plantilla.

[!INCLUDE [azure-monitor-samples](../../../includes/azure-monitor-resource-manager-samples.md)]


## <a name="create-a-sql-insights-monitoring-profile"></a>Creación de un perfil de supervisión de SQL Insights
En el ejemplo siguiente se crea un perfil de supervisión de SQL Insights, que incluye los datos de supervisión de SQL que se van a recopilar y la frecuencia de recopilación de los datos, además de especificar el área de trabajo a la que se enviarán los datos.


### <a name="template-file"></a>Archivo de plantilla

Vea el [archivo de plantilla en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/Create%20new%20profile/CreateNewProfile.armtemplate).

### <a name="parameter-file"></a>Archivo de parámetros

Vea el [archivo de parámetro en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/Create%20new%20profile/CreateNewProfile.parameters.json).


## <a name="add-a-monitoring-vm-to-a-sql-insights-monitoring-profile"></a>Incorporación de una máquina virtual de supervisión a un perfil de supervisión de SQL Insights
Una vez que haya creado un perfil de supervisión, debe asignar máquinas virtuales de Azure que se configurarán para recopilar datos de forma remota de los recursos de SQL que especifique en la configuración de esa máquina virtual.  Consulte la documentación de habilitación de SQL Insights para obtener más detalles.

En el ejemplo siguiente se configura una máquina virtual de supervisión para recopilar datos de los recursos de SQL especificados.


### <a name="template-file"></a>Archivo de plantilla

Vea el [archivo de plantilla en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/Add%20monitoring%20virtual%20machine/AddMonitoringVirtualMachine.armtemplate).

### <a name="parameter-file"></a>Archivo de parámetros

Vea el [archivo de parámetro en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/SQL/Add%20monitoring%20virtual%20machine/AddMonitoringVirtualMachine.parameters.json).


## <a name="create-an-alert-rule-for-sql-insights"></a>Creación de una regla de alerta para SQL Insights
En el ejemplo siguiente se creará una regla de alerta que abarcará los recursos de SQL incluidos en el ámbito del perfil de supervisión especificado.  Esta regla de alerta aparecerá en la interfaz de usuario de SQL Insights en el panel de contexto de la interfaz de usuario de alertas.  

El archivo de parámetro contiene valores de una de las plantillas de alerta suministradas en SQL Insights. Puede modificarlo para alertar sobre otros datos que se recopilan para SQL.  La plantilla no especifica un grupo de acciones para la regla de alerta.


#### <a name="template-file"></a>Archivo de plantilla

Vea el [archivo de plantilla en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/Alerts/log-metric-noag.armtemplate).

### <a name="parameter-file"></a>Archivo de parámetros

Vea el [archivo de parámetro en GitHub](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Workloads/Alerts/sql-cpu-utilization-percent.parameters.json).





## <a name="next-steps"></a>Pasos siguientes

* [Otras plantillas de ejemplo para Azure Monitor](../resource-manager-samples.md).
* [Más información sobre SQL Insights](sql-insights-overview.md).