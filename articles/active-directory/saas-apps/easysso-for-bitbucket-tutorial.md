---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EasySSO for BitBucket | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y EasySSO for BitBucket.
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
ms.openlocfilehash: 56f2eea9dc485c69b6070fda6e9519887f7b30cb
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107519663"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-easysso-for-bitbucket"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con EasySSO for BitBucket

En este tutorial aprenderá a integrar EasySSO for BitBucket con Azure Active Directory (Azure AD). Al integrar EasySSO for BitBucket con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a EasySSO for BitBucket.
* Permitir que los usuarios inicien sesión automáticamente en EasySSO for BitBucket con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción a EasySSO for BitBucket habilitada para el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* EasySSO for BitBucket admite el inicio de sesión único iniciado por SP e IdP.
* EasySSO for BitBucket admite el aprovisionamiento de usuarios "Just-In-Time".

## <a name="add-easysso-for-bitbucket-from-the-gallery"></a>Incorporación de EasySSO for BitBucket desde la galería

Para configurar la integración de EasySSO for BitBucket en Azure AD, debe agregar EasySSO for BitBucket desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **EasySSO for BitBucket** en el cuadro de búsqueda.
1. Seleccione **EasySSO for BitBucket** en los resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-easysso-for-bitbucket"></a>Configuración y prueba del inicio de sesión único de Azure AD para EasySSO for BitBucket

Configure y pruebe el inicio de sesión único de Azure AD con EasySSO for BitBucket mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de EasySSO for BitBucket.

Para configurar y probar el inicio de sesión único de Azure AD con EasySSO for BitBucket, complete los siguientes pasos:

1. [Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso), para permitir que los usuarios puedan utilizar esta característica.
    1. [Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user), para probar el inicio de sesión único de Azure AD con B. Simon.
    1. [Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user), para que B. Simon pueda usar el inicio de sesión único de Azure AD.
1. [Configuración del inicio de sesión único en EasySSO for BitBucket](#configure-easysso-for-bitbucket-sso), para configurar los valores de inicio de sesión único en la aplicación.
    1. [Creación de un usuario de prueba de EasySSO for BitBucket](#create-an-easysso-for-bitbucket-test-user), para tener un homólogo de B.Simon en EasySSO for BitBucket vinculado a la representación del usuario en Azure AD.
1. [Comprobación del inicio de sesión único](#test-sso), para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **EasySSO for BitBucket**, busque la sección **Administrar**. Seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, seleccione el icono con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Captura de pantalla de la página Configurar el inicio de sesión único con SAML con el icono de lápiz resaltado](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IdP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/easysso/saml`.

    b. En el cuadro de texto **Dirección URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/plugins/servlet/easysso/saml`.

1. Seleccione **Establecer direcciones URL adicionales** y lleve a cabo el siguiente paso si desea configurar la aplicación en el modo iniciado por **SP**:

    - En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<server-base-url>/login.jsp`.

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de EasySSO](mailto:support@techtime.co.nz) para obtener estos valores en caso de duda. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación EasySSO for BitBucket espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![Captura de pantalla de los atributos predeterminados](common/default-attributes.png)

1. La aplicación EasySSO for BitBucket también espera que se devuelvan algunos atributos más en la respuesta de SAML. Estos atributos se muestran en la tabla siguiente. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | ---------------| --------- |
    | urn:oid:0.9.2342.19200300.100.1.1 | user.userprincipalname |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:2.16.840.1.113730.3.1.241 | user.displayname |
    | urn:oid:2.5.4.4 | user.surname |
    | urn:oid:2.5.4.42 | user.givenname |
    
    Si los usuarios de Azure AD tienen **sAMAccountName** configurado, tiene que asignar **urn:oid:0.9.2342.19200300.100.1.1** en el atributo **sAMAccountName**.
    
1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en los vínculos de descarga de las opciones **Certificado (Base64)** o **XML de metadatos de federación**. Guarde cualquiera de ellos o ambos en el equipo. Los necesitará más adelante para configurar BitBucket EasySSO.

    ![Captura de pantalla de la sección Certificado de firma de SAML, con los vínculos de descarga resaltados](./media/easysso-for-bitbucket-tutorial/certificate.png)
    
    Si va a configurar EasySSO for BitBucket manualmente con un certificado, también necesitará copiar los valores de **Dirección URL de inicio de sesión** e **Identificador de Azure AD** y guardarlos en el equipo.

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, creará un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En **Nombre**, escriba `B.Simon`.
   1. En **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione la casilla **Mostrar contraseña** y, a continuación, anote la contraseña.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a EasySSO for BitBucket mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **EasySSO for BitBucket**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. En el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** en la lista **Usuarios** y, después, elija **Seleccionar** en la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, seleccione **Asignar**.

## <a name="configure-easysso-for-bitbucket-sso"></a>Configuración del inicio de sesión único en EasySSO for BitBucket

1. Para automatizar la configuración en Zoom, debe instalar la **extensión del navegador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Set up Zoom** (Configurar Zoom) para ir a la aplicación del mismo nombre. Ahí, escriba las credenciales de administrador para iniciar sesión en Zoom. La extensión del navegador configurará automáticamente la aplicación y automatizará los pasos del 3 al 10.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Zoom manualmente, en otra ventana del explorador web, inicie sesión en su sitio de la compañía de Zoom como administrador.

1. Acceda a la sección **Administración**.

    ![Captura de pantalla de la instancia de BitBucket, con icono de engranaje resaltado](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-1.png)
1. Busque y seleccione **EasySSO**.

    ![Captura de pantalla de la opción EasySSO](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-2.png)

1. Seleccione **SAML**. Esto le llevará a la sección de configuración de SAML.

    ![Captura de pantalla de la página de administración de EasySSO, con SAML resaltado](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-3.png)

1. Seleccione la pestaña **Certificates** (Certificados) y aparecerá la siguiente pantalla:

    ![Captura de pantalla de la pestaña de certificados, con varias opciones resaltadas](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-4.png)

1. Busque el **certificado (Base64)** o el **archivo de metadatos** que guardó en la sección anterior de este tutorial. Puede continuar de las siguientes maneras:

    - Use el **archivo de metadatos** de federación de la aplicación que descargó en un archivo local del equipo. Seleccione el botón de radio **Upload** (Cargar) y siga la ruta de acceso específica del sistema operativo.

    - Abra el **archivo de metadatos** de federación de la aplicación para ver el contenido del archivo, en cualquier editor de texto sin formato. Cópielo en el Portapapeles. Seleccione la opción **Input** (Entrada) y pegue el contenido del Portapapeles en el campo de texto.
 
    - Realice una configuración totalmente manual. Abra el **certificado (Base64)** de federación de la aplicación para ver el contenido del archivo, en cualquier editor de texto sin formato. Cópielo en el Portapapeles y péguelo en el campo de texto **IdP Token Signing Certificates** (Certificados de firma de tokens del IdP). A continuación, vaya a la pestaña **General** y rellene los campos **POST Binding URL** (Dirección URL de enlace POST) y **Entity ID** (Identificador de entidad) con los valores correspondientes de **Dirección URL de inicio de sesión** e **Identificador de Azure AD** que guardó anteriormente.
 
1. Haga clic en **Save** (Guardar) en la parte inferior de la página. Verá que el contenido de los archivos de metadatos o del certificado se analiza en los campos de configuración. La configuración de EasySSO for BitBucket se ha completado.

1. Para probar la configuración, vaya a la pestaña **Look & Feel** (Aspecto y configuración) y seleccione **SAML Login Button** (Botón de inicio de sesión de SAML). De esta forma, se habilita un botón independiente en la pantalla de inicio de sesión de BitBucket para probar específicamente la integración de SAML de Azure AD de un extremo a otro. También puede dejar este botón activo y configurar su selección de ubicación, color y traducción para el modo de producción.

    ![Captura de pantalla del separador de apariencia y aspecto de la página de SAML, con la opción de botón de inicio de sesión de SAML resaltada](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-5.png)
    > [!NOTE]
    >Si tiene algún problema, póngase en contacto con el [equipo de soporte técnico de EasySSO](mailto:support@techtime.co.nz).

### <a name="create-an-easysso-for-bitbucket-test-user"></a>Creación de un usuario de prueba de EasySSO for BitBucket

En esta sección, creará un usuario llamado Britta Simon en BitBucket. EasySSO for BitBucket admite el aprovisionamiento de usuarios Just-In-Time, que está deshabilitado de forma predeterminada. Para habilitarlo, debe marcar explícitamente la opción **Create user on successful login** (Crear usuario después de un inicio de sesión correcto) en la sección **General** de la configuración del complemento de EasySSO. Si la usuaria no existe en BitBucket, se crea uno tras la autenticación.

Sin embargo, si no desea habilitar el aprovisionamiento automático de usuarios en el primer inicio de sesión del usuario, los usuarios deben existir en los directorios de usuario que se utilizan en la instancia de BitBucket. Por ejemplo, este directorio podría ser LDAP o Atlassian Crowd.

![Captura de pantalla de la sección General de la configuración del complemento de EasySSO, con la opción de creación de usuario después de un inicio de sesión correcto resaltada](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-6.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de BitBucket, desde donde puede comenzar el flujo de inicio de sesión.

* Acceda directamente a la URL de inicio de sesión de BitBucket y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Se debería iniciar sesión automáticamente en la instancia de EasySSO for BitBucket para la que configuró el inicio de sesión único.

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de EasySSO for BitBucket en Mis aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de EasySSO for BitBucket para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado EasySSO for BitBucket, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).