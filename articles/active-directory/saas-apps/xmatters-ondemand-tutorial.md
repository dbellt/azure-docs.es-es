---
title: 'Tutorial: Integración de Azure Active Directory con xMatters OnDemand | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y xMatters OnDemand.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/07/2021
ms.author: jeedes
ms.openlocfilehash: f6a1fe629d3a73245570dddba1b983b7c0d62612
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111748902"
---
# <a name="tutorial-azure-active-directory-integration-with-xmatters-ondemand"></a>Tutorial: Integración de Azure Active Directory con xMatters OnDemand

En este tutorial, aprenderá a integrar xMatters OnDemand con Azure Active Directory (Azure AD). Al integrar xMatters OnDemand con Azure AD, puede realiza las siguientes operaciones:

* Controlar en Azure AD quién tiene acceso a xMatters OnDemand.
* Permitir a los usuarios iniciar sesión automáticamente en xMatters OnDemand con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con xMatters OnDemand, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único en xMatters OnDemand.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* xMatters OnDemand admite el inicio de sesión único iniciado por **IDP**.

## <a name="add-xmatters-ondemand-from-the-gallery"></a>Adición de xMatters OnDemand desde la galería

Para configurar la integración de xMatters OnDemand en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **xMatters OnDemand** en el cuadro de búsqueda.
1. Seleccione **xMatters OnDemand** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-xmatters-ondemand"></a>Configuración y prueba del inicio de sesión único de Azure AD para xMatters OnDemand

Configure y pruebe el inicio de sesión único de Azure AD con xMatters OnDemand mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de xMatters OnDemand.

Para configurar y probar el inicio de sesión único de Azure AD con xMatters OnDemand, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    2. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de xMatters OnDemand](#configure-xmatters-ondemand-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de xMatters OnDemand](#create-xmatters-ondemand-test-user)**: para tener un homólogo de Britta Simon en xMatters OnDemand que esté vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **xMatters OnDemand**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    | Identificador |
    | ---------- |
    | `https://<COMPANY_NAME>.au1.xmatters.com.au/` |
    | `https://<COMPANY_NAME>.cs1.xmatters.com/` |
    | `https://<COMPANY_NAME>.xmatters.com/` |
    | `https://www.xmatters.com` |
    | `https://<COMPANY_NAME>.xmatters.com.au/` |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:

    | URL de respuesta |
    | ---------- |
    | `https://<COMPANY_NAME>.au1.xmatters.com.au` |
    | `https://<COMPANY_NAME>.xmatters.com/sp/<INSTANCE_NAME>` |
    | `https://<COMPANY_NAME>.cs1.xmatters.com/sp/<INSTANCE_NAME>` |
    | `https://<COMPANY_NAME>.au1.xmatters.com.au/<INSTANCE_NAME>` |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte al cliente de xMatters OnDemand](https://www.xmatters.com/company/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

    > [!IMPORTANT]
    > Debe reenviar el certificado al [equipo de soporte técnico de xMatters OnDemand](https://www.xmatters.com/company/contact-us/). El equipo de soporte técnico de xMatters debe cargar el certificado para que se pueda finalizar la configuración del inicio de sesión único.

6. En la sección **Configurar xMatters OnDemand**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a xMatters OnDemand mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **xMatters OnDemand**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-xmatters-ondemand-sso"></a>Configuración del inicio de sesión único de xMatters OnDemand

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de xMatters OnDemand como administrador.

2. Haga clic en **Admin** (Administrador) y, a continuación, haga clic en **Company Details** (Datos de la empresa).

    ![Página de administración](./media/xmatters-ondemand-tutorial/admin.png "Página de administración")

3. En la página **Configuración de SAML** , realice los siguientes pasos:

    ![Sección de configuración de SAML](./media/xmatters-ondemand-tutorial/saml-configuration.png "Sección de configuración de SAML")

    a. Seleccione **Habilitar SAML**.

    b. En el cuadro de texto **Id. de proveedor de identidad**, pegue el valor del **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. En el cuadro de texto **Single Sign On URL** (Dirección URL de inicio de sesión único), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Logout URL Redirect** (Redirección URL de cierre de sesión), pegue la **dirección URL de cierre de sesión** que ha copiado de Azure Portal.

    e. Haga clic en **Choose File** (Elegir archivo) para cargar el **Certificado (Base 64)** que ha descargado de Azure Portal. 

    f. En la página Detalles de la empresa, en la parte superior, haga clic en **Guardar cambios**.

    ![Detalles de la compañía](./media/xmatters-ondemand-tutorial/save-button.png "Detalles de la compañía")

### <a name="create-xmatters-ondemand-test-user"></a>Creación de un usuario de prueba de xMatters OnDemand

1. Inicie sesión en su inquilino de **xMatters OnDemand**.

2. Vaya al **icono de usuarios** > **Users** (Usuarios) y, a continuación, haga clic en **Add Users** (Agregar usuarios).

    ![Usuarios](./media/xmatters-ondemand-tutorial/add-user.png "Usuarios")

3. En la sección **Add Users** (Agregar usuarios), rellene los campos obligatorios y haga clic en el botón **Add User** (Agregar usuario).

    ![Adición de un usuario](./media/xmatters-ondemand-tutorial/add-user-2.png "Agregar un usuario")

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de xMatters OnDemand para la que ha configurado el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de xMatters OnDemand en Aplicaciones, debería iniciar sesión automáticamente en la instancia de xMatters OnDemand para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado xMatters OnDemand, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).