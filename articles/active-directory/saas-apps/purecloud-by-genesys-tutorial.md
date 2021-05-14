---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con PureCloud by Genesys | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y PureCloud by Genesys.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2021
ms.author: jeedes
ms.openlocfilehash: da3e88706c4a7820920191df419a448a08b9f675
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736599"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purecloud-by-genesys"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con PureCloud by Genesys

En este tutorial, aprenderá a integrar PureCloud by Genesys con Azure Active Directory (Azure AD). Al integrar PureCloud by Genesis con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a PureCloud by Genesys.
* Permitir que los usuarios inicien sesión automáticamente en PureCloud by Genesys con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una, puede obtener una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en PureCloud by Genesys.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* PureCloud by Genesys admite el inicio de sesión único iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-purecloud-by-genesys-from-the-gallery"></a>Adición de PureCloud by Genesys desde la galería

Para configurar la integración de PureCloud by Genesys en Azure AD, deberá agregar PureCloud by Genesys desde la galería a la lista de aplicaciones SaaS administradas. Para ello, siga estos pasos.

1. Inicie sesión en Azure Portal mediante una cuenta profesional o educativa, o bien mediante una cuenta personal de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **PureCloud by Genesys** en el cuadro de búsqueda.
1. Seleccione **PureCloud by Genesys** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-purecloud-by-genesys"></a>Configuración y prueba del inicio de sesión único de Azure AD para PureCloud by Genesis

Configure y pruebe el inicio de sesión único de Azure AD con PureCloud by Genesys mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de PureCloud by Genesys.

Para configurar y probar el inicio de sesión único de Azure AD con PureCloud by Genesys, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en PureCloud by Genesys](#configure-purecloud-by-genesys-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de PureCloud by Genesys](#create-purecloud-by-genesys-test-user)** , para tener un homólogo de B.Simon en PureCloud by Genesys que esté vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Para habilitar el inicio de sesión único de Azure AD en Azure Portal, siga estos pasos:

1. En Azure Portal, en la página de integración de la aplicación **PureCloud by Genesys**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro **Identificador**, escriba las direcciones URL que correspondan a su región:
    
    | URL del identificador |
    |---|
    | https://login.mypurecloud.com/saml |
    | https://login.mypurecloud.de/saml |
    | https://login.mypurecloud.jp/saml |
    | https://login.mypurecloud.ie/saml |
    | https://login.mypurecloud.com.au/saml |
    |

    b. En el cuadro **URL de respuesta**, escriba las direcciones URL que correspondan a su región:

    | URL de respuesta |
    |---|
    | https://login.mypurecloud.com/saml |
    | https://login.mypurecloud.de/saml |
    | https://login.mypurecloud.jp/saml |
    | https://login.mypurecloud.ie/saml |
    | https://login.mypurecloud.com.au/saml |
    |

1. Seleccione **Establecer direcciones URL adicionales** y lleve a cabo el siguiente paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro **URL de inicio de sesión**, escriba las direcciones URL que correspondan a su región:
    
    |URL de inicio de sesión |
    |---|
    | https://login.mypurecloud.com |
    | https://login.mypurecloud.de |
    | https://login.mypurecloud.jp |
    | https://login.mypurecloud.ie |
    | https://login.mypurecloud.com.au |
    |

1. La aplicación PureCloud by Genesys espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de atributos de token SAML. En la siguiente captura se muestra la lista de los atributos predeterminados:

    ![imagen](common/default-attributes.png)

1. Además, la aplicación PureCloud by Genesys espera que se devuelvan algunos atributos más en la respuesta de SAML, como se muestra en la tabla siguiente. Estos atributos también se rellenan previamente, pero puede revisarlos según necesite.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | Email | user.userprincipalname |
    | OrganizationName | `Your organization name` |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar PureCloud by Genesys**, copie las direcciones URL adecuadas según sus requisitos.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado B.Simon en Azure Portal:

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba el nombre de usuario con el siguiente formato: username@companydomain.extension. Por ejemplo: `B.Simon@contoso.com`.
   1. Active la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, concederá acceso a B.Simon a PureCloud by Genesys para que use el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **PureCloud by Genesys**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-purecloud-by-genesys-sso"></a>Configuración del inicio de sesión único en PureCloud by Genesys

1. En otra ventana del explorador web, inicie sesión en PureCloud by Genesys como administrador.

1. Seleccione **Admin** (Administración) en la parte superior y, a continuación, vaya a **Single Sign-On** (Inicio de sesión único) en **Integrations** (Integraciones).

    ![Captura de pantalla que muestra la ventana de administración de PureCloud, donde puede seleccionar la opción de inicio de sesión único.](./media/purecloud-by-genesys-tutorial/configure-1.png)

1. Cambie a la pestaña **ADFS/Azure AD (Premium)** y, a continuación, siga estos pasos:

    ![Captura de pantalla que muestra la página de integraciones, donde puede especificar los valores descritos.](./media/purecloud-by-genesys-tutorial/configure-2.png)

    a. Seleccione **Browse** (Examinar) para cargar en **ADFS Certificate** (Certificado de ADFS) el certificado codificado en base 64 que descargó de Azure Portal.

    b. En el cuadro de texto **ADFS Issuer URI** (URI de emisor de ADFS), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro de texto **Target URI** (URI de destino), pegue el valor de **URL de inicio de sesión** que copió de Azure Portal.

    d. Para el valor de **Relying Party Identifier** (Identificador de usuario de confianza), vaya a Azure Portal y, en la página de integración de aplicaciones de **PureCloud by Genesys**, seleccione la pestaña **Propiedades** y copie el valor de **Id. de aplicación**. Péguelo en el cuadro de texto **Relying Party Identifier** (Identificador de usuario de confianza).

    ![Captura de pantalla que muestra el panel de propiedades, donde puede encontrar el valor de identificador de la aplicación.](./media/purecloud-by-genesys-tutorial/configure-6.png)

    e. Seleccione **Guardar**.

### <a name="create-purecloud-by-genesys-test-user"></a>Creación de un usuario de prueba de PureCloud by Genesys

Para permitir que los usuarios de Azure AD inicien sesión en PureCloud by Genesys, deben aprovisionarse en PureCloud by Genesys. En PureCloud by Genesys, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, siga estos pasos:**

1. Inicie sesión en PureCloud by Genesys como administrador.

1. Seleccione **Admin** (Administrador) en la parte superior y vaya a **People** (Personas) en **People & Permissions** (Personas y permisos).

    ![Captura de pantalla que muestra la ventana de administración de PureCloud, donde puede seleccionar la opción de personas.](./media/purecloud-by-genesys-tutorial/configure-3.png)

1. En la página **People** (Personas), haga clic en **Add Person** (Agregar persona).

    ![Captura de pantalla que muestra la página de personas en la que puede agregar una persona.](./media/purecloud-by-genesys-tutorial/configure-4.png)

1. En el cuadro de diálogo **Add People to the Organization** (Agregar personas a la organización), siga estos pasos:

    ![Captura de pantalla que muestra la página donde puede especificar los valores descritos.](./media/purecloud-by-genesys-tutorial/configure-5.png)

    a. En el cuadro **Full name** (Nombre completo), escriba el nombre del usuario. Por ejemplo: **B.Simon**.

    b. En el cuadro **Email** (Correo electrónico), escriba el correo electrónico del usuario. Por ejemplo, **b.simon\@contoso.com**.

    c. Seleccione **Crear**.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de PureCloud by Genesys, donde podrá iniciar el flujo de inicio de sesión.  

* Acceda directamente a la dirección URL de inicio de sesión de PureCloud by Genesys y comience el flujo de inicio de sesión desde ahí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de PureCloud by Genesys para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de PureCloud by Genesys en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de PureCloud by Genesys para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada la aplicación PureCloud by Genesys, podrá aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).