---
title: Actualización de credenciales de entidad de servicio
description: Actualización de credenciales de una entidad de servicio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: dnethi
ms.author: dinethi
ms.reviewer: mikeray
ms.date: 12/09/2020
ms.topic: how-to
ms.openlocfilehash: ff1e43ca78583e5ccda56b7c813ec580050e3288
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110495731"
---
# <a name="update-service-principal-credentials"></a>Actualización de credenciales de entidad de servicio

Cuando cambien las credenciales de la entidad de servicio, deberá actualizar los secretos en el controlador de datos.

Por ejemplo, si implementó el controlador de datos con un conjunto específico de valores para el identificador de inquilino de la entidad de servicio, el identificador de cliente y el secreto de cliente y, luego, cambia uno o varios de estos valores, debe actualizar los secretos en el controlador de datos.  A continuación, se indican las instrucciones para actualizar el identificador de inquilino, el identificador de cliente o el secreto de cliente. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="background"></a>Información previa

La entidad de servicio se creó en [Creación de una entidad de servicio](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal). 

## <a name="steps"></a>Pasos

1. Acceda al secreto de la entidad de servicio en el editor predeterminado.

   ```console
   kubectl edit secret/upload-service-principal-secret -n <name of namespace>
   ```

   Por ejemplo, para editar el secreto de la entidad de servicio en un controlador de datos del espacio de nombres `arc`, ejecute el siguiente comando:

   ```console
   kubectl edit secret/upload-service-principal-secret -n arc
   ```

   El comando `kubecl edit` abre el archivo .yml de credenciales en el editor predeterminado. 


1. Escriba el secreto de la entidad de servicio. 

   En el editor predeterminado, reemplace los valores de la sección de datos por la información actualizada de las credenciales.

   Por ejemplo:

   ```console
   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: v1
   data:
     authority: aHR0cHM6Ly9sb2dpbi5taWNyb3NvZnRvbmxpbmUuY29t
     clientId: NDNiNDcwYrFTGWYzOC00ODhkLTk0ZDYtNTc0MTdkN2YxM2Uw
     clientSecret: VFA2RH125XU2MF9+VVhXenZTZVdLdECXFlNKZi00Lm9NSw==
     tenantId: NzJmOTg4YmYtODZmMRFVBGTJLSATkxYWItMmQ3Y2QwMTFkYjQ3
   kind: Secret
   metadata:
     creationTimestamp: "2020-12-02T05:02:04Z"
     name: upload-service-principal-secret
     namespace: arc
     resourceVersion: "7235659"
     selfLink: /api/v1/namespaces/arc/secrets/upload-service-principal-secret
     uid: 7fb693ff-6caa-4a31-b83e-9bf22be4c112
   type: Opaque
   ```

   Edite los valores de `clientID`, `clientSecret` o `tenantID`, según sea necesario. 

> [!NOTE]
>Los valores deben estar codificados en Base64. No edite ninguna otra propiedad.

Si se proporciona un valor incorrecto para `clientId`, `clientSecret` o `tenantID`, verá un mensaje de error como el siguiente en los registros del contenedor de pod/controller `control-xxxx`:

```output
YYYY-MM-DD HH:MM:SS.mmmm | ERROR | [AzureUpload] Upload task exception: A configuration issue is preventing authentication - check the error message from the server for details.You can modify the configuration in the application registration portal. See https://aka.ms/msal-net-invalid-client for details.  Original exception: AADSTS7000215: Invalid client secret is provided.
```



## <a name="next-steps"></a>Pasos siguientes

[Recuperación del nombre de usuario y la contraseña para conectarse al controlador de datos de Arc](retrieve-the-username-password-for-data-controller.md)

[Creación de una entidad de servicio](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal)
