---
title: 'Tutorial: Integración de Azure Active Directory con Infinite Campus | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory e Infinite Campus.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/17/2021
ms.author: jeedes
ms.openlocfilehash: fc1a2658b32cb2a1be3a6d08a4210d3cf667cdc2
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468057"
---
# <a name="tutorial-azure-active-directory-integration-with-infinite-campus"></a>Tutorial: Integración de Azure Active Directory con Infinite Campus

En este tutorial, aprenderá a integrar Infinite Campus con Azure Active Directory (Azure AD). Al integrar Infinite Campus con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Infinite Campus.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Infinite Campus con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Infinite Campus, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Infinite Campus.
* Como mínimo, debe ser un administrador de Azure Active Directory y tener un rol de seguridad de productos Campus de "sistema de información de estudiantes (SIS)" para completar la configuración.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Infinite Campus admite el inicio de sesión único iniciado por **SP**.

## <a name="add-infinite-campus-from-the-gallery"></a>Adición de Infinite Campus desde la galería

Para configurar la integración de Infinite Campus en Azure AD, debe agregar Infinite Campus desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Infinite Campus** en el cuadro de búsqueda.
1. Seleccione **Infinite Campus** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-infinite-campus"></a>Configuración y prueba del inicio de sesión único de Azure AD para Infinite Campus

Configure y pruebe el inicio de sesión único de Azure AD con Infinite Campus mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Infinite Campus.

Para configurar y probar el inicio de sesión único de Azure AD con Infinite Campus, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Infinite Campus](#configure-infinite-campus-sso)** , para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Infinite Campus](#create-infinite-campus-test-user)** , para tener un homólogo de B.Simon en Infinite Campus que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Infinite Campus**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección Configuración básica de SAML, siga los pasos que se indican a continuación (tenga en cuenta que el dominio varía según el modelo de hospedaje, pero el valor **FULLY-QUALIFIED-DOMAIN** debe coincidir con la instalación de Infinite Campus):

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>/SIS`.

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/<DISTRICTNAME>`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>`.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección, va a permitir que B.Simon acceda a Infinite Campus mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Infinite Campus**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-infinite-campus-sso"></a>Configuración del inicio de sesión único de Infinite Campus

1. En otra ventana del explorador web, inicie sesión en Infinite Campus como Administrador de seguridad.

2. En el lado izquierdo del menú, haga clic en **System Administration** (Administración del sistema).

    ![El administrador](./media/infinitecampus-tutorial/admin.png)

3. Vaya a **User Security** (Seguridad de usuarios) > **SAML Management** (Administración de SAML) > **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO).

    ![SAML](./media/infinitecampus-tutorial/security.png)

4. En la página **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO), siga estos pasos:

    ![Inicio de sesión único](./media/infinitecampus-tutorial/configuration.png)

    a. Seleccione **Enable SAML v2 Single Sign On**(Habilitar inicio de sesión único de SAML v2).

    b. Edite el **nombre del atributo opcional** para que contenga el **nombre**.

    c. En la sección **Select an option to retrieve Identity Provider (IDP) server data** (Seleccionar una opción para recuperar los datos del servidor del proveedor de identidades), seleccione **Metadata URL** (URL de metadatos), pegue el valor de **App Federation Metadata Url** (Dirección URL de metadatos de federación de aplicaciones) que copió de Azure Portal en el cuadro y, después, haga clic en **Sync** (Sincronizar).

    d. Después de hacer clic en **Sync** (Sincronizar), los valores se rellenan automáticamente en la página **SSO Service Provider Configuration** (Configuración del proveedor de servicios de SSO). Estos valores se pueden comprobar para que coincidan con los valores del paso 4 anterior.

    e. Haga clic en **Save**(Guardar).

### <a name="create-infinite-campus-test-user"></a>Creación de un usuario de prueba de Infinite Campus

Infinite Campuss tiene una arquitectura centrada en datos demográficos. Póngase en contacto con el [equipo de soporte técnico de Infinite Campus](mailto:sales@infinitecampus.com) para agregar los usuarios a la plataforma de Infinite Campus.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Infinite Campus, desde donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de Infinite Campus y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Infinite Campus en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Infinite Campus, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
