---
title: Habilitación del acceso remoto a Power BI con Azure AD Application Proxy
description: En este artículo se explican los conceptos básicos sobre cómo integrar una instancia de Power BI local con Azure AD Application Proxy.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/25/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: 811eae15baec603029d19b24252a1487d65357ac
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108224560"
---
# <a name="enable-remote-access-to-power-bi-mobile-with-azure-ad-application-proxy"></a>Habilitación del acceso remoto a Power BI Mobile con Azure AD Application Proxy

En este artículo se describe cómo usar Azure AD Application Proxy para habilitar la conexión de la aplicación Power BI Mobile a Power BI Report Server (PBIRS) y SQL Server Reporting Services (SSRS) 2016 y versiones posteriores. A través de esta integración, los usuarios que están fuera de la red corporativa pueden acceder a sus informes de Power BI desde la aplicación Power BI Mobile y estar protegidos mediante la autenticación de Azure AD. Esta protección incluye [ventajas de seguridad](application-proxy-security.md#security-benefits) como el acceso condicional y la autenticación multifactor.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por hecho que ya se han implementado los servicios de informes y se ha [habilitado Application Proxy](../app-proxy/application-proxy-add-on-premises-application.md).

- La habilitación de Application Proxy requiere instalar un conector en un servidor de Windows y cumplir los [requisitos previos](../app-proxy/application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment) para que el conector pueda comunicarse con los servicios de Azure AD.
- Al publicar Power BI, se recomienda usar los mismos dominios internos y externos. Para más información sobre los dominios personalizados, consulte el artículo [Uso de dominios personalizados en el proxy de la aplicación de Azure AD](./application-proxy-configure-custom-domain.md).
- Esta integración está disponible para la aplicación **Power BI Mobile para iOS y Android**.

## <a name="step-1-configure-kerberos-constrained-delegation-kcd"></a>Paso 1: Configuración de la delegación restringida de Kerberos (KCD)

Para las aplicaciones locales que usan la autenticación de Windows, puede realizar el inicio de sesión único (SSO) mediante el protocolo de autenticación Kerberos y una característica denominada delegación restringida de Kerberos (KCD). Cuando está configurada, la KCD permite que el conector de Application Proxy obtenga un token de Windows para un usuario, incluso aunque el usuario no haya iniciado sesión en Windows directamente. Para más información sobre KCD, consulte [Información general sobre la delegación restringida de Kerberos](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)) y [Delegación restringida de Kerberos para el inicio de sesión único en las aplicaciones con Application Proxy](application-proxy-configure-single-sign-on-with-kcd.md).

En lo que respecta a Reporting Services, no hay mucho que configurar. Simplemente asegúrese de tener un nombre de entidad de seguridad de servicio (SPN) válido para permitir que se produzca la autenticación Kerberos adecuada. Asegúrese también de que el servidor de Reporting Services esté habilitado para la autenticación de negociación.

Para configurar KCD para Reporting Services, continúe con los pasos siguientes.

### <a name="configure-the-service-principal-name-spn"></a>Configuración del nombre de entidad de seguridad de servicio (SPN)

El SPN es un identificador único para un servicio que usa la autenticación Kerberos. Deberá asegurarse de tener un SPN HTTP adecuado presente para el servidor de informes. Para información sobre cómo configurar el nombre de entidad de seguridad de servicio (SPN) adecuado para el servidor de informes, consulte [Registrar un nombre principal de servicio (SPN) para un servidor de informes](/sql/reporting-services/report-server/register-a-service-principal-name-spn-for-a-report-server).
Puede comprobar que el SPN se agregó mediante la ejecución del comando Setspn con la opción -l. Para más información sobre este comando, vea [Setspn](https://social.technet.microsoft.com/wiki/contents/articles/717.service-principal-names-spn-setspn-syntax.aspx).

### <a name="enable-negotiate-authentication"></a>Habilitación de la autenticación de negociación

Para habilitar el uso de la autenticación Kerberos por parte de un servidor de informes, configure el tipo de autenticación del servidor de informes para que sea RSWindowsNegotiate. Configure esta opción mediante el archivo rsreportserver.config.

```xml
<AuthenticationTypes>
    <RSWindowsNegotiate />
    <RSWindowsKerberos />
    <RSWindowsNTLM />
</AuthenticationTypes>
```

Para más información, consulte [Modificar un archivo de configuración de Reporting Services](/sql/reporting-services/report-server/modify-a-reporting-services-configuration-file-rsreportserver-config) y [Configurar la autenticación de Windows en el servidor de informes](/sql/reporting-services/security/configure-windows-authentication-on-the-report-server).

### <a name="ensure-the-connector-is-trusted-for-delegation-to-the-spn-added-to-the-reporting-services-application-pool-account"></a>Asegúrese de que el conector es de confianza para delegación en el SPN que se agrega a la cuenta del grupo de aplicaciones Reporting Services
Configure KCD para que el servicio de Azure AD Application Proxy pueda delegar las identidades de usuario en la cuenta de agrupación de la aplicación Reporting Services. Para configurar KCD, habilite el conector del proxy de aplicación para recuperar los vales Kerberos para los usuarios que se autenticaron en Azure AD. A continuación, ese servidor pasa el contexto a la aplicación de destino, o a Reporting Services en este caso.

Para configurar KCD, repita los pasos siguientes para cada equipo de conexión:

1. Inicie sesión como administrador de dominio en un controlador de dominio y, después, abra **Usuarios y equipos de Active Directory**.
2. Busque el equipo en el que se ejecuta el conector.
3. Haga doble clic en el equipo y, después, seleccione la pestaña **Delegación**.
4. Establezca la configuración de delegación en **Confiar en este equipo para la delegación solo en los servicios especificados**. Después, seleccione **Usar cualquier protocolo de autenticación**.
5. Seleccione **Agregar** y, luego, **Usuarios o equipos**.
6. Escriba la cuenta de servicio que está usando para Reporting Services. Se trata de la cuenta a la que agregó el SPN en la configuración de Reporting Services.
7. Haga clic en **OK**. Haga clic en **Aceptar** de nuevo para guardar los cambios.

Para obtener más información, consulte [Delegación restringida de Kerberos para el inicio de sesión único para las aplicaciones con Proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md).

## <a name="step-2-publish-report-services-through-azure-ad-application-proxy"></a>Paso 2: Publicación de servicios de informes a través de Azure AD Application Proxy

Ahora ya está a punto para configurar Azure AD Application Proxy.

1. Publique los servicios de informes a través de Application Proxy con la configuración siguiente. Para obtener instrucciones paso a paso sobre cómo publicar una aplicación mediante Application Proxy, consulte [Publicación de aplicaciones con Azure AD Application Proxy](../app-proxy/application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).
   - **Dirección URL interna**: Escriba la dirección URL del servidor de informes al cual el conector puede acceder en la red corporativa. Asegúrese de que esta dirección URL sea accesible desde el servidor en el que está instalado el conector. Un procedimiento recomendado es el uso de un dominio de nivel superior como `https://servername/` para evitar problemas con subrutas publicadas a través de Application Proxy. Por ejemplo, use `https://servername/` y no `https://servername/reports/` o `https://servername/reportserver/`.
     > [!NOTE]
     > Se recomienda usar una conexión HTTPS segura con el servidor de informes. Consulte [Configuración de conexiones SSL en un servidor de informes en modo nativo](/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server) para obtener información sobre cómo hacerlo.
   - **Dirección URL externa**: escriba la dirección URL pública a la que se conectará la aplicación Power BI Mobile. Por ejemplo, puede ser similar a `https://reports.contoso.com` si se utiliza un dominio personalizado. Para usar un dominio personalizado, cargue un certificado para el dominio y apunte un registro DNS al dominio msappproxy.net predeterminado de la aplicación. Para obtener los pasos detallados, consulte [Uso de dominios personalizados en Azure AD Application Proxy](application-proxy-configure-custom-domain.md).

   - **Método de autenticación previa**: Azure Active Directory

2. Una vez publicada la aplicación, establezca la configuración de inicio de sesión único según los pasos siguientes:

   a. En la página de la aplicación en el portal, seleccione **Inicio de sesión único**.

   b. En **Modo de inicio de sesión único**, seleccione **Autenticación de Windows integrada**.

   c. Establezca el **SPN de la aplicación interno** en el valor establecido anteriormente.

   d. Elija la **Identidad de inicio de sesión delegada** que va a usar el conector en nombre de los usuarios. Para más información, consulte [Trabajar con diferentes identidades locales y de nube](application-proxy-configure-single-sign-on-with-kcd.md#working-with-different-on-premises-and-cloud-identities).

   e. Haga clic en **Guardar** para guardar los cambios.

Para finalizar la configuración de la aplicación, vaya a la sección **Usuarios y grupos** y asigne usuarios para que accedan a esta aplicación.

## <a name="step-3-modify-the-reply-uris-for-the-application"></a>Paso 3: Modificación del URI de respuesta para la aplicación

Para que la aplicación móvil de Power BI pueda conectarse y acceder a los servicios de informes, debe configurar el registro de aplicación que se creó automáticamente en el paso 2.

1. En la página **Introducción** de Azure Active Directory, haga clic en **Registros de aplicaciones**.
2. En la pestaña **Todas las aplicaciones**, busque la aplicación que creó en el paso 2.
3. Seleccione la aplicación y luego **Autenticación**.
4. Agregue los siguientes URI de redirección en función de la plataforma que use.

   Al configurar la aplicación para Power BI Mobile **iOS**, agregue los siguientes URI de redireccionamiento de tipo cliente público (móvil y escritorio):
   - `msauth://code/mspbi-adal%3a%2f%2fcom.microsoft.powerbimobile`
   - `msauth://code/mspbi-adalms%3a%2f%2fcom.microsoft.powerbimobilems`
   - `mspbi-adal://com.microsoft.powerbimobile`
   - `mspbi-adalms://com.microsoft.powerbimobilems`

   Al configurar la aplicación para Power BI Mobile **Android**, agregue los siguientes URI de redireccionamiento de tipo cliente público (móvil y escritorio):
   - `urn:ietf:wg:oauth:2.0:oob`
   - `mspbi-adal://com.microsoft.powerbimobile`
   - `msauth://com.microsoft.powerbim/g79ekQEgXBL5foHfTlO2TPawrbI%3D`
   - `msauth://com.microsoft.powerbim/izba1HXNWrSmQ7ZvMXgqeZPtNEU%3D`

   > [!IMPORTANT]
   > Se deben agregar los URI de redirección para que la aplicación funcione correctamente. Si va a configurar la aplicación para Power BI Mobile iOS y Android, agregue el siguiente URI de redireccionamiento de tipo cliente público (móvil y escritorio) a la lista de URI de redireccionamiento configurados para iOS: `urn:ietf:wg:oauth:2.0:oob`.

## <a name="step-4-connect-from-the-power-bi-mobile-app"></a>Paso 4: Conexión desde la aplicación Power BI Mobile

1. En la aplicación Power BI Mobile, conéctese a la instancia de Reporting Services. Para ello, escriba la **dirección URL externa** de la aplicación publicada mediante Application Proxy.

   ![Aplicación Power BI Mobile con dirección URL externa](media/application-proxy-integrate-with-power-bi/app-proxy-power-bi-mobile-app.png)

2. Seleccione **Conectar**. Se le dirigirá a la página de inicio de sesión de Azure Active Directory.

3. Escriba las credenciales válidas para el usuario y seleccione **Iniciar sesión**. Verá los elementos del servidor de Reporting Services.

## <a name="step-5-configure-intune-policy-for-managed-devices-optional"></a>Paso 5: Configuración de la directiva de Intune para dispositivos administrados (opcional)

Puede usar Microsoft Intune para administrar las aplicaciones cliente que utiliza el personal de su empresa. Intune permite usar funcionalidades como el cifrado de datos y requisitos de acceso adicionales. Para obtener más información sobre la administración de aplicaciones a través de Intune, consulte Administración de aplicaciones de Intune. Para habilitar la aplicación Power BI Mobile para que funcione con la directiva de Intune, siga estos pasos.

1. Vaya a **Azure Active Directory** y, luego, a **Registros de aplicaciones**.
2. Seleccione la aplicación configurada en el paso 3 al registrar la aplicación cliente nativa.
3. En la página de la aplicación, seleccione **Permisos de API**.
4. Haga clic en **Agregar un permiso**.
5. En **API que mi organización usa**, busque la opción "Administración de aplicaciones móviles de Microsoft" y selecciónela.
6. Adición del permiso **DeviceManagementManagedApps.ReadWrite** a la aplicación
7. Haga clic en **Conceder consentimiento de administrador** para conceder el permiso de acceso a la aplicación.
8. Configure la directiva de Intune que quiera consultando el [procedimiento de creación y asignación de directivas de protección de aplicaciones](/intune/app-protection-policies).

## <a name="troubleshooting"></a>Solución de problemas

Si la aplicación devuelve una página de error después de intentar cargar un informe durante varios minutos, puede que tenga que cambiar la configuración del tiempo de espera. De forma predeterminada, Application Proxy admite aplicaciones que tardan un máximo de 85 segundos en responder a una solicitud. Para aumentar este valor a 180 segundos, seleccione el tiempo de espera de back-end como **Largo** en la página de configuración de Application Proxy para la aplicación. Para obtener sugerencias sobre procedimientos para crear informes rápidos y de confianza, consulte los [procedimientos recomendados para informes de Power BI](/power-bi/power-bi-reports-performance).

El uso de Azure AD Application Proxy para permitir que la aplicación móvil de Power BI se conecte a la versión local de Power BI Report Server no es compatible con las directivas de acceso condicional que requieren la aplicación de Microsoft Power BI como una aplicación cliente aprobada.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación de las aplicaciones de cliente nativas para interactuar con el proxy de aplicaciones](application-proxy-configure-native-client-application.md)
- [Visualización de informes y KPI del servidor de informes local en las aplicaciones Power BI Mobile](/power-bi/consumer/mobile/mobile-app-ssrs-kpis-mobile-on-premises-reports)
