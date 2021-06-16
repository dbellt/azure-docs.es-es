---
title: Problemas de Internet Explorer (MSAL.js) | Azure
titleSuffix: Microsoft identity platform
description: Uso de la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) con el explorador Internet Explorer.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/16/2019
ms.author: mtillman
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: eda2d61d7af1b56a2e0069e8726aa1e29800e19a
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077129"
---
# <a name="known-issues-on-internet-explorer-browsers-msaljs"></a>Problemas conocidos en los exploradores Internet Explorer (MSAL.js)

La biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) se genera para [JavaScript ES5](https://fr.wikipedia.org/wiki/ECMAScript#ECMAScript_Edition_5_.28ES5.29), para que pueda ejecutarse en Internet Explorer. Hay, sin embargo, algunas cosas que debe saber.

## <a name="run-an-app-in-internet-explorer"></a>Ejecutar una aplicación en Internet Explorer
Si pretende usar MSAL.js en aplicaciones que pueden ejecutarse en Internet Explorer, deberá agregar una referencia a un polyfill de promesa antes de hacer referencia al script MSAL.js.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.3.4/bluebird.min.js" class="pre"></script>
```

Esto se debe a que Internet Explorer no admite las promesas de JavaScript de forma nativa.

## <a name="debugging-an-application-running-in-internet-explorer"></a>Depuración de una aplicación que se ejecuta en Internet Explorer

### <a name="running-in-production"></a>Ejecución en producción
La implementación de la aplicación en producción (por ejemplo, en las aplicaciones web de Azure) normalmente funciona bien, siempre que el usuario final haya aceptado los elementos emergentes. Lo probamos con Internet Explorer 11.

### <a name="running-locally"></a>Ejecución local
Si quiere ejecutar y depurar localmente la aplicación que se ejecuta en Internet Explorer, debe considerar los siguientes aspectos (suponga que quiere ejecutar su aplicación como *http://localhost:1234* ):

- Internet Explorer tiene un mecanismo de seguridad llamado "modo protegido", que impide que MSAL.js funcione correctamente. Entre los síntomas que pueden aparecer después de iniciar sesión, la página puede redirigirse a http://localhost:1234/null.

- Para ejecutar y depurar la aplicación localmente, deberá deshabilitar el "modo protegido". Para esto:

    1. Haga clic en las **herramientas** de Internet Explorer (el icono con forma de engranaje).
    1. Seleccione **Opciones de Internet** y la pestaña **Seguridad**.
    1. Haga clic en la zona **Internet** y desactive la opción **Habilitar modo protegido (requiere reiniciar Internet Explorer)** . Internet Explorer le advertirá que su equipo ya no está protegido. Haga clic en **OK**.
    1. Reinicie Internet Explorer.
    1. Ejecute y depure la aplicación.

Cuando haya terminado, restaure la configuración de seguridad de Internet Explorer.  Seleccione **Configuración** -> **Opciones de Internet** -> **Seguridad** -> **Restablecer todas las zonas al nivel predeterminado**.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre los [problemas conocidos al usar MSAL.js en Internet Explorer](msal-js-use-ie-browser.md).