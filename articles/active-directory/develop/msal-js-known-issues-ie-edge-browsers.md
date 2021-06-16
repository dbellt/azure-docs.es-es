---
title: Problemas en Internet Explorer y Microsoft Edge (MSAL.js) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información acerca de los problemas conocidos al usar la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) con Internet Explorer y Microsoft Edge.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 05/18/2020
ms.author: mtillman
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 9323558aec1cb330cc5253f8d380706854aaeae9
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077237"
---
# <a name="known-issues-on-internet-explorer-and-microsoft-edge-browsers-msaljs"></a>Problemas conocidos en los exploradores Internet Explorer y Microsoft Edge (MSAL.js)

## <a name="issues-due-to-security-zones"></a>Problemas debidos a zonas de seguridad
Hemos recibido varios informes de problemas con la autenticación en IE y Microsoft Edge (desde la actualización de la *versión de Microsoft Edge a 40.15063.0.0*). Estamos realizando un seguimiento de estos problemas y hemos informado al equipo de Microsoft Edge. Si bien Microsoft Edge trabaja en una solución, aquí tiene una descripción de los problemas que ocurren con mayor frecuencia y las posibles soluciones que se pueden implementar.

### <a name="cause"></a>Causa
La causa de la mayoría de estos problemas es la siguiente. El almacenamiento de sesión y el almacenamiento local están divididos por zonas de seguridad en el explorador Microsoft Edge. En esta versión particular de Microsoft Edge, cuando la aplicación se redirige a través de las zonas, se borran el almacenamiento de la sesión y el almacenamiento local. Específicamente, el almacenamiento de la sesión se borra de la navegación normal del explorador y, por ello, tanto la sesión como el almacenamiento local se borran en el modo InPrivate del explorador. MSAL.js guarda cierto estado en el almacenamiento de la sesión y comprueba este estado durante los flujos de autenticación. Cuando se borra el almacenamiento de la sesión, este estado se pierde y, por lo tanto, las experiencias se interrumpen.

### <a name="issues"></a>Issues

- **Bucles de redirección y recargas de páginas infinitos durante la autenticación**. Cuando los usuarios inician sesión en la aplicación desde Microsoft Edge, se les redirige de nuevo desde la página de inicio de sesión de AAD y se quedan en un bucle de redireccionamiento infinito que resulta en repetidas recargas de página. Esto suele ir acompañado de un error `invalid_state` en el almacenamiento de la sesión.

- **Bucles de adquisición de token infinitos y el error AADSTS50058**. Cuando una aplicación que se ejecuta en Microsoft Edge intenta adquirir un token para un recurso, la aplicación puede quedarse en un bucle infinito en plena llamada al token de adquisición junto con el siguiente error de AAD en el seguimiento de red:

    `Error :login_required; Error description:AADSTS50058: A silent sign-in request was sent but no user is signed in. The cookies used to represent the user's session were not sent in the request to Azure AD. This can happen if the user is using Internet Explorer or Edge, and the web app sending the silent sign-in request is in different IE security zone than the Azure AD endpoint (login.microsoftonline.com)`

- **La ventana emergente no se cierra o se bloquea cuando se usa el inicio de sesión a través de la ventana emergente para realizar la autenticación**. Cuando se realiza la autenticación mediante la ventana emergente de Microsoft Edge o IE (InPrivate), después de escribir las credenciales y realizar el inicio de sesión, si intervienen varios dominios de las zonas de seguridad en la navegación, la ventana emergente no se cierra porque MSAL.js pierde el identificador de la ventana emergente.  

### <a name="update-fix-available-in-msaljs-023"></a>Actualización: Corrección disponible en MSAL.js 0.2.3
Las correcciones para los problemas del bucle de redireccionamiento de la autenticación se han publicado en [MSAL.js 0.2.3](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases). Habilite la marca `storeAuthStateInCookie` en la configuración de MSAL.js para usar esta solución. De manera predeterminada, esta marca se establece en "false".

Cuando la marca `storeAuthStateInCookie` está habilitada, MSAL.js usará las cookies del navegador para almacenar el estado de solicitud requerido para la validación de los flujos de autenticación.

> [!NOTE]
> Esta solución aún no está disponible para los encapsulados msal-angular y msal-angularjs. Esta solución no soluciona el problema con las ventanas emergentes.

Use las soluciones que tiene a continuación.

#### <a name="other-workarounds"></a>Otras soluciones
Asegúrese de que su problema solo se produce en la versión específica del explorador Microsoft Edge y de que funciona en los otros exploradores antes de adoptar estas soluciones alternativas.  
1. Como primer paso para solucionar estos problemas, asegúrese de que el dominio de la aplicación y cualquier otro sitio involucrado en las redirecciones del flujo de autenticación se agreguen como sitios de confianza en la configuración de seguridad del explorador, para que pertenezcan a la misma zona de seguridad.
Para hacerlo, siga estos pasos:
    - Abra **Internet Explorer** y haga clic en la **configuración** (icono con forma de engranaje) en la esquina superior derecha.
    - A continuación, seleccione **Opciones de Internet**.
    - Seleccione la pestaña **Seguridad**
    - En la opción **Sitios de confianza**, haga clic en el botón de **sitios** y agregue las direcciones URL en el cuadro de diálogo que se abrirá.

2. Tal como se mencionó anteriormente, dado que solo se borra el almacenamiento de la sesión durante la navegación regular, puede configurar MSAL.js para que use el almacenamiento local en su lugar. Esto se puede establecer como el parámetro de configuración `cacheLocation` al inicializar MSAL.

Tenga en cuenta que esto no solucionará el problema de exploración de InPrivate, ya que tanto la sesión como el almacenamiento local están desactivados.

## <a name="issues-due-to-popup-blockers"></a>Problemas debidos a los bloqueadores de ventanas emergentes

Hay casos en que las ventanas emergentes se bloquean en IE o Microsoft Edge; por ejemplo, cuando aparece una segunda ventana emergente durante la [autenticación multifactor](../authentication/concept-mfa-howitworks.md). Recibirá una alerta en el explorador para permitir la ventana emergente una vez o siempre. Si elige permitir, el explorador abre automáticamente la ventana emergente y se devuelve un identificador `null`. Como resultado, la biblioteca no tiene un controlador para la ventana y no hay forma de cerrar la ventana emergente. Este problema no ocurre en Chrome cuando le solicita que permita las ventanas emergentes, porque no abre automáticamente las ventanas emergentes.

Como **solución**, los desarrolladores deberán permitir ventanas emergentes en IE y Microsoft Edge antes de comenzar a usar la aplicación para evitar este problema.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre el [uso de MSAL.js en Internet Explorer](msal-js-use-ie-browser.md).
