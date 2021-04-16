---
title: 'Conexión a la API de Azure Media Services v3: Python'
description: En este artículo se muestra cómo conectarse a la API de Media Services v3 con Python.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/18/2020
ms.author: inhenkel
ms.custom: devx-track-python
ms.openlocfilehash: 24e2ba4027dc818256dc9572f697fe7ec5a5a56b
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105960714"
---
# <a name="connect-to-media-services-v3-api---python"></a>Conexión a la API de Media Services v3: Python

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se muestra cómo conectarse al SDK de Python de Azure Media Services v3 con el método de inicio de sesión de la entidad de servicio.

## <a name="prerequisites"></a>Prerrequisitos

- Descargue Python desde [python.org](https://www.python.org/downloads/).
- Asegúrese de establecer la variable de entorno `PATH`.
- [Cree una cuenta de Media Services](./account-create-how-to.md). Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services.
- Siga los pasos descritos en el tema [API de acceso](./access-api-howto.md) y seleccione el método de autenticación de la entidad de servicio. Registre el id. de suscripción (`SubscriptionId`), el id. de cliente de la aplicación (`AadClientId`), la clave de autenticación (`AadSecret`) y el id. de inquilino (`AadTenantId`) que necesitará en pasos siguientes.

> [!IMPORTANT]
> Revise las [convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions).

## <a name="install-the-modules"></a>Instalación de los módulos

Para trabajar con Azure Media Services mediante Python, debe instalar estos módulos.

* El módulo `azure-mgmt-resource`, que incluye los módulos de Azure para Active Directory.
* El módulo `azure-mgmt-media`, que incluye las entidades de Media Services.

    Asegúrese de obtener la [versión más reciente del SDK de Media Services para Python](https://pypi.org/project/azure-mgmt-media/).

Abra una herramienta de línea de comandos y use los comandos siguientes para instalar los módulos.

```
pip3 install azure-mgmt-resource
pip3 install azure-mgmt-media==3.0.0
```

## <a name="connect-to-the-python-client"></a>Conexión con el cliente de Python

1. Cree un archivo con una extensión `.py`.
1. Abra el archivo en el editor que prefiera.
1. Agregue el código siguiente al archivo. El código importa los módulos necesarios y crea el objeto de credenciales de Active Directory que necesita para conectarse a Media Services.

      Establezca los valores de las variables en los valores que obtuvo de [Acceso a la API](./access-api-howto.md). Actualice las variables `ACCOUNT_NAME` y `RESOUCE_GROUP_NAME` a los nombres de cuenta de Media Services y de grupo de recursos que se usaron al crear esos recursos.

      ```
      import adal
      from azure.mgmt.media import AzureMediaServices
      from msrestazure.azure_active_directory import AdalAuthentication
      from msrestazure.azure_cloud import AZURE_PUBLIC_CLOUD

      RESOURCE = "https://management.core.windows.net/"
      # Tenant ID for your Azure Subscription (AadTenantId)
      TENANT_ID = "00000000-0000-0000-000000000000"
      # Your Service Principal App ID (AadClientId)
      CLIENT = "00000000-0000-0000-000000000000"
      # Your Service Principal Password (AadSecret)
      KEY = "00000000-0000-0000-000000000000"
      # Your Azure Subscription ID (SubscriptionId)
      SUBSCRIPTION_ID = "00000000-0000-0000-000000000000"
      # Your Azure Media Service (AMS) Account Name
      ACCOUNT_NAME = "amsaccount"
      # Your Resource Group Name
      RESOUCE_GROUP_NAME = "amsResourceGroup"

      LOGIN_ENDPOINT = AZURE_PUBLIC_CLOUD.endpoints.active_directory
      RESOURCE = AZURE_PUBLIC_CLOUD.endpoints.active_directory_resource_id

      context = adal.AuthenticationContext(LOGIN_ENDPOINT + "/" + TENANT_ID)
      credentials = AdalAuthentication(
          context.acquire_token_with_client_credentials, RESOURCE, CLIENT, KEY
      )

      # The AMS Client
      # You can now use this object to perform different operations to your AMS account.
      client = AzureMediaServices(credentials, SUBSCRIPTION_ID)

      print("signed in")

      # Now that you are authenticated, you can manipulate the entities.
      # For example, list assets in your AMS account
      print(client.assets.list(RESOUCE_GROUP_NAME, ACCOUNT_NAME).get(0))
      ```

1. Ejecute el archivo.

## <a name="next-steps"></a>Pasos siguientes

- Use el [SDK de Python](https://aka.ms/ams-v3-python-sdk).
- Revise la documentación de [referencia de Python](/python/api/overview/azure/mediaservices/management) de Media Services.
