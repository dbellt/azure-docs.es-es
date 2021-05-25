---
title: 'Tutorial: Integración de Azure Active Directory con SmartRecruiters | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SmartRecruiters.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/03/2021
ms.author: jeedes
ms.openlocfilehash: abe025436562a22fa31c436d8ce47bc2c8b5d31a
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734572"
---
# <a name="tutorial-azure-active-directory-integration-with-smartrecruiters"></a>Tutorial: Integración de Azure Active Directory con SmartRecruiters

En este tutorial, aprenderá a integrar SmartRecruiters con Azure Active Directory (Azure AD). Si integra SmartRecruiters con Azure AD, podrá realizar las siguientes operaciones:

* Controle en Azure AD quién tiene acceso a SmartRecruiters.
* Permita que los usuarios inicien sesión automáticamente en SmartRecruiters con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único en SmartRecruiters.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SmartRecruiters admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="add-smartrecruiters-from-the-gallery"></a>Adición de SmartRecruiters desde la galería

Para configurar la integración de SmartRecruiters en Azure AD, será preciso que agregue SmartRecruiters desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SmartRecruiters** en el cuadro de búsqueda.
1. Seleccione **SmartRecruiters** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-smartrecruiters"></a>Configuración y prueba del inicio de sesión único de Azure AD para SmartRecruiters

Configure y pruebe el inicio de sesión único de Azure AD con SmartRecruiters mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SmartRecruiters.

Para configurar y probar el inicio de sesión único de Azure AD con SmartRecruiters, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de SmartRecruiters](#configure-smartrecruiters-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SmartRecruiters](#create-smartrecruiters-test-user)** : para tener un homólogo de B. Simon en SmartRecruiters que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicaciones **SmartRecruiters**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://www.smartrecruiters.com/web-sso/saml/<companyname>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.smartrecruiters.com/web-sso/saml/<companyname>/callback`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.smartrecruiters.com/web-sso/saml/<companyname>/login`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up SmartRecruiters** (Configurar SmartRecruiters), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B. Simon acceda a SmartRecruiters mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SmartRecruiters**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-smartrecruiters-sso"></a>Configuración del inicio de sesión único de SmartRecruiters

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de SmartRecruiters como administrador.

1. Vaya a **Settings / Admin** (Configuración / Administrador).

    ![Captura de pantalla que muestra las opciones de configuración y administrador seleccionadas en un menú.](./media/smartrecruiters-tutorial/configure.png)

1. En la sección **Configuration** (Configuración), haga clic en **Web SSO**.

    ![Captura de pantalla que muestra la opción de S S O web seleccionada en la configuración.](./media/smartrecruiters-tutorial/configuration-section.png)

1. Alterne el conmutador **Enable Web SSO** (Habilitar SSO Web).

    ![Captura de pantalla que muestra el control para habilitar S S O web.](./media/smartrecruiters-tutorial/enable-web.png)

1. En **Identity Provider Configuration** (Configuración del proveedor de identidades), realice los pasos siguientes:

    ![Captura de pantalla que muestra la configuración del proveedor de identidades, donde puede especificar los valores descritos.](./media/smartrecruiters-tutorial/identity-provider.png)

    a. En el cuadro de texto **Identity Provider URL** (Dirección URL del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    b. Abra el **certificado (Base64)** que ha descargado de Azure Portal en el Bloc de notas, copie su contenido y péguelo en el cuadro de texto **Identity Provider certificate** (Certificado del proveedor de identidades).

1. Haga clic en **Save Web SSO configuration** (Guardar configuración SSO Web).

### <a name="create-smartrecruiters-test-user"></a>Creación de un usuario de prueba de SmartRecruiters

En esta sección, creará un usuario llamado Britta Simon en SmartRecruiters. Póngase en contacto con el [equipo de soporte técnico de SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/) para agregar los usuarios en la plataforma de SmartRecruiters. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de SmartRecruiters, desde donde puede comenzar el flujo de inicio de sesión.  

* Acceda directamente a la URL de inicio de sesión de SmartRecruiters y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Se debería iniciar sesión automáticamente en la instancia de SmartRecruiters para la que ha configurado el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de SmartRecruiters en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de SmartRecruiters para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SmartRecruiters, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
