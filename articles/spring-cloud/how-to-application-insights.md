---
title: Uso del agente In-Process de Java de Application Insights en Azure Spring Cloud
description: Supervisión de aplicaciones y microservicios mediante el agente In-Process de Java de Application Insights en Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 12/04/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: e9c0a2a6a6b40325e4bf87dc8cf52bc419b77621
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129070"
---
# <a name="application-insights-java-in-process-agent-in-azure-spring-cloud-preview"></a>Agente In-Process de Java de Application Insights en Azure Spring Cloud (versión preliminar)

En este documento se explica cómo supervisar las aplicaciones y los microservicios mediante el agente Java de Application Insights en Azure Spring Cloud. 

Con esta característica, puede hacer lo siguiente:

* Buscar datos de seguimiento con distintos filtros.
* Ver el mapa de dependencias de los microservicios.
* Comprobar el rendimiento de la solicitud.
* Supervisar las métricas en directo en tiempo real.
* Comprobar los errores de solicitud.
* Comprobar las métricas de aplicación.

Application Insights proporciona muchas perspectivas observables, entre las que se incluyen las siguientes:

* Mapa de aplicación
* Rendimiento
* Errores
* Métricas
* Live Metrics
* Disponibilidad

## <a name="enable-java-in-process-agent-for-application-insights"></a>Habilitar el agente In-Process de Java para Application Insights

Habilite la característica en versión preliminar del agente In-Process de Java mediante el procedimiento siguiente.

1. Vaya a la página de información general del servicio de la instancia de servicio.
2. Haga clic en la entrada **Application Insights** en la hoja de supervisión.
3. Haga clic en el botón **Habilitar Application Insights** para habilitar la integración de **Application Insights**.
4. Seleccione una instancia existente de Application Insights o cree una.
5. Haga clic en **Habilitar el agente In-Process de Java** para habilitar la característica del agente In-Process de Java en versión preliminar. Aquí también puede personalizar la frecuencia de muestreo de 0 a 100.
6.  Haga clic en **Guardar** para guardar el cambio.

## <a name="portal"></a>Portal

1. Vaya a la página **servicio | Información general** y seleccione **Application Insights** en la sección **Supervisión**. 
2. Haga clic en **Habilitar Application Insights** para habilitar Application Insights en Azure Spring Cloud.
3. Haga clic en **Habilitar el agente In-Process de Java** para habilitar la característica IPA de Java en versión preliminar. Cuando está habilitada una característica IPA en versión preliminar, puede configurar una frecuencia de muestreo opcional (valor predeterminado del 10,0 %).

  [ ![IPA 0](media/spring-cloud-application-insights/insights-process-agent-0.png)](media/spring-cloud-application-insights/insights-process-agent-0.png)

## <a name="using-the-application-insights-feature"></a>Uso de la característica Application Insights

Cuando está habilitada la característica **Application Insights**, puede hacer lo siguiente:

En el panel de navegación izquierdo, haga clic en **Application Insights** para ir a la página **Información general** de Application Insights. 

* Haga clic en **Mapa de aplicación** para ver el estado de las llamadas entre las aplicaciones.

  [ ![IPA 2](media/spring-cloud-application-insights/insights-process-agent-2-map.png)](media/spring-cloud-application-insights/insights-process-agent-2-map.png)

* Haga clic en el vínculo entre el servicio para los clientes y `petclinic` para ver más detalles, como una consulta de SQL.

* En el panel de navegación izquierdo, haga clic en **Rendimiento** para ver los datos de rendimiento de las operaciones de todas las aplicaciones, así como las dependencias y los roles.

  [ ![IPA 4](media/spring-cloud-application-insights/insights-process-agent-4-performance.png)](media/spring-cloud-application-insights/insights-process-agent-4-performance.png)

* En el panel de navegación izquierdo, haga clic en **Errores** para ver si sucede algo inesperado en las aplicaciones.

  [ ![IPA 6](media/spring-cloud-application-insights/insights-process-agent-6-failures.png)](media/spring-cloud-application-insights/insights-process-agent-6-failures.png)

* En el panel de navegación izquierdo, haga clic en **Métricas** y seleccione el espacio de nombres. Verá las métricas de Spring Boot y las métricas personalizadas, si las hay.

  [ ![IPA 7](media/spring-cloud-application-insights/insights-process-agent-5-metrics.png)](media/spring-cloud-application-insights/insights-process-agent-5-metrics.png)

* En el panel de navegación izquierdo, haga clic en **Live Metrics** para ver las métricas en tiempo real de las distintas dimensiones.

  [ ![IPA 8](media/spring-cloud-application-insights/petclinic-microservices-live-metrics.jpg)](media/spring-cloud-application-insights/petclinic-microservices-live-metrics.jpg)

* En el panel de navegación izquierdo, haga clic en **Disponibilidad** para supervisar la disponibilidad y la capacidad de respuesta de las aplicaciones web mediante la creación de [pruebas de disponibilidad en Application Insights](../azure-monitor/app/monitor-web-app-availability.md).

  [ ![IPA 9](media/spring-cloud-application-insights/petclinic-microservices-availability.jpg)](media/spring-cloud-application-insights/petclinic-microservices-availability.jpg)

## <a name="arm-template"></a>Plantilla de ARM

Para usar la plantilla de Azure Resource Manager, copie el siguiente contenido en `azuredeploy.json`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.AppPlatform/Spring",
            "name": "customize this",
            "apiVersion": "2020-07-01",
            "location": "[resourceGroup().location]",
            "resources": [
                {
                    "type": "monitoringSettings",
                    "apiVersion": "2020-11-01-preview",
                    "name": "default",
                    "properties": {
                        "appInsightsInstrumentationKey": "00000000-0000-0000-0000-000000000000",
                        "appInsightsSamplingRate": 88.0
                    },
                    "dependsOn": [
                        "[resourceId('Microsoft.AppPlatform/Spring', 'customize this')]"
                    ]
                }
            ],
            "properties": {}
        }
    ]
}
```

## <a name="cli"></a>CLI

Aplique la plantilla de ARM con el comando de la CLI:

* Para una instancia existente de Azure Spring Cloud:

```azurecli
az spring-cloud app-insights update [--app-insights/--app-insights-key] "assignedName" [--sampling-rate] "samplingRate" â€“name "assignedName" â€“resource-group "resourceGroupName"
```
* Para una instancia recién creada de Azure Spring Cloud:

```azurecli
az spring-cloud create/update [--app-insights]/[--app-insights-key] "assignedName" --disable-app-insights false --enable-java-agent true --name "assignedName" â€“resource-group "resourceGroupName"
```
* Para deshabilitar Application Insights:

```azurecli
az spring-cloud app-insights update --disable â€“name "assignedName" â€“resource-group "resourceGroupName"

```

## <a name="see-also"></a>Consulte también
* [Uso del seguimiento distribuido con Azure Spring Cloud](./how-to-distributed-tracing.md)
* [Análisis de registros y métricas](diagnostic-services.md)
* [Streaming de registros en tiempo real](./how-to-log-streaming.md)