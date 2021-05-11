---
title: Inicio de sesión único (SSO) basado en Kerberos en Azure Active Directory con Application Proxy
description: Explica cómo proporcionar el inicio de sesión único mediante Application Proxy de Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: how-to
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.custom: contperf-fy21q2
ms.openlocfilehash: ba29c7577bd725cf0c165e0cae8374d3c4bb647c
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108187125"
---
# <a name="kerberos-constrained-delegation-for-single-sign-on-sso-to-your-apps-with-application-proxy"></a>Delegación restringida de Kerberos para el inicio de sesión único (SSO) para las aplicaciones con Application Proxy

Puede proporcionar un inicio de sesión único para las aplicaciones locales publicadas a través de Proxy de aplicación que se protegen con la autenticación integrada de Windows. Estas aplicaciones requieren un vale Kerberos para el acceso. Proxy de aplicación usa la Delegación restringida de Kerberos (KCD) para admitir estas aplicaciones. 

Puede habilitar el inicio de sesión único a sus aplicaciones mediante la autenticación integrada de Windows (IWA) dando permiso a los conectores del proxy de aplicación en Active Directory para suplantar a los usuarios. Los conectores usan este permiso para enviar y recibir tokens en su nombre.

## <a name="how-single-sign-on-with-kcd-works"></a>Cómo funciona el inicio de sesión único con KCD
En este diagrama se explica el flujo que se crea cuando un usuario intenta tener acceso a una aplicación local que usa IWA.

![Diagrama de flujos de autenticación de Microsoft AAD](./media/application-proxy-configure-single-sign-on-with-kcd/authdiagram.png)

1. El usuario escribe la dirección URL para tener acceso a la aplicación local mediante Application Proxy.
2. Proxy de aplicación redirige la solicitud a los servicios de autenticación de Azure AD para realizar la autenticación previa. En este momento, Azure AD aplica cualquier autenticación correspondiente, así como directivas de autorización, como la autenticación multifactor. Si se valida el usuario, Azure AD crea un token y lo envía al usuario.
3. El usuario pasa el token a Proxy de aplicación.
4. El proxy de aplicación valida el token y recupera el nombre principal del usuario (UPN); a continuación, el conector extrae el UPN y el nombre de entidad de seguridad de servicio (SPN) a través de un canal seguro con autenticación dual.
5. El conector realiza la negociación de la delegación limitada de Kerberos (KCD) con AD local, suplantando al usuario para obtener un token de Kerberos para la aplicación.
6. Active Directory envía el token de Kerberos para la aplicación al conector.
7. El conector envía la solicitud original al servidor de aplicaciones, con el token de Kerberos que recibió de AD.
8. La aplicación envía la respuesta al conector y, después, se devuelve al servicio Application Proxy y, por último, al usuario.

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar a trabajar con el inicio de sesión único para aplicaciones con autenticación integrada de Windows, asegúrese de que el entorno está preparado con los siguientes ajustes y configuraciones:

* Sus aplicaciones, como las aplicaciones web de SharePoint, están establecidas para usar la autenticación de Windows integrada. Para más información, consulte [Habilitar la compatibilidad con la autenticación Kerberos](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd759186(v=ws.11)) o, para SharePoint, consulte [Planear la autenticación Kerberos en SharePoint 2013](/SharePoint/security-for-sharepoint-server/kerberos-authentication-planning).
* Todas las aplicaciones tienen [nombres de entidad de seguridad de servicio](https://social.technet.microsoft.com/wiki/contents/articles/717.service-principal-names-spns-setspn-syntax-setspn-exe.aspx).
* El servidor que ejecuta el conector y el que ejecuta la aplicación están unidos a dominio y forman parte del mismo dominio o dominios de confianza. Para obtener más información sobre la unión a un dominio, consulte [Unir un equipo a un dominio](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dd807102(v=ws.11)).
* El servidor que ejecuta el conector tiene acceso para leer el atributo TokenGroupsGlobalAndUniversal de los usuarios. Esta configuración predeterminada que podría verse afectada por la seguridad del entorno.

### <a name="configure-active-directory"></a>Configurar Active Directory
La configuración de Active Directory varía, en función de si el conector de Proxy de aplicación y el servidor de aplicaciones están en el mismo dominio o no.

#### <a name="connector-and-application-server-in-the-same-domain"></a>Conector y servidor de aplicaciones en el mismo dominio
1. En Active Directory, vaya a **Herramientas** > **Usuarios y equipos**.
2. Seleccione el servidor que ejecuta el conector.
3. Haga clic en el botón derecho y seleccione **Propiedades** > **Delegación**.
4. Seleccione **Confiar en este equipo para la delegación solo a los servicios especificados**. 
5. Seleccione **Usar cualquier protocolo de autenticación**.
6. En **Servicios a los que esta cuenta puede presentar credenciales delegadas**, agregue el valor de la identidad SPN del servidor de aplicaciones. Esto permite al conector del proxy de aplicación suplantar a los usuarios en AD en las aplicaciones definidas en la lista.

   ![Captura de pantalla de ventana Conector-Propiedades SVR](./media/application-proxy-configure-single-sign-on-with-kcd/properties.jpg)

#### <a name="connector-and-application-server-in-different-domains"></a>Conector y servidor de aplicaciones en dominios diferentes
1. Para obtener una lista de requisitos previos para trabajar con KCD entre dominios, vea [Delegación limitada Kerberos entre dominios](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831477(v=ws.11)).
2. Use la propiedad `principalsallowedtodelegateto` de la cuenta de servicio (cuenta de equipo o de usuario de dominio dedicada) de la aplicación web para habilitar la delegación de autenticación Kerberos desde el proxy de aplicación (conector). El servidor de aplicaciones se ejecuta en el contexto de `webserviceaccount` y el servidor de delegación es `connectorcomputeraccount`. Ejecute los siguientes comandos en un controlador de dominio (con Windows Server 2012 R2 o posterior) en el dominio de `webserviceaccount`. Use nombres planos (no UPN) para ambas cuentas.

   Si `webserviceaccount` es una cuenta de equipo, use estos comandos:

   ```powershell
   $connector= Get-ADComputer -Identity connectorcomputeraccount -server dc.connectordomain.com

   Set-ADComputer -Identity webserviceaccount -PrincipalsAllowedToDelegateToAccount $connector

   Get-ADComputer webserviceaccount -Properties PrincipalsAllowedToDelegateToAccount
   ```

   Si `webserviceaccount` es una cuenta de usuario, use estos comandos:

   ```powershell
   $connector= Get-ADComputer -Identity connectorcomputeraccount -server dc.connectordomain.com

   Set-ADUser -Identity webserviceaccount -PrincipalsAllowedToDelegateToAccount $connector

   Get-ADUser webserviceaccount -Properties PrincipalsAllowedToDelegateToAccount
   ```

## <a name="configure-single-sign-on"></a>Configurar inicio de sesión único 
1. Publique la aplicación según las instrucciones de [Publicar aplicaciones con el proxy de aplicación](../app-proxy/application-proxy-add-on-premises-application.md). Asegúrese de seleccionar **Azure Active Directory** como **Método de autenticación previa**.
2. Cuando la aplicación aparezca en la lista de aplicaciones empresariales, selecciónela y haga clic en **Inicio de sesión único**.
3. Establezca el modo de inicio de sesión único en **Autenticación integrada de Windows**.  
4. Escriba el **SPN de la aplicación interno** del servidor de aplicaciones. En este ejemplo, el SPN para nuestra aplicación publicada es http/www.contoso.com. Este SPN debe estar en la lista de servicios a los que el conector puede presentar credenciales delegadas. 
5. Elija la **Identidad de inicio de sesión delegada** que va a usar el conector en nombre de los usuarios. Para más información, vea [Trabajar con identidades en la nube y locales diferentes](#working-with-different-on-premises-and-cloud-identities)

   ![Configuración avanzada de aplicaciones](./media/application-proxy-configure-single-sign-on-with-kcd/cwap_auth2.png)  

## <a name="sso-for-non-windows-apps"></a>SSO para las aplicaciones no son de Windows

El flujo de la delegación de Kerberos del Proxy de aplicación de Azure AD se inicia cuando Azure AD autentica al usuario en la nube. Una vez que la solicitud se recibe en local, el conector de Proxy de aplicación de Azure AD emite un vale Kerberos en nombre del usuario mediante la interacción con Active Directory local. Este proceso se conoce como Delegación limitada de Kerberos (KCD). 

En la fase siguiente, se envía una solicitud a la aplicación back-end con este vale Kerberos. 

Hay varios mecanismos que definen cómo enviar el vale Kerberos en estas solicitudes. La mayoría de los servidores que no son de Windows esperan recibirlo en forma de token SPNEGO. Este mecanismo es compatible con Azure Active Directory Application Proxy pero está deshabilitado de forma predeterminada. Se puede configurar un conector para SPNEGO o para un token Kerberos estándar, pero no para ambos.

Si configura una máquina de conector para SPNEGO, asegúrese de que todos los demás conectores del grupo de conectores también estén configurados con SPNEGO. Las aplicaciones que esperan un token Kerberos estándar deben enrutarse a través de otros conectores que no estén configurados para SPNEGO. Algunas aplicaciones web aceptan ambos formatos sin necesidad de ningún cambio en la configuración. 
 

Para habilitar SPNEGO:

1. Abra un símbolo del sistema que se ejecute como administrador.
2. Desde el símbolo del sistema, ejecute los siguientes comandos en los servidores de conector que necesiten SPNEGO.

    ```
    REG ADD "HKLM\SOFTWARE\Microsoft\Microsoft AAD App Proxy Connector" /v UseSpnegoAuthentication /t REG_DWORD /d 1
    net stop WAPCSvc & net start WAPCSvc
    ```

Las aplicaciones que no son de Windows normalmente usan nombres de usuario o nombres de cuenta SAM en lugar de direcciones de correo electrónico de dominio. Si esta situación se aplica a sus aplicaciones, debe configurar el campo de identidad de inicio de sesión delegada para conectar las identidades de nube a las identidades de aplicación. 

## <a name="working-with-different-on-premises-and-cloud-identities"></a>Trabajar con diferentes identidades locales y de nube
Proxy de aplicación supone que los usuarios tienen exactamente la misma identidad en la nube y en local. Sin embargo, en algunos entornos, debido a las directivas corporativas o a las dependencias de la aplicación, es posible que las organizaciones tengan que usar id. alternativos para el inicio de sesión. En esos casos, puede seguir usando KCD para el inicio de sesión único. Configure una **Identidad de inicio de sesión delegada** para cada aplicación para especificar qué identidad se debe usar al realizar el inicio de sesión único.  

Esta funcionalidad permite a muchas organizaciones que tienen identidades locales y en la nube diferentes disponer de inicio de sesión único en aplicaciones locales desde aplicaciones en la nube sin que los usuarios tengan que escribir nombres de usuario y contraseñas diferentes. Esto incluye las organizaciones que:

* Tienen varios dominios internamente (joe@us.contoso.com, joe@eu.contoso.com) y un único dominio en la nube (joe@contoso.com).
* Tienen un nombre de dominio no enrutable internamente (joe@contoso.usa) y uno legal en la nube.
* No utilizan nombres de dominio internamente (joe)
* Usan distintos alias locales y en la nube. Por ejemplo, joe-johns@contoso.com en comparación con joej@contoso.com  

Con el proxy de aplicación, puede seleccionar qué identidad utilizar para obtener el vale de Kerberos. Esta configuración es por aplicación. Algunas de estas opciones son adecuadas para los sistemas que no aceptan el formato de dirección de correo electrónico, otras están pensadas para el inicio de sesión alternativo.

![Captura de pantalla de parámetro de identidad de inicio de sesión delegado](./media/application-proxy-configure-single-sign-on-with-kcd/app_proxy_sso_diff_id_upn.png)

Si se usa la identidad de inicio de sesión delegada, es posible que el valor no sea único para todos los dominios o bosques de la organización. Puede evitar este problema al publicar estas aplicaciones dos veces con dos grupos diferentes de conector. Puesto que cada aplicación tiene una audiencia de usuarios diferente, puede unir sus conectores a un dominio diferente.

Si se utiliza **Nombre de cuenta SAM local** para la identidad de inicio de sesión, el equipo que hospeda el conector debe agregarse al dominio en el que se encuentra la cuenta de usuario.

### <a name="configure-sso-for-different-identities"></a>Configurar el inicio de sesión único para distintas identidades
1. Configure Azure AD Connect para que la identidad principal sea la dirección de correo electrónico (correo). Esto se realiza como parte del proceso de personalización, al cambiar el campo **Nombre principal de usuario** en la configuración de sincronización. Esta configuración también determina la forma en que los usuarios inician sesión en dispositivos con Office 365 y Windows10, y otras aplicaciones que usan Azure AD como almacén de identidades.  
   ![Captura de pantalla de la identificación de usuarios - lista desplegable de Nombre principal de usuario](./media/application-proxy-configure-single-sign-on-with-kcd/app_proxy_sso_diff_id_connect_settings.png)  
2. En las opciones de Configuración de la aplicación correspondientes a la aplicación que quiere modificar, seleccione la información en **Identidad de inicio de sesión delegada** que quiere usar:

   * Nombre principal de usuario (por ejemplo, joe@contoso.com).
   * Nombre principal de usuario alternativo (por ejemplo, joed@contoso.local).
   * Parte nombre de usuario del nombre principal de usuario (por ejemplo, joe).
   * Parte nombre de usuario del nombre principal de usuario alternativo (por ejemplo, joed).
   * Nombre de cuenta SAM local (depende de la configuración del controlador de dominio).

### <a name="troubleshooting-sso-for-different-identities"></a>Solución de problemas de SSO para diferentes identidades
Si se produce un error en el proceso de inicio de sesión único, aparece en el registro de eventos del equipo de conexión, como se explica en [Solución de problemas](application-proxy-back-end-kerberos-constrained-delegation-how-to.md).
Pero, en algunos casos, la solicitud se envía correctamente a la aplicación del back-end mientras que esta aplicación responde en otras respuestas HTTP. La solución de problemas de estos casos debe empezar por examinar el número de evento 24029 en el equipo de conexión en el registro de eventos de sesión de Proxy de aplicación. La identidad del usuario que se usó para la delegación aparece en el campo “usuario” de los detalles del evento. Para activar el registro de sesión, seleccione **Mostrar registros analíticos y de depuración** en el menú de vista del Visor de eventos.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de una aplicación de proxy de aplicación para que use la delegación restringida de Kerberos](application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
* [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)