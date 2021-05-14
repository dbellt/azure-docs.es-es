---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Count Me In - Operations Dashboard | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Count Me In - Operations Dashboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/30/2020
ms.author: jeedes
ms.openlocfilehash: 44ea8afcb42365b2b01d81674d0f76781cca0ea9
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108801737"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-count-me-in---operations-dashboard"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Count Me In - Operations Dashboard

En este tutorial, aprenderá a integrar Meraki Dashboard con Azure Active Directory (Azure AD). Al integrar Count Me In - Operations Dashboard con Azure AD, puede:

- Controlar en Azure AD quién tiene acceso a Count Me In - Operations Dashboard.
- Permitir que los usuarios inicien sesión automáticamente en Count Me In - Operations Dashboard con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción habilitada para el inicio de sesión único (SSO) en Count Me In - Operations Dashboard.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Count Me In - Operations Dashboard admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-count-me-in---operations-dashboard-from-the-gallery"></a>Adición de Count Me In - Operations Dashboard desde la galería

Para configurar la integración de Count Me In - Operations Dashboard en Azure AD, deberá agregar Count Me In - Operations Dashboard desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Count Me In - Operations Dashboard** en el cuadro de búsqueda.
1. Seleccione **Count Me In - Operations Dashboard** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-count-me-in---operations-dashboard"></a>Configuración y prueba del inicio de sesión único de Azure AD para Count Me In - Operations Dashboard

Configure y pruebe el inicio de sesión único de Azure AD con Count Me In - Operations Dashboard con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Count Me In - Operations Dashboard.

Para configurar y probar el inicio de sesión único de Azure AD con Count Me In - Operations Dashboard, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Count Me In - Operations Dashboard](#configure-count-me-in-operations-dashboard-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Count Me In - Operations Dashboard](#create-count-me-in-operations-dashboard-test-user)** : para tener un homólogo de B.Simon en Count Me In - Operations Dashboard que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Count Me In - Operations Dashboard**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://api-us.localz.io/user/v1/saml/initsso?projectId=<PROJECT_ID>`

   b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `api-us.localz.io/<PROJECT_ID>`

   c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://api-us.localz.io/user/v1/saml/initsso?projectId=<PROJECT_ID>`

   > [!NOTE]
   > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Count Me In - Operations Dashboard](mailto:support@localz.co) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Count Me In - Operations Dashboard espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

   ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Count Me In - Operations Dashboard espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | Nombre           | Atributo de origen   |
   | -------------- | ------------------ |
   | roles asignados | user.assignedroles |

   > [!NOTE]
   > Count Me In - Operations Dashboard espera roles para los usuarios asignados a la aplicación. Configure estos roles en Azure AD para que se puedan asignar los roles correspondientes a los usuarios. Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Count Me In - Operations Dashboard**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Count Me In - Operations Dashboard mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Count Me In - Operations Dashboard**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-count-me-in-operations-dashboard-sso"></a>Configuración del inicio de sesión único de Count Me In - Operations Dashboard

Para configurar el inicio de sesión único en **Count Me In - Operations Dashboard**, es preciso enviar el **certificado (Base 64)** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de Count Me In - Operations Dashboard](mailto:support@localz.co). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-count-me-in-operations-dashboard-test-user"></a>Creación de un usuario de prueba de Count Me In - Operations Dashboard

En esta sección, creará un usuario llamado Britta Simon en Count Me In - Operations Dashboard. Colabore con el [equipo de soporte técnico de Count Me In - Operations Dashboard](mailto:support@localz.co) para agregar los usuarios en la plataforma de Count Me In - Operations Dashboard. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Count Me In - Operations Dashboard, donde podrá iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Count Me In - Operations Dashboard e inicie el flujo de inicio de sesión desde allí.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Count Me In - Operations Dashboard en el panel Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Count Me In - Operations Dashboard. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Count Me In - Operations Dashboard, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
