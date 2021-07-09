---
title: 'Tutorial: integración de Azure Active Directory con Optimizely | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Optimizely.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/24/2021
ms.author: jeedes
ms.openlocfilehash: 8e2a95839df560a15906d38a0d0721a8f156e1cb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472962"
---
# <a name="tutorial-azure-active-directory-integration-with-optimizely"></a>Tutorial: integración de Azure Active Directory con Optimizely

En este tutorial, obtendrá información sobre cómo integrar Optimizely con Azure Active Directory (Azure AD). Al integrar Optimizely con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Optimizely.
* Permitir que los usuarios inicien sesión automáticamente en Optimizely con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Optimizely.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Optimizely admite el inicio de sesión único iniciado por **SP**.

## <a name="add-optimizely-from-the-gallery"></a>Adición de Optimizely desde la galería

Para configurar la integración de Optimizely en Azure AD, es preciso agregar Optimizely desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Optimizely** en el cuadro de búsqueda.
1. Seleccione **Optimizely** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-optimizely"></a>Configuración y prueba del inicio de sesión único de Azure AD para Optimizely

Configure y pruebe el inicio de sesión único de Azure AD con Optimizely mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Optimizely.

Para configurar y probar el inicio de sesión único de Azure AD con Optimizely, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Optimizely](#configure-optimizely-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Optimizely](#create-optimizely-test-user)** : para tener un homólogo de B.Simon en Optimizely que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Optimizely**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://app.optimizely.net/<INSTANCE_NAME>`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba un valor con el siguiente patrón: `urn:auth0:optimizely:contoso`

    > [!NOTE]
    > Estos valores no son reales. El valor se actualizará con la dirección URL de inicio de sesión y el identificador reales, que se explican más adelante en el tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. La aplicación Optimizely espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

    ![Captura de pantalla que muestra el cuadro de diálogo "Atributos de usuario" con el botón "Editar" seleccionado.](common/edit-attribute.png)

6. Además de lo anterior, la aplicación Optimizely espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:

    | Nombre | Atributo de origen |
    | ---------------| --------------- |
    | email | user.mail |
    
    a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

    ![Captura de pantalla que muestra la sección "Notificaciones del usuario" con las acciones "Agregar nueva notificación" y "Guardar" resaltadas.](common/new-save-attribute.png)

    ![imagen](common/new-attribute-details.png)

    b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

    c. Deje **Espacio de nombres** en blanco.

    d. Seleccione **Atributo** como origen.

    e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

    f. Haga clic en **Aceptar**.

    g. Haga clic en **Save**(Guardar).

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Optimizely** (Configurar Optimizely), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Optimizely mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Optimizely**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-optimizely-sso"></a>Configuración del inicio de sesión único de Optimizely

1. Para configurar el inicio de sesión único en **Optimizely**, póngase en contacto con el administrador de cuentas de Optimizely y proporcione el **certificado (Base64)** descargado y las direcciones URL copiadas adecuadas.

2. En respuesta a su correo electrónico, Optimizely proporciona la URL de inicio de sesión (SSO iniciado por el proveedor de servicios) y los valores de Identificador (Id. de entidad del proveedor de servicios).

    a. Copie la **URL de inicio de sesión único iniciado por SP** que proporciona Optimizely y péguela en el cuadro de texto **URL de inicio de sesión** en la sección **Configuración básica de SAML** de Azure Portal.

    b. Copie el **identificador de entidad del proveedor de servicios** que proporciona Optimizely y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** de Azure Portal.

3. En una ventana de explorador diferente, inicie sesión en su aplicación de Optimizely como administrador.

4. Haga clic en el nombre de la cuenta de la parte superior derecha y, después, en **Configuración de la cuenta**.

    ![Captura de pantalla que muestra el nombre de cuenta seleccionado en la esquina superior derecha, con la opción "Configuración de cuenta" seleccionada en el menú.](./media/optimizely-tutorial/settings.png)

5. En la pestaña Cuenta, active la casilla **Habilitar SSO** que se encuentra debajo de Inicio de sesión único en la sección **Información general**.
  
    ![Inicio de sesión único de Azure AD](./media/optimizely-tutorial/account.png)

6. Haga clic en **Save**(Guardar).

### <a name="create-optimizely-test-user"></a>Creación de un usuario de prueba de Optimizely

En esta sección, creará un usuario llamado Britta Simon en Optimizely.

1. En la página principal, seleccione la pestaña **Colaboradores**.

2. Para agregar un nuevo colaborador, haga clic en **New Collaborator** (Nuevo colaborador).
   
    ![Captura de pantalla que muestra la página principal de Optimizely con la pestaña "Collaborators" (Colaboradores) y el botón "New Collaborator" (Nuevo colaborador) seleccionados.](./media/optimizely-tutorial/collaborator.png)

3. Escriba la dirección de correo electrónico y asígnele un rol. Haga clic en **Invitar**.

    ![Creación de un usuario de prueba de Azure AD](./media/optimizely-tutorial/invite-collaborator.png)

4. El usuario recibirá una invitación por correo electrónico, con la que debe iniciar sesión en Optimizely.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Optimizely, donde puede iniciar el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Optimizely y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Optimizely en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Optimizely, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
