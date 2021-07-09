---
title: 'Tutorial: Integración de Azure Active Directory con Pingboard | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Pingboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/01/2021
ms.author: jeedes
ms.openlocfilehash: e0d592274b64e292be6671de27b7d06cf5606e30
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111568627"
---
# <a name="tutorial-azure-active-directory-integration-with-pingboard"></a>Tutorial: Integración de Azure Active Directory con Pingboard

En este tutorial, aprenderá a integrar Pingboard en Azure Active Directory (Azure AD). Al integrar Pingboard en Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Pingboard.
* Permitir que los usuarios inicien sesión automáticamente en Pingboard con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción a Pingboard con el inicio de sesión único (SSO) habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Pingboard admite el inicio de sesión único iniciado por **SP** e **IDP**.

* Pingboard admite el [aprovisionamiento automático de usuarios](./pingboard-provisioning-tutorial.md). 

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-pingboard-from-the-gallery"></a>Adición de Pingboard desde la galería

Para configurar la integración de Pingboard en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Pingboard** en el cuadro de búsqueda.
1. Seleccione **Pingboard** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-pingboard"></a>Configuración y prueba del inicio de sesión único de Azure AD en Pingboard

Configure y pruebe el inicio de sesión único de Azure AD con Pingboard mediante una usuaria de prueba llamada **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Pingboard.

Para configurar y probar el inicio de sesión único de Azure AD con Pingboard, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Pingboard](#configure-pingboard-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Crear un usuario de prueba de Pingboard](#create-pingboard-test-user)** : para tener un homólogo de B.Simon en Pingboard que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Pingboard**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba la dirección URL: `http://app.pingboard.com/sp`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<ENTITY_ID>.pingboard.com/auth/saml/consume`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.pingboard.com/sign_in`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Pingboard](https://support.pingboard.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up Pingboard** (Configurar Pingboard), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Pingboard mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Pingboard**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-pingboard-sso"></a>Configuración del inicio de sesión único de Pingboard

1. Para configurar SSO en Pingboard, abra una nueva ventana del explorador e inicie sesión en su cuenta de Pingboard. Debe ser un administrador de Pingboard para configurar el inicio de sesión único.

2. En el menú superior, seleccione **Aplicaciones > Integraciones**.

    ![Configurar inicio de sesión único](./media/pingboard-tutorial/integration.png)

3. En la página **Integraciones**, busque el icono **"Azure Active Directory"** y haga clic en él.

    ![Integración de inicio de sesión único de Pingboard](./media/pingboard-tutorial/directory.png)

4. En el estado modal que sigue, haga clic en **"Configurar"** .

    ![Botón de configuración de Pingboard](./media/pingboard-tutorial/configure.png)

5. En la siguiente página, podrá ver el mensaje "La integración de inicio de sesión único de Azure SSO está habilitada". Abra el archivo XML de metadatos descargado en el Bloc de notas y pegue el contenido en **metadatos de IDP**.

    ![Pantalla de configuración de inicio de sesión único de Pingboard](./media/pingboard-tutorial/metadata.png)

6. El archivo se validará y, si todo está correcto, se habilitará el inicio de sesión único.

### <a name="create-pingboard-test-user"></a>Creación de un usuario de prueba de Pingboard

El objetivo de esta sección es crear una usuaria llamada Britta Simon en Pingboard. Pingboard admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada. [Aquí](pingboard-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión en el sitio de la compañía Pingboard como administrador.

2. Haga clic en el botón **“Add Employee”** (Agregar empleado) en la página **Directory** (Directorio).

    ![Agregar empleado](./media/pingboard-tutorial/test-user.png)

3. En la página del cuadro de diálogo **Agregar empleado**, realice los siguientes pasos:

    ![Invitar a contactos](./media/pingboard-tutorial/create-name.png)

    a. En el cuadro de texto **Nombre completo**, escriba el nombre completo de un usuario; por ejemplo, **Britta Simon**.

    b. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario; por ejemplo, **brittasimon@contoso.com** .

    c. En el cuadro de texto **Puesto**, escriba el puesto de Britta Simon.

    d. En la lista desplegable **Ubicación**, seleccione la ubicación de Britta Simon.

    e. Haga clic en **Agregar**.

4. Se muestra una pantalla de confirmación para confirmar la adición del usuario.

    ![confirmar](./media/pingboard-tutorial/confirm-user.png)

    > [!NOTE]
    > El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo y seguirá un vínculo para confirmar su cuenta antes de que se active.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Pingboard, donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Pingboard y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal y debería iniciar sesión automáticamente en la instancia de Pingboard para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Pingboard en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Pingboard para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Pingboard, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
