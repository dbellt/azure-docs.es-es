---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Onit | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Onit.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/31/2021
ms.author: jeedes
ms.openlocfilehash: b08121b7008dd40944eb1035ecb968ac0d01b8af
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111557951"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-onit"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Onit

En este tutorial, obtendrá información sobre cómo integrar Onit con Azure Active Directory (Azure AD). Al integrar Onit con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Onit.
* Permitir que los usuarios inicien sesión automáticamente en Onit con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Onit.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Onit admite el inicio de sesión único iniciado por **SP**.

## <a name="add-onit-from-the-gallery"></a>Adición de Onit desde la galería

Para configurar la integración de Onit en Azure AD, deberá agregar Onit desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Onit** en el cuadro de búsqueda.
1. Seleccione **Onit** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-onit"></a>Configuración y prueba del inicio de sesión único de Azure AD para Onit

Configure y pruebe el inicio de sesión único de Azure AD con Onit mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Onit.

Para configurar y probar el inicio de sesión único de Azure AD con Onit, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Onit](#configure-onit-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Onit](#create-onit-test-user)** , para tener un homólogo de B.Simon en Onit que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Onit**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.onit.com`

    b. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.onit.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Onit](https://www.onit.com/support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar Onit**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Onit mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Onit**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-onit-sso"></a>Configuración del inicio de sesión único de Onit

1. En otra ventana del explorador web, inicie sesión en como administrador en el sitio de Onit de la compañía.

2. En el menú de la parte superior, haga clic en **Administration**(Administración).
   
    ![Captura de pantalla que muestra el menú de la parte superior de la página "M S S S O Test" (Probar M S S S O) con la acción "Administración" seleccionada.](./media/onit-tutorial/admin.png "Administración")

3. Haga clic en **Edit Corporation**(Editar corporación).
   
    ![Edit Corporation](./media/onit-tutorial/corporation.png "Edit Corporation")
   
4. Haga clic en la pestaña **Security** (Seguridad).
    
    ![Editar información de la empresa](./media/onit-tutorial/security.png "Editar información de la compañía")

5. En la pestaña **Seguridad** , lleve a cabo estos pasos:

    ![Inicio de sesión único](./media/onit-tutorial/configuration.png "Inicio de sesión único")

    a. Como **Estrategia de autenticación**, seleccione **Inicio de sesión único y contraseña**.
    
    b. En el cuadro de texto **Idp Target URL** (Dirección URL de destino de IdP), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Idp logout URL** (Dirección URL de cierre de sesión de IdP), pegue el valor de **Dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Idp Cert Fingerprint (SHA1)** (Huella digital de certificado del Idp [SHA1]), pegue el valor de **Huella digital** del certificado que ha copiado de Azure Portal.

### <a name="create-onit-test-user"></a>Creación de un usuario de prueba de Onit

Para permitir que los usuarios de Azure AD inicien sesión en Onit, tienen que aprovisionarse en Onit. En el caso de Onit, el aprovisionamiento es una tarea manual.

**Siga estos pasos para configurar el aprovisionamiento de usuario:**

1. Inicie sesión en su sitio de la compañía de **Onit** como administrador.

2. Haga clic en **Agregar usuario**.

    ![Administración](./media/onit-tutorial/user.png "Administración")

3. En la página del cuadro de diálogo **Add User** (Agregar usuario), realice los siguientes pasos:

    ![Agregar usuario](./media/onit-tutorial/create-user.png "Agregar usuario")

    a. Escriba el **Nombre** y la **Dirección de correo electrónico** de una cuenta de Azure AD válida que quiera aprovisionar en los cuadros de texto relacionados.

    b. Haga clic en **Crear**.

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Onit, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Onit e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Onit en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Onit, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).

