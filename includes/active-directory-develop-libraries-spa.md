---
ms.openlocfilehash: a393cf978318c39081805cae7e38c3386ff156f5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "103010715"
---
| Lenguaje/marco de trabajo | Proyecto en<br/>GitHub                                                                                                    | Paquete                                                                      | Introducción<br/>iniciado                             | Inicio de sesión de usuarios                                         | Acceso a API web                                                 | Disponible con carácter general (GA) *o*<br/>Versión preliminar pública<sup>1</sup> |
|----------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|:-----------------------------------------------:|:-----------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| Angular              | [MSAL Angular v2](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)<sup>2</sup>         | [@azure/msal-angular](https://www.npmjs.com/package/@azure/msal-angular)     | —                                               | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | Versión preliminar pública                                               |
| Angular              | [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/msal-angular-v1/lib/msal-angular)<sup>3</sup> | [@azure/msal-angular](https://www.npmjs.com/package/@azure/msal-angular)     |[Tutorial](../articles/active-directory/develop/tutorial-v2-angular.md)| ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
| AngularJS            | [MSAL AngularJS](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angularjs)<sup>3</sup>         | [@azure/msal-angularjs](https://www.npmjs.com/package/@azure/msal-angularjs) | —                                               | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | Versión preliminar pública                                               |
| JavaScript           | [MSAL.js v2](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)<sup>2</sup>              | [@azure/msal-browser](https://www.npmjs.com/package/@azure/msal-browser)     | [Tutorial](../articles/active-directory/develop/tutorial-v2-javascript-auth-code.md) | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
|JavaScript|[MSAL.js 1.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core)<sup>3</sup> | [@azure/msal-core](https://www.npmjs.com/package/@azure/msal-core)    | [Tutorial](../articles/active-directory/develop/tutorial-v2-javascript-spa.md)| ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
| React                | [MSAL React](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)<sup>2</sup>                 | [@azure/msal-react](https://www.npmjs.com/package/@azure/msal-react)         | —                                               | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | Versión preliminar pública                                               |
<!--
| Vue | [Vue MSAL]( https://github.com/mvertopoulos/vue-msal) | [vue-msal]( https://www.npmjs.com/package/vue-msal) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Los términos de uso complementarios para las versiones preliminares de Microsoft Azure][preview-tos] se aplican a las bibliotecas en la *versión preliminar pública*.

<sup>2</sup> [Flujo de código de autenticación][auth-code-flow] solo con PCKE (recomendado). 

<sup>3</sup> [Flujo de concesión implícita][implicit-flow] solo.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[Microsoft-SDL]: https://www.microsoft.com/securityengineering/sdl/
[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[auth-code-flow]: ../articles/active-directory/develop/v2-oauth2-auth-code-flow.md
[implicit-flow]: ../articles/active-directory/develop/v2-oauth2-implicit-grant-flow.md