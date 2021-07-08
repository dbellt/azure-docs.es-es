---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Spotinst | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Spotinst.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2021
ms.author: jeedes
ms.openlocfilehash: 5c7944cbc16ca7e9210255b8c05c90a15d406739
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111982895"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-spotinst"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Spotinst

En este tutorial, aprenderá a integrar Spotinst con Azure Active Directory (Azure AD). Al integrar Spotinst con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Spotinst.
* Permitir que los usuarios inicien sesión automáticamente en Spotinst con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Spotinst.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Spotinst admite el inicio de sesión único iniciado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-spotinst-from-the-gallery"></a>Incorporación de Spotinst desde la galería

Para configurar la integración de Spotinst en Azure AD, es preciso agregar Spotinst desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Spotinst** en el cuadro de búsqueda.
1. Seleccione **Spotinst** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-spotinst"></a>Configuración y prueba del inicio de sesión único de Azure AD para Spotinst

Configure y pruebe el inicio de sesión único de Azure AD con Spotinst mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Spotinst.

Para configurar y probar el inicio de sesión único de Azure AD con Spotinst, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Spotinst](#configure-spotinst-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Spotinst](#create-spotinst-test-user)** , para tener un homólogo de B.Simon en Spotinst vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Spotinst**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por IDP, haga lo siguiente:

   1. Asegúrese de que la **URL de respuesta** esté establecida en https://console.spotinst.com/auth/saml.
   1. En **Estado de la retransmisión**, escriba el identificador de la organización de Spotinst, que también puede confirmar en la pestaña **SSO**.
   1. La **dirección URL de inicio de sesión** debe estar vacía.

1. Haga clic en **Save**(Guardar).

1. La aplicación Spotinst espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Spotinst espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | -----| --------------- |
    | Email | user.mail |
    | Nombre | user.givenname |
    | Apellidos | user.surname |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Spotinst**, copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Spotinst mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Spotinst**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-spotinst-sso"></a>Configuración del inicio de sesión único de Spotinst

1. En otra ventana del explorador web, inicie sesión en Spotinst como administrador de seguridad.

2. Haga clic en el **icono de usuario** en la parte superior derecha de la pantalla y haga clic en **Configuración**.

    ![Captura de pantalla que muestra la configuración seleccionada en el icono de usuario.](./media/spotinst-tutorial/settings.png)

3. Haga clic en la pestaña **Seguridad** en la parte superior y, a continuación, seleccione **Proveedores de identidades** y realice los pasos siguientes:

    ![Seguridad de Spotinst](./media/spotinst-tutorial/security.png)

    a. Copie el valor de **Estado de la retransmisión** de su instancia y péguelo en el cuadro de texto **Estado de la retransmisión** de la sección **Configuración básica de SAML** de Azure Portal.

    b. Haga clic en **EXAMINAR** para cargar el archivo XML de metadatos que descargó de Azure Portal.

    c. Haga clic en **GUARDAR**.

### <a name="create-spotinst-test-user"></a>Creación de un usuario de prueba de Spotinst

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en Spotinst.

1. Si ha configurado la aplicación en modo iniciado por **SP**, realice los siguientes pasos:

   a. En otra ventana del explorador web, inicie sesión en Spotinst como administrador de seguridad.

   b. Haga clic en el **icono de usuario** en la parte superior derecha de la pantalla y haga clic en **Configuración**.

    ![Captura de pantalla que muestra la configuración seleccionada en el icono de usuario.](./media/spotinst-tutorial/settings.png)

    c. Haga clic en **Usuarios** y seleccione **Agregar usuario**.

    ![Captura de pantalla que muestra la opción para agregar usuario seleccionada en Usuarios.](./media/spotinst-tutorial/add-user.png)

    d. En la sección Agregar usuario, realice los pasos siguientes:

    ![Captura de pantalla que muestra la sección para agregar usuario, donde puede especificar los valores descritos.](./media/spotinst-tutorial/new-user.png)

    1. En el cuadro de texto **Nombre completo**, escriba el nombre completo del usuario, por ejemplo, `BrittaSimon`.

    1. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico del usuario, en este caso, `brittasimon@contoso.com`.

    1. Seleccione los detalles específicos de su organización para el **rol de la organización, el rol de cuenta y las cuentas**.

2. Si ha configurado la aplicación en el modo iniciado por **IDP**, no debe realizar ninguna acción en esta sección. Spotinst admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. Al intentar acceder a Spotinst, se crea un nuevo usuario, en caso de que no exista.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Spotinst, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Spotinst y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal e iniciará sesión automáticamente en la instancia de Spotinst para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Spotinst en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Spotinst para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Spotinst, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
