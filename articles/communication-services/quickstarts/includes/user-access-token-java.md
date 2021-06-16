---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: c0b90279b4c3c65fdc96867f5d9fff9648320294
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430057"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-java-quickstarts/tree/main/access-token-quickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Kit de desarrollo de Java (JDK)](/azure/developer/java/fundamentals/java-jdk-install), versión 8 o posterior.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Un recurso de Communication Services implementado y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-java-application"></a>Creación de una aplicación Java

Abra el terminal o la ventana de comandos. Vaya al directorio en el que quiere crear la aplicación Java. Ejecute el siguiente comando para generar el proyecto de Java a partir de la plantilla maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Observará que la tarea "generar" creó un directorio con el mismo nombre que el objeto `artifactId`. En este directorio, el directorio src/main/java contiene el código fuente del proyecto, el directorio `src/test/java directory` contiene el origen de la prueba y el archivo `pom.xml` es el modelo de objetos del proyecto o POM.

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo **pom.xml** en el editor de texto. Agregue el siguiente elemento de dependencia al grupo de dependencias.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-identity</artifactId>
    <version>1.0.0</version>
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

import com.azure.communication.common.*;
import com.azure.communication.identity.*;
import com.azure.communication.identity.models.*;
import com.azure.core.credential.*;

import java.io.IOException;
import java.time.*;
import java.util.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Access Tokens Quickstart");
        // Quickstart code goes here
    }
}
```

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de `CommunicationIdentityClient` con la clave de acceso y el punto de conexión del recurso. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string). Además, puede inicializar el cliente con cualquier cliente HTTP personalizado que implemente la interfaz `com.azure.core.http.HttpClient`.

Agregue el código siguiente al método `main`:

```java
// Your can find your endpoint and access key from your resource in the Azure portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
String accessKey = "SECRET";

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
        .endpoint(endpoint)
        .credential(new AzureKeyCredential(accessKey))
        .buildClient();
```

También puede proporcionar toda la cadena de conexión mediante la función `connectionString()`, en vez de proporcionar un punto de conexión y una clave de acceso.
```java
// Your can find your connection string from your resource in the Azure portal
String connectionString = "<connection_string>";

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
    .connectionString(connectionString)
    .buildClient();
```

Si ha configurado una identidad administrada, consulte [Uso de identidades administradas](../managed-identity.md); también puede autenticarse con la identidad administrada.
```java
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
TokenCredential credential = new DefaultAzureCredentialBuilder().build();

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
        .endpoint(endpoint)
        .credential(credential)
        .buildClient();
```

## <a name="create-an-identity"></a>Creación de una identidad

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Almacene la identidad recibida con la asignación a los usuarios de la aplicación. Por ejemplo, almacenándolos en la base de datos del servidor de aplicaciones. La identidad es necesaria más adelante para emitir tokens de acceso.

```java
CommunicationUserIdentifier user = communicationIdentityClient.createUser();
System.out.println("\nCreated an identity with ID: " + user.getId());
```

## <a name="issue-access-tokens"></a>Emitir tokens de acceso

Use el método `getToken` para emitir un token de acceso para la identidad de Communication Services existente. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md). Se puede crear una nueva instancia del parámetro `user` en función de la representación de cadena de la identidad de Azure Communication Services.

```java
// Issue an access token with the "voip" scope for a user identity
List<CommunicationTokenScope> scopes = new ArrayList<>(Arrays.asList(CommunicationTokenScope.VOIP));
AccessToken accessToken = communicationIdentityClient.getToken(user, scopes);
OffsetDateTime expiresAt = accessToken.getExpiresAt();
String token = accessToken.getToken();
System.out.println("\nIssued an access token with 'voip' scope that expires at: " + expiresAt + ": " + token);
```

## <a name="create-an-identity-and-issue-token-in-one-call"></a>Creación de una identidad y emisión de un token en una llamada

También puede usar el método "createUserAndToken" para crear una entrada nueva en el directorio con un valor de `Id` único y emitir un token de acceso.

```java
List<CommunicationTokenScope> scopes = Arrays.asList(CommunicationTokenScope.CHAT);
CommunicationUserIdentifierAndToken result = communicationIdentityClient.createUserAndToken(scopes);
CommunicationUserIdentifier user = result.getUser();
System.out.println("\nCreated a user identity with ID: " + user.getId());
AccessToken accessToken = result.getUserToken();
OffsetDateTime expiresAt = accessToken.getExpiresAt();
String token = accessToken.getToken();
System.out.println("\nIssued an access token with 'chat' scope that expires at: " + expiresAt + ": " + token);
```

Los tokens de acceso son credenciales de corta duración que deben volver a emitirse. No hacerlo puede provocar una interrupción de la experiencia de los usuarios respecto a la aplicación. La propiedad `expiresAt` indica la duración del token de acceso.

## <a name="refresh-access-tokens"></a>Tokens de acceso de actualización

Para actualizar un token de acceso, use el objeto `CommunicationUserIdentifier` para volver a emitir:

```java
// Value existingIdentity represents identity of Azure Communication Services stored during identity creation
CommunicationUserIdentifier identity = new CommunicationUserIdentifier(existingIdentity);
AccessToken response = communicationIdentityClient.getToken(identity, scopes);
```

## <a name="revoke-access-tokens"></a>Revocación de los tokens de acceso

En algunos casos, puede revocar explícitamente los tokens de acceso. Por ejemplo, cuando el usuario de una aplicación cambia la contraseña que usa para autenticarse en el servicio. El método `revokeTokens` invalida todos los tokens de acceso activos emitidos para la identidad.

```java
communicationIdentityClient.revokeTokens(user);
System.out.println("\nSuccessfully revoked all access tokens for user identity with ID: " + user.getId());
```

## <a name="delete-an-identity"></a>Eliminación de una identidad

La eliminación de una identidad revoca todos los tokens de acceso activos e impide que se emitan tokens de acceso posteriores para la identidad. También quita todo el contenido conservado asociado a la identidad.

```java
communicationIdentityClient.deleteUser(user);
System.out.println("\nDeleted the user identity with ID: " + user.getId());
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