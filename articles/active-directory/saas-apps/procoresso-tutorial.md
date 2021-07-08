---
title: 'Tutorial: Integración de Azure Active Directory con Procore SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Procore SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/11/2021
ms.author: jeedes
ms.openlocfilehash: 9200c669df4e9ca756a037f7dd05823e9caf21df
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112062732"
---
# <a name="tutorial-azure-active-directory-integration-with-procore-sso"></a>Tutorial: Integración de Azure Active Directory con Procore SSO

En este tutorial aprenderá a integrar Procore SSO con Azure Active Directory (Azure AD). Al integrar Procore SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Procore SSO.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Procore SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Procore SSO, necesita lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Procore SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Procore SSO admite el inicio de sesión único iniciado por **IDP**.

## <a name="add-procore-sso-from-the-gallery"></a>Incorporación de Procore SSO desde la galería

Para configurar la integración de Procore SSO en Azure AD, tiene que agregar Procore SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Procore SSO** en el cuadro de búsqueda.
1. Seleccione **Procore SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-procore-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD en Procore SSO

Configure y pruebe el inicio de sesión único de Azure AD con Procore SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Procore SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Procore SSO, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Procore SSO](#configure-procore-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Procore SSO](#create-procore-sso-test-user)** , para tener un homólogo de B.Simon en Procore SSO que esté vinculado a la representación de usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Procore SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Procore SSO** (Configurar Procore SSO), copie la dirección URL adecuada según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Pantheon mediante el inicio de sesión único de Procore SSO.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Procore SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-procore-sso"></a>Configuración del inicio de sesión único de Procore SSO 

1. Para configurar el inicio de sesión único en **Procore SSO**, inicie sesión como administrador en su sitio de la compañía de Procore.

2. En la lista desplegable del cuadro de herramientas, haga clic en **Administrador** para abrir la página de configuración del SSO.

    ![Captura de pantalla que muestra el sitio de la compañía Procore con la opción de directorio seleccionada.](./media/procoresso-tutorial/admin.png)

3. Pegue los valores en los cuadros tal y como se describe a continuación:

    ![Captura de pantalla que muestra el cuadro de diálogo para agregar una persona.](./media/procoresso-tutorial/setting.png)   

    a. En el cuadro de texto **Dirección URL del emisor de inicio de sesión único**, pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Dirección URL de destino de inicio de sesión único de SAML**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra ahora el **XML de metadatos de la federación** que descargó antes de Azure Portal y copie el certificado en el etiqueta denominada **X509Certificate**. Pegue el valor copiado en el cuadro **Single Sign On x509 Certificate** (Certificado x509 de inicio de sesión único).

4. Haga clic en **Guardar cambios**.

5. Al finalizar esta configuración, tiene que enviar el **nombre de dominio** (por ejemplo, **contoso.com**) con el que inicia sesión en Procore al [equipo de soporte técnico de Procore](https://support.procore.com/) y ellos activarán el SSO federado para ese dominio.

### <a name="create-procore-sso-test-user"></a>Creación de un usuario de prueba de Procore SSO

Siga estos pasos para crear un usuario de prueba en Procore del lado de Procore SSO.

1. Inicie sesión como administrador en su sitio de la compañía de Procore.    

2. En la lista desplegable del cuadro de herramientas, haga clic en **Directorio** para abrir la página de directorio de la compañía.

    ![Captura de pantalla que muestra el sitio de la compañía Procore con la opción de directorio seleccionada en el cuadro de herramientas.](./media/procoresso-tutorial/directory.png)

3. Haga clic en la opción **Add a Person** (Agregar una persona) para abrir el formulario y especificar la ejecución de las opciones siguientes:

    ![En la captura de pantalla se muestra la sección para agregar una persona a Boylan Construction, donde puede especificar información de usuario.](./media/procoresso-tutorial/user.png)

    a. En el cuadro de texto **Nombre**, escriba el nombre del usuario, por ejemplo, **Britta**.

    b. En el cuadro de texto **Apellido**, escriba el apellido del usuario, por ejemplo, **Simon**.

    c. En el cuadro de texto **Dirección de correo electrónico**, escriba la dirección de correo del usuario con el formato siguiente BrittaSimon@contoso.com.

    d. Seleccione **Permission Template** (Plantilla de permisos) como **Apply Permission Template Later** (Aplicar plantilla de permisos más tarde).

    e. Haga clic en **Crear**.

4. Compruebe los datos del contacto recién agregado y actualícelos.

    ![Captura de pantalla que muestra una página de edición en la que puede comprobar la configuración de usuario.](./media/procoresso-tutorial/details.png)

5. Haga clic en **Save and Send Invitiation** (Guardar y enviar invitación), si se requiere una invitación por correo electrónico, o en **Guardar** (guardar directamente) para completar el registro del usuario.
    
    ![Captura de pantalla que muestra la configuración del proyecto actual, donde puede guardar y enviar la invitación.](./media/procoresso-tutorial/save.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal y debería iniciar sesión automáticamente en la instancia de Procore SSO para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Procore SSO en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Procore SSO para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado el inicio de sesión único de Procore SSO, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
