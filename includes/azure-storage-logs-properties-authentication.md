---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: ca8963ed8928745a6d5918c86021199432339c83
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104612280"
---
| Propiedad | Descripción |
|:--- |:---|
|**identity/type** | Tipo de autenticación que se ha usado para realizar la solicitud. Por ejemplo: `OAuth`, `Kerberos`, `SAS Key`, `Account Key` o `Anonymous` |
|**identity/tokenHash**|El hash SHA-256 del token de autenticación utilizado en la solicitud. <br>Cuando el tipo de autenticación es `Account Key`, el formato es "clave 1 \| clave 2 (hash SHA 256 de la clave)". Por ejemplo: `key1(5RTE343A6FEB12342672AFD40072B70D4A91BGH5CDF797EC56BF82B2C3635CE)`. <br>Cuando el tipo de autenticación es `SAS Key`, el formato es "clave 1 \| clave 2 (hash SHA 256 de la clave), SasSignature (hash SHA 256 del token de SAS)". Por ejemplo: `key1(0A0XE8AADA354H19722ED12342443F0DC8FAF3E6GF8C8AD805DE6D563E0E5F8A),SasSignature(04D64C2B3A704145C9F1664F201123467A74D72DA72751A9137DDAA732FA03CF)`. Cuando el tipo de autenticación es `OAuth`, el formato es "hash SHA 256 del token de OAuth". Por ejemplo: `B3CC9D5C64B3351573D806751312317FE4E910877E7CBAFA9D95E0BE923DW25C`<br> No hay campo tokenHash para otros tipos de autenticación. |
|**authorization/action** | Acción asignada a la solicitud. |
|**authorization/roleAssignmentId** | Identificador de la asignación de roles. Por ejemplo: `4e2521b7-13be-4363-aeda-111111111111`.|
|**authorization/roleDefinitionId** | Identificador de la definición de roles. Por ejemplo: `ba92f5b4-2d11-453d-a403-111111111111"`.|
|**principals/id** | Identificador de la entidad de seguridad. Por ejemplo: `a4711f3a-254f-4cfb-8a2d-111111111111`.|
|**principals/type** | Tipo de la entidad de seguridad. Por ejemplo: `ServicePrincipal`. |
|**requester/appID** | Identificador de la aplicación de Open Authorization (OAuth) que se usa como solicitante. <br> Por ejemplo: `d3f7d5fe-e64a-4e4e-871d-333333333333`.|
|**requester/audience** | Audiencia de OAuth de la solicitud. Por ejemplo: `https://storage.azure.com`. |
|**requester/objectId** | Identificador de objeto de OAuth de la solicitud. En el caso de la autenticación Kerberos, representa al identificador de objeto del usuario autenticado de Kerberos. Por ejemplo: `0e0bf547-55e5-465c-91b7-2873712b249c`. |
|**requester/tenantId** | Identificador de inquilino de OAuth de la identidad. Por ejemplo: `72f988bf-86f1-41af-91ab-222222222222`.|
|**requester/tokenIssuer** | Emisor de token de OAuth. Por ejemplo: `https://sts.windows.net/72f988bf-86f1-41af-91ab-222222222222/`.|
|**requester/upn** | Nombre principal de usuario (UPN) del solicitante. Por ejemplo: `someone@contoso.com`. |
|**requester/userName** | Este campo está reservado para uso interno exclusivamente.|
