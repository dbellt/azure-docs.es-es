---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Workday | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/06/2021
ms.author: jeedes
ms.openlocfilehash: db6df5f89a380f6a570b4aa2bbb0e317c38b57ac
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108767827"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workday"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Workday

En este tutorial, obtendrá información sobre cómo integrar Workday con Azure Active Directory (Azure AD). Al integrar Workday con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Workday.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Workday con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Workday.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Workday admite el inicio de sesión único iniciado por **SP**.

* Ahora se puede configurar la aplicación Workday Mobile con Azure AD para habilitar el inicio de sesión único. Para más información sobre cómo realizar la configuración, consulte [este vínculo](workday-mobile-tutorial.md).

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-workday-from-the-gallery"></a>Incorporación de Workday desde la galería

Para configurar la integración de Workday en Azure AD, hay que agregar Workday desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Workday** en el cuadro de búsqueda.
1. Seleccione **Workday** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-workday"></a>Configuración y prueba del inicio de sesión único de Azure AD para Workday

Configure y pruebe el inicio de sesión único de Azure AD con Workday mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Workday.

Para configurar y probar el inicio de sesión único de Azure AD con Workday, es preciso completar los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración de Workday](#configure-workday)**, para configurar las opciones del SSO en la aplicación.
    1. **[Creación de un usuario de prueba de Workday](#create-workday-test-user)**: para tener un homólogo de B. Simon en Workday que esté vinculado a la representación del usuario en Azure AD.
3. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Workday**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://impl.workday.com/<tenant>/login-saml2.flex`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://impl.workday.com/<tenant>/login-saml.htmld`

    c. En el cuadro de texto **URL de cierre de sesión**, escriba una dirección URL con el siguiente patrón: `https://impl.workday.com/<tenant>/login-saml.htmld`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la URL de inicio de sesión, la URL de respuesta y la URL de cierre de sesión. La URL de respuesta debe tener un subdominio (por ejemplo, www, wd2, wd3, wd3-impl, wd5 y wd5-impl).
    > Algo como `http://www.myworkday.com` funciona, pero `http://myworkday.com` no. Póngase en contacto con el [equipo de soporte técnico de Workday](https://www.workday.com/en-us/partners-services/services/support.html) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Workday espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**. La aplicación Workday espera que **nameidentifier** se asigne con **user.mail**, **UPN** etc., por lo que debe editar la asignación de atributos haciendo clic en el icono **Editar** y cambiar dicha asignación.

    ![Captura de pantalla que muestra User Attributes (Atributos de usuario) con el icono de edición seleccionado.](common/edit-attribute.png)

    > [!NOTE]
    > Aquí hemos asignado el Identificador de nombre con el UPN (user.userprincipalname) como valor predeterminado. Tiene que asignar el Identificador de nombre con el Identificador de usuario real en su cuenta de Workday (su correo electrónico, UPN etc.) para el correcto funcionamiento del inicio de sesión único.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. Para modificar las opciones de **Firma** según sus propios requisitos, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Certificado](common/edit-certificate.png) 

    ![Certificado de firma SAML](./media/workday-tutorial/signing-option.png)

    a. Seleccione **Opción de firma** como **Firmar respuesta y aserción SAML**.

    b. Haga clic en **Guardar**

1. En la sección **Set up Workday** (Configurar Workday), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a Workday mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Workday**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-workday"></a>Configuración de Workday

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Workday como administrador.

1. En el **cuadro de búsqueda**, busque el nombre **Edit Tenant Setup – Security** (Editar configuración de inquilino: seguridad) en la parte superior izquierda de la página principal.

    ![Editar seguridad del inquilino](./media/workday-tutorial/search-box.png "Editar seguridad del inquilino")


1. En la sección **SAML Setup** (Configuración de SAML), haga clic en **Import Identity Provider** (Importar proveedor de identidades).

    ![Configuración de SAML](./media/workday-tutorial/saml-setup.png "Configuración de SAML")

1. En la sección **Import Identity Provider** (Importar proveedor de identidades), realice los siguientes pasos:

    ![Importación del proveedor de identidades](./media/workday-tutorial/import-identity-provider.png)

    a. En el cuadro de texto **Identity Provider Name** (Nombre del proveedor de identidad), escriba `AzureAD`.

    b. En el cuadro de texto **Used for Environments** (Se usa para entornos), seleccione los nombres de entorno correspondientes en la lista desplegable.

    c. Haga clic en **Select files** (Seleccionar archivos) para cargar el archivo **XML de metadatos de federación** descargado.

    d. Haga clic en **OK** (Aceptar) y, a continuación, en **Done** (Listo).

1. Después de hacer clic en **Done** (Listo), se agregará una nueva fila en la sección **SAML Identity Providers** (Proveedores de identidades de SAML) y, a continuación, puede agregar los pasos siguientes para la fila recién creada.

    ![Proveedores de identidades de SAML.](./media/workday-tutorial/saml-identity-providers.png "Proveedores de identidades SAML")

    a. Haga clic en la casilla **Enable IDP Initiated Logout** (Habilitar el cierre de sesión iniciado por IDP).

    b. En el cuadro de texto **URL de respuesta de cierre de sesión**, escriba **http://www.workday.com** .

    c. Haga clic en la casilla **Enable Workday Initiated Logout** (Habilitar el cierre de sesión iniciado por Workday).

    d. En el cuadro de texto **Logout Request URL** (URL de solicitud de cierre de sesión), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    e. Haga clic en la casilla **Iniciado por SP**.

    f. En el cuadro de texto **Service Provider ID** (Id. de proveedor de servicios), escriba **http://www.workday.com**.

    Seleccione **Do Not Deflate SP-initiated Authentication Request** (No desinflar la solicitud de autenticación iniciada por SP).

1. Realice los pasos descritos en la imagen siguiente.

    ![Workday](./media/workday-tutorial/service-provider.png "Proveedores de identidades SAML")

    a. En el cuadro de texto **Service Provider ID (Will be Deprecated)** [Id. de proveedor de servicio (dejará de usarse)], escriba **http://www.workday.com** .

    b. En el cuadro de texto **IDP SSO Service URL (Will be Deprecated)** [URL de servicio de SSO de IDP (dejará de usarse)], escriba el valor de **URL de inicio de sesión**.

    c. Seleccione **Do Not Deflate SP-initiated Authentication Request (Will be Deprecated)** [No desinflar la solicitud de autenticación iniciada por SP (dejará de usarse)].

    d. Como **Método de firma de solicitud de autenticación**, seleccione **SHA256**.

    e. Haga clic en **OK**.

    > [!NOTE]
    > Asegúrese de configurar el inicio de sesión único correctamente. Si habilita el inicio de sesión único con una configuración incorrecta, es posible que no pueda entrar en la aplicación con sus credenciales y quede bloqueado fuera de esta. En esta situación, Workday proporciona una URL de inicio de sesión de respaldo donde los usuarios pueden iniciar sesión con su nombre de usuario y contraseña normal en el siguiente formato: [Su URL de Workday]/login.flex?redirect=n

### <a name="create-workday-test-user"></a>Creación de un usuario de prueba de Workday

1. Inicie sesión en el sitio de Workday como administrador.

1. Haga clic en **Profile** (Perfil) en la esquina superior derecha, seleccione **Home** (Inicio) y haga clic en **Directory** (Directorio) en la pestaña **Applications** (Aplicaciones). 

1. En la página **Directory** (Directorio), seleccione **Find Workers** (Buscar trabajadores) en la pestaña View (Ver).

    ![Búsqueda de trabajadores](./media/workday-tutorial/user-directory.png)

1.  En la página **Find Workers** (Buscar trabajadores), seleccione el usuario en los resultados.

1. En la página siguiente, seleccione **Job > Worker Security** (Trabajo > Seguridad del trabajador), y **Workday account** (Cuenta de Workday) debe coincidir con Azure Active Directory como valor de **Name ID** (Id. de nombre).

    ![Seguridad del trabajador](./media/workday-tutorial/worker-security.png)

> [!NOTE]
> Para más información sobre cómo crear un usuario de prueba de Workday, póngase en contacto con el [equipo de soporte técnico al cliente de Workday](https://www.workday.com/en-us/partners-services/services/support.html).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Workday, desde donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Workday e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Workday en Aplicaciones, se debería iniciar sesión automáticamente en la instancia de Workday para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Workday, puede aplicar el control de sesión, que protege a su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).