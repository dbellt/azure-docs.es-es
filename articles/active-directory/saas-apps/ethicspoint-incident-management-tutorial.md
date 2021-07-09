---
title: 'Tutorial: Integración de Azure Active Directory con EthicsPoint Incident Management (EPIM) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y EthicsPoint Incident Management (EPIM).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/31/2021
ms.author: jeedes
ms.openlocfilehash: 72a0defed198e36783f086a9e5934eed580ab756
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111754937"
---
# <a name="tutorial-azure-active-directory-integration-with-ethicspoint-incident-management-epim"></a>Tutorial: Integración de Azure Active Directory con EthicsPoint Incident Management (EPIM)

En este tutorial, aprenderá a integrar EthicsPoint Incident Management (EPIM) con Azure Active Directory (Azure AD). Al integrar EthicsPoint Incident Management (EPIM) con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a EthicsPoint Incident Management (EPIM).
* Permitir que los usuarios inicien sesión automáticamente en EthicsPoint Incident Management (EPIM) con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en EthicsPoint Incident Management (EPIM).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* EthicsPoint Incident Management (EPIM) admite el inicio de sesión único iniciado por **SP**.

## <a name="add-ethicspoint-incident-management-epim-from-the-gallery"></a>Adición de EthicsPoint Incident Management (EPIM) desde la galería

Para configurar la integración de EthicsPoint Incident Management (EPIM) en Azure AD, debe agregar EthicsPoint Incident Management (EPIM) desde la galería a su lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **EthicsPoint Incident Management (EPIM)** en el cuadro de búsqueda.
1. Seleccione **EthicsPoint Incident Management (EPIM)** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-ethicspoint-incident-management-epim"></a>Configuración y prueba del inicio de sesión único de Azure AD para EthicsPoint Incident Management (EPIM)

Configure y pruebe el inicio de sesión único de Azure AD con EthicsPoint Incident Management (EPIM) mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de EthicsPoint Incident Management (EPIM).

Para configurar el inicio de sesión único de Azure AD con EthicsPoint Incident Management (EPIM), haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de EthicsPoint Incident Management (EPIM)](#configure-ethicspoint-incident-management-epim-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de EthicsPoint Incident Management (EPIM)](#create-ethicspoint-incident-management-epim-test-user)** : para tener un homólogo de B.Simon en EthicsPoint Incident Management (EPIM) que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **EthicsPoint Incident Management (EPIM)** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<COMPANY_NAME>.navexglobal.com/adfs/services/trust`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SERVER_NAME>.navexglobal.com/adfs/ls/`

     c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón:

    | URL de inicio de sesión|
    |---|
    |`https://<COMPANY_NAME>.navexglobal.com`|
    |`https://<COMPANY_NAME>.ethicspointvp.com`|
    |
  
    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Para obtener estos valores, póngase en contacto con el [equipo de soporte técnico de EthicsPoint Incident Management (EPIM)](https://www.navexglobal.com/company/contact-us). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up EthicsPoint Incident Management (EPIM)** (Configurar EthicsPoint Incident Management [EPIM]), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a EthicsPoint Incident Management (EPIM) mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **EthicsPoint Incident Management (EPIM)** .
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ethicspoint-incident-management-epim-sso"></a>Configuración del inicio de sesión único de EthicsPoint Incident Management (EPIM)

Para configurar el inicio de sesión único en **EthicsPoint Incident Management (EPIM)** , debe enviar el archivo **XML de metadatos de federación** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de EthicsPoint Incident Management (EPIM)](https://www.navexglobal.com/company/contact-us). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-ethicspoint-incident-management-epim-test-user"></a>Creación de un usuario de prueba de EthicsPoint Incident Management (EPIM)

En esta sección, creará un usuario llamado Britta Simon en EthicsPoint Incident Management (EPIM). Trabaje con el [equipo de soporte técnico de EthicsPoint Incident Management (EPIM)](https://www.navexglobal.com/company/contact-us) para agregar los usuarios a la plataforma de EthicsPoint Incident Management (EPIM). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de EthicsPoint Incident Management (EPIM), donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de EthicsPoint Incident Management (EPIM) y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de EthicsPoint Incident Management (EPIM) en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de EthicsPoint Incident Management (EPIM). Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que ha configurado EthicsPoint Incident Management (EPIM), podrá aplicar el control de sesiones, que protege la información confidencial de la organización en tiempo real de posibles filtraciones e infiltraciones. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
