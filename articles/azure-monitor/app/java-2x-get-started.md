---
title: 'Inicio rápido: Análisis de aplicaciones web Java con Azure Application Insights'
description: 'Supervisión del rendimiento de aplicaciones web de Java con Application Insights. '
ms.topic: conceptual
ms.date: 11/22/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 043287d8a66ee6954e171dc0c09b42963f522fac
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110104517"
---
# <a name="quickstart-get-started-with-application-insights-in-a-java-web-project"></a>Inicio rápido: Introducción a Application Insights en un proyecto web de Java

> [!CAUTION]
> Este documento se aplica a Application Insights para Java 2.x, que ya no se recomienda.
>
> La documentación de la versión más reciente se puede encontrar en [Application Insights para Java 3.x](./java-in-process-agent.md).

En este inicio rápido, se usa el SDK de Application Insights para instrumentar solicitudes, realizar un seguimiento de dependencias y recopilar contadores de rendimiento, diagnosticar problemas de rendimiento y excepciones y escribir código para realizar un seguimiento de lo que hacen los usuarios con la aplicación.

Application Insights es un servicio de análisis extensible para desarrolladores web que ayuda a comprender el rendimiento y el uso de la aplicación activa. Application Insights es compatible con aplicaciones Java que se ejecutan en Linux, Unix o Windows.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Una aplicación Java en funcionamiento.

## <a name="get-an-application-insights-instrumentation-key"></a>Obtención de una clave de instrumentación de Application Insights

> [!IMPORTANT]
> Se recomiendan las [cadenas de conexión](./sdk-connection-string.md?tabs=java) por encima de las claves de instrumentación. Las nuevas regiones de Azure **requieren** el uso de cadenas de conexión en lugar de claves de instrumentación. La cadena de conexión identifica el recurso con el que se quieren asociar los datos de telemetría. También permite modificar los puntos de conexión que va a usar el recurso como destino de la telemetría. Tiene que copiar la cadena de conexión y agregarla al código de la aplicación o a una variable de entorno.
1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En Azure Portal, cree un nuevo recurso de Application Insights. Establezca el tipo de aplicación a una aplicación web de Java.

3. Busque la clave de instrumentación del nuevo recurso. Pronto tendrá que pegarla en el proyecto de código.

    ![En la información general de nuevos recursos, haga clic en Propiedades y copie la clave de instrumentación.](./media/java-get-started/instrumentation-key-001.png)

## <a name="add-the-application-insights-sdk-for-java-to-your-project"></a>Incorporación del SDK de Application Insights para Java al proyecto

*Elija el tipo de proyecto.*

# <a name="maven"></a>[Maven](#tab/maven)

Si su proyecto ya se ha configurado para usar Maven para la compilación, combine el siguiente código en el archivo *pom.xml*.

A continuación, actualice las dependencias del proyecto, para obtener los archivos binarios descargados.

```XML
    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-web-auto</artifactId>
        <!-- or applicationinsights-web for manual web filter registration -->
        <!-- or applicationinsights-core for bare API -->
        <version>2.6.2</version>
      </dependency>
    </dependencies>
```

# <a name="gradle"></a>[Gradle](#tab/gradle)

Si su proyecto ya se ha configurado para usar Gradle para la compilación, combine el siguiente código al archivo *build.gradle*.

A continuación, actualice las dependencias del proyecto, para obtener los archivos binarios descargados.

```gradle
    dependencies {
      compile group: 'com.microsoft.azure', name: 'applicationinsights-web-auto', version: '2.6.2'
      // or applicationinsights-web for manual web filter registration
      // or applicationinsights-core for bare API
    }
```

---

### <a name="questions"></a>Preguntas
* *¿Cuál es la relación entre los componentes `-web-auto`, `-web` y `-core`?*
  * `applicationinsights-web-auto` proporciona métricas que realizan el seguimiento de los recuentos de solicitudes de servlet HTTP y los tiempos de respuesta, mediante el registro automático del filtro de servlet de Application Insights en tiempo de ejecución.
  * `applicationinsights-web` también proporciona métricas que realizan el seguimiento de los recuentos de solicitudes de servlet HTTP y los tiempos de respuesta, pero requiere el registro manual del filtro de servlet de Application Insights en la aplicación.
  * `applicationinsights-core` solo proporciona la API básica, por ejemplo, si la aplicación no se basa en servlet.
  
* *¿Cómo se debe actualizar el SDK a la versión más reciente?*
  * A partir de noviembre de 2020, para la supervisión de aplicaciones Java se recomienda el uso de Java 3.x para Application Insights. Para más información sobre cómo empezar, consulte [Java 3.x para Application Insights](./java-in-process-agent.md).

## <a name="add-an-applicationinsightsxml-file"></a>Adición de un archivo *ApplicationInsights.xml*
Agregue *ApplicationInsights.xml* a la carpeta de recursos del proyecto o asegúrese de que se agrega a la ruta de acceso de la clase de implementación del proyecto. Copie en ella el siguiente XML.

Reemplace la clave de instrumentación por la que recibió de Azure Portal.

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">

   <!-- The key from the portal: -->
   <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>

   <!-- HTTP request component (not required for bare API) -->
   <TelemetryModules>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
   </TelemetryModules>

   <!-- Events correlation (not required for bare API) -->
   <!-- These initializers add context data to each event -->
   <TelemetryInitializers>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
      <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
   </TelemetryInitializers>

</ApplicationInsights>
```

Si lo desea, el archivo de configuración puede estar en cualquier ubicación a la que su aplicación tenga acceso.  La propiedad del sistema `-Dapplicationinsights.configurationDirectory` especifica el directorio que contiene *ApplicationInsights.xml*. Por ejemplo, un archivo de configuración ubicado en `E:\myconfigs\appinsights\ApplicationInsights.xml` se configuraría con la propiedad `-Dapplicationinsights.configurationDirectory="E:\myconfigs\appinsights"`.

* La clave de instrumentación se envía junto con todos los elementos de telemetría e indica a Application Insights que se muestre en el recurso.
* El componente de la solicitud HTTP es opcional. Envía automáticamente telemetría sobre las solicitudes y tiempos de respuesta en el portal.
* La correlación de eventos es una incorporación al componente de la solicitud HTTP. Asigna un identificador a cada solicitud recibida por el servidor. A continuación, agrega este identificador como una propiedad a todos los elementos de telemetría como la propiedad "Operation.Id". Le permite correlacionar la telemetría asociada a cada solicitud estableciendo un filtro en la [búsqueda de diagnóstico][diagnostic].

### <a name="alternative-ways-to-set-the-instrumentation-key"></a>Alternativas para establecer la clave de instrumentación
SDK de Application Insights busca la clave en este orden:

1. Propiedad del sistema: -DAPPINSIGHTS_INSTRUMENTATIONKEY=your_ikey
2. Variable de entorno: APPINSIGHTS_INSTRUMENTATIONKEY
3. Archivo de configuración: *ApplicationInsights.xml*

También se puede [configurar en el código](./api-custom-events-metrics.md#ikey):

```java
    String instrumentationKey = "00000000-0000-0000-0000-000000000000";

    if (instrumentationKey != null)
    {
        TelemetryConfiguration.getActive().setInstrumentationKey(instrumentationKey);
    }
```

## <a name="add-agent"></a>Adición del agente

[Instale el agente de Java](java-2x-agent.md) para capturar llamadas HTTP salientes, consultas de JDBC y registros de aplicaciones y mejorar la nomenclatura de las operaciones.

## <a name="run-your-application"></a>Ejecución de la aplicación
Ejecútela en modo de depuración en el equipo de desarrollo o bien publíquela en el servidor.

## <a name="view-your-telemetry-in-application-insights"></a>Visualización de la telemetría en Application Insights
Vuelva al recurso de Application Insights en el [Portal de Microsoft Azure](https://portal.azure.com).

Los datos de las solicitudes HTTP aparecen en la hoja de información general. (Si todavía no está ahí, espere unos segundos y, a continuación, haga clic en Actualizar).

![Captura de pantalla de información general con datos de ejemplo](./media/java-get-started/overview-graphs.png)

[Más información acerca de las métricas.][metrics]

Haga clic en cualquier gráfico para ver métricas agregadas más detalladas.

![Panel de errores de Application Insights con gráficos](./media/java-get-started/006-barcharts.png)

### <a name="instance-data"></a>Datos de instancia
Haga clic en un tipo de solicitud específico para ver las instancias individuales.

![Profundización en una vista de ejemplo concreta](./media/java-get-started/007-instance.png)

### <a name="analytics-powerful-query-language"></a>Análisis: Lenguaje de consulta eficaz
A medida que acumula más datos, puede ejecutar consultas tanto para agregar datos como para buscar instancias individuales.  [Analytics](../logs/log-query-overview.md) es una eficaz herramienta tanto para conocer el rendimiento y el uso, como para el diagnóstico.

![Ejemplo de Analytics](./media/java-get-started/0025.png)

## <a name="install-your-app-on-the-server"></a>Instalación de la aplicación en el servidor
Ahora puede publicar la aplicación en el servidor, dejar que la utilicen los usuarios y ver la telemetría en el portal.

* Asegúrese de que el firewall permite que la aplicación envíe datos de telemetría a estos puertos:

  * dc.services.visualstudio.com:443
  * f5.services.visualstudio.com:443

* Si el tráfico saliente debe enrutarse a través de un firewall, defina las propiedades de sistema `http.proxyHost` y `http.proxyPort`.

* En los servidores de Windows, instale:

  * [Microsoft Visual C++ Redistributable](https://www.microsoft.com/download/details.aspx?id=40784)

    (Este componente habilita los contadores de rendimiento.)

## <a name="azure-app-service-aks-vms-config"></a>Azure App Service, AKS y configuración de máquinas virtuales

El mejor enfoque y el más sencillo para supervisar las aplicaciones que se ejecutan en cualquiera de los proveedores de recursos de Azure es usar [Java 3.x para Application Insights](./java-in-process-agent.md).


## <a name="exceptions-and-request-failures"></a>Excepciones y errores de solicitud
El filtro web de Application Insights recopila automáticamente las excepciones no controladas y las solicitudes con error.

Para recopilar datos de otras excepciones, puede [insertar llamadas a trackException() en el código][apiexceptions].

## <a name="monitor-method-calls-and-external-dependencies"></a>Supervisión de llamadas a métodos y dependencias externas
[Instale el agente de Java](java-2x-agent.md) para registrar los métodos internos especificados y las llamadas realizadas a través de JDBC, con datos de tiempo.

También sirve para la nomenclatura automática de las operaciones.

## <a name="w3c-distributed-tracing"></a>Seguimiento distribuido de W3C

El SDK de Java de Application Insights ahora admite el [seguimiento distribuido de W3C](https://w3c.github.io/trace-context/).

La configuración del SDK de entrada se explica más en detalle en nuestro artículo sobre [correlación](correlation.md).

La configuración del SDK de salida se define en el archivo [AI-Agent.xml](java-2x-agent.md).

## <a name="performance-counters"></a>Contadores de rendimiento
Abra **Investigar** y **Métricas** para ver un intervalo de contadores de rendimiento.

![Captura de pantalla del panel de métricas con los bytes privados del proceso seleccionados](./media/java-get-started/011-perf-counters.png)

### <a name="customize-performance-counter-collection"></a>Personalizar la recopilación de contadores de rendimiento
Para deshabilitar la recopilación del conjunto estándar de contadores de rendimiento, agregue el siguiente código bajo el nodo raíz del archivo *ApplicationInsights.xml*:

```XML
    <PerformanceCounters>
       <UseBuiltIn>False</UseBuiltIn>
    </PerformanceCounters>
```

### <a name="collect-additional-performance-counters"></a>Recopilar contadores de rendimiento adicionales
Puede especificar contadores de rendimiento adicionales que se van a recopilar.

#### <a name="jmx-counters-exposed-by-the-java-virtual-machine"></a>Contadores JMX (expuestos por la máquina virtual de Java)

```XML
    <PerformanceCounters>
      <Jmx>
        <Add objectName="java.lang:type=ClassLoading" attribute="TotalLoadedClassCount" displayName="Loaded Class Count"/>
        <Add objectName="java.lang:type=Memory" attribute="HeapMemoryUsage.used" displayName="Heap Memory Usage-used" type="composite"/>
      </Jmx>
    </PerformanceCounters>
```

* `displayName` : el nombre mostrado en el portal de Application Insights.
* `objectName` : el nombre del objeto JMX.
* `attribute` : el atributo del nombre del objeto JMX que se va a capturar
* `type` (opcional): el tipo del atributo del objeto JMX:
  * Valor predeterminado: un tipo simple como int o long.
  * `composite`: los datos del contador de rendimiento tienen el formato 'Attribute.Data'
  * `tabular`: los datos del contador de rendimiento tienen el formato de una fila de tabla

#### <a name="windows-performance-counters"></a>Contadores de rendimiento de Windows
Cada [contador de rendimiento de Windows](/windows/win32/perfctrs/performance-counters-portal) es un miembro de una categoría (de la misma manera que un campo es un miembro de una clase). Las categorías puede ser globales, o pueden tener instancias con nombre o numeradas.

```XML
    <PerformanceCounters>
      <Windows>
        <Add displayName="Process User Time" categoryName="Process" counterName="%User Time" instanceName="__SELF__" />
        <Add displayName="Bytes Printed per Second" categoryName="Print Queue" counterName="Bytes Printed/sec" instanceName="Fax" />
      </Windows>
    </PerformanceCounters>
```

* displayName: el nombre mostrado en el portal de Application Insights.
* categoryName: la categoría de contador de rendimiento (objeto de rendimiento) con la que está asociada este contador de rendimiento.
* counterName: el nombre del contador de rendimiento.
* instanceName: el nombre de la instancia de categoría del contador de rendimiento o una cadena vacía (""), si la categoría contiene una sola instancia. Si categoryName es Proceso, y el contador de rendimiento que desea recopilar está en el proceso de JVM actual en que se ejecuta la aplicación, especifique `"__SELF__"`.

### <a name="unix-performance-counters"></a>Contadores de rendimiento de Unix
* [Instale collectd con el complemento de Application Insights](java-2x-collectd.md) para obtener una amplia variedad de datos de red y del sistema.

## <a name="get-user-and-session-data"></a>Obtención de datos de usuario y sesión
Bien, va a enviar telemetría desde el servidor web. Ahora, para obtener la visión completa de 360 grados de la aplicación, puede agregar mayor supervisión:

* [Agregue telemetría a las páginas web][usage] para supervisar las vistas de páginas y las métricas de usuario.
* [Configure las pruebas web][availability] para comprobar que la aplicación efectivamente está activa y responde adecuadamente.

## <a name="send-your-own-telemetry"></a>Envío de su propia telemetría
Ahora que ha instalado el SDK, puede utilizar la API para enviar su propia telemetría.

* [Realice el seguimiento de eventos y métricas personalizados][api] para saber qué hacen los usuarios con su aplicación.
* [Busque eventos y registros][diagnostic] para ayudar a diagnosticar problemas.

## <a name="availability-web-tests"></a>Pruebas web de disponibilidad
Application Insights puede probar su sitio web a intervalos regulares para comprobar que está activo y que responde correctamente.

[Obtenga más información sobre cómo configurar las pruebas web de disponibilidad][availability].

## <a name="questions-problems"></a>¿Tiene preguntas? ¿Tiene problemas?
[Solución de problemas de Java](java-2x-troubleshoot.md)

## <a name="next-steps"></a>Pasos siguientes
* [Supervisión de llamadas a dependencias](java-2x-agent.md)
* [Supervisión de contadores de rendimiento de Unix](java-2x-collectd.md)
* Incorporación de la [supervisión a las páginas web](javascript.md) para controlar los tiempos de carga, las llamadas de AJAX y la excepciones del explorador.
* Escritura de una [telemetría personalizada](./api-custom-events-metrics.md) para hacer un seguimiento del uso en el explorador o el servidor.
* Uso de [Analytics](../logs/log-query-overview.md) para realizar consultas eficaces sobre los datos de telemetría de la aplicación
* Para más información, visite [Azure para desarrolladores de Java](/java/azure).

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#trackexception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[javalogs]: java-2x-trace-logs.md
[metrics]: ../essentials/metrics-charts.md
[usage]: javascript.md