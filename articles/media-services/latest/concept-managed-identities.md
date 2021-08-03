---
title: Identidades administradas
description: Media Services puede usarse con identidades administradas de Azure.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.date: 05/17/2021
ms.author: inhenkel
ms.openlocfilehash: 02f832c7fc25003950d6a112d951c07d1332e08a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110454374"
---
# <a name="managed-identities"></a>Identidades administradas

Un desafío común para los desarrolladores es la administración de secretos y credenciales para proteger la comunicación entre distintos servicios. En Azure, las identidades administradas eliminan la necesidad de que los desarrolladores administren credenciales, al proporcionar una identidad para el recurso de Azure en Azure AD y usarla para obtener tokens de Azure Active Directory (Azure AD).

## <a name="media-services-managed-identity-scenarios"></a>Escenarios de identidad administrada de Media Services

Actualmente hay tres escenarios en los que las identidades administradas se pueden usar con Media Services:

- [Conceder a una cuenta de Media Services acceso a Key Vault para habilitar las claves administradas por el cliente](security-encrypt-data-managed-identity-cli-tutorial.md).
- [Conceder a una cuenta de Media Services acceso a las cuentas de almacenamiento para permitir a Media Services omitir las ACL de red de Azure Storage](security-access-storage-managed-identity-cli-tutorial.md).
- Permitir que otros servicios (por ejemplo, VM o [Azure Functions](security-function-app-managed-identity-cli-tutorial.md)) accedan a Media Services.

En los dos primeros escenarios, la identidad administrada se usa para conceder a la *cuenta de Media Services* acceso a otros servicios.  En el tercer escenario, *el servicio* tiene una identidad administrada que se usa para acceder a Media Services.

:::image type="content" source="media/diagrams/managed-identities-scenario-comparison.svg" alt-text="Comparación de los escenarios de identidades administradas":::

> [!NOTE]
> Estos escenarios se pueden combinar. Puede crear identidades administradas para la cuenta de Media Services (por ejemplo, para acceder a las claves administradas por el cliente) y el recurso de Azure Functions para acceder a la cuenta de Media Services.

## <a name="tutorials-and-how-tos"></a>Tutoriales y procedimientos

Pruebe estos tutoriales para obtener experiencia práctica con el uso de una identidad administrada con Media Services.

- [CLI: cifrar datos en una cuenta de Media Services con una clave de Key Vault](security-encrypt-data-managed-identity-cli-tutorial.md)
- [CLI: permitir que Media Services acceda a una cuenta de almacenamiento configurada para bloquear las solicitudes de direcciones IP desconocidas](security-access-storage-managed-identity-cli-tutorial.md)
- [CLI: proporcionar a una aplicación de funciones acceso a una cuenta de Media Services](security-function-app-managed-identity-cli-tutorial.md)
- [PORTAL: usar Azure Portal para emplear claves administradas por el cliente o BYOK con Media Services](security-customer-managed-keys-portal-tutorial.md)
- [POSTMAN/REST: usar claves administradas por el cliente o BYOK con la API de REST de Media Services](security-customer-managed-keys-rest-postman-tutorial.md)

## <a name="further-reading"></a>Lecturas adicionales

Para obtener más información sobre las funciones de las identidades administradas y su uso con las aplicaciones de Azure, consulte [Identidades administradas de Azure AD](../../active-directory/managed-identities-azure-resources/overview.md).

Para obtener más información sobre Azure Functions, consulte [Acerca de Azure Functions](../../azure-functions/functions-overview.md).
