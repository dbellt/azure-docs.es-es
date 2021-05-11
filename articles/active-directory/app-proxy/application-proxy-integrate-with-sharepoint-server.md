---
title: 'Habilitación del acceso remoto a SharePoint: Azure Active Directory Application Proxy'
description: En este documento se explican los conceptos básicos sobre cómo integrar un servidor de SharePoint en el entorno local con Azure Active Directory Application Proxy.
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
ms.openlocfilehash: 3bbe0480fcf6e4e1cf828cc61c5778fdd096ed8b
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186886"
---
# <a name="enable-remote-access-to-sharepoint-with-azure-active-directory-application-proxy"></a>Habilitación del acceso remoto a SharePoint con Azure Active Directory Application Proxy

En esta guía paso a paso se explica cómo integrar una granja de servidores de SharePoint local con Azure Active Directory (Azure AD) Application Proxy.

## <a name="prerequisites"></a>Requisitos previos

Para realizar la configuración, necesita los siguientes recursos:
- Una granja de servidores de SharePoint 2013, o posterior.
- Un inquilino de Azure AD con un plan que incluya el proxy de aplicación. Obtenga más información acerca de los [planes y precios de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).
- Un [dominio comprobado personalizado ](../fundamentals/add-custom-domain.md) en el inquilino de Azure AD.
- Un directorio local de Active Directory sincronizado con Azure AD Connect con el que los usuarios pueden [iniciar sesión en Azure](../hybrid/plan-connect-user-signin.md).
- Un conector del proxy de aplicación instalado y en ejecución en una máquina que se encuentre dentro del dominio corporativo.

La configuración de SharePoint con el proxy de aplicación requiere dos direcciones URL:
- Una dirección URL externa, visible para los usuarios finales y determinada en Azure AD. Esta dirección URL puede utilizar un dominio personalizado. Más información acerca del [uso de dominios personalizados en Azure AD Application Proxy](application-proxy-configure-custom-domain.md).
- Una dirección URL interna, que solo se conoce dentro del dominio corporativo y nunca se usa directamente.

> [!IMPORTANT]
> Para garantizar que los vínculos se asignan correctamente, siga estas recomendaciones para la dirección URL interna:
> - Utilice HTTPS.
> - No utilice puertos personalizados.
> - En el sistema de nombres de dominio (DNS) corporativo, cree un host (A), que apunte a SharePoint WFE (o al equilibrador de carga), no a un alias (CName).

En este artículo se usan los siguientes valores:
- Dirección URL interna: `https://sharepoint`
- Dirección URL externa: `https://spsites-demo1984.msappproxy.net/`
- Cuenta del grupo de aplicaciones para la aplicación web de SharePoint: `Contoso\spapppool`

## <a name="step-1-configure-an-application-in-azure-ad-that-uses-application-proxy"></a>Paso 1: Configuración de una aplicación en Azure AD que usa Application Proxy

En este paso, creará en su inquilino de Azure Active Directory una aplicación que usa el proxy de aplicación. Establezca la dirección URL externa y especifique la dirección URL interna. Ambas direcciones se usarán más adelante en SharePoint.

1. Cree la aplicación como se describe en la siguiente configuración. Para obtener instrucciones detalladas, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](../app-proxy/application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).
   * **Dirección URL interna**: la dirección URL interna de SharePoint que se establecerá más adelante en SharePoint, como `https://sharepoint`.
   * **Autenticación previa**: Azure Active Directory.
   * **Traducción de direcciones URL en encabezados**: No.
   * **Traducción de direcciones URL en el cuerpo de la aplicación**: No.

   ![Publicación de SharePoint como aplicación](./media/application-proxy-integrate-with-sharepoint-server/publish-app.png)

1. Después de publicar la aplicación, siga los pasos a continuación para configurar las opciones de inicio de sesión único:

   1. En la página de la aplicación en el portal, seleccione **Inicio de sesión único**.
   1. En **Modo de inicio de sesión único**, seleccione **Autenticación de Windows integrada**.
   1. Establezca el **SPN de la aplicación interno** en el valor que estableció antes. Para este ejemplo, el valor es `HTTP/sharepoint`.
   1. En **Identidad de inicio de sesión delegada**, seleccione la opción más adecuada para la configuración del bosque de Active Directory. Por ejemplo, si tiene un solo dominio de Active Directory en el bosque, seleccione **Nombre de cuenta SAM local** (como se muestra en la captura de pantalla siguiente). Sin embargo, si los usuarios no están en el mismo dominio que SharePoint y los servidores de conector de Application Proxy, seleccione **Nombre principal de usuario local** (no se muestra en la captura de pantalla).

   ![Configuración de la autenticación de Windows integrada para SSO](./media/application-proxy-integrate-with-sharepoint-server/configure-iwa.png)

1. Para finalizar la configuración de la aplicación, vaya a la sección **Usuarios y grupos** y asigne usuarios para que accedan a esta aplicación. 

## <a name="step-2-configure-the-sharepoint-web-application"></a>Paso 2: Configuración de la aplicación web de SharePoint

La aplicación web de SharePoint debe configurarse con Kerberos y las asignaciones de acceso alternativas apropiadas para funcionar correctamente con Azure Active Directory Application Proxy. Hay dos posibles opciones:

- Cree una nueva aplicación web y use solo la zona predeterminada. Esta es la opción preferida, ya que ofrece la mejor experiencia con SharePoint (por ejemplo, los vínculos de las alertas por correo electrónico que genera SharePoint siempre apuntan a la zona predeterminada).
- Extienda una aplicación web existente para configurar Kerberos en una zona no predeterminada.

> [!IMPORTANT]
> Independientemente de la zona que se use, para que Kerberos funcione correctamente la cuenta del grupo de aplicaciones de la aplicación web de SharePoint debe ser una cuenta de dominio.

### <a name="provision-the-sharepoint-web-application"></a>Aprovisionamiento de la aplicación web de SharePoint

- Si crea una aplicación web y usa solo la zona predeterminada (la opción preferida):

    1. Inicie el **shell de administración de SharePoint** y ejecute el siguiente script:

       ```powershell
       # This script creates a web application and configures the Default zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment. Note that the managed account must exist and it must be a domain account
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       $applicationPoolManagedAccount = "Contoso\spapppool"
            
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = New-SPWebApplication -Name "SharePoint - AAD Proxy" -Port 443 -SecureSocketsLayer -URL $externalUrl -ApplicationPool "SharePoint - AAD Proxy" -ApplicationPoolAccount (Get-SPManagedAccount $applicationPoolManagedAccount) -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Default -Internal
       ```

    2. Abra el sitio **Administración central de SharePoint**.
    1. En **Configuración del sistema**, seleccione **Configurar asignaciones de acceso alternativas**. Se abre el cuadro **Colección de asignaciones de acceso alternativas**.
    1. Filtre la visualización con la nueva aplicación web y confirme que se ve algo parecido a esto:

       ![Asignaciones de acceso alternativas de la aplicación web](./media/application-proxy-integrate-with-sharepoint-server/new-webapp-aam.png)

- Si extiende una aplicación web existente a una nueva zona (en caso de que no pueda usar la zona predeterminada):

    1. Inicie el shell de administración de SharePoint y ejecute el siguiente script:

       ```powershell
       # This script extends an existing web application to Internet zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment
       $webAppUrl = "http://spsites/"
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = Get-SPWebApplication $webAppUrl
       New-SPWebApplicationExtension -Name "SharePoint - AAD Proxy" -Identity $wa -SecureSocketsLayer -Zone Extranet -Url $externalUrl -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Extranet -Internal
       ```

    2. Abra el sitio **Administración central de SharePoint**.
    1. En **Configuración del sistema**, seleccione **Configurar asignaciones de acceso alternativas**. Se abre el cuadro **Colección de asignaciones de acceso alternativas**.
    1. Filtre la visualización con la aplicación web que se ha extendido y confirme que se muestra algo parecido a esto:

        ![Asignaciones de acceso alternativas de la aplicación extendida](./media/application-proxy-integrate-with-sharepoint-server/extend-webapp-aam.png)

### <a name="make-sure-the-sharepoint-web-application-is-running-under-a-domain-account"></a>Asegúrese de que la aplicación web de SharePoint se ejecuta en una cuenta de dominio

Para identificar la cuenta que ejecuta el grupo de aplicaciones de la aplicación web de SharePoint y asegurarse de que es una cuenta de dominio, siga estos pasos:

1. Abra el sitio **Administración central de SharePoint**.
1. Vaya a **Seguridad** y seleccione **Configurar cuentas de servicio**.
1. Seleccione **Grupo de aplicaciones web - YourWebApplicationName**.

   ![Opciones para configurar una cuenta de servicio](./media/application-proxy-integrate-with-sharepoint-server/service-web-application.png)

1. Confirme que **Seleccione una cuenta para este componente** devuelve una cuenta de dominio y recuérdela, ya que se necesitará en el paso siguiente.

### <a name="make-sure-that-an-https-certificate-is-configured-for-the-iis-site-of-the-extranet-zone"></a>Asegúrese de que hay un certificado HTTPS configurado para el sitio IIS de la zona Extranet.

Dado que la dirección URL interna usa el protocolo HTTPS (`https://SharePoint/`), debe establecerse un certificado en el sitio de Internet Information Services (IIS).

1. Abra la consola Windows PowerShell.
1. Ejecute el siguiente script para generar un certificado autofirmado y agregarlo al almacén My del equipo:

   ```powershell
   # Replace "SharePoint" with the actual hostname of the Internal URL of your Azure AD proxy application
   New-SelfSignedCertificate -DnsName "SharePoint" -CertStoreLocation "cert:\LocalMachine\My"
   ```

   > [!IMPORTANT]
   > Los certificados autofirmados son adecuados solo para pruebas. En los entornos de producción, se recomienda encarecidamente que use certificados emitidos por una entidad de certificación.

1. Abra la consola del administrador de Internet Information Services.
1. Expanda el servidor en la vista de árbol, expanda **Sitios**, seleccione el sitio **SharePoint – Proxy de AAD** y haga clic en **Enlaces**.
1. Seleccione el **enlace HTTPS** y después haga clic en **Editar**.
1. En el campo Certificado TLS/SSL, elija el certificado de **SharePoint** y después seleccione **Aceptar**.

Ahora puede tener acceso al sitio de SharePoint de forma externa mediante Azure AD Application Proxy.

## <a name="step-3-configure-kerberos-constrained-delegation"></a>Paso 3: Configuración de la delegación restringida de Kerberos

Inicialmente, los usuarios se autenticarán en Azure AD y, a continuación, en SharePoint mediante Kerberos a través del conector del proxy de Azure AD. Para permitir que el conector obtenga un token de Kerberos en nombre del usuario de Azure AD, debe configurar la delegación restringida de Kerberos (KCD) con la transición de protocolo. Para obtener más información acerca de KCD, consulte [Introducción a la delegación limitada de Kerberos](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)).

### <a name="set-the-spn-for-the-sharepoint-service-account"></a>Configuración de un SPN para la cuenta de servicio de SharePoint

En este artículo, la dirección URL interna es `https://sharepoint`, por lo que el nombre de entidad de servicio (SPN) es `HTTP/sharepoint`. Debe reemplazar esos valores por los que corresponden a su entorno.
Para registrar el SPN `HTTP/sharepoint` de la cuenta del grupo de aplicaciones de SharePoint `Contoso\spapppool`, ejecute el siguiente comando desde un símbolo del sistema como administrador del dominio:

`setspn -S HTTP/sharepoint Contoso\spapppool`

El comando `Setspn` busca el SPN antes de agregarlo. Si ya existe, se muestra el error **Valor de SPN duplicado**. En ese caso, considere la posibilidad de quitar el SPN existente si no está establecido en la cuenta del grupo de aplicaciones correcto. Puede comprobar que el SPN se agregó correctamente mediante la ejecución del comando `Setspn` con la opción -L. Para más información sobre este comando, vea [Setspn](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc731241(v=ws.11)).

### <a name="make-sure-the-connector-is-trusted-for-delegation-to-the-spn-that-was-added-to-the-sharepoint-application-pool-account"></a>Asegúrese de que el conector es de confianza para delegación en el SPN que se agregó a la cuenta del grupo de aplicaciones de SharePoint.

Configure KCD para que el servicio de proxy de aplicación de Azure AD pueda delegar las identidades de usuario en el servicio de SharePoint. Para configurar KCD, habilite el conector del proxy de aplicación para recuperar los vales Kerberos para los usuarios que se autenticaron en Azure AD. A continuación, ese servidor pasa el contexto a la aplicación de destino (SharePoint en este caso).

Para configurar KCD, siga estos pasos para cada equipo de conexión:

1. Inicie sesión como administrador de dominio en un controlador de dominio y, después, abra Usuarios y equipos de Active Directory.
1. Busque el equipo que ejecuta el conector del proxy de Azure AD. En este ejemplo, se trata del propio servidor de SharePoint.
1. Haga doble clic en el equipo y, después, seleccione la pestaña **Delegación**.
1. Asegúrese de que las opciones de delegación estén establecidas en **Confiar en este equipo para la delegación solo a los servicios especificados**. Después, seleccione **Usar cualquier protocolo de autenticación**.
1. Seleccione el botón **Agregar**, seleccione **Usuarios o equipos** y busque la cuenta del grupo de aplicaciones de SharePoint. Por ejemplo: `Contoso\spapppool`.
1. En la lista de los SPN, seleccione el que creó anteriormente para la cuenta de servicio.
1. Seleccione **Aceptar** y, después, otra vez **Aceptar** para guardar sus cambios.
  
   ![Configuración de delegación](./media/application-proxy-integrate-with-sharepoint-server/delegation-box2.png)

Ya está listo para iniciar sesión en SharePoint mediante la dirección URL externa y para autenticarse en Azure.

## <a name="troubleshoot-sign-in-errors"></a>Solución de errores de inicio de sesión

Si el inicio de sesión en el sitio no funciona, puede obtener más información sobre el problema en los registros del conector: desde la máquina que ejecuta el conector, abra el visor de eventos, vaya a **Registros de aplicaciones y servicios** > **Microsoft** > **AadApplicationProxy** > **Conector** e inspeccione el registro **Admin**.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de dominios personalizados en el proxy de la aplicación de Azure AD](application-proxy-configure-custom-domain.md)
* [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)