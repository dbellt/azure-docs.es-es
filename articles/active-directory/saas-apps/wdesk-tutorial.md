---
title: 'Tutorial: Integración de Azure Active Directory con Wdesk | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Wdesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/08/2021
ms.author: jeedes
ms.openlocfilehash: d18f1a9ffad76690ee03ab5f926b2f00d04c2543
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206896"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-wdesk"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Wdesk

En este tutorial, aprenderá a integrar Wdesk con Azure Active Directory (Azure AD). Al integrar Wdesk con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Wdesk.
* Permitir que los usuarios inicien sesión automáticamente en Wdesk con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Wdesk.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Wdesk admite el inicio de sesión único mediante **SP** e **IDP**.

## <a name="add-wdesk-from-the-gallery"></a>Adición de Wdesk desde la galería

Para configurar la integración de Wdesk en Azure AD, es preciso agregar Wdesk desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Wdesk** en el cuadro de búsqueda.
1. Seleccione **Wdesk** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-wdesk"></a>Configuración y prueba del inicio de sesión único de Azure AD en Wdesk

En esta sección, podrá configurar y probar el inicio de sesión único de Azure AD con Wdesk con un usuario de prueba llamado **Britta Simon**.
Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Wdesk.

Para configurar y probar el inicio de sesión único de Azure AD con Wdesk, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Wdesk](#configure-wdesk-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Wdesk](#create-wdesk-test-user)** , para tener un homólogo de B.Simon en Wdesk que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Wdesk**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<subdomain>.wdesk.com/auth/saml/sp/metadata/<instancename>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.wdesk.com/auth/saml/sp/consumer/<instancename>`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<subdomain>.wdesk.com/auth/login/saml/<instancename>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Estos valores se obtienen de portal de WDesk al configurar el SSO.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Wdesk** (Configurar Wdesk), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Wdesk mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Wdesk**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-wdesk-sso"></a>Configuración del inicio de sesión único de Wdesk

1. En otra ventana del explorador web, inicie sesión en Wdesk como administrador de seguridad.

1. En la parte inferior izquierda, haga clic en **Admin** (Administración) y elija **Account Admin** (Administrador de cuenta):
 
    ![Captura de pantalla que muestra el administrador de cuenta seleccionado en el menú Administrador.](./media/wdesk-tutorial/account.png)

1. En Wdesk Admin, navegue hasta **Security** (Seguridad) y, después, **SAML** > **SAML Settings** (Configuración de SAML):

    ![Captura de pantalla que muestra la opción de configuración de SAML seleccionada en la pestaña SAML.](./media/wdesk-tutorial/settings.png)

1. En **SAML User ID Settings** (Configuración de Id. de usuario de SAML), marque **SAML User ID is Wdesk Username** (Id. de usuario de SAML es nombreDeUsuario de Wdesk).

    ![Captura de pantalla que muestra la configuración de Id. de usuario de SAML, donde puede seleccionar S A M L User I D is Wdesk Username (I d. de usuario de S A M L es nombreDeUsuario de W desk).](./media/wdesk-tutorial/wdesk-username.png)

4. En **General Settings** (Configuración general), seleccione la casilla **Enable SAML Single Sign On** (Habilitar inicio de sesión único de SAML):

    ![Captura de pantalla que muestra Edit SAML Settings (Editar la configuración de SAML), donde puede seleccionar Enable SAML Single Sign-On (Habilitar inicio de sesión único de SAML).](./media/wdesk-tutorial/user-settings.png)

5. En **Service Provider Details** (Detalles del proveedor de servicios), siga estos pasos:

    ![Captura de pantalla que muestra los detalles del proveedor de servicios, donde puede especificar los valores descritos.](./media/wdesk-tutorial/service-provider.png)

    1. Copia el valor de **Login URL** (Dirección URL de inicio de sesión) y péguelo en el cuadro de texto **Dirección URL de inicio de sesión** de Azure Portal.

    1. Copia el valor de **Metadata Url** (Dirección URL de metadatos) y péguelo en el cuadro de texto **Identificador** de Azure Portal.

    1. Copia el valor de **Consumer url** (Dirección URL del consumidor ) y péguelo en el cuadro de texto **URL de respuesta** de Azure Portal.

    1. Haga clic en **Guardar** en Azure Portal para guardar los cambios.      

1. Haga clic en **Configure IdP Settings** (Configurar valores de IdP) para abrir el cuadro de diálogo **Edit IdP Settings** (Editar valores de IdP). Haga clic en **Choose File** (Elegir archivo) para buscar el archivo **Metadata.xml** que guardó en Azure Portal y cárguelo.
    
    ![Captura de pantalla que muestra Edit I d P Settings (Editar valores de I d P), donde puede cargar metadatos.](./media/wdesk-tutorial/metadata.png)
  
1. Haga clic en **Guardar cambios**.

    ![Captura de pantalla que muestra el botón Save changes (Guardar cambios).](./media/wdesk-tutorial/save.png)

### <a name="create-wdesk-test-user"></a>Creación de usuario de prueba de Wdesk

Para permitir que los usuarios de Azure AD inicien sesión en Wdesk, tienen que aprovisionarse en Wdesk. En el caso de Wdesk, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en Wdesk como administrador de seguridad.

2. Navegue hasta **Admin** (Administración)  > **Account Admin** (Administrador de cuenta).

     ![Captura de pantalla que muestra el administrador de cuenta seleccionado en el menú Administrador.](./media/wdesk-tutorial/account.png)

3. Haga clic en **Members** (Miembros) en **People** (Contactos).

4. A continuación, haga clic en **Add Member** (Agregar miembro) para abrir el cuadro de diálogo **(Agregar miembro)** . 
   
    ![Captura de pantalla que muestra la pestaña Members (Miembros), donde puede seleccionar Add Member (Agregar miembro).](./media/wdesk-tutorial/create-user-1.png)  

5. En el cuadro de texto **User** (Usuario), escriba el nombre del usuario así b.simon@contoso.com y haga clic en el botón **Continue** (Continuar).

    ![Captura de pantalla que muestra el cuadro de diálogo Add Member (Agregar miembro), donde puede especificar un usuario.](./media/wdesk-tutorial/create-user-3.png)

6.  Escriba los detalles que se muestran a continuación:
  
    ![Captura de pantalla que muestra el cuadro de diálogo Add Member (Agregar miembro), donde puede especificar la información básica de un usuario.](./media/wdesk-tutorial/create-user-4.png)
 
    a. En el cuadro de texto **E-mail** (Correo electrónico), escriba el correo electrónico del usuario con el siguiente formato b.simon@contoso.com.

    b. En el cuadro de texto **First Name** (Nombre), escriba el nombre del usuario, en este caso **B**.

    c. En el cuadro de texto **Last Name** (Apellidos), escriba el nombre de usuario, en este caso **Simon**.

7. Haga clic en el botón **Save Member** (Guardar miembro).  

    ![Captura de pantalla que muestra la opción Send welcome email (Enviar mensaje de correo de bienvenida) con el botón Save Member (Guardar miembro).](./media/wdesk-tutorial/create-user-5.png)

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Wdesk, donde puede comenzar el flujo de inicio de sesión.  

* Vaya a directamente a la dirección URL de inicio de sesión de Wdesk y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* En Azure Portal, haga clic en **Probar esta aplicación**. Al hacerlo, debería iniciar sesión automáticamente en la instancia de Wdesk para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Wdesk en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de Wdesk para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Wdesk, puede aplicar el control de sesión, que protege a la organización frente a la filtración y la infiltración de información confidencial en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).