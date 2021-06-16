---
title: 'Tutorial: Integración de Azure Active Directory con Jitbit Helpdesk | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Jitbit Helpdesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/09/2021
ms.author: jeedes
ms.openlocfilehash: 7a9d0e2c52b0996699c94e6d8eb5f44921f76709
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111985551"
---
# <a name="tutorial-azure-active-directory-integration-with-jitbit-helpdesk"></a>Tutorial: Integración de Azure Active Directory con Jitbit Helpdesk

En este tutorial, aprenderá a integrar Jitbit Helpdesk con Azure Active Directory (Azure AD). Si integra Jitbit Helpdesk con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Jitbit Helpdesk.
* Permitir que los usuarios inicien sesión automáticamente en Jitbit Helpdesk con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción que permita utilizar el inicio de sesión único (SSO) de Jitbit Helpdesk.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Jitbit Helpdesk permite utilizar el inicio de sesión único con **SP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-jitbit-helpdesk-from-the-gallery"></a>Incorporación de Jitbit Helpdesk desde la galería

Para configurar la integración de Jitbit Helpdesk en Azure AD, será preciso que agregue Jitbit Helpdesk desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Jitbit Helpdesk** en el cuadro de búsqueda.
1. Seleccione **Jitbit Helpdesk** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-jitbit-helpdesk"></a>Configuración y comprobación del inicio de sesión único de Azure AD con Jitbit Helpdesk

Configure y pruebe el inicio de sesión único de Azure AD con Jitbit Helpdesk utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso vincular un usuario de Azure AD con el usuario de Jitbit Helpdesk correspondiente.

Para configurar y probar el inicio de sesión único de Azure AD con Jitbit Helpdesk, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Jitbit Helpdesk](#configure-jitbit-helpdesk-sso)** : para configurar el inicio de sesión único en el lado de la aplicación.
    1. **[Creación de un usuario de prueba de Jitbit Helpdesk](#create-jitbit-helpdesk-test-user)** : para tener un homólogo de B.Simon en Jitbit Helpdesk que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Jitbit Helpdesk**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una de las direcciones URL con este patrón:
    | |
    | ----------------------------------------|
    | `https://<hostname>/helpdesk/User/Login`|
    | `https://<tenant-name>.Jitbit.com`|
    | |
    
    > [!NOTE] 
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Jitbit Helpdesk](https://www.jitbit.com/support/) para obtener este valor.

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `https://www.jitbit.com/web-helpdesk/`.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Jitbit Helpdesk** (Configuración de Jitbit Helpdesk), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon utilice el inicio de sesión único de Azure concediéndole acceso a Jitbit Helpdesk.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Jitbit Helpdesk**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-jitbit-helpdesk-sso"></a>Configuración del inicio de sesión único de Jitbit Helpdesk

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía de Jitbit Helpdesk.

1. En la barra de herramientas de la parte superior, haga clic en **Administración**.

    ![Administración](./media/jitbit-helpdesk-tutorial/settings.png "Administración")

1. Haga clic en **Configuración general**.

    ![Captura de pantalla que muestra el vínculo Configuración general.](./media/jitbit-helpdesk-tutorial/general.png "Usuarios, compañías y permisos")

1. En la sección **Configuración de autenticación** , realice estos pasos:

    ![Configuración de la autenticación](./media/jitbit-helpdesk-tutorial/authentication.png "Configuración de autenticación")

    a. Seleccione **Enable SAML 2.0 single sign on** (Habilitar inicio de sesión único de SAML 2.0) para iniciar sesión mediante inicio de sesión único (SSO) con **OneLogin**.

    b. En el cuadro de texto **EndPoint URL** (Dirección URL de punto de conexión), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el certificado codificado en **base-64** en el Bloc de notas, copie el contenido del mismo en el Portapapeles y péguelo en el cuadro de texto **Certificado X.509**.

    d. Haga clic en **Guardar cambios**.

### <a name="create-jitbit-helpdesk-test-user"></a>Creación de un usuario de prueba de Jitbit Helpdesk

Para permitir que los usuarios de Azure AD inicien sesión en Jitbit Helpdesk, deben aprovisionarse en Jitbit Helpdesk. En el caso de Jitbit Helpdesk, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **Jitbit Helpdesk**.

1. En el menú de la parte superior, haga clic en **Administration**(Administración).

    ![Administración](./media/jitbit-helpdesk-tutorial/settings.png "Administración")

1. Haga clic en **Usuarios, compañías y permisos**.

    ![Usuarios, compañías y permisos](./media/jitbit-helpdesk-tutorial/users.png "Usuarios, compañías y permisos")

1. Haga clic en **Agregar usuario**.

    ![Agregar usuario](./media/jitbit-helpdesk-tutorial/add.png "Agregar usuario")

1. En la sección Crear, escriba los datos de la cuenta de Azure AD que desee aprovisionar como sigue:

    ![Creación](./media/jitbit-helpdesk-tutorial/create-section.png "Crear")

   a. En el cuadro de texto **Nombre de usuario**, escriba el nombre de un usuario, por ejemplo, **Britta Simon**.

   b. En el cuadro de texto **Correo electrónico**, escriba el correo electrónico del usuario, en el ejemplo **BrittaSimon@contoso.com** .

   c. En el cuadro de texto **Nombre**, escriba el nombre del usuario, en este caso **Britta**.

   d. En el cuadro de texto **Apellidos**, escriba el apellido del usuario, en este caso **Simon**.

   e. Haga clic en **Crear**.

> [!NOTE]
> Puede usar cualquier herramienta de creación de cuentas de usuario de Jitbit Helpdesk u otras API proporcionadas por Jitbit Helpdesk para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. De este modo, accederá automáticamente a la dirección URL de inicio de sesión de Jitbit Helpdesk, donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la URL de inicio de sesión de Jitbit Helpdesk y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Si hace clic en el icono de Jitbit Helpdesk de Aplicaciones, accederá automáticamente a la URL de inicio de sesión de Jitbit Helpdesk. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Jitbit Helpdesk, podrá aplicar el control de sesión, que protege la información confidencial de la organización en tiempo real de posibles filtraciones. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).