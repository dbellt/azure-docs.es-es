---
title: 'Tutorial: Integración de Azure Active Directory con Veritas Enterprise Vault.cloud SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Veritas Enterprise Vault.cloud SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2021
ms.author: jeedes
ms.openlocfilehash: 2796928316197bd48723253dbc97dfcd34ac95e8
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222289"
---
# <a name="tutorial-azure-active-directory-integration-with-veritas-enterprise-vaultcloud-sso"></a>Tutorial: Integración de Azure Active Directory con Veritas Enterprise Vault.cloud SSO

En este tutorial, aprenderá a integrar Veritas Enterprise Vault.cloud SSO en Azure Active Directory (Azure AD). Si se integra Veritas Enterprise Vault.cloud SSO en Azure AD, es posible:

* Controlar en Azure AD quién tiene acceso a Veritas Enterprise Vault.cloud SSO.
* Permitir que los usuarios inicien sesión automáticamente en Veritas Enterprise Vault.cloud SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Veritas Enterprise Vault.cloud SSO, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único en Veritas Enterprise Vault.cloud SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Veritas Enterprise Vault.cloud SSO admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-veritas-enterprise-vaultcloud-sso-from-the-gallery"></a>Adición de Veritas Enterprise Vault.cloud SSO desde la galería

Para configurar la integración de Veritas Enterprise Vault.cloud SSO en Azure AD, deberá agregar Veritas Enterprise Vault.cloud SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la Galería**, escriba **Veritas Enterprise Vault.Cloud SSO** en el cuadro de búsqueda.
1. Seleccione **Veritas Enterprise Vault.Cloud SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-veritas-enterprise-vaultcloud-sso"></a>Configuración y prueba del inicio de sesión único de Azure AD para Veritas Enterprise Vault.cloud SSO

Configure y pruebe el inicio de sesión único de Azure AD con Veritas Enterprise Vault.cloud SSO mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Veritas Enterprise Vault.cloud SSO.

Para configurar y probar el inicio de sesión único de Azure AD con Veritas Enterprise Vault.cloud SSO, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Veritas Enterprise Vault.cloud SSO](#configure-veritas-enterprise-vaultcloud-sso-sso)** : permite configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Veritas Enterprise Vault.cloud SSO](#create-veritas-enterprise-vaultcloud-sso-test-user)** : para tener un homólogo de B. Simon en Veritas Enterprise Vault.cloud SSO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación de **Veritas Enterprise Vault.cloud SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://personal.ap.archive.veritas.com/CID=<CUSTOMERID>`

    b. En el cuadro **Identificador**, escriba una de las direcciones URL según el centro de datos:

    | Centro de datos| URL |
    |----------|----|
    | Norteamérica| `https://auth.lax.archivecloud.net` |
    | Europa | `https://auth.ams.archivecloud.net` |
    | Asia Pacífico| `https://auth.syd.archivecloud.net`|

    c. En el cuadro de texto **Dirección URL de respuesta**, escriba una de las direcciones URL según el centro de datos:

    | Centro de datos| URL |
    |----------|----|
    | Norteamérica| `https://auth.lax.archivecloud.net` |
    | Europa | `https://auth.ams.archivecloud.net` |
    | Asia Pacífico| `https://auth.syd.archivecloud.net`|

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Veritas Enterprise Vault.cloud SSO** (Configuración de Veritas Enterprise Vault.cloud SSO), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B. Simon para que use el inicio de sesión único de Azure concediéndole acceso a Veritas Enterprise Vault.cloud SSO.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Veritas Enterprise Vault.cloud SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-veritas-enterprise-vaultcloud-sso-sso"></a>Configuración del inicio de sesión único de Veritas Enterprise Vault.cloud SSO

Para configurar el inicio de sesión único en **Veritas Enterprise Vault.cloud SSO**, es preciso enviar el **certificado (Base64)** descargado y las direcciones URL adecuadas copiadas de Azure Portal al [equipo de soporte técnico de Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-veritas-enterprise-vaultcloud-sso-test-user"></a>Creación de un usuario de prueba de Veritas Enterprise Vault.cloud SSO

En esta sección, creará un usuario llamado Britta Simon en Veritas Enterprise Vault.cloud SSO. Trabaje con el [equipo de soporte técnico de Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html) para agregar los usuarios a la plataforma de Veritas Enterprise Vault.cloud SSO. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Veritas Enterprise Vault.cloud SSO, desde donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Veritas Enterprise Vault.cloud SSO y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Veritas Enterprise Vault.cloud SSO en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión único de Veritas Enterprise Vault.cloud SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Veritas Enterprise Vault.cloud SSO, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
