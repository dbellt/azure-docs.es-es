---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Standard for Success Accreditation | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Standard for Success Accreditation.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/21/2021
ms.author: jeedes
ms.openlocfilehash: a9ecec2456354c2d766d528d2c29ceb1833a7ad9
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110481772"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-standard-for-success-accreditation"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Standard for Success Accreditation

En este tutorial, aprenderá a integrar Standard for Success Accreditation con Azure Active Directory (Azure AD). Cuando integra Standard for Success Accreditation con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Standard for Success Accreditation.
* Permitir que los usuarios inicien sesión automáticamente en Standard for Success Accreditation con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Standard for Success Accreditation.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Standard for Success Accreditation admite el inicio de sesión único iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-standard-for-success-accreditation-from-the-gallery"></a>Adición de Standard for Success Accreditation desde la galería

Para configurar la integración de Standard for Success Accreditation en Azure AD, debe agregar Standard for Success Accreditation desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Standard for Success Accreditation** en el cuadro de búsqueda.
1. Seleccione **Standard for Success Accreditation** en el panel de resultados y, después, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-standard-for-success-accreditation"></a>Configuración y prueba del inicio de sesión único de Azure AD de Standard for Success Accreditation

Configure y pruebe el inicio de sesión único de Azure AD con Standard for Success Accreditation utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Standard for Success Accreditation.

Para configurar y probar el inicio de sesión único de Azure AD con Standard for Success Accreditation, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Standard for Success Accreditation](#configure-standard-for-success-accreditation-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Standard for Success Accreditation](#create-standard-for-success-accreditation-test-user)** , para tener un homólogo de B.Simon en Standard for Success Accreditation que esté vinculado a la representación de Azure AD del usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Standard for Success Accreditation**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://edu.sfsed.com/access/saml_consume?did=<INSTITUTIONID>`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://edu.sfsed.com/access/saml_int?did=<INSTITUTIONID>`

    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://edu.sfsed.com/access/saml_consume?did=<INSTITUTIONID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico al cliente de Standard for Success Accreditation](mailto:help_he@standardforsuccess.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar Standard for Success Accreditation**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Standard for Success Accreditation mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Standard for Success Accreditation**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-standard-for-success-accreditation-sso"></a>Configuración del inicio de sesión único de Standard for Success Accreditation

1. Abra una nueva ventana del explorador web e inicie sesión en el sitio de Standard for Success Accreditation como administrador con acceso de superusuario.

1. En el menú, haga clic en **Portal de administración**.

1. Desplácese hacia abajo a **Configuración de inicio de sesión único**, haga clic en el vínculo del **Inicio de sesión único de Microsoft Azure** y siga estos pasos.

    ![Página de inicio de sesión único de Microsoft Azure](./media/standard-for-success-accreditation-tutorial/configuration.png)

    a. Marque la casilla **Enable Azure Single Sign On** (Habilitar inicio de sesión único de Azure).

    b. Rellene el cuadro de texto **Id. de inquilino de Azure** con el valor del identificador de inquilino de Azure Portal.

    c. Rellene el identificador de aplicación en el cuadro de texto **Id. de aplicación**.

    d. En el cuadro de texto **Huella digital del certificado**, pegue el **valor de huella digital** que ha copiado de Azure Portal.

    e. Haga clic en **Save**(Guardar). 

### <a name="create-standard-for-success-accreditation-test-user"></a>Creación de un usuario de prueba de Standard for Success Accreditation

1.  Inicie sesión en Standard for Success Accreditation como administrador con privilegios de superusuario.

1. En el menú, haga clic en **Portal de administración -> Create New Evaluatee** (Crear nueva evaluación) y realice los pasos siguientes.

    ![creación de un usuario de prueba.](./media/standard-for-success-accreditation-tutorial/new-user.png)

    a. En el cuadro de texto **First Name** (Nombre), escriba B.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba Simon.

    c. En el cuadro de texto **University Email** (Correo electrónico de la universidad), escriba la dirección de correo electrónico de su organización.

    d. Desplácese hasta la parte inferior y haga clic en **Create User** (Crear usuario).


## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Standard for Success Accreditation, donde podrá iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Standard for Success Accreditation e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Standard for Success Accreditation para la que configurara el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Standard for Success Accreditation en Aplicaciones, si tiene la configuración del modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión y, si tiene la configuración del modo IDP, debería iniciar sesión automáticamente en la instancia de Standard for Success Accreditation para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Standard for Success Accreditation, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).


