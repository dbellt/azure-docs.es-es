---
title: 'Tutorial: Integración de Azure Active Directory con LoginRadius | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y LoginRadius.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/27/2021
ms.author: jeedes
ms.openlocfilehash: f4c0ab8cb09839d208a1508730d9439a0c22ce60
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111555729"
---
# <a name="tutorial-azure-active-directory-integration-with-loginradius"></a>Tutorial: Integración de Azure Active Directory con LoginRadius

En este tutorial, aprenderá a integrar LoginRadius con Azure Active Directory (Azure AD). Al integrar LoginRadius con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a LoginRadius.
* Permitir que los usuarios inicien sesión automáticamente en LoginRadius con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con LoginRadius, se necesitan los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en LoginRadius.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* LoginRadius admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-loginradius-from-the-gallery"></a>Incorporación de LoginRadius desde la galería

Para configurar la integración de LoginRadius con Azure AD, será preciso que agregue LoginRadius desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **LoginRadius** en el cuadro de búsqueda.
1. Seleccione **LoginRadius** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-loginradius"></a>Configuración y prueba del inicio de sesión único de Azure AD para LoginRadius

Configure y pruebe el inicio de sesión único de Azure AD con LoginRadius mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de LoginRadius.

Para configurar y probar el inicio de sesión único de Azure AD con LoginRadius, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en LoginRadius](#configure-loginradius-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de LoginRadius](#create-loginradius-test-user)** : para tener un homólogo de B.Simon en LoginRadius que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **LoginRadius**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

   1. En el cuadro de texto **Identificador (Id. de entidad)** , escriba la dirección URL: `https://lr.hub.loginradius.com/`

   1. En el cuadro de texto **Dirección URL de respuesta (URL del Servicio de consumidor de aserciones)** , escriba la URL de ACS de LoginRadius: `https://lr.hub.loginradius.com/saml/serviceprovider/AdfsACS.aspx` 

   1. En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://secure.loginradius.com/login`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdalo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up LoginRadius** (Configurar LoginRadius), copie las direcciones URL apropiadas en función de sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a LoginRadius utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **LoginRadius**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-loginradius-sso"></a>Configuración del inicio de sesión único de LoginRadius

En esta sección, habilitará el inicio de sesión único de Azure AD en la consola de administración de LoginRadius.

1. Inicie sesión en su cuenta de la [consola de administración](https://adminconsole.loginradius.com/login) de LoginRadius.

2. Vaya a la sección **Team Management** (Administración del equipo) en la [consola de administración de LoginRadius](https://secure.loginradius.com/account/team).

3. Seleccione la pestaña **Inicio de sesión único** y, a continuación, **Azure AD**:

   ![Captura de pantalla que muestra el menú de inicio de sesión único en la consola de administración del equipo de LoginRadius.](./media/loginradius-tutorial/azure-ad.png)

4. En la página de configuración de Azure AD, complete los pasos siguientes:

   ![Captura de pantalla que muestra la configuración de Azure Active Directory en la consola de administración del equipo de LoginRadius.](./media/loginradius-tutorial/single-sign-on.png)

    1. En **ID Provider Location** (Ubicación del proveedor de identidades), escriba el punto de conexión de inicio de sesión que obtenga de la cuenta de Azure AD.

    1. En **ID Provider Logout URL** (Dirección URL de cierre de sesión del proveedor de identidades), escriba el punto de conexión de cierre de sesión que obtenga de la cuenta de Azure AD.
 
    1. En **Certificado del proveedor de identidades**, escriba el certificado de Azure AD que obtenga de la cuenta de Azure AD. Escriba el valor del certificado con el encabezado y el pie de página. Ejemplo: `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`

    1. En **Service Provider Certificate** (Certificado del proveedor de servicios) y **Server Provider Certificate Key** (Clave de certificado del proveedor del servidor), escriba el certificado y la clave. 

       Puede crear un certificado autofirmado mediante la ejecución de los siguientes comandos en la línea de comandos (Linux/Mac):

       - Comando para obtener la clave del certificado de SP: `openssl genrsa -out lr.hub.loginradius.com.key 2048`

       - Comando para obtener el certificado de SP: `openssl req -new -x509 -key lr.hub.loginradius.com.key -out lr.hub.loginradius.com.cert -days 3650 -subj /CN=lr.hub.loginradius.com`

       > [!NOTE]
       > Asegúrese de especificar los valores de certificado y de clave del certificado con el encabezado y el pie de página:
       > - Formato de ejemplo del valor de certificado: `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`
       > - Formato de ejemplo del valor de clave de certificado: `-----BEGIN RSA PRIVATE KEY-----<certifciate key value>-----END RSA PRIVATE KEY-----`

5. En la sección **Asignación de datos**, seleccione los campos (campos SP) y especifique los campos correspondientes (campos IdP) de Azure AD.

    A continuación, se enumeran algunos nombres de campo para Azure AD.

    | Fields    | Clave de perfil                                                          |
    | --------- | -------------------------------------------------------------------- |
    | Email     | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
    | Nombre | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`    |
    | Apellidos  | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`      |

    > [!NOTE]
    > La asignación de campos **Email** es obligatoria. Las asignaciones de campos **FirstName** y **LastName** son opcionales.

### <a name="create-loginradius-test-user"></a>Creación de un usuario de prueba de LoginRadius

1. Inicie sesión en su cuenta de la [consola de administración](https://adminconsole.loginradius.com/login) de LoginRadius.

2. Vaya a la sección de administración del equipo en la consola de administración de LoginRadius.

   ![Captura de pantalla que muestra la consola de administración de LoginRadius.](./media/loginradius-tutorial/team-management.png)

3. Seleccione **Agregar un miembro del equipo** en el menú lateral para abrir el formulario. 

4. En el formulario **Agregar un miembro del equipo**, creará un usuario llamado Britta Simon en el sitio de LoginRadius al proporcionar los detalles del usuario y asignar los permisos que desea para el usuario. Para obtener más información sobre los permisos basados en roles, consulte la sección de [permisos de acceso de rol](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0) del documento de [administración de miembros del equipo](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección probará la configuración de inicio de sesión único de Azure AD mediante Aplicaciones.

1. En un explorador, vaya a https://accounts.loginradius.com/auth.aspx y seleccione **Fed SSO log in** (Inicio de sesión único Fed).
2. Escriba el nombre de la aplicación LoginRadius y, a continuación, seleccione **inicio de sesión**.
3. Debería abrirse una ventana emergente para solicitarle que inicie sesión en su cuenta de Azure AD.
4. Después de la autenticación, se cerrará la ventana emergente y se iniciará sesión en la consola de administración de LoginRadius.

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado LoginRadius, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
