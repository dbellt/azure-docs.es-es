---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BlueJeans for Azure AD | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y BlueJeans for Azure AD.
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
ms.openlocfilehash: 273d82c5d9427dd7717b567c04b8c18141e6eee2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063782"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-bluejeans-for-azure-ad"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con BlueJeans for Azure AD

En este tutorial aprenderá a integrar BlueJeans for Azure AD con Azure Active Directory (Azure AD). Al integrar BlueJeans for Azure AD con Azure AD, puede:

* Controlar desde Azure AD quién tiene acceso a BlueJeans for Azure AD.
* Permitir que los usuarios inicien sesión automáticamente en BlueJeans for Azure AD con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en BlueJeans for Azure AD.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* BlueJeans for Azure AD admite el inicio de sesión único iniciado por **SP**.

* BlueJeans for Azure AD admite el [aprovisionamiento **automatizado** de usuarios](bluejeans-provisioning-tutorial.md).

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-bluejeans-for-azure-ad-from-the-gallery"></a>Incorporación de BlueJeans for Azure AD desde la galería

Para configurar la integración de BlueJeans for Azure AD en Azure AD, es preciso que agregue BlueJeans for Azure AD desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **BlueJeans for Azure AD** en el cuadro de búsqueda.
1. Seleccione **BlueJeans for Azure AD** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-bluejeans-for-azure-ad"></a>Configuración y prueba del inicio de sesión único de Azure AD para BlueJeans for Azure AD

Configure y pruebe el inicio de sesión único de Azure AD con BlueJeans for Azure AD utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de BlueJeans for Azure AD.

Para configurar y probar el inicio de sesión único de Azure AD con BlueJeans for Azure AD, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en BlueJeans for Azure AD](#configure-bluejeans-for-azure-ad-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en BlueJeans for Azure AD](#create-bluejeans-for-azure-ad-test-user)** : para tener un homólogo de B.Simon en BlueJeans for Azure AD vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **BlueJeans for Azure AD**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.bluejeans.com`

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `http://samlsp.bluejeans.com`.

    a. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://bluejeans.com/sso/saml2/`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de BlueJeans for Azure AD](https://support.bluejeans.com/contact) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación BlueJeans espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación BlueJeans espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre |  Atributo de origen|
    | ---------| --------- |
    | Teléfono | user.telephonenumber |
    | title | user.jobtitle |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up BlueJeans for Azure AD** (Configurar BlueJeans for Azure AD), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a BlueJeans for Azure AD mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **BlueJeans for Azure AD**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-bluejeans-for-azure-ad-sso"></a>Configuración del inicio de sesión único en BlueJeans for Azure AD

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la empresa de **BlueJeans for Azure AD**.

2. Vaya a **ADMIN \> CONFIGURACIÓN DE GRUPO \> SEGURIDAD**.

    ![Captura de pantalla que muestra parte de una ventana del explorador con la pestaña Admin (Administrador) seleccionada, así como las opciones Group Settings (Configuración de grupo) y Security (Seguridad).](./media/bluejeans-tutorial/admin.png "Administración")

3. En la sección **SEGURIDAD**, realice estos pasos:

    ![SAML Single Sign On](./media/bluejeans-tutorial/security.png "SAML Single Sign On") (Inicio de sesión único de SAML)

    a. Seleccione **SAML Single Sign On**(Inicio de sesión único de SAML).

    b. Seleccione **Enable automatic provisioning**(Habilitar aprovisionamiento automático).

4. Continúe con los siguiente pasos:

    ![Ruta de acceso del certificado](./media/bluejeans-tutorial/certificate.png "Ruta de acceso del certificado")

    a. Haga clic en **Elegir archivo** para cargar el certificado codificado en base 64 que descargó de Azure Portal.

    b. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Password Change URL** (Dirección URL de cambio de contraseña), pegue el valor de **Cambiar dirección URL de contraseña** que copió de Azure Portal.

    d. En el cuadro de texto **URL de cierre de sesión**, pegue el valor de **Sign-Out URL** (Dirección URL de cierre de sesión) que copió de Azure Portal.

5. Continúe con los siguiente pasos:

    ![Save Changes](./media/bluejeans-tutorial/changes.png "Guardar cambios")

    a. En el cuadro de texto **Id. de usuario**, escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    b. En el cuadro de texto **Correo electrónico**, escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    c. Haga clic en **GURDAR CAMBIOS**.

### <a name="create-bluejeans-for-azure-ad-test-user"></a>Creación de un usuario de prueba en BlueJeans for Azure AD

El objetivo de esta sección es crear un usuario llamado B.Simon en BlueJeans for Azure AD. BlueJeans for Azure AD admite el aprovisionamiento automático de usuarios, habilitado de forma predeterminada. [Aquí](bluejeans-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. Inicie sesión como administrador en el sitio de la empresa de **BlueJeans for Azure AD**.

2. Vaya a **ADMIN \> ADMINISTRAR USUARIOS \> AGREGAR USUARIO**.

    ![Captura de pantalla que muestra parte de una ventana del explorador con la pestaña Admin (Administrador) seleccionada, así como las opciones Manage Users (Administrar usuarios) y Add Users (Agregar usuarios).](./media/bluejeans-tutorial/add-user.png "Administración")

    > [!IMPORTANT]
    > La pestaña **AGREGAR USUARIO** está disponible solo si en la pestaña **SEGURIDAD**, está desactivada la opción **Habilitar aprovisionamiento automático**.

3. En la sección **AGREGAR USUARIO**, lleve a cabo estos pasos:

    ![Captura de pantalla que muestra la sección Add user (Agregar usuario) en la que se especifica la información que se describe en este paso.](./media/bluejeans-tutorial/new-user.png "Agregar usuario")

    a. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    b. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

    c. En el cuadro de texto **Pick a BlueJeans for Azure AD Username** (Elegir un nombre de usuario de BlueJeans for Azure AD), escriba el nombre de usuario del usuario, como **Brittasimon**

    d. En el cuadro de texto **Crear una contraseña**, escriba su contraseña.

    e. En el cuadro de texto **Compañía**, escriba su compañía.

    f. En el cuadro de texto **Dirección de correo electrónico**, escriba el correo electrónico del usuario; por ejemplo, `b.simon@contoso.com`.

    g. En el cuadro de texto **Create a BlueJeans for Azure AD Meeting I.D** (Crear un id. de reunión de BlueJeans for Azure AD), escriba el identificador de la reunión.

    h. En el cuadro de texto **Pick a Moderator Passcode** (Elegir un código de acceso de moderador), escriba el código de acceso.

    i. Haga clic en **CONTINUE** (Continuar).

    ![Captura de pantalla que muestra la sección Add user (Agregar usuario) con la configuración y las características, y el botón Add user (Agregar usuario) seleccionado.](./media/bluejeans-tutorial/settings.png "Agregar usuario")

    J. Haga clic en **ADD USER** (Agregar usuario).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de BlueJeans for Azure AD que proporcione BlueJeans for Azure AD para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de BlueJeans for Azure AD, donde puede comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de BlueJeans for Azure AD y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de BlueJeans for Azure AD en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de BlueJeans for Azure AD. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado BlueJeans for Azure AD, podrá aplicar el control de sesión, que protege la información confidencial de su organización de la exfiltración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
