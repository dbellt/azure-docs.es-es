---
title: Migración al nuevo portal para desarrolladores desde el portal para desarrolladores heredado
titleSuffix: Azure API Management
description: Obtenga información sobre cómo migrar desde el portal para desarrolladores heredado al nuevo portal para desarrolladores en API Management.
services: api-management
documentationcenter: API Management
author: mikebudzynski
ms.service: api-management
ms.topic: article
ms.date: 04/15/2021
ms.author: apimpm
ms.openlocfilehash: e4f9f3822b58886f7d453d52402b078d8401133f
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107738889"
---
# <a name="migrate-to-the-new-developer-portal"></a>Migración al nuevo portal para desarrolladores

En este artículo se describen los pasos que debe seguir para migrar desde el portal heredado obsoleto al nuevo portal para desarrolladores en API Management.

> [!IMPORTANT]
> El portal para desarrolladores heredado está en desuso y solo recibirá actualizaciones de seguridad. Puede seguir utilizándolo como de costumbre hasta su retirada en octubre de 2023, momento en el que se quitará de todos los servicios de API Management.

![API Management developer portal](media/api-management-howto-developer-portal/cover.png)

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="improvements-in-new-developer-portal"></a>Mejoras en el nuevo portal para desarrolladores

El nuevo portal para desarrolladores resuelve muchas limitaciones del portal en desuso. Incluye un [editor visual de arrastrar y colocar para editar contenido](api-management-howto-developer-portal-customize.md) y un panel dedicado para que los diseñadores modifiquen el sitio web. Las páginas, las personalizaciones y la configuración se guardan como recursos de Azure Resource Manager en el servicio API Management, lo que le permite [automatizar las implementaciones del portal](automate-portal-deployments.md). Por último, el código base del portal está abierto, por lo que [puede ampliarlo con funcionalidad personalizada](api-management-howto-developer-portal.md#managed-vs-self-hosted).

## <a name="how-to-migrate-to-new-developer-portal"></a>Migración al nuevo portal para desarrolladores

El nuevo portal para desarrolladores no es compatible con el portal para desarrolladores en desuso y la migración automática no es posible. Debe volver a crear manualmente el contenido (páginas, texto, archivos multimedia) y personalizar la apariencia del nuevo portal. Los pasos precisos variarán en función de las personalizaciones y la complejidad del portal. Consulte [el tutorial del portal para desarrolladores](api-management-howto-developer-portal-customize.md) para obtener instrucciones. La configuración restante, como la lista de API, productos, usuarios o proveedores de identidades, se comparte automáticamente entre ambos portales.

> [!IMPORTANT]
> Si ha iniciado el nuevo portal para desarrolladores anteriormente pero no ha realizado ningún cambio, restablezca el contenido predeterminado para actualizarlo a la versión más reciente.

Al migrar desde el portal en desuso, tenga en cuenta los siguientes cambios:

- Si expone el portal para desarrolladores a través de un dominio personalizado, [asigne un dominio](configure-custom-domain.md) al nuevo portal para desarrolladores. Use la opción **Portal para desarrolladores** en la lista desplegable de Azure Portal.
- [Aplique una directiva de CORS](developer-portal-faq.md#cors) en las API para habilitar la consola de prueba interactiva.
- Si inserta CSS personalizado para aplicar estilo al portal, debe [replicar el estilo con el panel de diseño integrado](api-management-howto-developer-portal-customize.md). No se permite la inserción de CSS en el nuevo portal.
- Puede insertar JavaScript personalizado solo en la [versión autohospedada del nuevo portal](api-management-howto-developer-portal.md#managed-vs-self-hosted).
- Si API Management está en una red virtual y se expone a Internet a través de Application Gateway, [consulte este artículo de la documentación](api-management-howto-integrate-internal-vnet-appgateway.md) para obtener pasos de configuración precisos. Necesita:

    - Habilitar la conectividad con el punto de conexión de administración de API Management.
    - Habilitar la conectividad con el nuevo punto de conexión del portal.
    - Deshabilitar las reglas de Web Application Firewall seleccionadas.

- Si cambió las plantillas de notificación de correo electrónico predeterminadas para incluir una dirección URL de portal en desuso definida explícitamente, cámbielas para usar el parámetro de la dirección URL del portal o para apuntar a la dirección URL del nuevo portal. Si en su lugar las plantillas usan el parámetro de dirección URL del portal integrado, no se requieren cambios.
- Las características *Problemas* y *Aplicaciones* no se admiten en el nuevo portal para desarrolladores.
- La integración directa con Facebook, Microsoft, Twitter y Google como proveedores de identidades no se admite en el nuevo portal para desarrolladores. Puede realizar la integración con estos proveedores a través de Azure AD B2C.
- Si usa la delegación, cambie la dirección URL de retorno en las aplicaciones y use el [*punto de conexión Get Shared Access Token* API](/rest/api/apimanagement/2019-12-01/user/getsharedaccesstoken) en lugar del punto de conexión *Generate SSO URL*.
- Si usa Azure AD como proveedor de identidades:

    - Cambie la dirección URL de retorno en la aplicación para que apunte al nuevo dominio del portal para desarrolladores.
    - Modifique el sufijo de la dirección URL de retorno en la aplicación de `/signin-aad` a `/signin`.

- Si usa Azure AD B2C como proveedor de identidades:

    - Cambie la dirección URL de retorno en la aplicación para que apunte al nuevo dominio del portal para desarrolladores.
    - Modifique el sufijo de la dirección URL de retorno en la aplicación de `/signin-aad` a `/signin`.
    - Incluya el *Nombre propio*, el *Apellido* y el *Identificador de objeto del usuario* en las notificaciones de la aplicación.

- Si usa OAuth 2.0 en la consola de prueba interactiva, cambie la dirección URL de retorno en la aplicación para que apunte al nuevo dominio del portal para desarrolladores y modifique el sufijo:

    - De `/docs/services/[serverName]/console/oauth2/authorizationcode/callback` a `/signin-oauth/code/callback/[serverName]` para el flujo de concesión de código de autorización.
    - De `/docs/services/[serverName]/console/oauth2/implicit/callback` a `/signin-oauth/implicit/callback` para el flujo de concesión implícita.
- Si usa OpenID Connect en la consola de prueba interactiva, cambie la dirección URL de retorno en la aplicación para que apunte al nuevo dominio del portal para desarrolladores y modifique el sufijo:

    - De `/docs/services/[serverName]/console/openidconnect/authorizationcode/callback` a `/signin-oauth/code/callback/[serverName]` para el flujo de concesión de código de autorización.
    - De `/docs/services/[serverName]/console/openidconnect/implicit/callback` a `/signin-oauth/implicit/callback` para el flujo de concesión implícita.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)
- [Acceso al portal para desarrolladores y su personalización](api-management-howto-developer-portal-customize.md)