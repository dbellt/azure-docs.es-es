---
title: archivo de inclusión
description: archivo de inclusión
services: media-services
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 05/01/2019
ms.author: juliako
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 9a9aaf723b7b89b5b9c5611cea05c22c4003b99a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92755719"
---
### <a name="access-the-media-services-api"></a>Acceso a la API de Media Services

Debe usar la autenticación de la entidad de servicio de Azure AD para conectarse a las API de Azure Media Services. El siguiente comando crea una aplicación de Azure AD y asocia una entidad de servicio a la cuenta. Debe usar los valores devueltos para configurar la aplicación.

Antes de ejecutar el script, debe reemplazar `amsaccount` y `amsResourceGroup` por los nombres que eligió al crear estos recursos. `amsaccount` es el nombre de la cuenta de Azure Media Services a la que asociar la entidad de servicio.

Si tiene acceso a varias suscripciones, establezca primero la suscripción activa en la suscripción en la que se creó la cuenta de Media Services.

```azurecli
az account set --subscription subscriptionId
```

El siguiente comando devuelve una salida `json`:

```azurecli
az ams account sp create --account-name amsaccount --resource-group amsResourceGroup
```

Este comando genera una respuesta similar a esta:

```json
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.microsoftonline.com",
  "AadSecret": "00000000-0000-0000-0000-000000000000",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": "amsaccount",
  "ArmAadAudience": "https://management.core.windows.net/",
  "ArmEndpoint": "https://management.azure.com/",
  "Region": "West US 2",
  "ResourceGroup": "amsResourceGroup",
  "SubscriptionId": "00000000-0000-0000-0000-000000000000"
}
```

Si desea obtener un `xml` en la respuesta, use el siguiente comando:

```azurecli
az ams account sp create --account-name amsaccount --resource-group amsResourceGroup --xml
```
