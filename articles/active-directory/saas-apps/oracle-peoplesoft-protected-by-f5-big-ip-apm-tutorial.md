---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Oracle PeopleSoft - Protected by F5 BIG-IP APM | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Oracle PeopleSoft - Protected by F5 BIG-IP APM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2021
ms.author: jeedes
ms.openlocfilehash: a45b4d1c006762cdad7f818c22f59b9e18436b95
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146156"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Oracle PeopleSoft - Protected by F5 BIG-IP APM

En este tutorial, aprenderá a integrar Oracle PeopleSoft - Protected by F5 BIG-IP APM con Azure Active Directory (Azure AD). Al integrar Oracle PeopleSoft - Protected by F5 BIG-IP APM con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Oracle PeopleSoft - Protected by F5 BIG-IP APM.
* Permitir que los usuarios inicien sesión automáticamente en Oracle PeopleSoft - Protected by F5 BIG-IP APM con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).
* En este tutorial se incluyen las instrucciones para Oracle PeopleSoft ELM.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

1. Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
1. Una suscripción habilitada para el inicio de sesión único (SSO) en Oracle PeopleSoft - Protected by F5 BIG-IP APM.

1. La implementación de la solución conjunta requiere la licencia siguiente:

    1. F5 BIG-IP® Best bundle (o) 
    2. Licencia independiente de F5 BIG-IP Access Policy Manager™ (APM). 
    3. Licencia del complemento F5 BIG-IP Access Policy Manager™ (APM) en una instalación de F5 BIG-IP® Local Traffic Manager™ (LTM) ya existente.
    4. Además de la licencia anterior, el sistema F5 también puede tener licencia para: 
        * Una suscripción de filtrado de direcciones URL para usar la base de datos de categorías de URL. 
        * Una suscripción a F5 IP Intelligence para detectar y bloquear a los atacantes conocidos y el tráfico malintencionado. 
        * Un módulo de seguridad de hardware (HSM) de red para proteger y administrar las claves digitales para la autenticación segura.
1. El sistema F5 BIG-IP se aprovisiona con módulos de APM (LTM es opcional).
1. Aunque es opcional, es muy recomendable implementar los sistemas F5 en un [grupo de dispositivos de sincronización/conmutación por error](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/big-ip-device-service-clustering-administration-14-1-0.html) (S/F DG), que incluye el par activo en espera, con una dirección IP flotante para la alta disponibilidad (HA). Se puede lograr una mayor redundancia de la interfaz mediante el protocolo de control de agregación de vínculos (LACP). LACP administra las interfaces físicas conectadas como una sola interfaz virtual (grupo agregado) y detecta cualquier error de interfaz dentro del grupo.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Oracle PeopleSoft - Protected by F5 BIG-IP APM admite el inicio de sesión único comenzado por **SP e IDP**.

## <a name="add-oracle-peoplesoft---protected-by-f5-big-ip-apm-from-the-gallery"></a>Incorporación de Oracle PeopleSoft - Protected by F5 BIG-IP APM desde la galería

Para configurar la integración de Oracle PeopleSoft - Protected by F5 BIG-IP APM en Azure AD, es preciso agregar Oracle PeopleSoft - Protected by F5 BIG-IP APM desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Oracle PeopleSoft - Protected by F5 BIG-IP APM** en el cuadro de búsqueda.
1. Seleccione **Oracle PeopleSoft - Protected by F5 BIG-IP APM** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Configuración y prueba del SSO de Azure AD para Oracle PeopleSoft - Protected by F5 BIG-IP APM

Configure y pruebe el inicio de sesión único de Azure AD con Oracle PeopleSoft - Protected by F5 BIG-IP APM mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Oracle PeopleSoft - Protected by F5 BIG-IP APM.

Para configurar y probar el inicio de sesión único de Azure AD con Oracle PeopleSoft - Protected by F5 BIG-IP APM, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Oracle PeopleSoft - Protected by F5 BIG-IP APM](#configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Oracle PeopleSoft - Protected by F5 BIG-IP APM](#create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user)** , para tener un homólogo de B.Simon en Oracle PeopleSoft - Protected by F5 BIG-IP APM que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Oracle PeopleSoft - Protected by F5 BIG-IP APM**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<FQDN>.peoplesoft.f5.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/post/acs`

    c. En el cuadro de texto **URL de cierre de sesión**, escriba una dirección URL con el siguiente patrón: `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/redirect/slr`.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<FQDN>.peoplesoft.f5.com/`

    > [!NOTE]
    >Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador, la dirección URL de respuesta y la dirección URL de cierre de sesión reales. Póngase en contacto con el [equipo de soporte técnico de Oracle PeopleSoft - Protected by F5 BIG-IP APM](https://support.f5.com) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Oracle PeopleSoft - Protected by F5 BIG-IP APM espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Oracle PeopleSoft - Protected by F5 BIG-IP APM espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | ------------------ | --------- |
    | EMPLID | user.employeeid |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, descargue el archivo **XML de metadatos de federación** y el **Certificado (Base64)** y guárdelos en su equipo.

    ![Vínculo de descarga del certificado](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/both-certificate.png)

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

En esta sección, va a permitir que B.Simon acceda a Oracle PeopleSoft - Protected by F5 BIG-IP APM mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Oracle PeopleSoft - Protected by F5 BIG-IP APM**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso"></a>Configuración del inicio de sesión único en Oracle PeopleSoft - Protected by F5 BIG-IP APM

### <a name="f5-saml-sp-configuration"></a>Configuración de SP de SAML en F5

Importe el certificado de metadatos en F5; se usará más adelante en el proceso de instalación. Vaya a **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados SSL). Seleccione **Import** (Importar) en la esquina derecha.

![Configuración de SP de SAML en F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sp-configuration.png)

#### <a name="setup-the-saml-idp-connector"></a>Configuración del conector IDP de SAML 

1. Vaya a **Access > Federation > SAML: Service Provider > External Idp Connectors** (Acceso > Federación > SAML: Proveedor de servicios > Conectores de IDP externos) y haga clic en **Create > From Metadata** (Crear > Desde metadatos).

    ![Conector de IDP de SAML de F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/saml-idp-connector.png)

1. En la página siguiente, haga clic en **Browse** (Examinar) para cargar el archivo XML.

1. En el cuadro **Identity Provider Name** (Nombre del proveedor de identidad), proporcione un nombre válido y, a continuación, haga clic en **OK** (Aceptar).

    ![Nuevo conector de IDP de SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-idp.png)

1. Realice los pasos necesarios en la pestaña **Security Settings** (Configuración de seguridad) y, a continuación, haga clic en **OK** (Aceptar).

    ![Editar el conector de IDP de SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp.png)

#### <a name="setup-the-saml-sp"></a>Configuración del SP de SAML

1. Vaya a **Access > Federation > SAML Service Provider > Local SP Services** (Acceso > Federación > Proveedor de servicios de SAML > Servicios del SP local) y haga clic en **Create** (Crear). Complete la siguiente información y haga clic en **OK** (Aceptar).

    * Nombre: `<Name>`
    * Entity ID (Identificador de entidad): `https://<FQDN>.peoplesoft.f5.com`
    * Configuración del nombre del SP
        * Scheme (Esquema): `https`
        * Host: `<FQDN>.peoplesoft.f5.com`
        * Description (Descripción): `<Description>`

    ![Nuevos servicios de SP de SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-sp-service.png)

1. Seleccione la configuración de SP, PeopleSoftAppSSO, y haga clic en **Bind/UnBind IdP Connectors** (Enlazar/Desenlazar conectores de IDP).
Haga clic en **Add New Row** (Agregar nueva fila) y seleccione el **Conector del IdP externo** creado en el paso anterior, haga clic en **Update**(Actualizar) y, a continuación, haga clic en **OK** (Aceptar).

    ![Crear servicios de SP de SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp-use-sp.png)

## <a name="configuring-application"></a>Configuración de la aplicación

### <a name="create-a-new-pool"></a>Creación de un nuevo grupo
1. Vaya a **Local Traffic > Pools > Pool List** (Tráfico local > Grupos > Lista de grupos), haga clic en **Create** (Crear), complete la siguiente información y, a continuación, haga clic en **Finished** (Finalizado).

    * Nombre: `<Name>`
    * Description (Descripción): `<Description>`
    * Health Monitors (Monitores de estado): `http`
    * Address (Dirección): `<Address>`
    * Service Port (Puerto de servicio): `<Service Port>`

    ![Creación de nuevo grupo](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/create-pool.png)


### <a name="create-a-new-client-ssl-profile"></a>Creación de un nuevo perfil SSL de cliente

Vaya a **Local Traffic > Profiles > SSL > Client > +** (Tráfico local > Perfiles > SSL > Cliente > +), complete la siguiente información y, a continuación, haga clic en **Finished** (Finalizado).

* Nombre: `<Name>`
* Certificate (Certificado): `<Certificate>`
* Clave: `<Key>`

![Nuevo perfil SSL de cliente](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/client-ssl-profile.png)

### <a name="create-a-new-virtual-server"></a>Creación de un nuevo servidor virtual

1. Vaya a **Local Traffic > Virtual Servers > Virtual Server List > +** (Tráfico local > Servidores virtuales > Lista de servidores virtuales > +), complete la siguiente información y haga clic en **Finished** (Finalizado).
    * Nombre: `<Name>`
    * Destination Address/Mask (Dirección de destino/Máscara): `<Address>`
    * Puerto de servicio: Puerto 443 HTTPS
    * HTTP Profile (Client) (Perfil HTTP [cliente]): http

    ![Creación de un nuevo servidor virtual](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-list.png)

1. Rellene los siguientes valores de la página siguiente:

    * SSL Profile (Client) (Perfil SSL [cliente]): `<SSL Profile>`
    * Source Address Translation (Traducción de direcciones de origen): Asignación automática
    * Access Profile (Perfil de acceso): `<Access Profile>`
    * Default Pool (Grupo predeterminado): `<Pool>`


    ![Creación de un nuevo grupo de servidores virtuales de PeopleSoft](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-people-soft.png)

## <a name="setting-up-peoplesoft-application-to-support-f5-big-ip-apm-as-the-single-sign-on-solution"></a>Configuración de la aplicación PeopleSoft para que admita F5 BIG-IP APM como solución de inicio de sesión único

>[!Note]
> Referencia: https://docs.oracle.com/cd/E12530_01/oam.1014/e10356/people.htm

1. Inicie sesión en la consola de PeopleSoft `https://<FQDN>.peoplesoft.f5.com/:8000/psp/ps/?cmd=start` con las credenciales de administrador (por ejemplo, PS/PS).

    ![Autoservicio de administrador](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-console.png)

1. En la aplicación PeopleSoft, cree **OAMPSFT** como un nuevo perfil de usuario y asóciele un rol de seguridad baja, como **PeopleSoft User** (Usuario de PeopleSoft).
Vaya a **PeopleTools > Security > User Profiles > User Profiles** (PeopleTools > Seguridad > Perfiles de usuario > Perfiles de usuario) para crear un nuevo perfil de usuario, por ejemplo: **OAMPSFT** y agregue **PeopleSoft User** (Usuario de PeopleSoft).

    ![Usuario de PeopleSoft](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/user-profile.png)

1. Acceda al perfil web y escriba **OAMPSFT** como el identificador de usuario de acceso público.

    ![Perfiles de usuario](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/web-profile-configuration.png)

1. En **PeopleTools Application Designer** (Diseñador de aplicaciones de PeopleTools), abra el registro **FUNCLIB_LDAP**.

    ![Configuración de perfil web](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/funclib.png)

1. Actualice el encabezado de usuario con **PS_SSO_UID** para la función **OAMSSO_AUTHENTICATION**.
En la función **getWWWAuthConfig()** , reemplace el valor asignado a **&defaultUserId** por el valor **OAMPSFT** que definimos en el perfil web. Guarde la definición del registro.

    ![OAMSSO_AUTHENTICATION](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/record.png)

1. Acceda a la página **Signon PeopleCode** (PeopleTools > Security > Security Objects > Signon PeopleCode) (PeopleCode de conexión [PeopleTools > Seguridad > Objetos de seguridad > PeopleCode de conexión]) y habilite la función **OAMSSO_AUTHENTICATION**, el PeopleCode de conexión para el inicio de sesión único de Oracle Access Manager.

    ![OAMSSO_AUTHENTICATION ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-on-people-soft.png)

## <a name="setting-up-f5-big-ip-apm-to-populate-ps_sso_uid-http-header-with-the-peoplesoft-user-id"></a>Configuración de F5 Big-IP APM para rellenar el encabezado HTTP "PS_SSO_UID" con el identificador de usuario de PeopleSoft

### <a name="configuring-per-request-policy"></a>Configuración de la directiva Por solicitud
1. Vaya a **Access > Profile/Policies > Per-Request Policies** (Acceso > Perfil/Directivas > Directivas Por solicitud), haga clic en **Create** (Crear), complete la siguiente información y haga clic en **Finished** (Finalizado).

    * Nombre: `<Name>`
    * Tipo de perfil: All
    * Languages (Idiomas): `<Language>`

    ![Configuración de la directiva Por solicitud ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/per-request.png)

1. Haga clic en **Edit** (Editar) la directiva Por solicitud `<Name>` ![Editar la directiva Por solicitud PeopleSoftSSO ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso.png)

    `Header Name: <Header Name>`   
    `Header Value: <Header Value>`

### <a name="assign-per-request-policy-to-the-virtual-server"></a>Asignación de la directiva Por solicitud al servidor virtual

Vaya a **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp** (Tráfico local > Servidores virtuales > Lista de servidores virtuales > PeopleSoftApp). Especifique en `<Name>` la directiva Por solicitud.

![PeopleSoftSSO como directiva Por solicitud ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso-1.png)

## <a name="setting-up-f5-big-ip-apm-to-support-single-logout-from-peoplesoft-application"></a>Configuración de F5 Big-IP APM para admitir el cierre de sesión único desde la aplicación PeopleSoft

Para agregar compatibilidad con el cierre de sesión único para todos los usuarios de PeopleSoft, siga estos pasos:

1. Determine la dirección URL de cierre de sesión correcta para el portal PeopleSoft.
    * Para determinar la dirección que usa la aplicación PeopleSoft para finalizar una sesión de usuario, debe abrir el portal con cualquier explorador web y habilitar las herramientas de depuración del explorador, tal como se muestra en el ejemplo siguiente:

        ![Dirección URL de cierre de sesión del portal de PeopleSoft ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-out.png)

    * Busque el elemento con el identificador `PT_LOGOUT_MENU` y guarde la ruta de acceso de la dirección URL con los parámetros de consulta. En nuestro ejemplo, se obtiene el siguiente valor: `/psp/ps/?cmd=logout`.

1. Cree una regla iRule de LTM que redirigirá al usuario a la dirección URL de cierre de sesión de APM: `/my.logout.php3`.

    * Vaya a **Local Traffic > iRule** (Tráfico local > iRule), haga clic en **Create** (Crear), complete la siguiente información y haga clic en **Finished** (Finalizado).

        Nombre: `<Name>`  
        Definición:  
                    _when HTTP_REQUEST { switch -glob -- [HTTP::uri] { `/psp/ps/?cmd=logout` { HTTP::redirect `/my.logout.php3` } } }_

1. Asigne el iRule creado al servidor virtual.

    * Vaya a **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp > Resources** (Tráfico local > Servidores virtuales > Lista de servidores virtuales > PeopleSoftApp > Recursos). Haga clic en el botón **Manage...** (Administrar):   

    * Especifique `<Name>` como iRule habilitado y haga clic en **Finished** (Finalizado).

        ![_iRule_PeopleSoftApp ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/irule-people-soft.png)

    * Asigne en el cuadro de texto **Name** (Nombre) el valor `<Name>`. 

        ![_iRule_PeopleSoftApp finalizado](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/common-irule.png)


### <a name="create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user"></a>Creación de un usuario de prueba en Oracle PeopleSoft - Protected by F5 BIG-IP APM

En esta sección, va a crear un usuario llamado B.Simon en Oracle PeopleSoft - Protected by F5 BIG-IP APM. Trabaje con el [equipo de soporte técnico de Oracle PeopleSoft - Protected by F5 BIG-IP APM](https://support.f5.com) para agregar usuarios a la plataforma de Oracle PeopleSoft - Protected by F5 BIG-IP APM. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Oracle PeopleSoft - Protected by F5 BIG-IP APM, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Oracle PeopleSoft - Protected by F5 BIG-IP APM e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Oracle PeopleSoft - Protected by F5 BIG-IP APM para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Oracle PeopleSoft - Protected by F5 BIG-IP APM en Aplicaciones, si está configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión y, si está configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Oracle PeopleSoft - Protected by F5 BIG-IP APM para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Oracle PeopleSoft - Protected by F5 BIG-IP APM, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).