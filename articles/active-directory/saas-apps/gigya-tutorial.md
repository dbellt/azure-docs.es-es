---
title: 'Tutorial: Integración de Azure Active Directory con Gigya | Microsoft Doc'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Gigya.
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
ms.openlocfilehash: f2ce885de4c3745f48cbc9f9df69a1d9e868f9bb
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146102"
---
# <a name="tutorial-azure-active-directory-integration-with-gigya"></a>Tutorial: integración de Azure Active Directory con Gigya

En este tutorial aprenderá a integrar Gigya con Azure Active Directory (Azure AD). Al integrar Gigya con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Gigya.
* Permitir que los usuarios inicien sesión automáticamente en Gigya con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Gigya.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Gigya admite el inicio de sesión único iniciado por **SP**.

## <a name="add-gigya-from-the-gallery"></a>Incorporación de Gigya desde la galería

Para configurar la integración de Gigya en Azure AD, es preciso agregar Gigya desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Gigya** en el cuadro de búsqueda.
1. Seleccione **Gigya** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-gigya"></a>Configuración y prueba del inicio de sesión único de Azure AD para Gigya

Configure y pruebe el inicio de sesión único de Azure AD con Gigya mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Gigya.

Para configurar y probar el inicio de sesión único de Azure AD con Gigya, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Gigya](#configure-gigya-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Gigya](#create-gigya-test-user)** : para tener un homólogo de B.Simon en Gigya vinculado a la representación del usuario de Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Gigya**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `http://<companyname>.gigya.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://fidm.gigya.com/saml/v2.0/<companyname>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de clientes de Gigya](https://developers.gigya.com/display/GD/Opening+A+Support+Incident). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Gigya**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección va a permitir que B.Simon acceda a Gigya mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Gigya**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-gigya-sso"></a>Configuración del inicio de sesión único en Gigya

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Gigya como administrador.

2. Vaya **Settings \> SAML Login** (Configuración > Inicio de sesión de SAML), y luego haga clic en el botón **Add** (Agregar).
   
    ![SAML Login (Inicio de sesión de SAML)](./media/gigya-tutorial/login.png "Inicio de sesión SAML")

3. En la sección **Inicio de sesión de SAML**, siga estos pasos:
   
    ![Configuración de SAML](./media/gigya-tutorial/configuration.png "Configuración de SAML")
   
    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración.
   
    b. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal. 
   
    c. En el cuadro de texto **Single Sign-On Service URL** (Dirección URL del servicio de inicio de sesión único de SAML), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.
   
    d. En el cuadro de texto **Name ID Format** (Formato de identificador de nombre), pegue el valor de **Formato de identificador de nombre** que ha copiado de Azure Portal.
   
    e. Abra el certificado codificado en base 64 descargado de Azure Portal en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado X.509**.
   
    f. Haga clic en **Guardar configuración**.

## <a name="create-gigya-test-user"></a>Creación de usuario de prueba de Gigya

Para permitir que los usuarios de Azure AD inicien sesión en Gigya, deben aprovisionarse en Gigya. En el caso de Gigya, el aprovisionamiento es una tarea manual.

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>Para aprovisionar cuentas de usuario, realice los siguientes pasos:

1. Inicie sesión en el sitio de la empresa **Gigya** como administrador.

2. Vaya a **Admin \> Manage Users** (Administrador > Administrar usuarios) y, a continuación, haga clic en **Invite Users** (Invitar a usuarios).
   
    ![Administración de usuarios](./media/gigya-tutorial/users.png "Administrar usuarios")

3. En el cuadro de diálogo Invite Users (Invitar a usuarios), realice los pasos siguientes:
   
    ![Invite Users (Invitar a usuarios)](./media/gigya-tutorial/invite-user.png "Invitar a usuarios")
   
    a. En el cuadro de texto **Correo electrónico**, escriba el alias de correo electrónico de una cuenta válida de Azure Active Directory que desee aprovisionar.
    
    b. Haga clic en **Invitar usuario**.
      
    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo electrónico con un vínculo para confirmar la cuenta antes de que se active.
    > 

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Gigya, desde donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la URL de inicio de sesión de Gigya y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Gigya en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Gigya, podrá aplicar el control de sesión, que protege la información confidencial de su organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).