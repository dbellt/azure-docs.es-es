---
title: Tutorial para la configuración de Azure Active Directory B2C con Zscaler
titleSuffix: Azure AD B2C
description: Aprenda cómo integrar la autenticación de Azure AD B2C con Zscaler.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/09/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 9cd193eb6ff2858440f1cd9a62bdd53d58d6047d
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256300"
---
# <a name="tutorial-configure-zscaler-private-access-with-azure-active-directory-b2c"></a>Tutorial: Configuración de Zscaler Private Access con Azure Active Directory B2C

En este tutorial, aprenderá a integrar la autenticación de Azure Active Directory B2C (Azure AD B2C) con [Zscaler Private Access (ZPA)](https://www.zscaler.com/products/zscaler-private-access). ZPA ofrece acceso seguro y basado en directivas a aplicaciones y recursos privados sin el costo, los inconvenientes o los riesgos de seguridad que presenta una red privada virtual (VPN). La oferta de acceso híbrido seguro de Zscaler, en combinación con Azure AD B2C, permite una superficie sin ataques en aplicaciones orientadas al consumidor.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, va a necesitar:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).  
- [Un inquilino de Azure AD B2C](./tutorial-create-tenant.md) vinculado a la suscripción de Azure.  
- [Una suscripción a ZPA](https://azuremarketplace.microsoft.com/marketplace/apps/aad.zscalerprivateaccess?tab=Overview).

## <a name="scenario-description"></a>Descripción del escenario

La integración de ZPA incluye los siguientes componentes:

- **Azure AD B2C**: el proveedor de identidades (IdP) responsable de comprobar las credenciales del usuario. También es responsable de suscribir nuevos usuarios.  
- **ZPA**: el servicio responsable de proteger la aplicación web al exigir [acceso de confianza cero](https://www.microsoft.com/security/blog/2018/12/17/zero-trust-part-1-identity-and-access-management/#:~:text=Azure%20Active%20Directory%20%28Azure%20AD%29%20provides%20the%20strong%2C,to%20express%20their%20access%20requirements%20in%20simple%20terms.).  
- **Aplicación web**: hospeda el servicio al que el usuario intenta acceder.

En el diagrama siguiente se muestra cómo ZPA se integra con Azure AD B2C.

![Diagrama de la arquitectura de Zscaler, que muestra cómo se integra ZPA con Azure AD B2C.](media/partner-zscaler/zscaler-architecture-diagram.png)

En la tabla siguiente se describe la secuencia:

|Paso | Descripción |
| :-----:| :-----------|
| 1 | El usuario llega a un portal de usuarios de ZPA o a una aplicación de acceso mediante el explorador a ZPA.
| 2 | ZPA necesita la información de contexto del usuario para decidir si le permite acceder a la aplicación web. Para autenticar al usuario, ZPA realiza una redirección SAML a la página de inicio de sesión de Azure AD B2C.  
| 3 | El usuario llega a la página de inicio de sesión de Azure AD B2C. Los nuevos usuarios deben registrarse para crear una cuenta. Un usuario existente inicia la sesión con las credenciales que ya tiene. Azure AD B2C valida la identidad del usuario.
| 4 | Tras una autenticación correcta, Azure AD B2C redirige al usuario de vuelta a ZPA junto con la aserción de SAML. ZPA comprueba la aserción de SAML y establece el contexto del usuario.
| 5 | ZPA evalúa las directivas de acceso para el usuario. Si al usuario se le permite acceder a la aplicación web, se permitirá la conexión.

## <a name="onboard-to-zpa"></a>Incorporación a ZPA

En este tutorial se supone que ya tiene una configuración de ZPA que funciona. Si está comenzando con ZPA, consulte la [guía de configuración paso a paso de ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa).

## <a name="integrate-zpa-with-azure-ad-b2c"></a>Integración de ZPA con Azure AD B2C

### <a name="step-1-configure-azure-ad-b2c-as-an-idp-on-zpa"></a>Paso 1: Configuración de Azure AD B2C como un IdP en ZPA

Para configurar Azure AD B2C como un [IdP de ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign), haga lo siguiente:

1. Inicie sesión en el [portal de administración de ZPA](https://admin.private.zscaler.com).

1. Vaya a **Administration** > **IdP Configuration** (Administración > Configuración del IdP).

1. Seleccione **Add IdP Configuration** (Agregar configuración del IdP).

   Se abre el panel **Add IdP Configuration** (Agregar configuración del IdP).

   ![Captura de pantalla de la pestaña "IdP Information" (Información de IdP), en el panel "Add IdP Configuration" (Agregar configuración del IdP).](media/partner-zscaler/add-idp-configuration.png)

1. Seleccione la pestaña **IdP Information** (Información de IdP) y haga lo siguiente:

   a. En el cuadro **Name** (Nombre), escriba **Azure AD B2C**.  
   b. En **Single Sign-On** (Inicio de sesión único), seleccione **User** (Usuario).  
   c. En la lista desplegable **Domains** (Dominios), seleccione los dominios de autenticación que desea asociar a este IdP.

1. Seleccione **Next** (Siguiente).

1. Seleccione la pestaña **SP Metadata** (Metadatos del SP) y haga lo siguiente:

   a. En **Service Provider URL** (Dirección URL del proveedor de servicios), copie o anote el valor para su uso posterior.  
   b. En **Service Provider Entity ID** (Id. de entidad del proveedor de servicios), copie o anote el valor para su uso posterior.

   ![Captura de pantalla de la pestaña "SP Metadata" (Metadatos del SP) en el panel "Add IdP Configuration" (Agregar configuración del IdP).](media/partner-zscaler/sp-metadata.png)

1. Seleccione **Pause**(Pausar).

Después de configurar Azure AD B2C, se reanudará el resto de la configuración del IdP.

### <a name="step-2-configure-custom-policies-in-azure-ad-b2c"></a>Paso 2: Configuración de directivas personalizadas en Azure AD B2C

>[!Note]
>Este paso solo es necesario si todavía no ha configurado las directivas personalizadas. Si ya tiene una o varias directivas personalizadas, puede omitirlo.

Para configurar directivas personalizadas en el inquilino de Azure AD B2C, consulte [Introducción a las directivas personalizadas en Azure Active Directory B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy).

### <a name="step-3-register-zpa-as-a-saml-application-in-azure-ad-b2c"></a>Paso 3: Registro de ZPA como aplicación SAML en Azure AD B2C

Para configurar una aplicación SAML en Azure AD B2C, consulte [Registrar una aplicación SAML en Azure AD B2C](./saml-service-provider.md). 

En el paso ["Cargar la directiva"](./saml-service-provider.md#upload-your-policy), copie o anote la dirección URL de metadatos de SAML del IdP que se usa en Azure AD B2C. Lo necesitará más adelante.

Siga las instrucciones del paso ["Configurar la aplicación en Azure AD B2C"](./saml-service-provider.md#configure-your-application-in-azure-ad-b2c). En el paso 4.2, actualice las propiedades del manifiesto de la aplicación tal como se indica a continuación:

- En **identifierUris**: use el valor de Service Provider Entity ID (Id. de entidad del proveedor de servicios) que copió o anotó anteriormente en el "Paso 1.6.b".  
- En **samlMetadataUrl**: omita esta propiedad, porque ZPA no hospeda ninguna dirección URL de metadatos de SAML.  
- En **replyUrlsWithType**: use el valor de Service Provider URL (Dirección URL del proveedor de servicios) que copió o anotó anteriormente en el "Paso 1.6.a".  
- En **logoutUrl**: omita esta propiedad, porque ZPA no admite una dirección URL de cierre de sesión.

El resto de los pasos no son de interés para este tutorial.

### <a name="step-4-extract-the-idp-saml-metadata-from-azure-ad-b2c"></a>Paso 4: Extracción de los metadatos de SAML del IdP de Azure AD B2C

A continuación, debe obtener una dirección URL de metadatos de SAML con el siguiente formato:

```https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/Samlp/metadata```

Tenga en cuenta que `<tenant-name>` es el nombre de su inquilino de Azure AD B2C y `<policy-name>` es el nombre de la directiva de SAML personalizada que creó en el paso anterior.

Por ejemplo, la URL podrían ser `https://safemarch.b2clogin.com/safemarch.onmicrosoft.com/B2C_1A_signup_signin_saml//Samlp/metadata`.

Abra un explorador web y vaya a la dirección URL de metadatos de SAML. Haga clic con el botón derecho en cualquier parte de la página, seleccione **Guardar como** y, a continuación, guarde el archivo en el equipo para usarlo en el paso siguiente.

### <a name="step-5-complete-the-idp-configuration-on-zpa"></a>Paso 5: Finalización de la configuración del IdP en ZPA

Finalice la [configuración del IdP en el portal de administración de ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign) que hizo parcialmente en el "Paso 1: Configuración de Azure AD B2C como IdP en ZPA".

1. En el [portal de administración de ZPA](https://admin.private.zscaler.com), vaya a **Administration** > **IdP Configuration** (Administración > Configuración del IdP).

1. Seleccione el IdP que configuró en el paso 1 y, a continuación, seleccione **Resume** (Reanudar).

1. En el panel **Add IdP Configuration** (Agregar configuración del IdP), seleccione la pestaña **Create IdP** (Crear IdP) y, a continuación, haga lo siguiente:

   a. En **IdP Metadata File** (Archivo de metadatos de IdP), cargue el archivo de metadatos que guardó anteriormente en el "Paso 4: Extracción de los metadatos de SAML del IdP de Azure AD B2C".  
   b. Compruebe que el valor de **Status** (Estado) para la configuración del IdP es **Enabled** (Habilitado).  
   c. Seleccione **Guardar**.

   ![Captura de pantalla de la pestaña "Create IdP" (Crear IdP) en el panel "Add IdP Configuration (Agregar configuración del IdP).](media/partner-zscaler/create-idp.png)

## <a name="test-the-solution"></a>Probar la solución

Vaya a un portal de usuarios de ZPA o a una aplicación de acceso mediante el explorador y pruebe el proceso de registro o de inicio de sesión. La autenticación SAML debería realizarse correctamente.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy)
- [Registrar una aplicación SAML en Azure AD B2C](./saml-service-provider.md)
- [Guía para la configuración paso a paso de ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa)
- [Configuración de un IdP para el inicio de sesión único](https://help.zscaler.com/zpa/configuring-idp-single-sign)