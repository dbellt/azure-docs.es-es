---
title: Versiones de diseño de página
titleSuffix: Azure AD B2C
description: Historial de versiones de diseño de página de personalización de interfaz de usuario en directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/05/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 8469e05b82a651760829761ca57af3bdb1b256a9
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443413"
---
# <a name="page-layout-versions"></a>Versiones de diseño de página

Los paquetes de diseño de página se actualizan periódicamente para incluir correcciones y mejoras en sus elementos de la página. El registro de cambios siguiente especifica los cambios introducidos en cada versión.

## <a name="jquery-version"></a>versión jQuery

El diseño de página de Azure AD B2C utiliza la siguiente versión de la [biblioteca de jQuery](https://jquery.com/):

|Desde la versión de diseño de página  |versión jQuery  |
|---------|---------|
|2.1.4 | 3.5.1 |
|1.2.0 | 3.4.1 |
|1.1.0 | 1.10.2 |

## <a name="self-asserted-page-selfasserted"></a>Página autoafirmada (selfasserted)

**2.1.4**
- Se actualizó JQuery a la versión 3.5.1.
- Se actualizó HandlebarJS a la versión 4.7.6.

**2.1.3**
- Revisiones de seguridad.

**2.1.2**
- Se ha corregido el problema de codificación de la localización para idiomas como el español y el francés.

**2.1.1**

- Se ha agregado una UXString `heading` además de `intro` para que se muestre en la página como un título. Está oculta de forma predeterminada.
- Se ha agregado compatibilidad para guardar contraseñas en iCloud Keychain.
- Se ha agregado compatibilidad con el uso de la Directiva o el parámetro de QueryString `pageFlavor` para seleccionar el diseño (clásico, oceanBlue o slateGray).
- Se han agregado declinaciones de responsabilidades en la página autoafirmada.
- El foco se sitúa ahora en el primer campo editable cuando se carga la página.
- El foco se sitúa ahora en el primer campo de error cuando varios campos tienen errores.
- El foco se sitúa ahora en el botón "cambiar" después de comprobar el código de verificación del correo electrónico.

**2.1.0**

- Correcciones de localización y accesibilidad.

**2.0.0**

- Se ha agregado compatibilidad para los [controles de visualización](display-controls.md) en las directivas personalizadas.

**1.2.0**

- Los campos de nombre de usuario/correo electrónico y contraseña ahora usan el elemento HTML `form` para permitir que Microsoft Edge e Internet Explorer (IE) guarden correctamente esta información.
- Se ha agregado un retraso configurable en la validación de entradas de usuario para mejorar la experiencia del usuario.
- Correcciones de accesibilidad
- Se ha corregido un problema de accesibilidad para que Narrator lea ahora los mensajes de error. 
- El foco se sitúa ahora en el campo de contraseña después de comprobar el correo electrónico.
- Se ha eliminado `autofocus` del control CheckBox. 
- Se ha agregado compatibilidad con un control de pantalla para la comprobación del número de teléfono.
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content).
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome.
- Se ha agregado compatibilidad con la personalización de marca de empresa en las páginas del flujo de usuario.

**1.1.0**

- Se ha quitado la alerta de cancelación.
- Clase CSS para elementos de error.
- Se ha mejorado la lógica para mostrar u ocultar errores.
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="unified-sign-in-sign-up-page-with-password-reset-link-unifiedssp"></a>Página de registro de inicio de sesión unificado con el vínculo de restablecimiento de contraseña (unifiedssp)

> [!TIP]
> Si localiza la página para que admita varias configuraciones regionales o idiomas en un flujo de usuario. El artículo sobre los [identificadores de localización](localization-string-ids.md) proporciona la lista de identificadores de localización que puede utilizar para la versión de la página que seleccione.

**2.1.4**
- Se actualizó JQuery a la versión 3.5.1.
- Se actualizó HandlebarJS a la versión 4.7.6.

**2.1.3**
- Revisiones de seguridad.
- Correcciones de errores leves.

**2.1.2**
- Se ha corregido el problema de codificación de la localización para idiomas como el español y el francés.
- Permite que el vínculo "contraseña olvidada" se use como intercambio de notificaciones. Para más información, consulte [Autoservicio de restablecimiento de contraseña](add-password-reset-policy.md#self-service-password-reset-recommended).

**2.1.1**
- Se ha agregado una UXString `heading` además de `intro` para que se muestre en la página como un título. Está oculta de forma predeterminada.
- Se ha agregado compatibilidad con el uso de la Directiva o el parámetro de QueryString `pageFlavor` para seleccionar el diseño (clásico, oceanBlue o slateGray).
- Se ha agregado compatibilidad para guardar contraseñas en iCloud Keychain.
- El foco se sitúa ahora en el primer campo de error cuando varios campos tienen errores.
- El foco se sitúa ahora en el primer campo editable cuando se carga la página.
- Se ha agregado una nueva ubicación para el vínculo de selección del proveedor de notificaciones `bottomUnderFormClaimsProviderSelections`.
- Se han eliminado UXStrings que ya no se usan.

**2.1.0**

- Se ha agregado compatibilidad con varios vínculos de registro.
- Se ha agregado compatibilidad con la validación de entradas de usuario según las reglas de predicado definidas en la directiva.

**1.2.0**

- Los campos de nombre de usuario/correo electrónico y contraseña ahora usan el elemento HTML `form` para permitir que Microsoft Edge e Internet Explorer (IE) guarden correctamente esta información.
- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome.
- Se ha agregado compatibilidad con la personalización de marca del inquilino en las páginas del flujo de usuario.

**1.1.0**

- Se ha agregado el control para mantener la sesión iniciada (KMSI)

**1.0.0**

- Versión inicial

## <a name="mfa-page-multifactor"></a>Página de MFA (multifactor)

**1.2.4**
- Se actualizó JQuery a la versión 3.5.1.
- Se actualizó HandlebarJS a la versión 4.7.6.

**1.2.3**
- Se permite la invalidación de la cadena de información sobre herramientas a través de la localización del lenguaje.
- Revisiones de seguridad.
- Correcciones de errores leves.

**1.2.2**
- Se ha corregido un problema con el rellenado automático del código de verificación cuando se usa iOS.
- Se ha corregido un problema con la redirección de un token al usuario de confianza desde Android WebView. 
- Se ha agregado una UXString `heading` además de `intro` para que se muestre en la página como un título. Está oculta de forma predeterminada.  
- Se ha agregado compatibilidad con el uso de la Directiva o el parámetro de QueryString `pageFlavor` para seleccionar el diseño (clásico, oceanBlue o slateGray).

**1.2.1**

- Correcciones de accesibilidad en plantillas predeterminadas

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome.
- Se ha agregado compatibilidad con la personalización de marca del inquilino en las páginas del flujo de usuario.

**1.1.0**

- Se ha quitado el botón “Confirmar código”.
- El campo de entrada del código ahora solo acepta la entrada de hasta seis (6) caracteres.
- La página intentará comprobar el código introducido automáticamente cuando se introduzca un código de 6 dígitos, sin tener que hacer clic en ningún botón.
- Si el código es incorrecto, el campo de entrada se borra automáticamente.
- Después de tres (3) intentos con un código incorrecto, B2C reenvía un error al usuario de confianza.
- Correcciones de accesibilidad
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="exception-page-globalexception"></a>Página de excepciones (globalexception)

**1.2.1**
- Se actualizó JQuery a la versión 3.5.1.
- Se actualizó HandlebarJS a la versión 4.7.6.

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.1.0**

- Corrección de accesibilidad.
- Se ha quitado el mensaje predeterminado cuando no hay ningún contacto de la directiva.
- Se ha quitado la hoja CSS predeterminada.

**1.0.0**

- Versión inicial

## <a name="other-pages-providerselection-claimsconsent-unifiedssd"></a>Otras páginas (ProviderSelection, ClaimsConsent, UnifiedSSD)

**1.2.1**
- Se actualizó JQuery a la versión 3.5.1.
- Se actualizó HandlebarJS a la versión 4.7.6.

**1.2.0**

- Correcciones de accesibilidad
- Ahora puede agregar el atributo `data-preload="true"` [en las etiquetas HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) para controlar el orden de carga de CSS y JavaScript.
  - Cargue los archivos CSS vinculados al mismo tiempo que la plantilla HTML para que no "vacile" durante la carga de los archivos.
  - Controle el orden en el que se capturan y ejecutan las etiquetas `script` antes de la carga de la página.
- El campo de correo electrónico es ahora `type=email` y los teclados para móviles proporcionarán las sugerencias correctas.
- Compatibilidad con la traducción de Chrome

**1.0.0**

- Versión inicial

## <a name="next-steps"></a>Pasos siguientes

Para obtener detalles sobre cómo personalizar la interfaz de usuario de las aplicaciones en directivas personalizadas, vea [Personalización de la interfaz de usuario de la aplicación mediante una directiva personalizada](customize-ui-with-html.md).
