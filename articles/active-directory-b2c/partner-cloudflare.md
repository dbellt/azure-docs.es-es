---
title: Tutorial para configurar Azure Active Directory B2C con Cloudflare Web Application Firewall
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con el firewall de aplicaciones web de Cloudflare para proteger las aplicaciones de ataques malintencionados.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/24/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 8494e9048ef298fbb3c2e70de10ec793f2bddec5
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108161616"
---
# <a name="tutorial-configure-clouldflare-with-azure-active-directory-b2c"></a>Tutorial: configuración de Cloudflare con Azure Active Directory B2C

En este tutorial de ejemplo, aprenderá a habilitar la solución [Web Application Firewall (WAF) Cloudflare](https://www.cloudflare.com/waf/) para el inquilino de Azure Active Directory (AD) B2C con dominio personalizado. Cloudflare WAF ayuda a su organización a protegerse contra ataques maliciosos que tienen como objetivo explotar vulnerabilidades como SQL y XSS.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- Una cuenta de [Cloudflare](https://dash.cloudflare.com/sign-up).

## <a name="scenario-description"></a>Descripción del escenario

La integración de Cloudflare WAF incluye los siguientes componentes:

- **Inquilino de Azure AD B2C**: el servidor de autorización, responsable de comprobar las credenciales del usuario en función de las directivas personalizadas definidas en el inquilino.  También se conoce como proveedor de identidades.

- [**Azure Front Door:** ](../frontdoor/front-door-overview.md) es el responsable de habilitar dominios personalizados para el inquilino de Azure B2C. Todo el tráfico de Cloudflare WAF se enrutará a Azure Front Door antes de llegar al inquilino de Azure AD B2C.

- **Cloudflare:** es el firewall de aplicaciones web, que administra todo el tráfico que se envía al servidor de autorización.

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

Para usar dominios personalizados en Azure AD B2C, es necesario usar la característica de dominio personalizado que proporciona Azure Front Door. Obtenga información sobre la [habilitación de dominios personalizados de Azure AD B2C](./custom-domain.md?pivots=b2c-user-flow).  

Una vez que el dominio de Azure AD B2C personalizado se haya configurado correctamente mediante Azure Front Door, [pruebe el dominio personalizado](./custom-domain.md?pivots=b2c-custom-policy#test-your-custom-domain) antes de continuar.  

## <a name="onboard-with-cloudflare"></a>Incorporación con Cloudflare

[Regístrese](https://dash.cloudflare.com/sign-up) y cree una cuenta de Cloudflare. Para habilitar WAF, se requiere un número mínimo de [SKU Pro](https://www.cloudflare.com/plans/).

### <a name="configure-dns"></a>Configurar el DNS

1. Para habilitar WAF para un dominio, debe **activar la configuración del proxy** desde la consola DNS para la entrada CNAME, tal como se muestra en el ejemplo de dominio id.contosobank.co.uk.

   ![En la imagen se muestra cómo seleccionar la configuración del proxy.](./media/partner-cloudflare/select-proxy-settings.png)

1. Alterne la opción **Estado del proxy** disponible en el panel DNS.

1. Después de cambiar el estado a **Proxied**, aparecerá de color naranja. La configuración final debe tener este aspecto:

   ![Imagen que muestra cómo seleccionar el estado Proxied](./media/partner-cloudflare/select-proxied.png)

### <a name="configure-waf"></a>Configuración de WAF

[Configure WAF](https://www.cloudflare.com/waf/) en la configuración de Cloudflare.

### <a name="configure-firewall-rule"></a>Configurar de la regla de firewall

Agregue, actualice o quite las reglas de firewall mediante la opción de firewall disponible en el panel superior de la consola. Por ejemplo, la siguiente configuración de firewall habilita una opción CAPTCHA para todas las solicitudes entrantes del dominio *contosobank.co.uk* antes de enviar la solicitud a Azure Front Door. Obtenga más información sobre la [configuración de las reglas de firewall](https://support.cloudflare.com/hc/articles/360016473712-Cloudflare-Firewall-Rules).

![Esta imagen muestra cómo aplicar un captcha.](./media/partner-cloudflare/configure-firewall-rule.png)

### <a name="test-the-settings"></a>Prueba de la configuración

1. Después de guardar la regla, es necesario completar la opción CAPTCHA cada vez que se solicita acceso al dominio personalizado.

   ![Esta imagen muestra cómo Cloudflare WAF aplica un CAPTCHA.](./media/partner-cloudflare/enforce-captcha.png)

   > [!NOTE]
   > Cloudflare también proporciona [**otras opciones**](https://support.cloudflare.com/hc/en-us/articles/200172706-Configuring-Custom-Pages-Error-and-Challenge-) para crear páginas de bloque personalizadas.  

2. El usuario accede a la directiva de Azure AD B2C después de completar correctamente el CAPTCHA.

   ![La imagen muestra el inicio de sesión de la directiva de Azure AD B2C.](./media/partner-cloudflare/azure-ad-b2c-policy.png)


## <a name="additional-resources"></a>Recursos adicionales

- [Solución de problemas de la página personalizada de Cloudflare](https://support.cloudflare.com/hc/en-us/articles/200172706-Configuring-Custom-Pages-Error-and-Challenge-#5QWV2KVjLnaAQ8L4tjiguw)

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy&tabs=applications)

## <a name="next-steps"></a>Pasos siguientes 
 
- [Configuración de un dominio personalizado en Azure AD B2C](./custom-domain.md?pivots=b2c-user-flow)
