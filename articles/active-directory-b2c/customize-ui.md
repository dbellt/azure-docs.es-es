---
title: Personalizar la interfaz del usuario
titleSuffix: Azure AD B2C
description: Obtenga información acerca de cómo personalizar la interfaz de usuario en las aplicaciones que usan Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/26/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: ba24afb329b5d588ab7a71976f56f50eb475b04b
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110575414"
---
# <a name="customize-the-user-interface-in-azure-active-directory-b2c"></a>Personalización de la interfaz de usuario en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

La personalización de marca y la personalización de la interfaz de usuario que Azure Active Directory B2C (Azure AD B2C) muestra a los clientes ayuda a proporcionar una experiencia de usuario sin problemas en la aplicación. Estas experiencias incluyen las opciones de registro, inicio de sesión, edición de perfiles y restablecimiento de contraseñas. En este artículo, personalizará sus páginas de Azure AD B2C con la plantilla de página, así como la personalización de marca de la empresa. 

> [!TIP]
> Para personalizar otros aspectos de las páginas del flujo de usuario más allá de la plantilla de página, el logotipo del banner, la imagen de fondo o el color de fondo, consulte [Personalización de la interfaz de usuario con una plantilla HTML](customize-ui-with-html.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="overview"></a>Información general

Azure AD B2C proporciona varias plantillas integradas que puede elegir para dar a sus páginas de experiencia del usuario una apariencia profesional. Estas plantillas de página también pueden servir como punto de partida para su propia personalización con la característica de [personalización de marca](#company-branding).

> [!NOTE]
> Los exploradores admitidos para la plantilla clásica incluyen versiones actuales y anteriores de Internet Explorer, Microsoft Edge, Google Chrome, Mozilla Firefox y Safari. Las plantillas Ocean Blue y Slate Gray pueden tener compatibilidad limitada con versiones anteriores del explorador, como Internet Explorer 11 y 10. Se recomienda probar la aplicación con los exploradores que pretende admitir.

### <a name="ocean-blue"></a>Azul océano

Ejemplo de la plantilla Azul océano que se representa en la página de inicio de sesión y registro:

![Captura de pantalla de la plantilla Azul océano](media/customize-ui/template-ocean-blue.png)

### <a name="slate-gray"></a>Gris pizarra

Ejemplo de la plantilla Gris pizarra que se representa en la página de inicio de sesión y registro:

![Captura de pantalla de la plantilla Gris pizarra](media/customize-ui/template-slate-gray.png)

### <a name="classic"></a>Clásico

Ejemplo de la plantilla Clásica que se representa en la página de inicio de sesión y registro:

![Captura de pantalla de la plantilla Clásica](media/customize-ui/template-classic.png)

### <a name="company-branding"></a>Personalización de marca de empresa

Puede personalizar las páginas de Azure AD B2C con un logotipo de banner, una imagen de fondo y un color de fondo mediante [Personalización de marca de empresa](../active-directory/fundamentals/customize-branding.md) de Azure Active Directory. La personalización de marca de una empresa incluye las páginas de registro, inicio de sesión, edición de perfiles y restablecimiento de contraseñas. 

En el ejemplo siguiente se muestra una página de *inicio de sesión y registro* con un logotipo personalizado y una imagen de fondo que usa la plantilla Azul océano:

![Página de registro o inicio de sesión con personalización de marca ofrecida por Azure AD B2C](media/customize-ui/template-ocean-blue-branded.png)


## <a name="select-a-page-template"></a>Selección de una plantilla de página

::: zone pivot="b2c-user-flow"

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione un flujo de usuario que quiera personalizar.
1. En **Personalizar** en el menú de la izquierda, seleccione **Diseños de página** y, después, seleccione una **Plantilla**.
    ![Lista desplegable de selección de plantilla en la página flujo de usuario de Azure Portal](./media/customize-ui/select-page-template.png)

Al elegir una plantilla, la que se seleccione se aplicará a todas las páginas de su flujo de usuario. El URI de cada página está visible en el campo **URI de página personalizado**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Para seleccionar una plantilla de página, establezca el elemento `LoadUri` de las [definiciones de contenido](contentdefinitions.md). En el ejemplo siguiente se muestran los identificadores de definición de contenido y el elemento `LoadUri` correspondiente. 

Azul océano:

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/AzureBlue/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/AzureBlue/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```

Gris pizarra:

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/MSA/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/MSA/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/MSA/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/MSA/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/MSA/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```

Clásica: 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/default/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/default/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/default/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/default/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```
::: zone-end


## <a name="configure-company-branding"></a>Configuración de la personalización de marca de la compañía

Antes de personalizar las páginas del flujo de usuario, debe configurar la personalización de marca de empresa en Azure Active Directory y, a continuación, habilitarla en los flujos de usuario en Azure AD B2C.

Empiece por establecer el logotipo del banner, la imagen de fondo y el color de fondo en **Personalización de marca de empresa**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Administrar**, seleccione **Personalización de marca de empresa**.
1. Siga los pasos de [Incorporación de la personalización de marca en la página de inicio de sesión de Azure Active Directory de la organización](../active-directory/fundamentals/customize-branding.md).

Tenga en cuenta las siguientes cuestiones a la hora de configurar la personalización de marca de empresa en Azure AD B2C:

* La personalización de marca de empresa en Azure AD B2C está actualmente limitada a la personalización de la **imagen de fondo**, el **logotipo del banner** y el **color de fondo**. Las demás propiedades del panel de personalización de marca de empresa como, por ejemplo, la **Configuración avanzada**, *no son compatibles*.
* En las páginas del flujo de usuario se muestra el color de fondo antes de cargar la imagen de fondo. Se recomienda elegir un color de fondo que coincida en la medida de lo posible con los colores de la imagen de fondo, para conseguir una experiencia de carga más fluida.
* El logotipo del banner aparece en los mensajes de correo electrónico de verificación que se envían a los usuarios cuando inician un flujo de usuario de inicio de sesión.



## <a name="enable-company-branding-in-user-flow-pages"></a>Habilitación de la personalización de marca de empresa en las páginas del flujo de usuario

::: zone pivot="b2c-user-flow"

Después de configurar la personalización de marca de la empresa, habilítela en los flujos de usuario.

1. En el menú izquierdo de Azure Portal, seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Flujos de usuario (directivas)** .
1. Seleccione el flujo de usuario para el que desea habilitar la personalización de marca de empresa. La personalización de marca de la empresa **no es compatible** con los tipos de flujo de usuario *Inicio de sesión* estándar y *Edición de perfiles* estándar.
1. En **Personalizar**, seleccione **Diseños de página** y, a continuación, seleccione la página cuya marca quiere personalizar. Por ejemplo, seleccione **Página unificada de inicio de sesión o de registro**.
1. En **Versión de Diseño de página (versión preliminar)** , elija la versión **1.2.0** o posterior.
1. Seleccione **Guardar**.

Si desea personalizar la marca de todas las páginas del flujo de usuario, establezca la versión de cada diseño de página incluido en el flujo de usuario.

![Selección del diseño de página de Azure AD B2C en Azure Portal](media/customize-ui/portal-02-page-layout-select.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

Después de configurar la personalización de marca de la empresa, habilítela en la directiva personalizada. Configure la [versión de diseño de página](contentdefinitions.md#migrating-to-page-layout) con la versión `contract` de página para *todas* las definiciones de contenido de la directiva personalizada. El formato del valor debe contener la palabra `contract`: _urn:com:microsoft:aad:b2c:elements:**contract**:page-name:version_. Para especificar un diseño de página en las directivas personalizadas que utilizan un valor de **DataUri** anterior. Para más información, aprenda a [migrar al diseño de página](contentdefinitions.md#migrating-to-page-layout) con la versión de la página.

En el siguiente ejemplo se muestran las definiciones de contenido con su correspondiente contrato de página y la plantilla de página *Azul océano*: 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/AzureBlue/exception.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/AzureBlue/multifactor-1.0.0.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

Encuentre más información sobre cómo personalizar la interfaz de usuario de las aplicaciones en [Personalización de la interfaz de usuario de la aplicación en Azure Active Directory B2C](customize-ui-with-html.md).
