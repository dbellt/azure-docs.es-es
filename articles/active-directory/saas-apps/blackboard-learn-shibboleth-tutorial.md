---
title: 'Tutorial: Integración de Azure Active Directory con Blackboard Learn - Shibboleth | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Blackboard Learn - Shibboleth.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/04/2021
ms.author: jeedes
ms.openlocfilehash: 099ca4681dcbcc31962a56c08f5599a96c00bcb6
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112008118"
---
# <a name="tutorial-azure-active-directory-integration-with-blackboard-learn---shibboleth"></a>Tutorial: Integración de Azure Active Directory con Blackboard Learn - Shibboleth

En este tutorial, aprenderá a integrar Blackboard Learn - Shibboleth con Azure Active Directory (Azure AD). Al integrar Blackboard Learn - Shibboleth con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Blackboard Learn - Shibboleth.
* Permitir que los usuarios inicien sesión automáticamente en Blackboard Learn - Shibboleth con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:
 
* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único de Blackboard Learn - Shibboleth.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Blackboard Learn - Shibboleth admite el inicio de sesión único iniciado por **SP**.

## <a name="add-blackboard-learn---shibboleth-from-the-gallery"></a>Adición de Blackboard Learn - Shibboleth desde la galería

Para configurar la integración de Blackboard Learn - Shibboleth en Azure AD, es preciso agregar Blackboard Learn - Shibboleth desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Blackboard Learn - Shibboleth** en el cuadro de búsqueda.
1. En el panel de resultados, seleccione **Blackboard Learn - Shibboleth** y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-blackboard-learn---shibboleth"></a>Configuración y prueba del inicio de sesión único de Azure AD para Blackboard Learn - Shibboleth

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con Blackboard Learn - Shibboleth mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Blackboard Learn - Shibboleth.

Para configurar y probar el inicio de sesión único de Azure AD con Blackboard Learn - Shibboleth, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Blackboard Learn - Shibboleth](#configure-blackboard-learn---shibboleth-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Blackboard Learn - Shibboleth](#create-blackboard-learn---shibboleth-test-user)** : para tener un homólogo de B.Simon en Blackboard Learn - Shibboleth que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

En esta sección, habilitará el inicio de sesión único de Azure AD en Azure Portal.

Para configurar el inicio de sesión único de Azure AD con Blackboard Learn - Shibboleth, realice los pasos siguientes:

1. En Azure Portal, en la página de integración de la aplicación **Blackboard Learn - Shibboleth**, seleccione **Inicio de sesión único**.

2. En el cuadro de diálogo **Seleccionar un método de inicio de sesión único**, seleccione el modo **SAML/WS-Fed** para habilitar el inicio de sesión único.

3. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono del lapicero para abrir el cuadro de diálogo **Configuración básica de SAML**.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<yourblackoardlearnserver>.blackboardlearn.com/Shibboleth.sso/Login`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<yourblackoardlearnserver>.blackboardlearn.com/shibboleth-sp`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<yourblackoardlearnserver>.blackboardlearn.com/Shibboleth.sso/SAML2/POST`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con [el equipo de soporte al cliente de Blackboard Learn - Shibboleth](https://www.blackboard.com/forms/contact-us_form.aspx) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Blackboard Learn - Shibboleth** (Configurar Blackboard Learn - Shibboleth), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, le concederá a B.Simon acceso a Blackboard Learn - Shibboleth para que pueda usar el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Blackboard Learn - Shibboleth**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-blackboard-learn---shibboleth-sso"></a>Configuración del inicio de sesión único en Blackboard Learn - Shibboleth

Para configurar el inicio de sesión único de Blackboard Learn - Shibboleth, consulte este [documento](https://help.blackboard.com/Learn/Administrator/SaaS/Authentication/Implement_Authentication/SAML_Authentication_Provider_Type).

### <a name="create-blackboard-learn---shibboleth-test-user"></a>Creación de un usuario de prueba en Blackboard Learn - Shibboleth

En esta sección, creará un usuario llamado Britta Simon en Blackboard Learn - Shibboleth. Trabaje con el [equipo de soporte técnico de Blackboard Learn - Shibboleth](https://www.blackboard.com/forms/contact-us_form.aspx) para agregar los usuarios a la plataforma de Blackboard Learn - Shibboleth. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Blackboard Learn - Shibboleth, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Blackboard Learn - Shibboleth e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Blackboard Learn - Shibboleth en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Blackboard Learn - Shibboleth para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Blackboard Learn - Shibboleth, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
