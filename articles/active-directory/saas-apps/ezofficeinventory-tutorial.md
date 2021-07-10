---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EZOfficeInventory | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y EZOfficeInventory.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/02/2021
ms.author: jeedes
ms.openlocfilehash: 8306d593b74c6b7a51d59ba59e40378ed30677f4
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111556317"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ezofficeinventory"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EZOfficeInventory

En este tutorial aprenderá a integrar EZOfficeInventory con Azure Active Directory (Azure AD). Al integrar EZOfficeInventory con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a EZOfficeInventory.
* Permitir que los usuarios inicien sesión automáticamente en EZOfficeInventory con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en EZOfficeInventory.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* EZOfficeInventory admite el SSO iniciado por **SP**.
* EZOfficeInventory admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-ezofficeinventory-from-the-gallery"></a>Incorporación de EZOfficeInventory desde la galería

Para configurar la integración de EZOfficeInventory en Azure AD, deberá agregar EZOfficeInventory desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **EZOfficeInventory** en el cuadro de búsqueda.
1. Seleccione **EZOfficeInventory** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-ezofficeinventory"></a>Configuración y prueba del SSO de Azure AD para EZOfficeInventory

Configure y pruebe el inicio de sesión único de Azure AD con EZOfficeInventory mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de EZOfficeInventory.

Para configurar y probar el inicio de sesión único de Azure AD con EZOfficeInventory, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de EZOfficeInventory](#configure-ezofficeinventory-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de EZOfficeInventory](#create-ezofficeinventory-test-user)** , para tener un homólogo de B.Simon en EZOfficeInventory vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación de **EZOfficeInventory**, busque la sección **Administrar** y seleccione **inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.ezofficeinventory.com/users/sign_in`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de EZOfficeInventory Client](mailto:support@ezofficeinventory.com) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación EZOfficeInventory espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación EZOfficeInventory espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | first_name | user.givenname |
    | last_name | user.surname |
    | email | user.mail |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar EZOfficeInventory**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a EZOfficeInventory mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **EZOfficeInventory** .
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ezofficeinventory-sso"></a>Configuración del inicio de sesión único de EZOfficeInventory

1. Para automatizar la configuración en EZOfficeInventory, es preciso instalar la **extensión de inicio de sesión seguro de mis aplicaciones**, para lo que es preciso hacer clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up EZOfficeInventory** (Configurar EZOfficeInventory) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en EZOfficeInventory. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar EZOfficeInventory manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de EZOfficeInventory como administrador y haga lo siguiente:

1. En la esquina superior derecha de la página, haga clic en su **perfil** y vaya a **Configuración** > **Complementos**.

    ![Captura de pantalla que muestra la página "Settings" (Configuración) con la acción "Add Ons" (Complementos) seleccionada.](./media/ezofficeinventory-tutorial/settings.png)

1. Desplácese hacia abajo hasta la sección **SAML Integration** (Integración de SAML) y realice los pasos siguientes:

    ![Configuración de EZOfficeInventory](./media/ezofficeinventory-tutorial/integration.png)

    a. Marque la opción **Enabled** (Habilitado).

    b. En el cuadro de texto **Identity Provider URL** (URL del proveedor de identidades), pegue el valor de **Login URL** (Dirección URL de inicio de sesión) que ha copiado de Azure Portal.

    c. Abra el certificado codificado en Base64 con el Bloc de notas, copie su contenido y péguelo en el cuadro de texto **Identity Provider Certificate** (Certificado del proveedor de identidades).

    d. En el cuadro de texto **Login Button Text** (Texto del botón de inicio de sesión), escriba el texto del botón de inicio de sesión.

    e. En el cuadro de texto **First Name** (Nombre), escriba **first_name**.

    f. En el cuadro de texto **Last Name** (Apellidos), escriba **last_name**.

    g. En el cuadro de texto **Email** (Correo electrónico), escriba **email**.

    h. Seleccione el rol según su requisito en la opción **EZOfficeInventory Role By default** (Rol predeterminado de EZOfficeInventory).

    i. Haga clic en **Update**(Actualizar).

### <a name="create-ezofficeinventory-test-user"></a>Creación de un usuario de prueba de EZOfficeInventory

En esta sección, se crea un usuario llamado Britta Simon en EZOfficeInventory. EZOfficeInventory admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si aún no existe un usuario en EZOfficeInventory, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de EZOfficeInventory, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de EZOfficeInventory e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de EZOfficeInventory en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado EZOfficeInventory, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
