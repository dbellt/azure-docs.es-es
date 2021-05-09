---
title: Solución Azure Networking Analytics en Azure Monitor | Microsoft Docs
description: Puede usar la solución Azure Networking Analytics en Azure Monitor para revisar los registros de los grupos de seguridad de red de Azure y los registros de Azure Application Gateway.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/21/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b1434a166e715acfdfa1d2510c335cdc42f00c34
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108320176"
---
# <a name="azure-networking-monitoring-solutions-in-azure-monitor"></a>Soluciones de supervisión de redes de Azure en Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Azure Monitor ofrece las siguientes soluciones para supervisar las redes:
* Monitor de rendimiento de red (NPM)
    * Supervisión del mantenimiento de la red
* Azure Application Gateway Analytics para revisar
    * Registros de Azure Application Gateway
    * Métricas de Azure Application Gateway
* Soluciones para supervisar y auditar la actividad de red en la red de la nube
    * [Análisis de tráfico](../../networking/network-monitoring-overview.md#traffic-analytics) 
    * Azure Network Security Group Analytics

## <a name="network-performance-monitor-npm"></a>Network Performance Monitor (NPM) (monitor de rendimiento de red)

La solución de administración [Network Performance Monitor](../../networking/network-monitoring-overview.md) supervisa el mantenimiento, la disponibilidad y accesibilidad de las redes.  Se utiliza para supervisar la conectividad entre:

* Nube pública y entorno local
* Centros de datos y ubicaciones de usuario (sucursales)
* Subredes que hospedan distintos niveles de una aplicación de varios niveles.

Para más información consulte [Network Performance Monitor](../../networking/network-monitoring-overview.md).

## <a name="network-security-group-analytics"></a>Network Security Group Analytics

1. Agregue la solución de administración a Azure Monitor.
2. Habilite los diagnósticos para que se dirijan a un área de trabajo de Log Analytics en Azure Monitor. No se requiere escribir los registros en Azure Blob Storage.

Si los registros de diagnóstico no están habilitados, las hojas del panel de ese recurso están en blanco y muestran un mensaje de error.

## <a name="azure-application-gateway-analytics"></a>Azure Application Gateway Analytics

1. Habilite los diagnósticos para que se dirijan a un área de trabajo de Log Analytics en Azure Monitor.
2. Use el resumen detallado del recurso mediante la plantilla de libro de Application Gateway.

Si los registros de diagnóstico no están habilitados en Application Gateway, solo los datos de métricas predeterminados se rellenarían en el libro.


> [!NOTE]
> En enero de 2017, cambió la forma de enviar registros de Application Gateway y los grupos de seguridad de red a un área de trabajo de Log Analytics. Si consulta la solución **Azure Networking Analytics (en desuso)** , remítase a la sección [Migración desde la solución Networking Analytics antigua](#migrating-from-the-old-networking-analytics-solution) para conocer los pasos que debe seguir.
>
>

## <a name="review-azure-networking-data-collection-details"></a>Revisión de los detalles de recopilación de datos de redes de Azure
Las soluciones de administración Azure Application Gateway Analytics y Network Security Group Analytics recopilan registros de diagnósticos directamente de Azure Application Gateways y Network Security Groups. No es necesario escribir los registros en Azure Blob Storage y no se requiere ningún agente para la recopilación de datos.

En la siguiente tabla se muestran los métodos de recopilación de datos y otros detalles sobre cómo se reúnen los datos análisis de Azure Application Gateway y Network Security Group.

| Plataforma | Agente directo | Agente System Center Operations Manager | Azure | ¿Se requiere Operations Manager? | Se envían los datos del agente de Operations Manager a través del grupo de administración | Frecuencia de recopilación |
| --- | --- | --- | --- | --- | --- | --- |
| Azure |  |  |&#8226; |  |  |Cuando se inicia sesión |


### <a name="enable-azure-application-gateway-diagnostics-in-the-portal"></a>Habilitación de los diagnósticos de Azure Application Gateway en el portal

1. En Azure Portal, navegue hasta el recurso de Application Gateway que se va a supervisar.
2. Seleccione *Configuración de diagnóstico* para abrir la página siguiente.

   ![Captura de pantalla de Configuración de diagnóstico del recurso de Application Gateway.](media/azure-networking-analytics/diagnostic-settings-1.png)

   [ ![Captura de pantalla de la página para configurar las opciones de diagnóstico.](media/azure-networking-analytics/diagnostic-settings-2.png)](media/azure-networking-analytics/application-gateway-diagnostics-2.png#lightbox)

5. Haga clic en la casilla *Enviar a Log Analytics*.
6. Seleccione un área de trabajo de Log Analytics existente o cree una nueva.
7. En **Registro**, haga clic en la casilla correspondiente a cada uno de los tipos de registro que quiera recopilar.
8. Haga clic en *Guardar* para habilitar el registro de diagnósticos en Azure Monitor.

#### <a name="enable-azure-network-diagnostics-using-powershell"></a>Habilitación de los diagnósticos de red de Azure con PowerShell

El siguiente script de PowerShell proporciona un ejemplo de cómo habilitar el registro de recursos para puertas de enlace de aplicaciones.

```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$gateway = Get-AzApplicationGateway -Name 'ContosoGateway'

Set-AzDiagnosticSetting -ResourceId $gateway.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

#### <a name="accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights"></a>Acceso a Azure Application Gateway Analytics por medio de Azure Monitor Network Insights

Se puede acceder a Application Insights por medio de la pestaña Conclusiones del recurso de Application Gateway.

![Captura de pantalla de Conclusiones de Application Gateway ](media/azure-networking-analytics/azure-appgw-insights.png
)

La pestaña "Ver métricas detalladas" abre el libro rellenado previamente que resume los datos de Application Gateway.

[ ![Captura de pantalla de libro de Application Gateway](media/azure-networking-analytics/azure-appgw-workbook.png)](media/azure-networking-analytics/application-gateway-workbook.png#lightbox)

### <a name="new-capabilities-with-azure-monitor-network-insights-workbook"></a>Nuevas funcionalidades con los libros de Azure Monitor Network Insights

> [!NOTE]
> No hay costos adicionales asociados a los libros de Azure Monitor Network Insights. El área de trabajo de Log Analytics se sigue facturando por el uso.

Los libros de Network Insights le permiten aprovechar las funcionalidades más recientes de Azure Monitor y Log Analytics, lo que incluye:

* Consola centralizada para la supervisión y solución de problemas con datos de [métricas](../insights/network-insights-overview.md#resource-health-and-metrics) y registros.

* Lienzo flexible que admite la creación de [visualizaciones](../visualize/workbooks-overview.md#visualizations) enriquecidas personalizadas.

* La capacidad de usar y [compartir plantillas de libro](../visualize/workbooks-overview.md#workbooks-versus-workbook-templates) con una comunidad más amplia.

Para más información acerca de las funcionalidades de la nueva solución de libros, consulte [Libros de Azure Monitor](../visualize/workbooks-overview.md)

## <a name="migrating-from-azure-gateway-analytics-solution-to-azure-monitor-workbooks"></a>Migración de la solución Azure Gateway Analytics a libros de Azure Monitor

> [!NOTE]
> Los libros de Azure Monitor Network Insights son la solución recomendada para acceder a los análisis de métricas y registros de los recursos de Application Gateway.

1. Asegúrese de que la [configuración de diagnóstico esté habilitada](#enable-azure-application-gateway-diagnostics-in-the-portal) para almacenar los registros en un área de trabajo de Log Analytics. Si ya está configurado, los libros de Azure Monitor Network Insights podrán usar datos desde la misma ubicación y no se requieren cambios adicionales.

> [!NOTE]
> Todos los datos anteriores están disponibles en el libro desde el momento en que se ha habilitado originalmente la configuración de diagnóstico. No es necesario realizar ninguna transferencia de datos.

2. Acceda al [libro de información predeterminada](#accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights) del recurso de Application Gateway. Toda la información existente compatible con la solución de análisis de Application Gateway ya estará presente en el libro. Puede ampliarla mediante la incorporación de [visualizaciones](../visualize/workbooks-overview.md#visualizations) personalizadas basadas en datos de métricas y registros.

3. Después de ver toda la información sobre las métricas y registros, para quitar la solución de análisis de Azure Gateway del área de trabajo, puede eliminarla de la página de recursos de la solución.

[ ![Captura de pantalla de la opción Eliminar de la solución Azure Application Gateway Analytics.](media/azure-networking-analytics/azure-appgw-analytics-delete.png)](media/azure-networking-analytics/application-gateway-analytics-delete.png#lightbox)

## <a name="azure-network-security-group-analytics-solution-in-azure-monitor"></a>Solución Azure Network Security Group Analytics de Azure Monitor

![Símbolo de Azure Network Security Group Analytics](media/azure-networking-analytics/azure-analytics-symbol.png)

> [!NOTE]
> La solución de análisis Grupo de seguridad de red se traslada a la compatibilidad con la comunidad ya que su funcionalidad se ha reemplazado por [Análisis de tráfico](../../network-watcher/traffic-analytics.md).
> - La solución ahora está disponible en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/oms-azurensg-solution/) y pronto ya no estarán disponibles en Azure Marketplace.
> - Para los clientes existentes que ya agregaron la solución a su área de trabajo, seguirá funcionando sin cambios.
> - Microsoft seguirá admitiendo el envío de registros de recursos de grupos de seguridad de red al área de trabajo mediante Configuración de diagnóstico.

Para los grupos de seguridad de red se admiten los siguientes registros:

* NetworkSecurityGroupEvent
* NetworkSecurityGroupRuleCounter

### <a name="install-and-configure-the-solution"></a>Instalación y configuración de la solución
Para instalar y configurar la solución Azure Networking Analytics, siga estas instrucciones:

1. Habilite la solución Azure Network Security Group Analytics mediante el proceso que se describe en [Incorporación de soluciones de Azure Monitor desde la Galería de soluciones](./solutions.md).
2. Habilite el registro de diagnósticos para los recursos de [Network Security Group](../../virtual-network/virtual-network-nsg-manage-log.md) que desea supervisar.

### <a name="enable-azure-network-security-group-diagnostics-in-the-portal"></a>Habilitación de los diagnósticos de Azure Network Security Group en el portal

1. En Azure Portal, navegue hasta el recurso Grupo de seguridad de red que se va a supervisar.
2. Seleccione *Registros de diagnósticos* para abrir la página siguiente.

   ![Captura de pantalla de la página de registros de diagnóstico de un recurso de grupo de seguridad de red que muestra la opción para activar los diagnósticos.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics01.png)
3. Haga clic en *Activar diagnósticos* para abrir la página siguiente.

   ![Captura de pantalla de la página para configurar las opciones de diagnóstico. El estado está habilitado, la opción de envío a Log Analytics está activada y hay dos tipos de registro seleccionados.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics02.png)
4. Para activar los diagnósticos, haga clic en *Activar* en *Estado*.
5. Haga clic en la casilla *Send to Log Analytics* (Enviar a Log Analytics).
6. Seleccione un área de trabajo de Log Analytics existente o cree un área de trabajo.
7. En **Registro**, haga clic en la casilla correspondiente a cada uno de los tipos de registro para recopilar.
8. Haga clic en *Guardar* para habilitar el registro de diagnósticos en Log Analytics.

### <a name="enable-azure-network-diagnostics-using-powershell"></a>Habilitación de los diagnósticos de red de Azure con PowerShell

El siguiente script de PowerShell proporciona un ejemplo de cómo habilitar el registro de recursos para grupos de seguridad de red.
```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$nsg = Get-AzNetworkSecurityGroup -Name 'ContosoNSG'

Set-AzDiagnosticSetting -ResourceId $nsg.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="use-azure-network-security-group-analytics"></a>Uso de Azure Network Security Group Analytics
Tras hacer clic en el icono **Azure Network Security Group Analytics** en Overview (Información general), puede ver resúmenes de los registros y desplazarse hasta los detalles de las categorías siguientes:

* Flujos bloqueados de grupos de seguridad de red
  * Reglas de los grupos de seguridad de red con flujos bloqueados
  * Direcciones MAC con flujos bloqueados
* Flujos permitidos en los grupos de seguridad de red
  * Reglas de los grupos de seguridad de red con flujos permitidos
  * Direcciones MAC con flujos permitidos

![Captura de pantalla de iconos con datos para los flujos bloqueados del grupo de seguridad de red, incluidas las reglas con flujos bloqueados y las direcciones MAC con flujos bloqueados.](media/azure-networking-analytics/log-analytics-nsg01.png)

![Captura de pantalla de iconos con datos para los flujos permitidos del grupo de seguridad de red, incluidas las reglas con flujos permitidos y las direcciones MAC con flujos permitidos.](media/azure-networking-analytics/log-analytics-nsg02.png)

En el panel **Azure Network Security Group Analytics**, revise la información de resumen en una de las hojas y haga clic en una para obtener información detallada sobre la página de búsqueda de registros.

En cualquiera de las páginas de búsqueda de registros, puede ver los resultados por tiempo, resultados detallados y el historial de búsqueda de registros. También puede filtrar por las facetas para restringir los resultados.

## <a name="migrating-from-the-old-networking-analytics-solution"></a>Migración desde la solución Networking Analytics antigua
En enero de 2017, cambió la forma de enviar registros de Azure Application Gateway y grupos de seguridad de red de Azure a un área de trabajo de Log Analytics. Estos cambios proporcionan las siguientes ventajas:
+ Los registros se escriben directamente en Azure Monitor sin necesidad de usar ninguna cuenta de almacenamiento.
+ Menor latencia desde el momento en el que los registros se generan hasta que están disponibles en Azure Monitor
+ Menos pasos de configuración
+ Un formato común para todos los tipos de diagnósticos de Azure

Para usar las soluciones actualizadas:

1. [Configure los diagnósticos para que se envíen directamente a Azure Monitor desde Azure Application Gateway](#enable-azure-application-gateway-diagnostics-in-the-portal).
2. [Configure los diagnósticos para que se envíen directamente a Azure Monitor desde los grupos de seguridad de red de Azure](#enable-azure-network-security-group-diagnostics-in-the-portal).
2. Habilite las soluciones *Azure Application Gateway Analytics* y *Azure Network Security Group Analytics* mediante el proceso que se describe en [Adición de soluciones de Azure Monitor desde la Galería de soluciones](solutions.md).
3. Actualice cualquier consulta guardada, panel o alerta para usar el nuevo tipo de datos.
   + El tipo es AzureDiagnostics. Puede usar ResourceType para filtrar registros de redes de Azure.

     | En lugar de: | Uso: |
     | --- | --- |
     | NetworkApplicationgateways &#124; donde OperationName=="ApplicationGatewayAccess" | AzureDiagnostics &#124; donde ResourceType=="APPLICATIONGATEWAYS" y OperationName=="ApplicationGatewayAccess" |
     | NetworkApplicationgateways &#124; donde OperationName=="ApplicationGatewayPerformance" | AzureDiagnostics &#124; donde ResourceType=="APPLICATIONGATEWAYS" y OperationName=="ApplicationGatewayPerformance" |
     | NetworkSecuritygroups | AzureDiagnostics &#124; donde ResourceType=="NETWORKSECURITYGROUPS" |

   + Para todos los campos que tengan un sufijo \_s, \_d o \_g en el nombre, cambie el primer carácter a minúsculas.
   + Para todos los campos que tengan un sufijo \_o en el nombre, los datos se dividen en campos individuales según los nombres de campo anidados.
4. Quite la solución *Azure Networking Analytics (en desuso)* .
   + Si usa PowerShell, utilice `Set-AzureOperationalInsightsIntelligencePack -ResourceGroupName <resource group that the workspace is in> -WorkspaceName <name of the log analytics workspace> -IntelligencePackName "AzureNetwork" -Enabled $false`.

Los datos recopilados antes del cambio no aparecen en la nueva solución. Puede seguir consultando estos datos con el tipo y los nombres de campo anteriores.

## <a name="troubleshooting"></a>Solución de problemas
[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="next-steps"></a>Pasos siguientes
* Use las [consultas de registro de Azure Monitor](../logs/log-query-overview.md) para ver datos de diagnósticos detallados de Azure.

