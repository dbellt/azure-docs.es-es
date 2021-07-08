---
title: 'Tutorial: Integración de Azure Active Directory con Sprinklr | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Sprinklr.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: jeedes
ms.openlocfilehash: 9e683abfc9296211547647f47a46b82ed34d823f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462139"
---
# <a name="tutorial-azure-active-directory-integration-with-sprinklr"></a>Tutorial: Integración de Azure Active Directory con Sprinklr

En este tutorial, obtendrá información sobre cómo integrar Sprinklr con Azure Active Directory (Azure AD). Al integrar Sprinklr con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Sprinklr.
* Permitir que los usuarios inicien sesión automáticamente en Sprinklr con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Sprinklr.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Sprinklr admite el SSO iniciado por **SP**.

## <a name="add-sprinklr-from-the-gallery"></a>Incorporación de Sprinklr desde la galería

Para configurar la integración de Sprinklr en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Sprinklr** en el cuadro de búsqueda.
1. Seleccione **Sprinklr** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-sprinklr"></a>Configuración y prueba del SSO de Azure AD para Sprinklr

Configure y pruebe el SSO de Azure AD con Sprinklr mediante un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Sprinklr.

Para configurar y probar el SSO de Azure AD con Sprinklr, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del SSO en Sprinklr](#configure-sprinklr-sso)** : para definir los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Sprinklr](#create-sprinklr-test-user)** : para tener un homólogo de B.Simon en Sprinklr vinculado a la representación del usuario de Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Sprinklr**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.sprinklr.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.sprinklr.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Sprinklr](https://www.sprinklr.com/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Sprinklr** (Configurar Sprinklr), copie las direcciones URL adecuada según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Sprinklr mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Sprinklr**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sprinklr-sso"></a>Configuración del SSO en Sprinklr

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de Sprinklr como administrador.

1. Vaya a **Administración \> Configuración**.

    ![Administración](./media/sprinklr-tutorial/settings.png "Administración")

1. Vaya a **Administrar socios \> Inicio de sesión único** en el panel de la izquierda.

    ![Administrar asociado](./media/sprinklr-tutorial/users.png "Administrar socio")

1. Haga clic en **+Add Single Sign On**(+ Agregar inicios de sesión únicos).

    ![Captura de pantalla que muestra el botón para agregar inicios de sesión únicos.](./media/sprinklr-tutorial/add-user.png "Inicios de sesión únicos")

1. Siga estos pasos en la página **Inicio de sesión único** :

    ![Captura de pantalla que muestra la página de inicio de sesión único, donde puede escribir los valores descritos.](./media/sprinklr-tutorial/configuration.png "Inicios de sesión únicos")

    a. En el cuadro de texto **Name** (Nombre), escriba el nombre de la configuración (por ejemplo, **WAADSSOTest**).

    b. Seleccione **Habilitado**.

    c. Seleccione **Use new SSO Certificate**(Usar el nuevo certificado de SSO).

    d. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de proveedor de identidades**.

    e. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    f. En el cuadro de texto **URL de inicio de sesión del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    g. En el cuadro de texto **URL de cierre de sesión del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    h. Como **Tipo de Id. de usuario de SAML**, seleccione **La aserción contiene el nombre de usuario de sprinklr.com del usuario**.

    i. Para **Ubicación de Id. de usuario de SAML**, seleccione **El Id. de usuario está en el elemento NameIdentifier de la instrucción Subject**.

    j. Haga clic en **Save**(Guardar).

    ![SAML](./media/sprinklr-tutorial/save-configuration.png "SAML")

### <a name="create-sprinklr-test-user"></a>Creación de un usuario de prueba de Sprinklr

1. Inicie sesión en su sitio de la compañía de Sprinklr como administrador.

1. Vaya a **Administración \> Configuración**.

    ![Administración](./media/sprinklr-tutorial/settings.png "Administración")

1. Vaya a **Administrar clientes \> Usuarios** en el panel de la izquierda.

    ![Captura de pantalla que muestra el botón para agregar usuario en la opción de configuración, usuarios.](./media/sprinklr-tutorial/client.png "Configuración")

1. Haga clic en **Agregar usuario**.

    ![Captura de pantalla que muestra el cuadro de diálogo para editar usuario, donde puede especificar los valores descritos.](./media/sprinklr-tutorial/search-users.png "Configuración")

1. En el cuadro de diálogo **Edit user** (Editar usuario), realice los siguientes pasos:

    ![Editar usuario](./media/sprinklr-tutorial/update-users.png "Edit user")

    a. En los cuadros de texto **Correo electrónico**, **Nombre** y **Apellido**, escriba la información de una cuenta de usuario de Azure AD que desee aprovisionar.

    b. Seleccione **Password Disabled**(Contraseña deshabilitada).

    c. Seleccione **Language** (Lenguaje).

    d. Seleccione **User Type**(Tipo de usuario).

    e. Haga clic en **Update**(Actualizar).

    > [!IMPORTANT]
    > **Password Disabled** (Contraseña deshabilitada) debe estar seleccionada para que los usuarios puedan iniciar sesión a través de un proveedor de identidades. 

1. Vaya a **Role**(Rol) y luego lleve a cabo los siguientes pasos:

    ![Roles de asociados](./media/sprinklr-tutorial/role.png "Roles de socios")

    a. En la lista **Global**, seleccione **ALL_Permissions** (Todos los permisos).  

    b. Haga clic en **Update**(Actualizar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Sprinklr ofrecida por Sprinklr para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Sprinklr, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Sprinklr e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Sprinklr en Aplicaciones, se le redirigirá a la URL de inicio de sesión de esa plataforma. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Sprinklr, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
