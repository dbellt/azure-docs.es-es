---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: paolamvhz
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/12/2021
ms.topic: include
ms.custom: include file
ms.author: pvicencio
ms.openlocfilehash: b11be98718db282e219f3d1d151ceab235efe5e2
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109805421"
---
Introducción a Azure Communication Services mediante el SDK de SMS de Communication Services para Java para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](/java/azure/jdk/), versión 8 o posterior.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `mvn -v` para comprobar que Maven está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Abra la ventana de terminal o de comandos y navegue hasta el directorio en el que quiere crear la aplicación de Java. Ejecute el siguiente comando para generar el proyecto de Java a partir de la plantilla maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

El objetivo "generar" creó un directorio con el mismo nombre que el elemento artifactId. En este directorio, el directorio **src/main/java** contiene el código fuente del proyecto, el **directorio src/test/java** contiene el origen de la prueba, y el archivo **pom.xml** es el modelo de objetos del proyecto o POM.

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo **pom.xml** en el editor de texto. Agregue el siguiente elemento de dependencia al grupo de dependencias.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core</artifactId>
    <version>1.16.0</version> <!-- {x-version-update;com.azure:azure-core;dependency} -->
</dependency>
```

Abra **/src/main/java/com/communication/quickstart/App.java** en un editor de texto, agregue directivas de importación y quite la instrucción `System.out.println("Hello world!");`:

```java
package com.communication.quickstart;

import com.azure.communication.sms.models.*;
import com.azure.core.credential.AzureKeyCredential;
import com.azure.communication.sms.*;
import com.azure.core.util.Context;
import java.util.Arrays;

public class App
{
    public static void main( String[] args )
    {
        // Quickstart code goes here
    }
}

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de SMS de Azure Communication Services para Java:

| Nombre                                                             | Descripción                                                                                     |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| SmsClientBuilder              | Esta clase crea SmsClient. Se proporciona con el punto de conexión, la credencial y un cliente HTTP. |
| SmsClient                    | Esta clase es necesaria para la funcionalidad de los SMS, que se utiliza para enviar mensajes SMS.                |
| SmsSendOptions               | Esta clase proporciona opciones para agregar etiquetas personalizadas y configurar informes de entrega. Si deliveryReportEnabled se establece en True, se emitirá un evento cuando la entrega se realice correctamente. |        
| SmsSendResult                | Esta clase contiene el resultado del servicio SMS.                                          |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de `SmsClient` con su cadena de conexión. (La credencial es la `Key` de Azure Portal. Aprenda a [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string). Además, puede inicializar el cliente con cualquier cliente HTTP personalizado que implemente la interfaz `com.azure.core.http.HttpClient`.

Agregue el código siguiente al método `main`:

```java
// You can find your endpoint and access key from your resource in the Azure portal
String endpoint = "https://<resource-name>.communication.azure.com/";
AzureKeyCredential azureKeyCredential = new AzureKeyCredential("<access-key-credential>");

SmsClient smsClient = new SmsClientBuilder()
                .endpoint(endpoint)
                .credential(azureKeyCredential)
                .buildClient();
```

También puede proporcionar toda la cadena de conexión mediante la función connectionString() en lugar de proporcionar el punto de conexión y la clave de acceso.
```java
// You can find your connection string from your resource in the Azure portal
String connectionString = "https://<resource-name>.communication.azure.com/;<access-key>";

SmsClient smsClient = new SmsClientBuilder()
            .connectionString(connectionString)
            .buildClient();
```

## <a name="send-a-11-sms-message"></a>Envío de un SMS de un remitente a un solo destinatario

Para enviar un SMS a un solo destinatario, llame al método `send` desde el SmsClient con el número de teléfono de un solo destinatario. También puede incluir parámetros opcionales para especificar si el informe de entregas debe estar habilitado y establecer etiquetas personalizadas.

```java
SmsSendResult sendResult = smsClient.send(
                "<from-phone-number>",
                "<to-phone-number>",
                "Weekly Promotion");

System.out.println("Message Id: " + sendResult.getMessageId());
System.out.println("Recipient Number: " + sendResult.getTo());
System.out.println("Send Result Successful:" + sendResult.isSuccessful());
```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number>` por el número de teléfono al que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

## <a name="send-a-1n-sms-message-with-options"></a>Envío de un SMS de un remitente a varios destinatarios con opciones
Para enviar un SMS a una lista de destinatarios, llame al método `send` con una lista de los números de teléfono de los destinatarios. También puede incluir parámetros opcionales para especificar si el informe de entregas debe estar habilitado y establecer etiquetas personalizadas.
```java
SmsSendOptions options = new SmsSendOptions();
options.setDeliveryReportEnabled(true);
options.setTag("Marketing");

Iterable<SmsSendResult> sendResults = smsClient.sendWithResponse(
    "<from-phone-number>",
    Arrays.asList("<to-phone-number1>", "<to-phone-number2>"),
    "Weekly Promotion",
    options /* Optional */,
    Context.NONE).getValue();

for (SmsSendResult result : sendResults) {
    System.out.println("Message Id: " + result.getMessageId());
    System.out.println("Recipient Number: " + result.getTo());
    System.out.println("Send Result Successful:" + result.isSuccessful());
}
```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number-1>` y `<to-phone-number-2>` por los números de teléfono a los que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

El método `setDeliveryReportEnabled` se utiliza para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

El método `setTag` se usa para aplicar una etiqueta al informe de entrega.

## <a name="run-the-code"></a>Ejecución del código

Navegue hasta el directorio que contiene el archivo **pom.xml** y compile el proyecto mediante el siguiente comando `mvn`.

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
