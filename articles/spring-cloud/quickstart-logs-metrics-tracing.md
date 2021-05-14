---
title: 'Inicio rápido: supervisión de aplicaciones de Azure Spring Cloud con registros, métricas y seguimiento'
description: Use el streaming de registro, el análisis de registros, las métricas y el seguimiento para supervisar aplicaciones de ejemplo de PetClinic en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 04/23/2021
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 9275eb9d5f3206a5acd41fa970c4acd1f4663a37
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321580"
---
# <a name="quickstart-monitoring-azure-spring-cloud-apps-with-logs-metrics-and-tracing"></a>Inicio rápido: supervisión de aplicaciones de Azure Spring Cloud con registros, métricas y seguimiento

::: zone pivot="programming-language-csharp"
Con la funcionalidad de supervisión integrada de Azure Spring Cloud, puede depurar y supervisar problemas complejos. Azure Spring Cloud integra el [seguimiento distribuido](https://steeltoe.io/docs/3/tracing/distributed-tracing) de Steeltoe con [Application Insights](../azure-monitor/app/app-insights-overview.md) de Azure. Esta integración proporciona una eficaz funcionalidad de registros, métricas y seguimiento distribuido desde Azure Portal.

En los procedimientos siguientes se explica cómo usar el streaming de registro, Log Analytics, las métricas y el seguimiento distribuido con la aplicación de ejemplo que implementó en las guías de inicio rápido anteriores.

## <a name="prerequisites"></a>Requisitos previos

* Complete las guías de inicio rápido anteriores de esta serie:

  * [Aprovisionamiento del servicio Azure Spring Cloud](./quickstart-provision-service-instance.md).
  * [Configuración del servidor de configuración de Azure Spring Cloud](./quickstart-setup-config-server.md).
  * [Compile e implemente las aplicaciones](./quickstart-deploy-apps.md).

## <a name="logs"></a>Registros

Hay dos maneras de ver los registros en Azure Spring Cloud: **Streaming de registros** de registros en tiempo real por instancia de aplicación o **Log Analytics** para registros agregados con la funcionalidad de consulta avanzada.

### <a name="log-streaming"></a>Secuencias de registro

El streaming de registro se puede usar en la CLI de Azure con el siguiente comando.

```azurecli
az spring-cloud app logs -n solar-system-weather -f
```

Verá un resultado similar al del siguiente ejemplo:

```output
=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Executing action method Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather) - Validation state: Valid
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController[0]

=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Retrieved weather data from 4 planets
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]

=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Executing ObjectResult, writing value of type 'System.Collections.Generic.KeyValuePair`2[[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e],[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]][]'.
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]
```

> [!TIP]
> Use `az spring-cloud app logs -h` para explorar más parámetros y la funcionalidad de las secuencias de registro.

### <a name="log-analytics"></a>Log Analytics

1. En Azure Portal, vaya a la página **Servicio | Información general** y seleccione **Registros** en la sección **Supervisión**. Seleccione **Ejecutar** en una de las consultas de ejemplo de Azure Spring Cloud.

   [ ![Entrada de Log Analytics](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png#lightbox)
    
1. Edite la consulta para quitar las cláusulas WHERE que limitan la presentación a los registros de advertencias y errores.

1. A continuación, seleccione `Run` y verá los registros. Para más instrucciones sobre la escritura de consultas, consulte la [documentación de Azure Log Analytics](../azure-monitor/logs/get-started-queries.md).

   [ ![Consulta de Log Analytics: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query-steeltoe.png#lightbox)

1. Para más información sobre el lenguaje de consulta que se utiliza en Log Analytics, consulte [Consultas de registros de Azure Monitor](/azure/data-explorer/kusto/query/). Para consultar todos los registros de Log Analytics desde un cliente centralizado, vea [Azure Data Explorer](/azure/data-explorer/query-monitor-data).

## <a name="metrics"></a>Métricas

1. En Azure Portal, vaya a la página **Servicio | Información general** y seleccione **Métricas** en la sección **Supervisión**. Para agregar la primera métrica, seleccione una de las métricas de .NET en **Rendimiento (.NET)** o **Solicitud (.NET)** , en la lista desplegable **Métrica** y `Avg` en **Agregación** para ver la línea de tiempo de esa métrica.

   [ ![Entrada de métricas: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu-steeltoe.png#lightbox)
    
1. Haga clic en **Agregar filtro** en la barra de herramientas y seleccione `App=solar-system-weather` para ver el uso de la CPU solo para la aplicación **solar-system-weather**.

   [ ![Usar un filtro en métricas: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter-steeltoe.png#lightbox)

1. Descarte el filtro que creado en el paso anterior, seleccione **Apply Splitting** (Aplicar separación) y seleccione `App` en **Valores** para ver el uso que las diferentes aplicaciones hacen de la CPU.

   [ ![Aplicar separación en métricas: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split-steeltoe.png#lightbox)

## <a name="distributed-tracing"></a>Seguimiento distribuido

1. En Azure Portal, vaya a la página **Servicio | Información general** y seleccione **Seguimiento distribuido** en la sección **Supervisión**. Luego, seleccione la pestaña **View application map** (Ver mapa de la aplicación) de la derecha.

   [ ![Entrada de Seguimiento distribuido: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png#lightbox)

1. Ya puede ver el estado de las llamadas entre las aplicaciones. 

   [ ![Información general sobre el seguimiento distribuido: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview-steeltoe.png#lightbox)
    
1. Seleccione el vínculo entre **solar-system-weather** y **planet-weather-provider** para ver más detalles, como las llamadas más lentas por métodos HTTP.

   [ ![Seguimiento distribuido: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call-steeltoe.png#lightbox)
    
1. Por último, seleccione **Investigate Performance** (Investigar rendimiento) para explorar un análisis del rendimiento integrado más eficaz.

   [ ![Rendimiento del seguimiento distribuido: Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance-steeltoe.png#lightbox)
::: zone-end

::: zone pivot="programming-language-java"
Con la funcionalidad de supervisión integrada de Azure Spring Cloud, puede depurar y supervisar problemas complejos. Azure Spring Cloud integra [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) con [Application Insights](../azure-monitor/app/app-insights-overview.md) de Azure. Esta integración proporciona una eficaz funcionalidad de registros, métricas y seguimiento distribuido desde Azure Portal. En los procedimientos siguientes se explica cómo usar el streaming de registro, Log Analytics, las métricas y el seguimiento distribuido con aplicaciones de PetClinic implementadas.

## <a name="prerequisites"></a>Requisitos previos

Complete los pasos anteriores: 

* [Aprovisionamiento de una instancia de Azure Spring Cloud](./quickstart-provision-service-instance.md)
* [Configure el servidor de configuración](./quickstart-setup-config-server.md)
* [Compile e implemente las aplicaciones](./quickstart-deploy-apps.md).

## <a name="logs"></a>Registros

Hay dos maneras de ver los registros en Azure Spring Cloud: **Streaming de registros** de registros en tiempo real por instancia de aplicación o **Log Analytics** para registros agregados con la funcionalidad de consulta avanzada.

### <a name="log-streaming"></a>Secuencias de registro

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

El streaming de registro se puede usar en la CLI de Azure con el siguiente comando.

```azurecli
az spring-cloud app logs -s <service instance name> -g <resource group name> -n gateway -f
```

Verá registros como este:

[ ![Streaming de registro desde la CLI de Azure](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png#lightbox)

> [!TIP]
> Use `az spring-cloud app logs -h` para explorar más parámetros y funcionalidades de las secuencias de registro.

Para más información sobre el lenguaje de consulta que se utiliza en Log Analytics, consulte [Consultas de registros de Azure Monitor](/azure/data-explorer/kusto/query/). Para consultar todos los registros de Log Analytics desde un cliente centralizado, eche un vistazo a [Azure Data Explorer](/azure/data-explorer/query-monitor-data).

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

Para obtener los registros mediante Azure Toolkit for IntelliJ:

1. Seleccione **Azure Explorer** (Explorador de Azure) y, luego, **Spring Cloud**.

1. Haga clic con el botón derecho en la aplicación en ejecución.

1. Seleccione **Streaming Logs** (Registros de streaming) en la lista desplegable.

   ![Selección de los registros de streaming](media/spring-cloud-intellij-howto/streaming-logs.png)
    
1. Seleccione **Instancia**.

   ![Selección de la instancia](media/spring-cloud-intellij-howto/select-instance.png)
    
1. El registro de streaming se verá en la ventana de salida.

   ![Salida del registro de streaming](media/spring-cloud-intellij-howto/streaming-log-output.png)

 Para más información sobre el lenguaje de consulta que se utiliza en Log Analytics, consulte [Consultas de registros de Azure Monitor](/azure/data-explorer/kusto/query/). Para consultar todos los registros de Log Analytics desde un cliente centralizado, eche un vistazo a [Azure Data Explorer](/azure/data-explorer/query-monitor-data).

---

### <a name="log-analytics"></a>Log Analytics

1. Vaya a la página **servicio | Información general** y seleccione **Registros** en la sección **Supervisión**. Haga clic en **Ejecutar** en una de las consultas de ejemplo de Azure Spring Cloud.

   [ ![Entrada del portal de Log Analytics](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/logs-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/logs-entry.png#lightbox)

1. Luego verá los registros filtrados. Para más instrucciones sobre la escritura de consultas, consulte la [documentación de Azure Log Analytics](../azure-monitor/logs/get-started-queries.md).

   [ ![Consulta de Log Analytics](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/logs-query.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/logs-query.png#lightbox)

## <a name="metrics"></a>Métricas

Vaya a la hoja `Metrics` (puede ver las métricas aportadas por aplicaciones de Spring Boot, módulos de Spring Cloud y dependencias). En el gráfico siguiente se muestra `gateway-requests` (Spring Cloud Gateway), `hikaricp_connections` (Conexiones JDBC) y `http_client_requests`.
 
[ ![Hoja Métricas](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-metrics.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-metrics.jpg#lightbox)

Spring Boot registra un gran número de métricas principales: JVM, CPU, Tomcat, Logback... La configuración automática de Spring Boot permite la instrumentación de las solicitudes que controla Spring MVC.
Los tres controladores REST `OwnerResource`, `PetResource` y `VisitResource` los ha instrumentado la anotación Micrometer de `@Timed` en el nivel de clase.

* La aplicación `customers-service` tiene las siguientes métricas personalizadas habilitadas:
  * @Timed: `petclinic.owner`
  * @Timed: `petclinic.pet`
* La aplicación `visits-service` tiene las siguientes métricas personalizadas habilitadas:
  * @Timed: `petclinic.visit`

Estas métricas personalizadas se pueden ver en la hoja `Metrics`: [ ![ Métricas personalizadas](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-custom-metrics.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-custom-metrics.jpg#lightbox)

Puede usar la característica Prueba de disponibilidad de Application Insights y supervise la disponibilidad de las aplicaciones:

[ ![Prueba de disponibilidad](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-availability.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-availability.jpg#lightbox)

Vaya a la hoja `Live Metrics` (puede ver las métricas en directo en pantalla con latencias bajas, inferiores a 1 segundo: [ ![ Métricas en directo ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-live-metrics.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-live-metrics.jpg#lightbox))

## <a name="tracing"></a>Traza

Abra la instancia de Application Insights creada por Azure Spring Cloud y empiece a supervisar aplicaciones de microservicios.

Vaya a la hoja `Application Map`: [ ![ Mapa de aplicación ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/distributed-tracking-new-ai-agent.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/distributed-tracking-new-ai-agent.jpg#lightbox)

Vaya a la hoja `Performance`: [ ![ Hoja Rendimiento ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-performance.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-performance.jpg#lightbox)

Vaya a la hoja `Performance/Dependenices` (puede ver el número de rendimiento de las dependencias, especialmente las llamadas SQL: [ ![ hoja Rendimiento/Dependencias](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-insights-on-dependencies.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-insights-on-dependencies.jpg#lightbox))

Haga clic en una llamada SQL para ver la transacción de un extremo a otro en contexto: [ ![ transacción de un extremo a otro de SQL](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-end-to-end-transaction-details.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-end-to-end-transaction-details.jpg#lightbox)

Vaya a la hoja `Failures/Exceptions` (puede ver una colección de excepciones: [ ![ Errores o excepciones](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-failures-exceptions.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/petclinic-microservices-failures-exceptions.jpg#lightbox))

Haga clic en una excepción para ver la transacción de un extremo a otro y el seguimiento de la pila en contexto: [ ![ seguimiento de la pila de un extremo a otro](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/end-to-end-transaction-details.jpg) ](media/spring-cloud-quickstart-logs-metrics-tracing/update-logs-metrics-tracing/end-to-end-transaction-details.jpg#lightbox)

::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

En estas guías de inicio rápido, ha creado recursos de Azure que seguirán acumulando cargos si permanecen en la suscripción. Si prevé que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos desde el portal o con el siguiente comando en Cloud Shell:

```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

En una guía de inicio rápido anterior, también estableció el nombre del grupo de recursos predeterminado. Si no desea continuar con la siguiente guía de inicio rápido, borre ese nombre predeterminado con el siguiente comando de la CLI:

```azurecli
az configure --defaults group=
```

## <a name="next-steps"></a>Pasos siguientes

Para explorar más funcionalidades de supervisión de Azure Spring Cloud, consulte:

> [!div class="nextstepaction"]
> [Servicios de diagnóstico](diagnostic-services.md)
>
> [Seguimiento distribuido](./how-to-distributed-tracing.md)
>
> [Streaming de registros en tiempo real](./how-to-log-streaming.md)
