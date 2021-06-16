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
ms.openlocfilehash: 524770186c9bb391b9beb63620b13c99bf6dc4d0
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430310"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-python-quickstarts/tree/main/access-tokens-quickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7 o 3.6+.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

   ```console
   mkdir access-tokens-quickstart && cd access-tokens-quickstart
   ```

1. Use un editor de texto para crear un archivo denominado **issue-access-tokens.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

   ```python
   import os
   from azure.communication.identity import CommunicationIdentityClient, CommunicationUserIdentifier

   try:
      print("Azure Communication Services - Access Tokens Quickstart")
      # Quickstart code goes here
   except Exception as ex:
      print("Exception:")
      print(ex)
   ```

### <a name="install-the-package"></a>Instalar el paquete

En el directorio de aplicaciones, instale Identity SDK de Azure Communication Services para Python con el comando `pip install`.

```console
pip install azure-communication-identity
```

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue este código dentro del bloque `try`:

```python
# This code demonstrates how to fetch your connection string
# from an environment variable.
connection_string = os.environ["COMMUNICATION_SERVICES_CONNECTION_STRING"]

# Instantiate the identity client
client = CommunicationIdentityClient.from_connection_string(connection_string)
```

Como alternativa, si ha configurado una identidad administrada, consulte [Uso de identidades administradas](../managed-identity.md); también puede autenticarse con la identidad administrada.
```python
endpoint = os.environ["COMMUNICATION_SERVICES_ENDPOINT"]
client = CommunicationIdentityClient(endpoint, DefaultAzureCredential())
```

## <a name="create-an-identity"></a>Creación de una identidad

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `create_user` para crear una nueva entrada en el directorio con un `Id` único. Almacene la identidad recibida con la asignación a los usuarios de la aplicación. Por ejemplo, almacenándolos en la base de datos del servidor de aplicaciones. La identidad es necesaria más adelante para emitir tokens de acceso.

```python
identity = client.create_user()
print("\nCreated an identity with ID: " + identity.properties['id'])
```

## <a name="issue-access-tokens"></a>Emitir tokens de acceso

Use el método `get_token` para emitir un token de acceso para la identidad de Communication Services existente. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md). Se puede crear una nueva instancia del parámetro `CommunicationUserIdentifier` en función de la representación de cadena de la identidad de Azure Communication Services.

```python
# Issue an access token with the "voip" scope for an identity
token_result = client.get_token(identity, ["voip"])
expires_on = token_result.expires_on.strftime("%d/%m/%y %I:%M %S %p")
print("\nIssued an access token with 'voip' scope that expires at " + expires_on + ":")
print(token_result.token)
```

Los tokens de acceso son credenciales de corta duración que deben volver a emitirse. No hacerlo puede provocar una interrupción de la experiencia de los usuarios respecto a la aplicación. La propiedad de respuesta `expires_on` indica la duración del token de acceso.

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Cree una identidad y emita un token de acceso dentro de la misma solicitud.

Use el método `create_user_and_token` para crear una identidad de Communication Services y emitir un token de acceso para ella. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md).

```python
# Issue an identity and an access token with the "voip" scope for the new identity
identity_token_result = client.create_user_and_token(["voip"])
identity = identity_token_result[0].properties['id']
token = identity_token_result[1].token
expires_on = identity_token_result[1].expires_on.strftime("%d/%m/%y %I:%M %S %p")
print("\nCreated an identity with ID: " + identity)
print("\nIssued an access token with 'voip' scope that expires at " + expires_on + ":")
print(token)
```

## <a name="refresh-access-tokens"></a>Tokens de acceso de actualización

Para actualizar un token de acceso, use el objeto `CommunicationUserIdentifier` para volver a emitir:

```python
# Value existingIdentity represents identity of Azure Communication Services stored during identity creation
identity = CommunicationUserIdentifier(existingIdentity)
token_result = client.get_token( identity, ["voip"])
```

## <a name="revoke-access-tokens"></a>Revocación de los tokens de acceso

En algunos casos, puede revocar explícitamente los tokens de acceso. Por ejemplo, cuando el usuario de una aplicación cambia la contraseña que usa para autenticarse en el servicio. El método `revoke_tokens` invalida todos los tokens de acceso activos emitidos para la identidad.

```python
client.revoke_tokens(identity)
print("\nSuccessfully revoked all access tokens for identity with ID: " + identity.properties['id'])
```

## <a name="delete-an-identity"></a>Eliminación de una identidad

La eliminación de una identidad revoca todos los tokens de acceso activos e impide que se emitan tokens de acceso posteriores para la identidad. También quita todo el contenido conservado asociado a la identidad.

```python
client.delete_user(identity)
print("\nDeleted the identity with ID: " + identity.properties['id'])
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, vaya al directorio que contiene el archivo *issue-access-tokens.py* y ejecute el siguiente comando `python` para ejecutar la aplicación.

```console
python ./issue-access-tokens.py
```
