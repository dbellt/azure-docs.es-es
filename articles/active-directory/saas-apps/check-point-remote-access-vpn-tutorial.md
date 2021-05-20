---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Remote Access VPN | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Check Point Remote Access VPN.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2021
ms.author: jeedes
ms.openlocfilehash: 8c619398b357833aa5b5d996a255fb1df7bd3bb6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776188"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-check-point-remote-access-vpn"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Remote Access VPN

En este tutorial, aprenderá a integrar Check Point Remote Access VPN con Azure Active Directory (Azure AD). Al integrar Check Point Remote Access VPN con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Check Point Remote Access VPN.
* Permitir que los usuarios inicien sesión automáticamente en Check Point Remote Access VPN con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) de Check Point Remote Access VPN.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Check Point Remote Access VPN admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-check-point-remote-access-vpn-from-the-gallery"></a>Adición de Check Point Remote Access VPN desde la galería

Para configurar la integración de Check Point Remote Access VPN en Azure AD, es preciso agregar dicha VPN desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Check Point Remote Access VPN** en el cuadro de búsqueda.
1. Seleccione **Check Point Remote Access VPN** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-check-point-remote-access-vpn"></a>Configuración y prueba del inicio de sesión único de Azure AD para Check Point Remote Access VPN

Configure y pruebe el inicio de sesión único de Azure AD con Check Point Remote Access VPN con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Check Point Remote Access VPN.

Para configurar y probar el inicio de sesión único de Azure AD con Check Point Remote Access VPN, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración de Check Point Remote Access VPN](#configure-check-point-remote-access-vpn-sso)** , para permitir que los usuarios usen esta característica.

    1. **[Creación de un usuario de prueba de Check Point Remote Access VPN](#create-check-point-remote-access-vpn-test-user)** , para tener un homólogo de B.Simon en Check Point Remote Access VPN que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Check Point Remote Access VPN**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/saml-vpn/spPortal/ACS/ID/<IDENTIFIER_UID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/saml-vpn/spPortal/ACS/Login/<IDENTIFIER_UID>`

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/saml-vpn/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y las direcciones URL de inicio de sesión y de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Check Point Remote Access VPN](mailto:support@checkpoint.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Check Point Remote Access VPN**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Check Point Remote Access VPN mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Check Point Remote Access VPN**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-check-point-remote-access-vpn-sso"></a>Configuración del inicio de sesión único de Check Point Remote Access VPN

### <a name="configure-an-external-user-profile-object"></a>Configuración de un objeto de perfil de usuario externo

> [!NOTE]
> Esta sección solo es necesaria si no quiere usar una instancia de Active Directory local (LDAP).

**Configure un perfil de usuario genérico en SmartDashboard heredado**:

1. En SmartConsole, vaya a **Manage & Settings > Blades** (Administrar y configuración > Hojas).

1. En la sección **Mobile Access** (Acceso móvil), haga clic en **Configure in SmartDashboard** (Configurar en SmartDashboard). Se abre la instancia de SmartDashboard heredada.

1. En el panel **Objects** (Objetos) de red, haga clic en **Users** (Usuarios).

1. Haga clic con el botón derecho en un espacio vacío y seleccione **New > External User Profile > Match all users** (Nuevo > Perfil de usuario externo > Coincidir con todos los usuarios).

1. Configure las propiedades del **perfil de usuario externo**:

    1. En la página **General Properties** (Propiedades generales).
        * En el campo del nombre del **perfil de usuario externo**, deje el nombre predeterminado `generic`*.
        * En el campo **Expiration Date** (Fecha de expiración), establezca la fecha aplicable.

    1. En la página **Autenticación**:
        * En la lista desplegable **Authentication Scheme** (Esquema de autenticación), seleccione `undefined`.
    1. En las páginas **Location**, **Time** y **Encryption** (Ubicación, Hora y Cifrado):
        * Configuración de otras opciones aplicables
    1. Haga clic en **OK**.

1. En la barra de herramientas superior, haga clic en **Update** (Actualizar) (o presione Ctrl + S).

1. Cierre SmartDashboard.

1. En SmartConsole, instale la directiva de control de acceso.

### <a name="configure-remote-access-vpn"></a>Configuración de Remote Access VPN

1. Abra el objeto de la puerta de enlace de seguridad aplicable.

1. En la página de propiedades generales, habilite la hoja de software **IPSec VPN** (VPN con IPSec).

1. En el árbol izquierdo, haga clic en la página **IPSec VPN** (VPN con IPSec).

1. En la sección **This Security Gateway participates in the following VPN communities** (Esta puerta de enlace de seguridad participa en las siguientes comunidades de VPN), haga clic en **Add** (Agregar) y seleccione **Remote Access Community** (Comunidad de acceso remoto).

1. En el árbol izquierdo, haga clic en **VPN clients > Remote Access** (Clientes VPN > Acceso remoto).

1. Habilite el **modo de visitante de soporte técnico**.

1. En el árbol izquierdo, haga clic en **VPN clients > Office Mode** (Clientes VPN > Modo de oficina).

1. Seleccione **Allow Office Mode** (Permitir modo de oficina) y seleccione el método de modo de oficina correspondiente.

1. En el árbol izquierdo, haga clic en **VPN clients > SAML Portal Settings** (Clientes VPN > Configuración del portal de SAML).

1. Asegúrese de que la dirección URL principal contenga el nombre de dominio completo de la puerta de enlace.
Este nombre de dominio debe terminar con un sufijo DNS registrado por la organización.
Por ejemplo: `https://gateway1.company.com/saml-vpn`

1. Asegúrese de que el explorador de los usuarios finales confíe en el certificado.

1. Haga clic en **OK**.


### <a name="configure-an-identity-provider-object"></a>Configuración de un objeto de proveedor de identidades

1. Realice los pasos siguientes para cada puerta de enlace de seguridad que participa en Remote Access VPN.

1. En la vista **Gateways & Servers** (Puertas de enlace y servidores) de SmartConsole, haga clic en **New > More > User/Identity > Identity Provider** (Nuevo > Más > Usuario/Identidad > Proveedor de identidades).

    ![captura de pantalla del nuevo proveedor de identidades.](./media/check-point-remote-access-vpn-tutorial/identity-provider.png)

1. Realice los pasos siguientes en la ventana **New Identity Provider** (Nuevo proveedor de identidades).

    ![captura de pantalla de la sección Identity Provider (Proveedor de identidades).](./media/check-point-remote-access-vpn-tutorial/new-identity-provider.png)

    a. En el **Gateway** (Puerta de enlace), seleccione la puerta de enlace de seguridad, que debe realizar la autenticación SAML.

    b. En el campo **Service** (Servicio), seleccione **Remote Access VPN** en el menú desplegable.

    c. Copie el valor del **identificador de entidad** y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    d. Copie el valor de la **dirección URL de respuesta** y péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    e. Seleccione **Importar archivo de metadatos** para cargar el archivo **XML de metadatos de federación** descargado desde Azure Portal.

    > [!NOTE]
    > También puede seleccionar **Insert Manually** (Insertar manualmente) para pegar manualmente los valores de **Id. de entidad** y **Dirección URL de inicio de sesión** en los campos correspondientes y cargar el **archivo de certificado** desde Azure Portal.

    f. Haga clic en **OK**.

### <a name="configure-the-identity-provider-as-an-authentication-method"></a>Configuración del proveedor de identidades como método de autenticación

1. Abra el objeto de la puerta de enlace de seguridad aplicable.

1. En la página **VPN Clients > Authentication** (Clientes VPN > Autenticación):

    a. Desactive la casilla **Allow older clients to connect to this gateway** (Permitir que los clientes más antiguos se conecten a esta puerta de enlace).

    b. Agregue un nuevo objeto o edite un dominio kerberos existente.

    ![Captura de pantalla para agregar un nuevo objeto](./media/check-point-remote-access-vpn-tutorial/add-new-object.png)

1. Escriba un nombre y un nombre para mostrar y agregue o edite un método de autenticación. En caso de que la opción de inicio de sesión se use en las GW que participan en MEP, para permitir una experiencia fluida del usuario, el nombre debe comenzar con el prefijo "SAMLVPN_". 

    ![Captura de pantalla sobre la opción de inicio de sesión](./media/check-point-remote-access-vpn-tutorial/login-option.png)

1. Seleccione la opción **Proveedor de identidades**, haga clic en el botón verde `+` y seleccione el objeto Proveedor de identidades aplicable.

    ![Captura de pantalla para seleccionar el objeto Proveedor de identidades aplicable](./media/check-point-remote-access-vpn-tutorial/green-button.png)

1. En la ventana Multiple Logon Options (Opciones de inicio de sesión múltiples), en el panel izquierdo, haga clic en **User Directories** (Directorios de usuario) y, a continuación, seleccione **Manual Configuration** (Configuración manual).
Hay dos opciones:
    1. Si no desea usar un perfil de Active Directory local (LDAP), seleccione solo los perfiles de usuarios externos y haga clic en OK (Aceptar).
    2. Si desea usar una instancia de Active Directory local (LDAP), seleccione solo usuarios LDAP y, en tipo de búsqueda LDAP, seleccione correo electrónico. A continuación, haga clic en Aceptar.

    ![Captura de pantalla de la configuración manual](./media/check-point-remote-access-vpn-tutorial/manual-configuration.png)

1. Configure los valores necesarios en la base de datos de administración:

    1.  Cierre SmartConsole.

    2.  Conéctese con la herramienta GuiDBEdit al servidor de administración (consulte [sk13009](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails&solutionid=sk13009)).

    3.  En el panel superior izquierdo, vaya a **Edit > Network Objects** (Editar > Objetos de red).

    4.  En el panel superior derecho, seleccione el objeto **Security Gateway** (Puerta de enlace de seguridad).

    5.  En el panel inferior, vaya a **realms_for_blades > vpn**.

    6.  Si no desea usar una instancia de Active Directory local (LDAP), establezca **do_ldap_fetch** en **false** y **do_generic_fetch** en **true**. A continuación, haga clic en **Aceptar**. Si desea usar una instancia de Active Directory local (LDAP), establezca do_ **ldap_fetch** en **true** y **do_generic_fetch** en **false**. A continuación, haga clic en **Aceptar**.

    7.  Repita los pasos iv-vi para todas las puertas de enlace de seguridad aplicables.

    8.  Guarde todos los cambios (haga clic en el menú **File** > **Save All** [Archivo > Guardar todo]).

1. Cierre la herramienta GuiDBEdit.

1. Cada puerta de enlace de seguridad y cada hoja de software tienen una configuración independiente. Revise la configuración de cada puerta de enlace de seguridad y de cada hoja de software que usan la autenticación (VPN, acceso móvil y reconocimiento de identidad).

    * Asegúrese de seleccionar la opción de **usuarios de LDAP**  para las hojas de software que utilizan LDAP.

    * Asegúrese de seleccionar la opción **External user profiles** (Perfiles de usuarios externos) solo para las hojas de software que no usan LDAP.

1. Instale la directiva de control de acceso en cada puerta de enlace de seguridad.

### <a name="vpn-ra-client-installation-and-configuration"></a>Instalación y configuración del cliente de VPN-RA

1. Instale el cliente de VPN.

1. Establezca el modo de explorador del proveedor de identidades (opcional). De forma predeterminada, el cliente de Windows usa su explorador insertado y el cliente macOS usa Safari para autenticarse en el portal del proveedor de identidades.
Para que el cliente de Windows cambie este comportamiento y use Internet Explorer en su lugar:

    1.  En la máquina cliente, abra un editor de texto sin formato como administrador.
    2.  Abra el archivo trac.defaults en el editor de texto.
        * En Windows de 32 bits: ``%ProgramFiles%\CheckPoint\Endpoint Connect\trac.defaults``
        * En Windows de 64 bits: ``%ProgramFiles(x86)%\CheckPoint\Endpoint Connect\trac.defaults``
    3.  Cambie el atributo idp_browser_mode de "embedded" a "IE":
    4.  Guarde el archivo.
    5.  Reinicie el servicio cliente de Check Point Endpoint Security VPN.
Abra el símbolo del sistema de Windows como administrador y ejecute estos comandos:

        `# net stop TracSrvWrapper `

        `# net start TracSrvWrapper`
 

1. Inicie la autenticación con el explorador ejecutándose en segundo plano:

    1.  En la máquina cliente, abra un editor de texto sin formato como administrador.
    2.  Abra el archivo trac.defaults en el editor de texto.
        * En Windows de 32 bits: `%ProgramFiles%\CheckPoint\Endpoint Connect\trac.defaults`
        * En Windows de 64 bits: `%ProgramFiles(x86)%\CheckPoint\Endpoint Connect\trac.defaults`

        * En macOS: `/Library/Application Support/Checkpoint/Endpoint Security/Endpoint Connect/Trac.defaults`

    3.  Cambie el valor de **idp_show_browser_primary_auth_flow** a **false**.
    4.  Guarde el archivo.
    5.  Reinicie el servicio cliente de Check Point Endpoint Security VPN.
        * En los clientes de Windows, abra el símbolo del sistema como administrador y ejecute estos comandos:

            `# net stop TracSrvWrapper`
        
            `# net start TracSrvWrapper`

        * En los clientes macOS

            `sudo launchctl stop com.checkpoint.epc.service`

            `sudo launchctl start com.checkpoint.epc.service`


### <a name="create-check-point-remote-access-vpn-test-user"></a>Creación de un usuario de prueba de Check Point Remote Access VPN

En esta sección, creará un usuario llamado Britta Simon en Check Point Remote Access VPN. Trabaje con el [equipo de soporte técnico de Check Point Remote Access VPN](mailto:support@checkpoint.com) para agregar los usuarios a la plataforma Check Point Remote Access VPN. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

1. Abra el cliente VPN y haga clic en **Connect to…** (Conectar a...)

    ![Captura de pantalla de la conexión](./media/check-point-remote-access-vpn-tutorial/connect.png)

1. Seleccione **Site** (Sitio) la lista desplegable y haga clic en **Connect** (Conectar).

    ![Captura de pantalla para seleccionar el sitio](./media/check-point-remote-access-vpn-tutorial/site.png)

1. En el menú emergente de inicio de sesión de Azure AD, inicie sesión con las credenciales que ha creado en la sección **Creación de un usuario de prueba de Azure AD**.

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Check Point Remote Access VPN, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


