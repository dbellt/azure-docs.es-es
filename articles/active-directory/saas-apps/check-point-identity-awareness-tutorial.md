---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Identity Awareness | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Check Point Identity Awareness.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/08/2021
ms.author: jeedes
ms.openlocfilehash: 68705d4d1f870820e30563d2a197ac56349d2445
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107520474"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-check-point-identity-awareness"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Identity Awareness

En este tutorial, aprenderá a integrar Check Point Identity Awareness con Azure Active Directory (Azure AD). Al integrar Check Point Identity Awareness con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Check Point Identity Awareness.
* Permitir que los usuarios inicien sesión automáticamente en Check Point Identity Awareness con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción a Check Point Identity Awareness con el inicio de sesión único habilitado.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Check Point Identity Awareness admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-check-point-identity-awareness-from-the-gallery"></a>Adición de Check Point Identity Awareness desde la galería

Para configurar la integración de Check Point Identity Awareness en Azure AD, es preciso agregar Check Point Identity Awareness desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Check Point Identity Awareness** en el cuadro de búsqueda.
1. Seleccione **Check Point Identity Awareness** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-check-point-identity-awareness"></a>Configuración y prueba del inicio de sesión único de Azure AD para Check Point Identity Awareness

Configure y pruebe el inicio de sesión único de Azure AD con Check Point Identity Awareness con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Check Point Identity Awareness.

Para configurar y probar el inicio de sesión único de Azure AD con Check Point Identity Awareness, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Check Point Identity Awareness](#configure-check-point-identity-awareness-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Check Point Identity Awareness](#create-check-point-identity-awareness-test-user)** , para tener un homólogo de B.Simon en Check Point Identity Awareness que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Check Point Identity Awareness**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/connect/spPortal/ACS/ID/<IDENTIFIER_UID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/connect/spPortal/ACS/Login/<IDENTIFIER_UID>`

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<GATEWAY_IP>/connect`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y las direcciones URL de inicio de sesión y de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Check Point Identity Awareness](mailto:support@checkpoint.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Check Point Identity Awareness**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Check Point Identity Awareness mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Check Point Identity Awareness**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-check-point-identity-awareness-sso"></a>Configuración del inicio de sesión único de Check Point Identity Awareness

1. Inicie sesión en el sitio de la compañía de Check Point Identity Awareness como administrador.

1. En la vista **Gateways & Servers** (Puertas de enlace y servidores) de SmartConsole, haga clic en **New > More > User/Identity > Identity Provider** (Nuevo > Más > Usuario/Identidad > Proveedor de identidades).

    ![captura de pantalla del nuevo proveedor de identidades.](./media/check-point-identity-awareness-tutorial/identity-provider.png)

1. Realice los pasos siguientes en la ventana **New Identity Provider** (Nuevo proveedor de identidades).

    ![captura de pantalla de la sección Identity Provider (Proveedor de identidades).](./media/check-point-identity-awareness-tutorial/new-identity-provider.png)

    a. En el **Gateway** (Puerta de enlace), seleccione la puerta de enlace de seguridad, que debe realizar la autenticación SAML.

    b. En el campo **Service** (Servicio), seleccione **Identity Awareness** en la lista desplegable.

    c. Copie el valor del **identificador de entidad** y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    d. Copie el valor de la **dirección URL de respuesta** y péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    e. Seleccione **Import the Metadata File** (Importar el archivo de metadatos) para cargar el **certificado (Base64)** descargado de Azure Portal.

    > [!NOTE]
    > También puede seleccionar **Insert Manually** (Insertar manualmente) para pegar manualmente los valores de **Id. de entidad** y **Dirección URL de inicio de sesión** en los campos correspondientes y cargar el **archivo de certificado** desde Azure Portal.

    f. Haga clic en **OK**.

### <a name="create-check-point-identity-awareness-test-user"></a>Creación de un usuario de prueba de Check Point Identity Awareness

En esta sección, creará un usuario llamado Britta Simon en Check Point Identity Awareness. Trabaje con el [equipo de soporte técnico de Check Point Identity Awareness](mailto:support@checkpoint.com) para agregar los usuarios a la plataforma de Check Point Identity Awareness. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Check Point Identity Awareness, desde donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de Check Point Identity Awareness y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Check Point Identity Awareness en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Check Point Identity Awareness, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


