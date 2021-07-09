---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Terraform Cloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Terraform Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/01/2021
ms.author: jeedes
ms.openlocfilehash: ec6de7ea5c9da4402126990d1c165d4f4ad6d12f
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111557463"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-terraform-cloud"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Terraform Cloud

En este tutorial aprenderá a integrar Terraform Cloud con Azure Active Directory (Azure AD). Al integrar Terraform Cloud con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Terraform Cloud.
* Permitir que los usuarios inicien sesión automáticamente en Terraform Cloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Terraform Cloud.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Terraform Cloud admite el SSO iniciado por **SP e IDP**.
* Terraform Cloud admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-terraform-cloud-from-the-gallery"></a>Incorporación de Terraform Cloud desde la galería

Para configurar la integración de Terraform Cloud en Azure AD, necesita agregar Terraform Cloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Terraform Cloud** en el cuadro de búsqueda.
1. Seleccione **Terraform Cloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-terraform-cloud"></a>Configuración y prueba del inicio de sesión único de Azure AD para Terraform Cloud

Configure y pruebe el inicio de sesión único de Azure AD con Terraform Cloud mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Terraform Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con Terraform Cloud, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Terraform Cloud](#configure-terraform-cloud-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Terraform Cloud](#create-terraform-cloud-test-user)** : para tener un homólogo de B.Simon en Terraform Cloud vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Terraform Cloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://app.terraform.io/sso/saml/samlconf-<ID>/metadata`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.terraform.io/sso/saml/samlconf-<ID>/acs`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://app.terraform.io/session`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico al cliente de Terraform Cloud](mailto:tf-cloud@hashicorp.support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

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

En esta sección va a permitir que B.Simon acceda a Terraform Cloud mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Terraform Cloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-terraform-cloud-sso"></a>Configuración del inicio de sesión único en Terraform Cloud

1. Para automatizar la configuración en Terraform Cloud, debe instalar la **extensión del navegador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al navegador, haga clic en **Set up Terraform Cloud** (Configurar Terraform Cloud) para ir a la aplicación Terraform Cloud. Desde aquí, escriba las credenciales de administrador para iniciar sesión en Terraform Cloud. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Terraform Cloud manualmente, en otra ventana del explorador web, inicie sesión en el sitio de la empresa Terraform Cloud como administrador.

2. Vaya a **Settings > SSO > Edit Settings** (Configuración > Inicio de sesión único > Editar configuración).

    ![Configuración de Terraform Cloud](./media/terraform-cloud-tutorial/sso-settings.png)

3. En la página **Editar SSO**, realice los pasos siguientes:

    ![Editar SSO en Terraform Cloud](./media/terraform-cloud-tutorial/edit-sso.png)

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **URL de inicio de sesión** que ha copiado de Azure Portal.

    b. En el cuadro de texto **Entity ID or Identifier** (Id. de entidad o identificador), pega el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    c. Abra el archivo del **certificado** que descargó de Azure Portal en el Bloc de notas, copie el contenido y péguelo en el cuadro de texto **Certificado público**.

    d. Haga clic en **Save settings** (Guardar configuración).

### <a name="create-terraform-cloud-test-user"></a>Creación de un usuario de prueba en Terraform Cloud

En esta sección se crea un usuario llamado B.Simon en Terraform Cloud. Terraform Cloud admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si no existe un usuario en Terraform Cloud, se crea después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Terraform Cloud, donde podrá iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Terraform Cloud e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Terraform Cloud para la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Terraform Cloud en Aplicaciones, si tiene la configuración del modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión. Si tiene la configuración del modo IDP, debería iniciar sesión automáticamente en la instancia de Terraform Cloud para la que configuró el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada la aplicación Terraform Cloud, podrá aplicar el control de sesión, que protege a su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
