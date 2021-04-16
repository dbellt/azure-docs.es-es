---
title: Petición de consentimiento inesperada al iniciar sesión en una aplicación | Microsoft Docs
description: Cómo solucionar problemas cuando un usuario ve una solicitud de consentimiento para una aplicación que ha integrado con Azure AD que no esperaba
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: iangithinji
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 37e4384b5a1b400f11b7b7d6ab15beec4d2f8de9
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378066"
---
# <a name="unexpected-consent-prompt-when-signing-in-to-an-application"></a>Petición de consentimiento inesperada al iniciar sesión en una aplicación

Muchas aplicaciones que se integran con Azure Active Directory requieren permisos para acceder a diversos recursos para poder ejecutarse. Cuando estos recursos también se integran con Azure Active Directory, suelen solicitarse permisos para acceder a ellos mediante el marco de consentimiento de Azure AD. 

Esto da como resultado que la primera vez que se usa una aplicación aparece una solicitud de consentimiento, que suele ser una operación única. 

> [!VIDEO https://www.youtube.com/embed/a1AjdvNDda4]

## <a name="scenarios-in-which-users-see-consent-prompts"></a>Escenarios en los que los usuarios ven solicitudes de consentimiento

Se pueden esperar más solicitudes en distintos escenarios:

* El conjunto de permisos que requiere la aplicación ha cambiado.

* El usuario que originalmente otorgó su consentimiento a la aplicación no era administrador, y ahora un usuario diferente (no administrador) está usando la aplicación por primera vez.

* El usuario que originalmente otorgó su consentimiento a la aplicación era administrador, pero no otorgó el consentimiento en nombre de toda la organización.

* La aplicación usa [consentimiento incremental y dinámico](../azuread-dev/azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent) para solicitar permisos adicionales después de haberse otorgado inicialmente el consentimiento. Esto se suele usar cuando las características opcionales de una aplicación adicional requieren permisos más allá de los que se requieren para la funcionalidad de línea base.

* Se ha revocado el consentimiento después de que inicialmente se otorgó.

* El desarrollador ha configurado la aplicación para que requiera una solicitud de consentimiento cada vez que se utilice (nota: esto no es recomendable).

## <a name="next-steps"></a>Pasos siguientes

-   [Aplicaciones, permisos y consentimiento en Azure Active Directory (punto de conexión v1.0)](../develop/quickstart-register-app.md)

-   [Ámbitos, permisos y consentimiento en Azure Active Directory (punto de conexión v2.0)](../develop/v2-permissions-and-consent.md)
