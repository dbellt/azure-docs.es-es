---
title: 'Tutorial: Integración de Azure Active Directory con SilkRoad Life Suite | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y SilkRoad Life Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/09/2021
ms.author: jeedes
ms.openlocfilehash: 07bff698bd0c99536a123916b9d70792a6afc6bc
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894925"
---
# <a name="tutorial-azure-active-directory-integration-with-silkroad-life-suite"></a>Tutorial: Integración de Azure Active Directory con SilkRoad Life Suite

En este tutorial, aprenderá a integrar SilkRoad Life Suite con Azure Active Directory (Azure AD). La integración de SilkRoad Life Suite con Azure AD permite:

* Controlar en Azure AD quién tiene acceso a SilkRoad Life Suite.
* Habilitar que los usuarios inicien sesión automáticamente en SilkRoad Life Suite con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con SilkRoad Life Suite, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en SilkRoad Life Suite.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SilkRoad Life Suite admite el inicio de sesión único iniciado por **SP**.

## <a name="add-silkroad-life-suite-from-the-gallery"></a>Adición de SilkRoad Life Suite desde la galería

Para configurar la integración de SilkRoad Life Suite en Azure AD, deberá agregar SilkRoad Life Suite desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SilkRoad Life Suite** en el cuadro de búsqueda.
1. Seleccione **SilkRoad Life Suite** en el panel de resultados y, después, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-silkroad-life-suite"></a>Configuración y prueba del inicio de sesión único de Azure AD para SilkRoad Life Suite

Configure y pruebe el inicio de sesión único de Azure AD con SilkRoad Life Suite mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SilkRoad Life Suite.

Para configurar y probar el inicio de sesión único de Azure AD con SilkRoad Life Suite, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de SilkRoad Life Suite](#configure-silkroad-life-suite-sso)** : para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en SilkRoad Life Suite](#create-silkroad-life-suite-test-user)** : para tener un homólogo de B. Simon en SilkRoad Life Suite que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **SilkRoad Life Suite**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    > [!NOTE]
    > Obtendrá el **archivo de metadatos del proveedor de servicios**, que se explicará más adelante.

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Captura de pantalla que muestra la sección Configuración básica de SAML con el vínculo Cargar el archivo de metadatos.](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Captura de pantalla que muestra un cuadro de diálogo en el que puede seleccionar y cargar un archivo.](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, los valores de **Identificador** y **URL de respuesta** se rellenan automáticamente en la sección Configuración básica de SAML.

    > [!Note]
    > Si los valores de **Identificador** y **URL de respuesta** no se rellenan automáticamente, hágalo manualmente a medida que lo necesite.

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.silkroad-eng.com/Authentication/`

5. En la sección **Configuración básica de SAML**, si no tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    a. En el cuadro **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    | URL del identificador |
    |-----|
    |`https://<SUBDOMAIN>.silkroad-eng.com/Authentication/SP`|
    |`https://<SUBDOMAIN>.silkroad.com/Authentication/SP`|
    

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:

    | URL de respuesta |
    |-----|
    |`https://<SUBDOMAIN>.silkroad-eng.com/Authentication/`|
    |`https://<SUBDOMAIN>.silkroad.com/Authentication/`|

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.silkroad-eng.com/Authentication/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de atención al cliente de SilkRoad Life Suite](https://www.silkroad.com/locations/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Set up SilkRoad Life Suite** (Configurar SilkRoad Life Suite), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B. Simon para que pueda usar el inicio de sesión único de Azure, para lo que le concederá acceso a SilkRoad Life Suite.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SilkRoad Life Suite**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-silkroad-life-suite-sso"></a>Configuración del inicio de sesión único de SilkRoad Life Suite

1. Inicie sesión en su sitio de la compañía SilkRoad como administrador.

    > [!NOTE]
    > Para obtener acceso a la aplicación de autenticación de SilkRoad Life Suite para configurar la federación con Microsoft Azure AD, póngase en contacto con el soporte técnico o el representante de servicios de SilkRoad.

1. Vaya a **Proveedor de servicios** y, luego, haga clic en **Detalles de federación**.

    ![Captura de pantalla que muestra la opción de detalles de federación seleccionada en el proveedor de servicios.](./media/silkroad-life-suite-tutorial/details.png)

1. Haga clic en **Download Federation Metadata** (Descargar los metadatos de federación). Después, guarde el archivo de metadatos en el equipo. Use los metadatos de federación descargados como **archivo de metadatos del proveedor de servicios** en la sección **Configuración básica de SAML** de Azure Portal.

    ![Captura de pantalla que muestra el vínculo para descargar metadatos de federación.](./media/silkroad-life-suite-tutorial/metadata.png)

1. En la aplicación **SilkRoad**, haga clic en **Authentication Sources** (Orígenes de autenticación).

    ![Captura de pantalla que muestra la opción de orígenes de autenticación seleccionada.](./media/silkroad-life-suite-tutorial/sources.png) 

1. Haga clic en **Add Authentication Source**(Agregar origen de autenticación).

    ![Captura de pantalla que muestra el vínculo para agregar origen de autenticación.](./media/silkroad-life-suite-tutorial/add-source.png)

1. En la sección **Add Authentication Source** (Agregar origen de autenticación), realice los siguientes pasos:

    ![Captura de pantalla que muestra la sección para agregar origen de autenticación con el botón para crear el proveedor de identidades mediante datos de archivo seleccionada.](./media/silkroad-life-suite-tutorial/metadata-file.png)
  
    a. En **Option 2 - Metadata File** (Opción 2 - Archivo de metadatos), haga clic en **Examinar** para cargar el archivo de metadatos descargado de Azure Portal.
  
    b. Haga clic en **Create Identity Provider using File Data**(Crear proveedor de identidades con los datos del archivo).

1. En la sección **Authentication Sources** (Orígenes de autenticación), haga clic en **Edit** (Editar).

    ![Captura de pantalla que muestra la sección de orígenes de autenticación con la opción para editar seleccionada.](./media/silkroad-life-suite-tutorial/edit-source.png)

1. En el cuadro de diálogo **Edit Authentication Source** (Editar origen de autenticación), realice los siguientes pasos:

    ![Captura de pantalla que muestra el cuadro de diálogo para editar el origen de autenticación, donde puede especificar los valores descritos.](./media/silkroad-life-suite-tutorial/authentication.png)

    a. En **Enabled** (Habilitado), seleccione **Yes** (Sí).

    b. En el cuadro de texto **EntityId** (Identificador de entidad), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. En el cuadro de texto **IdP Description** (Descripción de IdP), escriba una descripción para la configuración (por ejemplo, **SSO de Azure AD**).

    d. En el cuadro de texto **Archivo de metadatos**, cargue el archivo de **metadatos** que descargó de Azure Portal.
  
    e. En el cuadro de texto **IdP Name** (Nombre de IdP), escriba un nombre que sea específico para su configuración (por ejemplo, *Azure SP*).
  
    f. En el cuadro de texto **Logout Service URL** (Dirección URL de cierre de sesión), pegue el valor de la **URL de cierre de sesión** que ha copiado de Azure Portal.

    g. En el cuadro de texto **Sign-on Service URL** (Dirección URL de inicio de sesión), pegue el valor de **URL de inicio de sesión** que ha copiado de Azure Portal.

    h. Haga clic en **Save**(Guardar).

1. Deshabilite todos los demás orígenes de autenticación.

    ![Captura de pantalla que muestra la sección de orígenes de autenticación, donde puede deshabilitar otros orígenes. ](./media/silkroad-life-suite-tutorial/manage-source.png)

### <a name="create-silkroad-life-suite-test-user"></a>Creación de un usuario de prueba en SilkRoad Life Suite

En esta sección, creará el usuario Britta Simon en SilkRoad Life Suite. Trabaje con el [equipo de atención al cliente de SilkRoad Life Suite](https://www.silkroad.com/locations/) para agregar los usuarios a la plataforma SilkRoad Life Suite. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de SilkRoad Life Suite, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de SilkRoad Life Suite e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Si hace clic en el icono de SilkRoad Life Suite en Aplicaciones, se le redireccionará a la dirección URL de inicio de sesión de SilkRoad Life Suite. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Tras configurar SilkRoad Life Suite, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
