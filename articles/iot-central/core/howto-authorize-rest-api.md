---
title: Autorización de la API de REST en Azure IoT Central
description: Autenticación y autorización de llamadas API de REST en IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 03/24/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 73ef942b42858a3219502fe09c3b9281be81f964
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776805"
---
# <a name="how-to-authenticate-and-authorize-iot-central-rest-api-calls"></a>Autenticación y autorización de llamadas API de REST en IoT Central

La API de REST de IoT Central permite desarrollar aplicaciones cliente que se integran con las aplicaciones de IoT Central. Use la API de REST para trabajar con recursos de la aplicación de IoT Central, como plantillas de dispositivo, dispositivos, trabajos, usuarios y roles.

Cada llamada API de REST en IoT Central requiere un encabezado de autorización que IoT Central usa para determinar la identidad del autor de la llamada y los permisos que se conceden al autor de la llamada dentro de la aplicación.

En este artículo se describen los tipos de token que puede usar en el encabezado de autorización y cómo obtenerlos.

## <a name="token-types"></a>Tipos de token

Para acceder a la aplicación de IoT Central con la API de REST, puede usar lo siguiente:

- _Token de portador de Azure Active Directory_. Un token de portador se asocia a una cuenta de usuario de Azure Active Directory. El token concede al autor de la llamada los mismos permisos que el usuario tiene en la aplicación de IoT Central.
- Token de API de IoT Central. Un token de API está asociado a un rol en la aplicación de IoT Central.

Para obtener más información sobre los usuarios y roles de IoT Central, vea [Administrar usuarios y roles en la aplicación de IoT Central](howto-manage-users-roles.md).

## <a name="get-a-bearer-token"></a>Obtención de un token de portador

Para obtener un token de portador para la cuenta de usuario de Azure Active Directory, use los siguientes comandos de la CLI de Azure:

```azurecli
az login
az account get-access-token --resource https://apps.azureiotcentral.com
```

> [!IMPORTANT]
> El comando `az login` es necesario incluso si usa Cloud Shell.

La salida JSON del comando anterior es similar al ejemplo siguiente:

```json
{
  "accessToken": "eyJ0eX...fNQ",
  "expiresOn": "2021-03-22 11:11:16.072222",
  "subscription": "{your subscription id}",
  "tenant": "{your tenant id}",
  "tokenType": "Bearer"
}
```

El token de portador es válido durante una hora aproximadamente, pero después debe crear otro.

## <a name="get-an-api-token"></a>Obtención de un token de API

Para obtener un token de API, puede usar la interfaz de usuario de IoT Central o una llamada API de REST.

En la interfaz de usuario de IoT Central:

1. Vaya a **Administración > Tokens de API** .
1. Seleccione **+ Generar token**.
1. Escriba un nombre para el token y seleccione un rol.
1. Seleccione **Generar**.
1. IoT Central muestra el token similar al ejemplo siguiente:

    `SharedAccessSignature sr=5782ed70...&sig=dvZZE...&skn=operator-token&se=1647948035850`

    Esta pantalla es la única vez que puede ver el token de API; si lo pierde, debe generar uno nuevo.

Un token de API es válido durante un año aproximadamente. Puede generar tokens para roles integrados y personalizados en la aplicación de IoT Central.

Puede eliminar tokens de API en la interfaz de usuario de IoT Central si necesita revocar el acceso.

Mediante la API de REST:

1. Use la API de REST para recuperar una lista de identificadores de rol de la aplicación:

    ```http
    GET https://{your app subdomain}.azureiotcentral.com/api/roles?api-version=1.0
    ```

    La respuesta a esta solicitud es similar al ejemplo siguiente:

    ```json
    {
      "value": [
        {
          "displayName": "Administrator",
          "id": "ca310b8d-2f4a-44e0-a36e-957c202cd8d4"
        },
        {
          "displayName": "Operator",
          "id": "ae2c9854-393b-4f97-8c42-479d70ce626e"
        },
        {
          "displayName": "Builder",
          "id": "344138e9-8de4-4497-8c54-5237e96d6aaf"
        }
      ]
    }
    ```

1. Use la API de REST para crear un token de API para un rol. Por ejemplo, para crear un token de API llamado `operator-token` para el rol de operador:

    ```http
    PUT https://{your app subdomain}.azureiotcentral.com/api/roles/operator-token?api-version=1.0
    ```

    Cuerpo de la solicitud:

    ```json
    {
      "roles": [
        {
          "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
        }
      ]
    }
    ```

    La respuesta al comando anterior es similar al siguiente elemento JSON:

    ```json
    {
      "expiry": "2022-03-22T12:01:27.889Z",
      "id": "operator-token",
      "roles": [
        {
          "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
        }
      ],
      "token": "SharedAccessSignature sr=e8a...&sig=jKY8W...&skn=operator-token&se=1647950487889"
    }
    ```

    Esta respuesta es la única vez que tiene acceso al token de API, si lo pierde, debe generar uno nuevo.

Puede usar la API de REST para enumerar y eliminar tokens de API en una aplicación.

## <a name="use-a-bearer-token"></a>Uso de un token de portador

Para usar un token de portador al realizar una llamada API de REST, el encabezado de autorización tiene un aspecto similar al del ejemplo siguiente:

`Authorization: Bearer eyJ0eX...fNQ`

## <a name="use-an-api-token"></a>Uso de un token de API

Para usar un token de API al realizar una llamada API de REST, el encabezado de autorización tiene un aspecto similar al del ejemplo siguiente:

`Authorization: SharedAccessSignature sr=e8a...&sig=jKY8W...&skn=operator-token&se=1647950487889`

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a autorizar las llamadas API de REST, el siguiente paso sugerido es [Uso de la API de REST de IoT Central para administrar usuarios y roles](howto-manage-users-roles-with-rest-api.md).
