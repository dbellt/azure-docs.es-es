---
title: 'Tutorial: Integración de Azure Active Directory con SharePoint local | Microsoft Docs'
description: Aprenda a implementar la autenticación federada entre Azure Active Directory y SharePoint local.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: jeedes
ms.openlocfilehash: be61dad30d7c8670442612faf928d5aa82e64d56
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110065753"
---
# <a name="tutorial-implement-federated-authentication-between-azure-active-directory-and-sharepoint-on-premises"></a>Tutorial: Implementación de la autenticación federada entre Azure Active Directory y SharePoint local

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, configurará la autenticación federada entre Azure Active Directory y SharePoint local. El objetivo es permitir que los usuarios inicien sesión en Azure Active Directory y usen su identidad para acceder a los sitios locales de SharePoint.

## <a name="prerequisites"></a>Requisitos previos

Para realizar la configuración, necesita los siguientes recursos:
* Un inquilino de Azure Active Directory. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una granja de servidores de SharePoint 2013, o posterior.

En este artículo se usan los siguientes valores:
- Nombre de la aplicación empresarial (en Azure AD): `SharePoint corporate farm`
- Identificador de confianza (en Azure AD) o dominio (en SharePoint): `urn:sharepoint:federation`
- loginUrl (a Azure AD): `https://login.microsoftonline.com/dc38a67a-f981-4e24-ba16-4443ada44484/wsfed`
- Dirección URL del sitio de SharePoint: `https://spsites.contoso.local/`
- Dirección URL de respuesta del sitio de SharePoint: `https://spsites.contoso.local/_trust/`
- Nombre de configuración de confianza de SharePoint: `AzureADTrust`
- UserPrincipalName del usuario de prueba de Azure AD: `AzureUser1@demo1984.onmicrosoft.com`

## <a name="configure-an-enterprise-application-in-azure-active-directory"></a>Configuración de una aplicación empresarial en Azure Active Directory

Para configurar la federación en Azure AD, debe crear una aplicación empresarial dedicada. Su configuración se simplifica mediante la plantilla preconfigurada `SharePoint on-premises`, que se puede encontrar en la galería de aplicaciones.

### <a name="create-the-enterprise-application"></a>Creación de la aplicación empresarial

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/).
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.
1. En el cuadro de búsqueda, escriba **SharePoint local**. Seleccione **SharePoint local** en el panel de resultados.
1. Especifique un nombre de la aplicación (en este tutorial, es `SharePoint corporate farm`) y haga clic en **Crear** para agregar la aplicación.
1. En la nueva aplicación empresarial, seleccione **Propiedades** y compruebe el valor de **¿Asignación de usuarios?** . En este escenario, establezca su valor en **No** y haga clic en **Guardar.**

### <a name="configure-the-enterprise-application"></a>Configuración de la aplicación empresarial

En esta sección, configurará la autenticación SAML y definirá las notificaciones que se enviarán a SharePoint después de una autenticación correcta.

1. En la información general de la aplicación empresarial `SharePoint corporate farm`, seleccione **2. Set up single sign-on** (Configurar inicio de sesión único) y elija **SAML** en el siguiente cuadro de diálogo.
 
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono **Editar** en el panel **Configuración básica de SAML**.

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    1. En el cuadro **Identificador**, asegúrese de que este valor exista: `urn:sharepoint:federation`.

    1. En el cuadro **Dirección URL de respuesta**, escriba una dirección URL con este patrón: `https://spsites.contoso.local/_trust/`.

    1. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con este patrón: `https://spsites.contoso.local/`.
    
    1. Seleccione **Guardar**.

1. En la sección **User Attributes & Claims** (Atributos y notificaciones del usuario), elimine los siguientes tipos de notificaciones, que no sirven para nada dado que SharePoint no las usará para conceder permisos:
    - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`
    - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
    - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

1. La configuración debería ser ahora similar a la siguiente:

    ![Configuración básica de SAML](./media/sharepoint-on-premises-tutorial/azure-active-directory-app-saml-ids.png)

1. Copie la información que necesitará más adelante en SharePoint:

    - En la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** . Esta es la clave pública del certificado de firma que usa Azure AD para firmar el token SAML. SharePoint lo necesitará para comprobar la integridad de los tokens SAML entrantes.

    - En la sección **Set up SharePoint corporate farm** (Configurar SharePoint corporate farm), copie el valor de **URL de inicio de sesión** en un bloc de notas y reemplace la cadena final **/saml2** por **/wsfed**.
     
    > [!IMPORTANT]
    > Asegúrese de reemplazar **/saml2** por **/wsfed** para que Azure AD emita un token SAML 1.1, como solicita SharePoint.

    - En la sección **Set up SharePoint corporate farm** (Configurar SharePoint corporate farm), copie el valor de **URL de cierre de sesión**

## <a name="configure-sharepoint-to-trust-azure-active-directory"></a>Configuración de SharePoint para que confíe en Azure Active Directory

### <a name="create-the-trust-in-sharepoint"></a>Creación de la confianza en SharePoint

En este paso, creará un elemento SPTrustedLoginProvider para almacenar la configuración que necesita SharePoint para confiar en Azure AD. Para ello, necesita la información de Azure AD que copió anteriormente. Inicie el shell de administración de SharePoint y ejecute el siguiente script para crearlo:

```powershell
# Path to the public key of the Azure AD SAML signing certificate (self-signed), downloaded from the Enterprise application in the Azure AD portal
$signingCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("C:\AAD app\SharePoint corporate farm.cer")
# Unique realm (corresponds to the "Identifier (Entity ID)" in the Azure AD Enterprise application)
$realm = "urn:sharepoint:federation"
# Login URL copied from the Azure AD enterprise application. Make sure to replace "saml2" with "wsfed" at the end of the URL:
$loginUrl = "https://login.microsoftonline.com/dc38a67a-f981-4e24-ba16-4443ada44484/wsfed"

# Define the claim types used for the authorization
$userIdentifier = New-SPClaimTypeMapping -IncomingClaimType `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` -IncomingClaimTypeDisplayName "name" -LocalClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"
$role = New-SPClaimTypeMapping "http://schemas.microsoft.com/ws/2008/06/identity/claims/role" -IncomingClaimTypeDisplayName "Role" -SameAsIncoming

# Let SharePoint trust the Azure AD signing certificate
New-SPTrustedRootAuthority -Name "Azure AD signing certificate" -Certificate $signingCert

# Create a new SPTrustedIdentityTokenIssuer in SharePoint
$trust = New-SPTrustedIdentityTokenIssuer -Name "AzureADTrust" -Description "Azure AD" -Realm $realm -ImportTrustCertificate $signingCert -ClaimsMappings $userIdentifier, $role -SignInUrl $loginUrl -IdentifierClaim $userIdentifier.InputClaimType
```

### <a name="configure-the-sharepoint-web-application"></a>Configuración de la aplicación web de SharePoint

En este paso, configurará una aplicación web en SharePoint para que confíe en la aplicación empresarial de Azure AD creada anteriormente. Hay reglas importantes que es necesario tener en cuenta:
- La zona predeterminada de la aplicación web de SharePoint debe tener habilitada la autenticación de Windows. Esta opción es obligatoria para el rastreador de búsqueda.
- La dirección URL de SharePoint que usará la autenticación de Azure AD debe establecerse con HTTPS.

1. Cree o amplíe la aplicación web. En este artículo se describen dos configuraciones posibles:

    - Si crea una aplicación web que usa tanto la autenticación de Windows como la de Azure AD en la zona predeterminada:

        1. Inicie el **shell de administración de SharePoint** y ejecute el siguiente script:
            ```powershell
            # This script creates a new web application and sets Windows and Azure AD authentication on the Default zone
            # URL of the SharePoint site federated with Azure AD
            $trustedSharePointSiteUrl = "https://spsites.contoso.local/"
            $applicationPoolManagedAccount = "Contoso\spapppool"

            $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$true
            $sptrust = Get-SPTrustedIdentityTokenIssuer "AzureADTrust"
            $trustedAp = New-SPAuthenticationProvider -TrustedIdentityTokenIssuer $sptrust    
            
            New-SPWebApplication -Name "SharePoint - Azure AD" -Port 443 -SecureSocketsLayer -URL $trustedSharePointSiteUrl -ApplicationPool "SharePoint - Azure AD" -ApplicationPoolAccount (Get-SPManagedAccount $applicationPoolManagedAccount) -AuthenticationProvider $winAp, $trustedAp
            ```
        1. Abra el sitio **Administración central de SharePoint**.
        1. En **Configuración del sistema**, seleccione **Configurar asignaciones de acceso alternativas**. Se abre el cuadro **Colección de asignaciones de acceso alternativas**.
        1. Filtre la visualización con la nueva aplicación web y confirme que se ve algo parecido a esto:
    
           ![Asignaciones de acceso alternativas de la aplicación web](./media/sharepoint-on-premises-tutorial/sp-alternate-access-mappings-new-web-app.png)

    - Si amplía una aplicación web existente para usar la autenticación de Azure AD en una nueva zona:

        1. Inicie el shell de administración de SharePoint y ejecute el siguiente script:

            ```powershell
            # This script extends an existing web application to set Azure AD authentication on a new zone
            # URL of the default zone of the web application
            $webAppDefaultZoneUrl = "http://spsites/"
            # URL of the SharePoint site federated with ADFS
            $trustedSharePointSiteUrl = "https://spsites.contoso.local/"
            $sptrust = Get-SPTrustedIdentityTokenIssuer "AzureADTrust"
            $ap = New-SPAuthenticationProvider -TrustedIdentityTokenIssuer $sptrust
            $wa = Get-SPWebApplication $webAppDefaultZoneUrl
            
            New-SPWebApplicationExtension -Name "SharePoint - Azure AD" -Identity $wa -SecureSocketsLayer -Zone Internet -Url $trustedSharePointSiteUrl -AuthenticationProvider $ap
            ```
        
        1. Abra el sitio **Administración central de SharePoint**.
        1. En **Configuración del sistema**, seleccione **Configurar asignaciones de acceso alternativas**. Se abre el cuadro **Colección de asignaciones de acceso alternativas**.
        1. Filtre la visualización con la aplicación web que se ha extendido y confirme que se muestra algo parecido a esto:
    
            ![Asignaciones de acceso alternativas de la aplicación web ampliada](./media/sharepoint-on-premises-tutorial/sp-alternate-access-mappings-extended-zone.png)

Una vez creada la aplicación web, puede crear una colección de sitios raíz y agregar su cuenta de Windows como administrador de la colección del sitio principal.

1. Creación de un certificado para el sitio de SharePoint

    Dado que la dirección URL de SharePoint usa el protocolo HTTPS (`https://spsites.contoso.local/`), debe establecerse un certificado en el sitio de Internet Information Services (IIS) correspondiente. Siga estos pasos para generar un certificado autofirmado:
    
    > [!IMPORTANT]
    > Los certificados autofirmados son adecuados solo para pruebas. En los entornos de producción, se recomienda encarecidamente que use certificados emitidos por una entidad de certificación.
    
    1. Abra la consola Windows PowerShell.
    1. Ejecute el siguiente script para generar un certificado autofirmado y agregarlo al almacén My del equipo:
    
        ```powershell
        New-SelfSignedCertificate -DnsName "spsites.contoso.local" -CertStoreLocation "cert:\LocalMachine\My"
        ```
    
1. Establecimiento del certificado en el sitio de IIS
    1. Abra la consola del administrador de Internet Information Services.
    1. Expanda el servidor en la vista de árbol, expanda **Sitios**, seleccione el sitio **SharePoint - Azure AD** y elija **Enlaces**.
    1. Seleccione el **enlace HTTPS** y después haga clic en **Editar**.
    1. En el campo Certificado TLS/SSL, elija el certificado que usará (por ejemplo, **spsites.contoso.local** creado anteriormente) y seleccione **Aceptar**.
    
    > [!NOTE]
    > Si tiene varios servidores front-end web, debe repetir esta operación en cada uno.

Ahora la configuración básica de la confianza entre SharePoint y Azure AD ha finalizado. Veamos cómo iniciar sesión en el sitio de SharePoint como usuario de Azure Active Directory.

## <a name="sign-in-as-a-member-user"></a>Inicio de sesión como usuario miembro

Azure Active Directory tiene [dos tipos de usuarios](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-user-properties): usuarios invitados y usuarios miembros. Empecemos con un usuario miembro, que es simplemente un usuario que se aloja en su organización.

### <a name="create-a-member-user-in-azure-active-directory"></a>Creación de un usuario miembro en Azure Active Directory

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory**. En el panel **Administrar**, seleccione **Usuarios**.

1. Seleccione **Todos los usuarios** > **Nuevo usuario** en la parte superior de la pantalla.

1. Seleccione **Crear usuario** y, en las propiedades del usuario, siga estos pasos.

    1. En el cuadro **Nombre**, escriba el nombre del usuario. Vamos a utilizar **TestUser**.
  
    1. En el cuadro **Nombre de usuario**, escriba `AzureUser1@<yourcompanytenant>.onmicrosoft.com`. En este ejemplo se muestra `AzureUser1@demo1984.onmicrosoft.com`:

       ![Cuadro de diálogo Usuario](./media/sharepoint-on-premises-tutorial/azure-active-directory-new-user.png)

    1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que aparece en el cuadro **Contraseña**.

    1. Seleccione **Crear**.

    1. Ahora puede compartir el sitio con `AzureUser1@demo1984.onmicrosoft.com` y permitir que este usuario tenga acceso a él.

### <a name="grant-permissions-to-the-azure-active-directory-user-in-sharepoint"></a>Concesión de permisos al usuario de Azure Active Directory en SharePoint

Inicie sesión en la colección de sitios raíz de SharePoint con su cuenta de Windows (administrador de la colección de sitios) y haga clic en **Compartir**.  
En el cuadro de diálogo, debe escribir el valor exacto de userprincipalname, por ejemplo `AzureUser1@demo1984.onmicrosoft.com`, y tener cuidado de seleccionar el resultado de la notificación de **nombre** (mueva el mouse sobre un resultado para ver su tipo de notificación).

> [!IMPORTANT]
> Tenga cuidado de escribir el valor exacto del usuario al que quiere invitar y elija el tipo de notificación adecuado de la lista; de lo contrario, el uso compartido no funcionará.

![Resultados del selector de personas sin AzureCP](./media/sharepoint-on-premises-tutorial/sp-people-picker-search-no-azurecp.png)

Esta limitación se debe a que SharePoint no valida la entrada del selector de personas, lo que puede resultar confuso y provocar errores ortográficos o que los usuarios elijan accidentalmente el tipo de notificación incorrecto.  
Para corregir esta situación, se puede usar una solución de código abierto denominada [AzureCP](https://yvand.github.io/AzureCP/) para conectar SharePoint 2019/2016/2013 con Azure Active Directory y resolver la entrada en el inquilino de Azure Active Directory. Para más información, consulte [AzureCP](https://yvand.github.io/AzureCP/).

A continuación, se muestra la misma búsqueda con AzureCP configurado: SharePoint devuelve usuarios reales en función de la entrada:

![Resultados del selector de personas con AzureCP](./media/sharepoint-on-premises-tutorial/sp-people-picker-search-with-azurecp.png)

> [!IMPORTANT]
> AzureCP no es un producto de Microsoft y no dispone de soporte técnico por parte del equipo de Soporte técnico de Microsoft. Para descargar, instalar y configurar AzureCP en la granja de servidores de SharePoint local, consulte el sitio web de [AzureCP](https://yvand.github.io/AzureCP/). 

El usuario de Azure Active Directory `AzureUser1@demo1984.onmicrosoft.com` puede usar ahora su identidad para iniciar sesión en el sitio de SharePoint `https://spsites.contoso.local/`.

## <a name="grant-permissions-to-a-security-group"></a>Concesión de permisos a un grupo de seguridad

### <a name="add-the-group-claim-type-to-the-enterprise-application"></a>Adición del tipo de notificación de grupo a la aplicación empresarial

1. En la información general de la aplicación empresarial `SharePoint corporate farm`, seleccione **2. Set up single sign-on** (Configurar inicio de sesión único). 

1. En la sección **User Attributes & Claims** (Atributos y notificaciones del usuario), siga estos pasos si no hay ninguna notificación de grupo existente:

    1. Seleccione **Agregar una notificación de grupo**, elija **Grupos de seguridad** y asegúrese de que **Atributo de origen** esté establecido en **Id. de grupo**.
    1. Active **Personalizar nombre de la notificación del grupo** y, luego, active **Emitir grupos como notificaciones de roles** y haga clic en **Guardar**.
    1. La sección **User Attributes & Claims** (Atributos y notificaciones del usuario) tiene el siguiente aspecto:

    ![Notificaciones para usuarios y grupos](./media/sharepoint-on-premises-tutorial/azure-active-directory-claims-with-group.png)

### <a name="create-a-security-group-in-azure-active-directory"></a>Creación de un grupo de seguridad en Azure Active Directory

Vamos a crear un grupo de seguridad en Azure Active Directory:

1. Seleccione **Azure Active Directory** > **Grupos**.

1. Seleccione **Nuevo grupo**.

1. Rellene los campos **Tipo de grupo** (Seguridad), **Nombre de grupo** (por ejemplo, `AzureGroup1`) y **Tipo de pertenencia**. Agregue el usuario que creó anteriormente como miembro y seleccione **Crear**:

    ![Creación de un grupo de seguridad de Azure AD](./media/sharepoint-on-premises-tutorial/azure-active-directory-new-group.png)
  
### <a name="grant-permissions-to-the-security-group-in-sharepoint"></a>Concesión de permisos al grupo de seguridad en SharePoint

Los grupos de seguridad de Azure AD se identifican con su atributo `Id`, que es un GUID (por ejemplo, `E89EF0A3-46CC-45BF-93A4-E078FCEBFC45`).  
Sin un proveedor de notificaciones personalizado, los usuarios deben escribir el valor exacto (`Id`) del grupo en el selector de personas y seleccionar el tipo de notificación correspondiente. Esta solución no es fácil de usar ni confiable.  
Para evitar esta situación, en este artículo se usa el proveedor de notificaciones de terceros [AzureCP](https://yvand.github.io/AzureCP/) para buscar el grupo de una manera fácil en SharePoint:

![Grupo de Azure AD de búsqueda del selector de personas](./media/sharepoint-on-premises-tutorial/sp-people-picker-search-azure-active-directory-group.png)

## <a name="manage-guest-users-access"></a>Administración del acceso de usuarios invitados

Existen dos tipos de cuentas de invitado:

- Cuentas de invitado B2B: esos usuarios se alojan en un inquilino externo de Azure Active Directory.
- Cuentas de invitado de MSA: esos usuarios se alojan en un proveedor de identificación de Microsoft (Hotmail, Outlook) o en un proveedor de cuentas de redes sociales (Google o similar).

De forma predeterminada, Azure Active Directory establece el "Identificador de usuario único" y la notificación "nombre" en el atributo `user.userprincipalname`.  
Desafortunadamente, este atributo es ambiguo para las cuentas de invitado, como se muestra en la tabla siguiente:

| Atributo de origen establecido en Azure AD | Propiedad real usada por Azure AD para invitados B2B | Propiedad real usada por Azure AD para invitados de MSA | Propiedad en la que SharePoint puede confiar para validar la identidad |
|--|--|--|--|
| `user.userprincipalname` | `mail`, por ejemplo, `guest@PARTNERTENANT` | `userprincipalname`, por ejemplo, `guest_outlook.com#EXT#@TENANT.onmicrosoft.com` | Ambigua |
| `user.localuserprincipalname` | `userprincipalname`, por ejemplo, `guest_PARTNERTENANT#EXT#@TENANT.onmicrosoft.com` | `userprincipalname`, por ejemplo, `guest_outlook.com#EXT#@TENANT.onmicrosoft.com` | `userprincipalname` |

Como conclusión, para que todas las cuentas de invitado se identifiquen con el mismo atributo, las notificaciones de identificador de la aplicación empresarial deben actualizarse para usar el atributo `user.localuserprincipalname` en lugar de `user.userprincipalname`.

### <a name="update-the-application-to-use-a-consistent-attribute-for-all-guest-users"></a>Actualización de la aplicación para usar un atributo coherente con todos los usuarios invitados

1. En la información general de la aplicación empresarial `SharePoint corporate farm`, seleccione **2. Set up single sign-on** (Configurar inicio de sesión único).
 
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono **Editar** para abrir el cuadro de diálogo **User Attributes & Claims** (Atributos y notificaciones del usuario).

1. Ahora, la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) tiene el siguiente aspecto:

    1. Seleccione **Identificador de usuario único (id. de nombre)** , cambie su propiedad **Atributo de origen** por **user.localuserprincipalname** y haga clic en **Guardar**.
    
    1. Seleccione `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` , cambie su propiedad **Atributo de origen** por **user.localuserprincipalname** y haga clic en **Guardar**.
    
    1. La sección **User Attributes & Claims** (Atributos y notificaciones del usuario) tiene el siguiente aspecto:
    
    ![Atributos y notificaciones del usuario para invitados](./media/sharepoint-on-premises-tutorial/azure-active-directory-claims-guests.png)

### <a name="invite-guest-users-in-sharepoint"></a>Invitación a los usuarios invitados en SharePoint

> [!NOTE]
> En esta sección se da por supuesto que se usa el proveedor de notificaciones AzureCP.

En la sección anterior, actualizó la aplicación empresarial para usar un atributo coherente con todas las cuentas de invitado.  
Ahora, la configuración de AzureCP debe actualizarse para que refleje ese cambio y usar el atributo `userprincipalname` en las cuentas de invitado:

1. Abra el sitio **Administración central de SharePoint**.
1. En **Seguridad**, seleccione **AzureCP global configuration** (Configuración global de AzureCP).
1. En la sección **User identifier property** (Propiedad de identificador de usuario): establezca **User identifier for 'Guest' users:** (Identificador de usuario para usuarios "invitados":) en **UserPrincipalName**.
1. Haga clic en Aceptar.

![Configuración de cuentas de invitados de AzureCP](./media/sharepoint-on-premises-tutorial/sp-azurecp-attribute-for-guests.png)

Ahora puede invitar a cualquier usuario invitado en los sitios de SharePoint.

## <a name="configure-the-federation-for-multiple-web-applications"></a>Configuración de la federación para varias aplicaciones web

La configuración funciona para una sola aplicación web, pero se necesitará configuración adicional si va a usar el mismo proveedor de identidades de confianza con varias aplicaciones web. Por ejemplo, suponga que tiene una aplicación web independiente `https://otherwebapp.contoso.local/` y quiere habilitar en ella la autenticación de Azure Active Directory. Para ello, configure SharePoint para pasar el parámetro WReply de SAML y agregue las direcciones URL en la aplicación empresarial.

### <a name="configure-sharepoint-to-pass-the-saml-wreply-parameter"></a>Configuración de SharePoint para pasar el parámetro WReply de SAML

1. En el servidor de SharePoint, abra el shell de administración de SharePoint 201x y ejecute los siguientes comandos. Use el mismo nombre para el emisor de tokens de identidad de confianza que utilizó anteriormente.

```powershell
$t = Get-SPTrustedIdentityTokenIssuer "AzureADTrust"
$t.UseWReplyParameter = $true
$t.Update()
```

### <a name="add-the-urls-in-the-enterprise-application"></a>Adición de las direcciones URL en la aplicación empresarial

1. En Azure Portal, seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione el nombre de la aplicación empresarial creada anteriormente y seleccione **Inicio de sesión único**.

1. En la página **Configuración del inicio de sesión único con SAML**, edite la sección **Configuración básica de SAML**.

1. En la sección **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** , agregue la dirección URL (por ejemplo, `https://otherwebapp.contoso.local/`) de todas las aplicaciones web adicionales que deben iniciar la sesión de los usuarios con Azure Active Directory y haga clic en **Guardar**.

![Especificación de aplicaciones web adicionales](./media/sharepoint-on-premises-tutorial/azure-active-directory-app-reply-urls.png)
