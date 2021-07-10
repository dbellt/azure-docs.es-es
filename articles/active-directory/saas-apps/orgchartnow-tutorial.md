---
title: 'Tutorial: Integración de Azure Active Directory con OrgChart Now | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y OrgChart Now.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: jeedes
ms.openlocfilehash: c7b2f67c1b753b3dec237f89165d11fa9c58f9e2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110096823"
---
# <a name="tutorial-azure-active-directory-integration-with-orgchart-now"></a>Tutorial: Integración de Azure Active Directory con OrgChart Now

En este tutorial, aprenderá a integrar OrgChart Now con Azure Active Directory (Azure AD). Al integrar OrgChart Now con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a OrgChart Now.
* Permitir que los usuarios inicien sesión automáticamente en OrgNow con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en OrgChart Now.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* OrgChart Now admite el inicio de sesión único iniciado por **SP** e **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-orgchart-now-from-the-gallery"></a>Adición de OrgChart Now desde la galería

Para configurar la integración de OrgChart Now en Azure AD, deberá agregar OrgChart Now de la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **OrgChart Now** en el cuadro de búsqueda.
1. Seleccione **OrgChart Now** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-orgchart-now"></a>Configuración y prueba del inicio de sesión único de Azure AD para OrgChart Now

Configure y pruebe el inicio de sesión único de Azure AD con OrgChart Now mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de OrgChart Now.

Para configurar y probar el inicio de sesión único de Azure AD con OrgChart Now, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en OrgChart Now](#configure-orgchart-now-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba en OrgChart Now](#create-orgchart-now-test-user)** : para tener un homólogo de B.Simon en OrgChart Now vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD 

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **OrgChart Now**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    En el cuadro de texto **Identificador**, escriba la dirección URL `https://sso2.orgchartnow.com`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://sso2.orgchartnow.com/Shibboleth.sso/Login?entityID=<YourEntityID>&target=https://sso2.orgchartnow.com`

    > [!NOTE]
    > `<YourEntityID>` es el **identificador de Azure AD** que se copió de la sección **Set up OrgChart Now** (Configurar OrgChart Now) que se describe más adelante en el tutorial.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up OrgChart Now** (Configurar OrgChart Now), copie las direcciones URL que necesite.

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

En esta sección, permitirá que B.Simon use el inicio de sesión único de Azure concediéndole acceso a OrgChart Now.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **OrgChart Now**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-orgchart-now-sso"></a>Configuración del inicio de sesión único de OrgChart Now

Para configurar el inicio de sesión único en **OrgChart Now**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de OrgChart Now](mailto:ocnsupport@officeworksoftware.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-orgchart-now-test-user"></a>Creación del usuario de prueba en OrgChart Now

Para permitir que los usuarios de Azure AD inicien sesión en OrgChart Now, deben aprovisionarse en OrgChart Now. 

1. OrgChart Now admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. Al intentar acceder a OrgChart Now, se crea un nuevo usuario, en caso de que no exista. La característica de aprovisionamiento de usuarios Just-in-Time solo crea un usuario de **solo lectura** cuando la solicitud de inicio de sesión único procede de un IDP reconocido y el correo electrónico de la aserción de SAML no se encuentra en la lista de usuarios. Para esta característica de aprovisionamiento automático, es necesario crear un grupo de acceso llamado **General** en OrgChart Now. Siga los pasos siguientes para crear un grupo de acceso:

    a. Después de hacer clic en el **engranaje** en la esquina superior derecha de la interfaz de usuario, vaya a la opción **Manage Groups** (Administrar grupos).

      ![Grupos de OrgChart Now](./media/orgchartnow-tutorial/groups.png)   

    b. Seleccione el icono **Add** (Agregar) y asigne al grupo el nombre **General**; a continuación, haga clic en **OK** (Aceptar). 

      ![OrgChart Now: agregar](./media/orgchartnow-tutorial/general.png)

    c. Seleccione las carpetas a las que desea que tengan acceso los usuarios generales o de solo lectura:
    
      ![Carpetas de OrgChart Now](./media/orgchartnow-tutorial/folders.png)

    d. **Bloquee** las carpetas para que solo los usuarios administrativos pueden modificarlas. Después, presione **OK** (Aceptar).

      ![OrgChart Now: bloquear](./media/orgchartnow-tutorial/lock.png)

2. Para crear usuarios **administradores** y usuarios de **lectura/escritura**, debe crear manualmente un usuario con el fin de obtener acceso a su nivel de privilegios mediante inicio de sesión único. Para aprovisionar una cuenta de usuario, realice estos pasos:

    a. Inicie sesión ahora en OrgChart Now como administrador de seguridad.

    b. Haga clic en **Settings** (Configuración) en la esquina superior derecha y, luego, vaya a **Manage Users** (Administrar usuarios).

      ![Configuración de OrgChart Now](./media/orgchartnow-tutorial/settings.png)

    c. Haga clic en **Add** (Agregar) y realice los siguientes pasos:

      ![OrgChart Now: administrar](./media/orgchartnow-tutorial/manage-users.png)

    1. En el cuadro de texto **User ID** (Id. de usuario), escriba el identificador de usuario, por ejemplo, **brittasimon\@contoso.com**.

    1. En el cuadro de texto **Email Address** (Dirección de correo electrónico), escriba el correo electrónico del usuario; por ejemplo, **brittasimon\@contoso.com**.

    1. Haga clic en **Agregar**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de OrgChart Now, donde puede comenzar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de OrgChart Now y comience el flujo de inicio de sesión desde ahí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de OrgChart Now para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de OrgChart Now en Mis aplicaciones, si se configuró en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se configuró en modo IDP, se debería iniciar sesión automáticamente en la instancia de OrgChart Now para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado OrgChart Now, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la exfiltración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
