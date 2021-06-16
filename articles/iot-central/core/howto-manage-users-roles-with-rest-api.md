---
title: Use la API REST para administrar usuarios y roles en Azure IoT Central
description: Uso de la API REST de IoT Central para administrar usuarios y roles en una aplicación
author: dominicbetts
ms.author: dobett
ms.date: 03/24/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 34be4920b343117ed895313c45a66e54b2de9ab3
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950582"
---
# <a name="how-to-use-the-iot-central-rest-api-to-manage-users-and-roles"></a>Uso de la API REST de IoT Central para administrar usuarios y roles

La API REST de IoT Central permite desarrollar aplicaciones cliente que se integran con las aplicaciones de IoT Central. Puede usar la API REST para administrar usuarios y roles en su aplicación de IoT Central.

Cada llamada a la API REST de IoT Central requiere un encabezado de autorización. Para obtener más información, vea los [procedimientos de autenticación y autorización de llamadas a la API REST de IoT Central](howto-authorize-rest-api.md).

Para ver la documentación de referencia sobre la API REST de IoT Central, consulte [Referencia de la API REST de Azure IoT Central](/rest/api/iotcentral/).

## <a name="manage-roles"></a>Administración de roles

La API REST permite enumerar los roles definidos en su aplicación de IoT Central. Use la siguiente solicitud para recuperar una lista de identificadores de rol de la aplicación:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/roles?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente, que incluye los tres roles integrados y un rol personalizado:

```json
{
  "value": [
    {
      "id": "ca310b8d-2f4a-44e0-a36e-957c202cd8d4",
      "displayName": "Administrator"
    },
    {
      "id": "ae2c9854-393b-4f97-8c42-479d70ce626e",
      "displayName": "Operator"
    },
    {
      "id": "344138e9-8de4-4497-8c54-5237e96d6aaf",
      "displayName": "Builder"
    },
    {
      "id": "16f8533f-6b82-478f-8ba8-7e676b541b1b",
      "displayName": "Example custom role"
    }
  ]
}
```

## <a name="manage-users"></a>Administrar usuarios

La API REST permite:

- Enumerar los usuarios de una aplicación.
- Recuperar los detalles de un usuario individual.
- Creación de un usuario
- Modificar un usuario.
- Eliminar un usuario

### <a name="list-users"></a>Enumerar usuarios

Use la siguiente solicitud para recuperar una lista de usuarios de la aplicación:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/users?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente. Los valores de rol identifican el identificador de rol al que está asociado el usuario:

```json
{
  "value": [
    {
      "id": "91907508-04fe-4349-91b5-b872f3055a95",
      "type": "email",
      "roles": [
        {
          "role": "ca310b8d-2f4a-44e0-a36e-957c202cd8d4"
        }
      ],
      "email": "user1@contoso.com"
    },
    {
      "id": "dc1c916b-a652-49ea-b128-7c465a54c759",
      "type": "email",
      "roles": [
        {
          "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
        }
      ],
      "email": "user2@contoso.com"
    },
    {
      "id": "3ab9375e-d2d9-42da-b419-6ae86a938321",
      "type": "email",
      "roles": [
        {
          "role": "344138e9-8de4-4497-8c54-5237e96d6aaf"
        }
      ],
      "email": "user3@contoso.com"
    },
    {
      "id": "fc5a250b-83fb-433d-892c-e0a144f68c2b",
      "type": "email",
      "roles": [
        {
          "role": "16f8533f-6b82-478f-8ba8-7e676b541b1b"
        }
      ],
      "email": "user4@contoso.com"
    }
  ]
}
```

### <a name="get-a-user"></a>Obtener un usuario

Use la siguiente solicitud para recuperar los detalles de un usuario individual desde la aplicación:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/users/dc1c916b-a652-49ea-b128-7c465a54c759?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente. El valor de rol indica el identificador de rol al que está asociado el usuario:

```json
{
  "id": "dc1c916b-a652-49ea-b128-7c465a54c759",
  "type": "email",
  "roles": [
    {
      "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
    }
  ],
  "email": "user2@contoso.com"
}
```

### <a name="create-a-user"></a>Creación de un usuario

Use la siguiente solicitud para crear un usuario en la aplicación. El identificador y el correo electrónico deben ser únicos en la aplicación:

```http
PUT https://{your app subdomain}.azureiotcentral.com/api/users/user-001?api-version=1.0
```

En el siguiente cuerpo de la solicitud, el valor `role` es para el rol de operador que recuperó anteriormente:

```json
{
  "id": "user-001",
  "type": "email",
  "roles": [
    {
      "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
    }
  ],
  "email": "user5@contoso.com"
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente. El valor de rol indica el rol al que está asociado el usuario:

```json
{
  "id": "user-001",
  "type": "email",
  "roles": [
    {
      "role": "ae2c9854-393b-4f97-8c42-479d70ce626e"
    }
  ],
  "email": "user5@contoso.com"
}
```

### <a name="change-the-role-of-a-user"></a>Cambio del rol de un usuario

Use la siguiente solicitud para cambiar el rol asignado al usuario. En este ejemplo se usa el identificador del rol de generador que recuperó anteriormente:

```http
PATCH https://{your app subdomain}.azureiotcentral.com/api/users/user-001?api-version=1.0
```

Cuerpo de la solicitud. El valor es para el rol de generador que recuperó anteriormente:

```json
{
  "roles": [
    {
      "role": "344138e9-8de4-4497-8c54-5237e96d6aaf"
    }
  ]
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "id": "user-001",
  "type": "email",
  "roles": [
    {
      "role": "344138e9-8de4-4497-8c54-5237e96d6aaf"
    }
  ],
  "email": "user5@contoso.com"
}
```

### <a name="delete-a-user"></a>Eliminar un usuario

Use la siguiente solicitud para eliminar un usuario:

```http
DELETE https://{your app subdomain}.azureiotcentral.com/api/users/user-001?api-version=1.0
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe administrar usuarios y roles con la API REST, el siguiente paso recomendado consiste en [administrar aplicaciones de IoT Central mediante la API REST](/learn/modules/manage-iot-central-apps-with-rest-api/).