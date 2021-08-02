---
title: JavaScript y versiones de diseño de página
titleSuffix: Azure AD B2C
description: Aprenda a habilitar JavaScript y a usar versiones de diseño de página en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/07/2021
ms.custom: project-no-code, devx-track-js
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 652551635b84c18020bf928194923d0e6ca86149
ms.sourcegitcommit: 89c889a9bdc2e72b6d26ef38ac28f7a6c5e40d27
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111565293"
---
# <a name="javascript-and-page-layout-versions-in-azure-active-directory-b2c"></a>JavaScript y versiones de diseño de página en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

Azure AD B2C proporciona un conjunto de contenido empaquetado que contiene HTML, CSS y JavaScript para los elementos de la interfaz de usuario de los flujos de usuarios y las directivas personalizadas. Para habilitar JavaScript para las aplicaciones:

::: zone pivot="b2c-user-flow"

* Seleccione un [diseño de página](page-layout.md)
* Habilítelo en el flujo de usuario mediante Azure Portal
* Use [b2clogin.com](b2clogin.md) en las solicitudes

::: zone-end

::: zone pivot="b2c-custom-policy"

* Seleccione un [diseño de página](page-layout.md)
* Agregue un elemento a la [directiva personalizada](custom-policy-overview.md)
* Use [b2clogin.com](b2clogin.md) en las solicitudes

::: zone-end

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]


## <a name="select-a-page-layout-version"></a>Selección de una versión de diseño de página

Si tiene pensado habilitar el código cliente de JavaScript, los elementos en los que se basa el código JavaScript deben ser inmutables. Si no son inmutables, cualquier cambio podría provocar un comportamiento inesperado en las páginas de usuario. Para evitar estos problemas, exija el uso de un diseño de página y especifique una versión de este para garantizar que las definiciones de contenido en las que ha basado JavaScript sean inmutables. Incluso si no piensa habilitar JavaScript, puede especificar una versión del diseño de página para las páginas.

::: zone pivot="b2c-user-flow"

Para especificar una versión de diseño de página para las páginas de flujo de usuario: 

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Diseños de página**. Elija un **Nombre de diseño** y, a continuación, elija la **Versión de Diseño de página**.

Para información sobre los distintos diseños de página, consulte el [registro de cambios en los diseños de página](page-layout.md).

![Configuración del diseño de página del portal que muestra el menú desplegable de la versión del diseño de página](media/javascript-and-page-layout/page-layout-version.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

Seleccione un [diseño de página](contentdefinitions.md#select-a-page-layout) para los elementos de la interfaz de usuario de su aplicación.

Defina una [versión de diseño de página](contentdefinitions.md#migrating-to-page-layout) con la versión `contract` de página para *todas* las definiciones de contenido de la directiva personalizada. El formato del valor debe contener la palabra `contract`: _urn:com:microsoft:aad:b2c:elements:**contract**:page-name:version_. Aprenda a [migrar al diseño de página](contentdefinitions.md#migrating-to-page-layout) con la versión de la página.

En el ejemplo siguiente se muestran los identificadores de definición de contenido y el elemento **DataUri** correspondiente con el contrato de página: 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```
::: zone-end

## <a name="enable-javascript"></a>Habilitar JavaScript

::: zone pivot="b2c-user-flow"

En **Propiedades** del flujo de usuario puede habilitar JavaScript. La habilitación de JavaScript también exige el uso de un diseño de página. Después, puede establecer la versión del diseño de página para el flujo de usuario tal y como se describe en la sección siguiente.

![Página de propiedades del flujo de usuario con la opción Habilitar JavaScript resaltada](media/javascript-and-page-layout/javascript-settings.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

Habilite la ejecución del script agregando el elemento **ScriptExecution** al elemento [RelyingParty](relyingparty.md).

1. Abra el archivo de la directiva personalizada. Por ejemplo, *SignUpOrSignin.xml*.
1. Agregue el elemento **ScriptExecution** al elemento **RelyingParty**:

    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
      <UserJourneyBehaviors>
        <ScriptExecution>Allow</ScriptExecution>
      </UserJourneyBehaviors>
      ...
    </RelyingParty>
    ```
1. Guarde y cargue el archivo.

::: zone-end

## <a name="guidelines-for-using-javascript"></a>Directrices para usar JavaScript

Siga estas directrices cuando personalice la interfaz de la aplicación con JavaScript:

- No enlace un evento de clic con elementos HTML `<a>`.
- No tome una dependencia del código Azure AD B2C ni de los comentarios.
- No cambie el orden ni la jerarquía de los elementos HTML de Azure AD B2C. Use una directiva de Azure AD B2C para controlar el orden de los elementos de la interfaz de usuario.
- Puede llamar a cualquier servicio RESTful con estas consideraciones:
    - Deberá establecer el servicio RESTful CORS para que permita llamadas HTTP del lado cliente.
    - Asegúrese de que el servicio RESTful es seguro y solo usa el protocolo HTTPS.
    - No use JavaScript directamente para llamar a puntos de conexión de Azure AD B2C.
- Puede incrustar el archivo JavaScript o puede vincular archivos JavaScript externos. Cuando use un archivo JavaScript externo, asegúrese de usar la dirección URL absoluta y no una relativa.
- Marcos de JavaScript:
    - Azure AD B2C usa una [versión específica de jQuery](page-layout.md#jquery-version). No incluya otra versión de jQuery. Usar más de una versión en la misma página provoca problemas.
    - No se admite el uso de RequireJS.
    - Azure AD B2C no admite la mayoría de los marcos de JavaScript.
- La configuración de Azure AD B2C puede leerse mediante una llamada a los objetos `window.SETTINGS` y `window.CONTENT`, como el idioma actual de la interfaz de usuario. No cambie el valor de estos objetos.
- Para personalizar el mensaje de error de Azure AD B2C, use la localización en una directiva.
- Si se puede lograr algo con una política, es la forma recomendada.

## <a name="javascript-samples"></a>Ejemplos de JavaScript

### <a name="show-or-hide-a-password"></a>Mostrar u ocultar una contraseña

Una forma habitual de ayudar a sus clientes para que se registren correctamente es permitirles ver la contraseña que escriben. Esta opción permite que los usuarios se registren permitiéndoles ver y realizar correcciones en su contraseña si es necesario. Cualquier campo de contraseña escrita tiene una casilla con la etiqueta **Mostrar contraseña**.  Esto permite al usuario ver la contraseña en texto sin formato. Incluya este fragmento de código en la plantilla de registro o de inicio de sesión para una página autoafirmada:

```Javascript
function makePwdToggler(pwd){
  // Create show-password checkbox
  var checkbox = document.createElement('input');
  checkbox.setAttribute('type', 'checkbox');
  var id = pwd.id + 'toggler';
  checkbox.setAttribute('id', id);

  var label = document.createElement('label');
  label.setAttribute('for', id);
  label.appendChild(document.createTextNode('show password'));

  var div = document.createElement('div');
  div.appendChild(checkbox);
  div.appendChild(label);

  // Add show-password checkbox under password input
  pwd.insertAdjacentElement('afterend', div);

  // Add toggle password callback
  function toggle(){
    if(pwd.type === 'password'){
      pwd.type = 'text';
    } else {
      pwd.type = 'password';
    }
  }
  checkbox.onclick = toggle;
  // For non-mouse usage
  checkbox.onkeydown = toggle;
}

function setupPwdTogglers(){
  var pwdInputs = document.querySelectorAll('input[type=password]');
  for (var i = 0; i < pwdInputs.length; i++) {
    makePwdToggler(pwdInputs[i]);
  }
}

setupPwdTogglers();
```

### <a name="add-terms-of-use"></a>Agregar términos de uso

Incluya el código siguiente en la página donde quiera la casilla **Términos de uso**. Normalmente, esta casilla suele ser necesaria en las páginas de registro de cuenta sociales o de cuentas locales.

```Javascript
function addTermsOfUseLink() {
    // find the terms of use label element
    var termsOfUseLabel = document.querySelector('#api label[for="termsOfUse"]');
    if (!termsOfUseLabel) {
        return;
    }

    // get the label text
    var termsLabelText = termsOfUseLabel.innerHTML;

    // create a new <a> element with the same inner text
    var termsOfUseUrl = 'https://docs.microsoft.com/legal/termsofuse';
    var termsOfUseLink = document.createElement('a');
    termsOfUseLink.setAttribute('href', termsOfUseUrl);
    termsOfUseLink.setAttribute('target', '_blank');
    termsOfUseLink.appendChild(document.createTextNode(termsLabelText));

    // replace the label text with the new element
    termsOfUseLabel.replaceChild(termsOfUseLink, termsOfUseLabel.firstChild);
}
```

En el código, reemplace `termsOfUseUrl` con el vínculo al contrato de los términos de uso. Para su directorio, cree un nuevo atributo de usuario denominado **termsOfUse** y, después, incluya **termsOfUse** como un atributo de usuario.

## <a name="next-steps"></a>Pasos siguientes

Encuentre más información sobre cómo personalizar la interfaz de usuario de las aplicaciones en [Personalización de la interfaz de usuario de la aplicación en Azure Active Directory B2C](customize-ui-with-html.md).
