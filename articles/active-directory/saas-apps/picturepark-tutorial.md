---
title: 'Tutorial: Integración de Azure Active Directory con Picturepark | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Picturepark.
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
ms.openlocfilehash: 066b534476bd3b6b49059ab6db764b33d3d39f17
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111571095"
---
# <a name="tutorial-azure-active-directory-integration-with-picturepark"></a>Tutorial: Integración de Azure Active Directory con Picturepark

En este tutorial, obtendrá información sobre cómo integrar Picturepark con Azure Active Directory (Azure AD). La integración de Picturepark con Azure AD permite:

* Controlar en Azure AD quién tiene acceso a Picturepark.
* Habilitar que los usuarios inicien sesión automáticamente en Picturepark con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Picturepark, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Picturepark.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Picturepark admite el inicio de sesión único iniciado por **SP**.

## <a name="add-picturepark-from-the-gallery"></a>Incorporación de Picturepark desde la galería

Para configurar la integración de Picturepark en Azure AD, será preciso que agregue Picturepark desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Picturepark** en el cuadro de búsqueda.
1. Seleccione **Picturepark** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-picturepark"></a>Configuración y prueba del inicio de sesión único de Azure AD en Picturepark

Configure y pruebe el inicio de sesión único de Azure AD con Picturepark mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Picturepark.

Para configurar y probar el inicio de sesión único de Azure AD con Picturepark, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Picturepark](#configure-picturepark-sso)** : para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Picturepark](#create-picturepark-test-user)** : el objetivo es tener un homólogo de B. Simon en Picturepark que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Picturepark**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con uno de los patrones siguientes:

    | URL del identificador |
    |---|
    |`https://<COMPANY_NAME>.current-picturepark.com`|
    |`https://<COMPANY_NAME>.picturepark.com`|
    |`https://<COMPANY_NAME>.next-picturepark.com`|
    |
    
    b. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY_NAME>.picturepark.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador y Dirección URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de cliente de Picturepark](https://picturepark.com/company/picturepark-customer-support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

    ![Edición del certificado de firma de SAML](common/edit-certificate.png)

6. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** en el equipo.

    ![Copia del valor de la huella digital](common/copy-thumbprint.png)

7. En la sección **Configurar Picturepark**, copie las direcciones URL que necesite. Para **Dirección URL de inicio de sesión**, use un valor con el siguiente patrón: `https://login.microsoftonline.com/_my_directory_id_/wsfed`

    > [!Note]
    > _my_directory_id_ es el identificador de inquilino de la suscripción de Azure AD.

    ![Copiar direcciones URL de configuración](./media/picturepark-tutorial/configure.png)

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

En esta sección, va a permitir que B.Simon acceda a Picturepark mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Picturepark**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-picturepark-sso"></a>Configuración del inicio de sesión único en Picturepark

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía Picturepark como administrador.

2. En la barra de herramientas en la parte superior, haga clic en **Herramientas administrativas** y, a continuación, haga clic en **Consola de administración**.
   
    ![Management Console (Consola de administración)](./media/picturepark-tutorial/tools.png "Consola de administración")

3. Haga clic en **Autenticación** y, a continuación, haga clic en **Proveedores de identidad**.
   
    ![Autenticación](./media/picturepark-tutorial/identity-provider.png "Authentication")

4. En la sección **Configuración del proveedor de identidades**, siga estos pasos:
   
    ![Identity provider configuration (Configuración del proveedor de identidades)](./media/picturepark-tutorial/add-configuration.png "Configuración del proveedor de identidades")
   
    a. Haga clic en **Agregar**.
  
    b. Escriba un nombre para su configuración.
   
    c. Seleccione **Establecer como predeterminado**.
   
    d. En el cuadro de texto **Issuer URI**, (Identificador URI del emisor) pegue el valor de **Dirección URL de inicio de sesión**, que ha copiado de Azure Portal.
   
    e. En el cuadro de texto **Trusted Issuer Thumb Print** (Huella digital del emisor de confianza), pegue el valor de **huella digital** que copió de la sección **Certificado de firma SAML**. 

5. Haga clic en **JoinDefaultUsersGroup**.

6. Para establecer el atributo **Emailaddress** en el cuadro de texto **Notificación**, escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` y haga clic en **Guardar**.

      ![Configuración](./media/picturepark-tutorial/claim.png "Configuración")

### <a name="create-picturepark-test-user"></a>Creación de un usuario de prueba de Picturepark

Para permitir que los usuarios de Azure AD inicien sesión en Picturepark, deben aprovisionarse en Picturepark. En el caso de Picturepark, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en el inquilino de **Picturepark**.

1. En la barra de herramientas en la parte superior, haga clic en **Herramientas administrativas** y, a continuación, haga clic en **Usuarios**.
   
    ![Usuarios](./media/picturepark-tutorial/user.png "Usuarios")

1. En la pestaña **Usuarios**, haga clic en **Nuevo**.
   
    ![Administración de usuarios](./media/picturepark-tutorial/new-user.png "Administración de usuarios")

1. En el cuadro de diálogo **Crear usuario**, siga estos pasos para un usuario válido de Azure Active Directory que desea aprovisionar:
   
    ![Crear usuario](./media/picturepark-tutorial/details.png "Crear usuario")
   
    a. En el cuadro de texto **Dirección de correo electrónico**, escriba la **dirección de correo electrónico** del usuario `BrittaSimon@contoso.com`.  
   
    b. En los cuadros de texto **Contraseña** y **Confirmar contraseña**, escriba la **contraseña** de BrittaSimon. 
   
    c. En el cuadro de texto **Nombre**, escriba el **nombre** de la usuaria **Britta**. 
   
    d. En el cuadro de texto **Apellido**, escriba el **apellido** de la usuaria **Simon**.
   
    e. En el cuadro de texto **Empresa**, escriba el **nombre de la empresa** del usuario. 
   
    f. En el cuadro de texto **País**, seleccione el **país o región** del usuario.
  
    g. En el cuadro de texto **ZIP** (Código postal), escriba el **código postal** de la ciudad.
   
    h. En el cuadro de texto **Ciudad**, escriba el **nombre de la ciudad** del usuario.

    i. Seleccione un **idioma**.
   
    j. Haga clic en **Crear**.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Picturepark ofrecida por Picturepark para aprovisionar cuentas de usuario de Azure AD.
>

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Picturepark, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Picturepark y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Picturepark en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Picturepark. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Tras configurar Picturepark, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
