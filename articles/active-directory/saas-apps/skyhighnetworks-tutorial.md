---
title: 'Tutorial: Integración de Azure Active Directory con MVISION Cloud Azure AD SSO Configuration | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y MVISION Cloud Azure AD SSO Configuration.
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
ms.openlocfilehash: c427353a68f06a31c214684e4e4bdd8021b68da8
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203746"
---
# <a name="tutorial-integrate-mvision-cloud-azure-ad-sso-configuration-with-azure-active-directory"></a>Tutorial: Integración de MVISION Cloud Azure AD SSO Configuration con Azure Active Directory

En este tutorial aprenderá a integrar MVISION Cloud Azure AD SSO Configuration con Azure Active Directory (Azure AD). Al integrar MVISION Cloud Azure AD SSO Configuration con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a MVISION Cloud Azure AD SSO Configuration.
* Permitir que los usuarios inicien sesión automáticamente en MVISION Cloud Azure AD SSO Configuration con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) de MVISION Cloud Azure AD SSO Configuration.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* MVISION Cloud Azure AD SSO Configuration admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="add-mvision-cloud-azure-ad-sso-configuration-from-the-gallery"></a>Incorporación de MVISION Cloud Azure AD SSO Configuration desde la galería

Para configurar la integración de MVISION Cloud Azure AD SSO Configuration en Azure AD, debe agregar MVISION Cloud Azure AD SSO Configuration desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la Galería**, escriba **MVISION Cloud Azure AD SSO Configuration** en el cuadro de búsqueda.
1. Seleccione **MVISION Cloud Azure AD SSO Configuration** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-mvision-cloud-azure-ad-sso-configuration"></a>Configuración y prueba del inicio de sesión único de Azure AD para MVISION Cloud Azure AD SSO Configuration

Configure y pruebe el inicio de sesión único de Azure AD con MVISION Cloud Azure AD SSO Configuration mediante un usuario de prueba llamado **Britta Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de MVISION Cloud Azure AD SSO Configuration.

Para configurar y probar el inicio de sesión único de Azure AD con MVISION Cloud Azure AD SSO Configuration, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    4. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de MVISION Cloud Azure AD SSO Configuration](#configure-mvision-cloud-azure-ad-sso-configuration-sso)** , para configurar las opciones de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de MVISION Cloud Azure AD SSO Configuration](#create-mvision-cloud-azure-ad-sso-configuration-test-user)** , para tener un homólogo de Britta Simon en MVISION Cloud Azure AD SSO Configuration que esté vinculado a la representación de Azure AD del usuario.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Datadog**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<ENV>.myshn.net/shndash/response/saml-postlogin`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up MVISION Cloud Azure AD SSO Configuration** (Configurar MVISION Cloud Azure AD SSO Configuration), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, concederá a B. Simon acceso a MVISION Cloud Azure AD SSO Configuration para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **MVISION Cloud Azure AD SSO Configuration**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-mvision-cloud-azure-ad-sso-configuration-sso"></a>Configuración del inicio de sesión único de MVISION Cloud Azure AD SSO Configuration

Para configurar el inicio de sesión único en **MVISION Cloud Azure AD SSO Configuration**, es preciso enviar el **Certificado (Base64)** descargado y las direcciones URL copiadas adecuadas de Azure Portal al [equipo de soporte técnico de MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-mvision-cloud-azure-ad-sso-configuration-test-user"></a>Creación del usuario de prueba de MVISION Cloud Azure AD SSO Configuration

En esta sección, creará un usuario llamado B.Simon en MVISION Cloud Azure AD SSO Configuration. Trabaje con el [equipo de soporte técnico de MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com) para agregar los usuarios a la plataforma de MVISION Cloud Azure AD SSO Configuration. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de MVISION Cloud Azure AD SSO Configuration, desde donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de MVISION Cloud Azure AD SSO Configuration y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Debería iniciarse sesión automáticamente en la instancia de MVISION Cloud Azure AD SSO Configuration para la que ha configurado el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de MVISION Cloud Azure AD SSO Configuration en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para poner en marcha el flujo de inicio de sesión y, si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de MVISION Cloud Azure AD SSO Configuration para la que configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado MVISION Cloud Azure AD SSO Configuration, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración y la infiltración de la información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).