---
ms.openlocfilehash: 15e809e1fa361f758a3df7e5b56d4fb80400f656
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "103007906"
---
| Plataforma          | Proyecto en<br/>GitHub                                                                          | Paquete                                                                               | Introducción<br/>iniciado                    | Inicio de sesión de usuarios                                         | Acceso a API web                                                 | Disponible con carácter general (GA) *o*<br/>Versión preliminar pública<sup>1</sup> |
|-------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|:--------------------------------------:|:-----------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| Android (Java)    | [MSAL de Android](https://github.com/AzureAD/microsoft-authentication-library-for-android)        | [MSAL](https://mvnrepository.com/artifact/com.microsoft.identity.client/msal)         | [Guía de inicio rápido](../articles/active-directory/develop/quickstart-v2-android.md) | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
| Android (Kotlin)  | [MSAL de Android](https://github.com/AzureAD/microsoft-authentication-library-for-android)        | [MSAL](https://mvnrepository.com/artifact/com.microsoft.identity.client/msal)         | —                                      | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
| iOS (Swift/Obj-C) | [MSAL para iOS y macOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [MSAL](https://cocoapods.org/pods/MSAL)                                               | [Tutorial](../articles/active-directory/develop/tutorial-v2-ios.md)         | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
| Xamarin (.NET)    | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)             | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) | —                                      | ![La biblioteca puede solicitar tokens de id. para el inicio de sesión de usuario.][y] | ![La biblioteca puede solicitar tokens de acceso para las API web protegidas.][y] | GA                                                           |
<!--
| React Native |[React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth/blob/main/docs/config-examples/azure-active-directory.md) | [react-native-app-auth](https://www.npmjs.com/package/react-native-app-auth) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Los términos de uso complementarios para las versiones preliminares de Microsoft Azure][preview-tos] se aplican a las bibliotecas en la *versión preliminar pública*.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->

[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/