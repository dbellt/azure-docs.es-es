---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: jbeauregardb
manager: vravikumar
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 06/01/2021
ms.topic: include
ms.custom: include file
ms.author: jbeauregardb
ms.openlocfilehash: b00fc19d811295f9ae29d69c73117d81acff57c9
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593151"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](/azure/developer/java/fundamentals/java-jdk-install), versión 8 o posterior.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).
- Cree un [recurso de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource) en Azure Portal.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Abra el terminal o la ventana de comandos. Vaya al directorio en el que quiere crear la aplicación Java. Ejecute el siguiente comando para generar el proyecto de Java a partir de la plantilla maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Observará que la tarea "generar" creó un directorio con el mismo nombre que el objeto `artifactId`. En este directorio, el directorio src/main/java contiene el código fuente del proyecto, el directorio `src/test/java directory` contiene el origen de la prueba y el archivo `pom.xml` es el modelo de objetos del proyecto o POM.

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo **pom.xml** en el editor de texto. Agregue los siguientes elementos de dependencia al grupo de dependencias.

```xml
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-communication-identity</artifactId>
      <version>1.0.0</version>
    </dependency>
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-monitor-opentelemetry-exporter</artifactId>
      <version>1.0.0-beta.4</version>
    </dependency>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Vaya al directorio */src/main/java/com/communication/quickstart*.
1. Abra el archivo *App.java* en el editor.
1. Reemplace la instrucción `System.out.println("Hello world!");`.
1. Agregue directivas `import`.

Use el código siguiente para empezar:

```java
package com.communication.quickstart;

import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.SimpleSpanProcessor;
import com.azure.monitor.opentelemetry.exporter.*;
import com.azure.communication.identity.*;

public class App
{
    public static void main( String[] args ) throws Exception
    {
        System.out.println("Azure Communication Services - Export Telemetry to Application Insights");
        // Quickstart code goes here
    }
}
```
## <a name="setting-up-the-telemetry-tracer-with-communication-identity-sdk-calls"></a>Configuración del seguimiento de telemetría con llamadas del SDK de identidad de comunicación

Inicialice un objeto `CommunicationIdentityClient` con su cadena de conexión. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

```java

    CommunicationIdentityClient client = new CommunicationIdentityClientBuilder()
        .connectionString("<COMMUNICATION-RESOURCE-CONNECTION-STRING>")
        .buildClient();
```

En primer lugar, para crear el intervalo que le permitirá hacer un seguimiento de las solicitudes en Azure Monitor, tendrá que crear una instancia de un objeto `AzureMonitorTraceExporter`. Tendrá que indicar la cadena de conexión del recurso de Application Insights.

```java
    AzureMonitorTraceExporter exporter = new AzureMonitorExporterBuilder()
        .connectionString("<APPLICATION-INSIGHTS-CONNECTION-STRING>")
        .buildTraceExporter();
```

A continuación, este exportador le permitirá crear las siguientes instancias para que el seguimiento de las solicitudes sea posible. Agregue el siguiente código después de crear `AzureMonitorTraceExporter`:

```java

    SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
        .addSpanProcessor(SimpleSpanProcessor.create(exporter))
        .build();

    OpenTelemetrySdk openTelemetrySdk = OpenTelemetrySdk.builder()
        .setTracerProvider(tracerProvider)
        .buildAndRegisterGlobal();

    Tracer tracer = openTelemetrySdk.getTracer("Sample");
```
Una vez inicializado el seguimiento, puede crear el intervalo que se encarga de hacer el seguimiento de las solicitudes.

```java
    Span span = tracer.spanBuilder("sample-span").startSpan();
    final Scope scope = span.makeCurrent();
    try {
        // Thread bound (sync) calls will automatically pick up the parent span and you don't need to pass it explicitly.
        client.createUser();
    } finally {
        span.end();
        scope.close();
    }
    Thread.sleep(10000);
```

## <a name="run-the-code"></a>Ejecución del código

Navegue hasta el directorio que contiene el archivo *pom.xml* y compile el proyecto mediante el siguiente comando `mvn`.

```console
mvn compile
```

A continuación, compile el paquete.

```console
mvn package
```

Ejecute el siguiente comando `mvn` para ejecutar la aplicación.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```