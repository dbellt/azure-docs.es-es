---
title: 'Tutorial: Integración de Azure Active Directory con Menlo Security | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Menlo Security.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/19/2021
ms.author: jeedes
ms.openlocfilehash: 21495c37b50fbab80546be50f4900766a1862285
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475603"
---
# <a name="tutorial-azure-active-directory-integration-with-menlo-security"></a>Tutorial: Integración de Azure Active Directory con Menlo Security

En este tutorial, aprenderá a integrar Menlo Security con Azure Active Directory (Azure AD). Al integrar Menlo Security con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Menlo Security.
* Permitir que los usuarios inicien sesión automáticamente en Menlo Security con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Menlo Security.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Menlo Security admite el inicio de sesión único iniciado por **SP**.

## <a name="add-menlo-security-from-the-gallery"></a>Adición de Menlo Security desde la galería

Para configurar la integración de Menlo Security en Azure AD, debe agregar Menlo Security desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Menlo Security** en el cuadro de búsqueda.
1. Seleccione **Menlo Security** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-menlo-security"></a>Configuración y prueba del inicio de sesión único de Azure AD para Menlo Security

Configure y pruebe el inicio de sesión único de Azure AD con Menlo Security con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Menlo Security.

Para configurar y probar el inicio de sesión único de Azure AD con Menlo Security, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Menlo Security](#configure-menlo-security-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Menlo Security](#create-menlo-security-test-user)** : para tener un homólogo de B.Simon en Menlo Security que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Menlo Security**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.menlosecurity.com/account/login`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.menlosecurity.com/safeview-auth-server/saml/metadata`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de Menlo Security](https://www.menlosecurity.com/menlo-contact) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Menlo Security**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección, va a permitir que B.Simon acceda a Menlo Security mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Menlo Security**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-menlo-security-sso"></a>Configuración del inicio de sesión único de Menlo Security

1. Para configurar el inicio de sesión único en **Menlo Security**, regístrese en el sitio web de **Menlo Security** como administrador.

2. En **Settings** (Configuración), vaya a **Authentication** (Autenticación) y realice las siguientes acciones:
    
    ![Configurar inicio de sesión único](./media/menlosecurity-tutorial/authentication.png)

    a. Active la casilla **Enable user authentication using SAML** (Habilitar la autenticación de usuario mediante SAML).

    b. Seleccione **Allow External Access** (Permitir acceso externo) en **Yes** (Sí).

    c. En **SAML Provider** (Proveedor de SAML), seleccione **Azure Active Directory**.

    d. **SAML 2.0 Endpoint** (Punto de conexión de SAML 2.0): pegue el valor de **Dirección URL de inicio de sesión**, que ha copiado de Azure Portal.

    e. **Service Identifier (Issuer)** [Identificador de servicio (emisor)]: pegue el **Identificador de Azure AD** que ha copiado de Azure Portal.

    f. **X.509 Certificate** (Certificado X.509): abra el **Certificado (Base64)** descargado de Azure Portal en el Bloc de notas y péguelo en este cuadro.

    g. Haga clic en **Guardar** para guardar la configuración.

### <a name="create-menlo-security-test-user"></a>Creación de un usuario de prueba de Menlo Security

En esta sección, creará un usuario llamado Britta Simon en Menlo Security. Colabore con el [equipo de soporte de cliente de Menlo Security](https://www.menlosecurity.com/menlo-contact) para agregar los usuarios a la plataforma de Menlo Security. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Menlo Security, donde podrá comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Menlo Security e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Menlo Security en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Menlo Security. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Menlo Security, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
