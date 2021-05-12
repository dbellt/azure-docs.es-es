---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TeamzSkill | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TeamzSkill.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2020
ms.author: jeedes
ms.openlocfilehash: 0a8cdbb298ff15b5502fead93aee0d6d200f06bb
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803105"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-teamzskill"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TeamzSkill

En este tutorial, aprenderá a integrar TeamzSkill con Azure Active Directory (Azure AD). Al integrar TeamzSkill con Azure AD, puede hacer lo siguiente:

- Controlar en Azure AD quién tiene acceso a TeamzSkill.
- Permitir que los usuarios inicien sesión automáticamente en TeamzSkill con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción habilitada para el inicio de sesión único en TeamzSkill.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- TeamzSkill admite el inicio de sesión único iniciado por **SP e IDP**.
- TeamzSkill admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-teamzskill-from-the-gallery"></a>Incorporación de TeamzSkill desde la galería

Para configurar la integración de TeamzSkill en Azure AD, es preciso que agregue TeamzSkill desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TeamzSkill** en el cuadro de búsqueda.
1. Seleccione **TeamzSkill** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-teamzskill"></a>Configuración y prueba del SSO de Azure AD para TeamzSkill

Configure y pruebe el inicio de sesión único de Azure AD con TeamzSkill mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de TeamzSkill.

Para configurar y probar el inicio de sesión único de Azure AD con TeamzSkill, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en TeamzSkill](#configure-teamzskill-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba en TeamzSkill](#create-teamzskill-test-user)** , para tener un homólogo de B.Simon en TeamzSkill que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **TeamzSkill**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

   a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

   b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

   En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CUSTOMER_SUBDOMAIN>.teamzskill.com/login/callback`

   > [!NOTE]
   > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de TeamzSkill](mailto:support@teamzskill.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación TeamzSkill espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

   ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación TeamzSkill espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | Nombre       | Atributo de origen   |
   | ---------- | ------------------ |
   | Firstname  | user.givenname     |
   | Lastname   | user.surname       |
   | jobtitle   | user.jobtitle      |
   | department | user.department    |
   | employeeid | user.employeeid    |
   | postalcode | user.postalcode    |
   | country    | user.country       |
   | rol       | user.assignedroles |

   > [!NOTE]
   > TeamzSkill espera roles para los usuarios asignados a la aplicación. Configure estos roles en Azure AD para que se puedan asignar los roles correspondientes a los usuarios. Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar TeamzSkill**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a TeamzSkill mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TeamzSkill**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-teamzskill-sso"></a>Configuración del inicio de sesión único en TeamzSkill

1. Abra una nueva ventana del explorador web e inicie sesión en TeamzSkill como administrador.

1. Haga clic en el icono de perfil de usuario y seleccione **Company settings** (Configuración de la empresa).

   ![Configuración de la empresa en Teamzskill](./media/teamzskill-tutorial/settings.png)

1. En la página **Settings** (Configuración), realice los pasos siguientes:

   ![Configuración de Teamzskill](./media/teamzskill-tutorial/metadata.png)

   a. Vaya a **Company > Single Sign-On** (Empresa > Inicio de sesión único) y, a continuación, seleccione la pestaña **Metadata Upload** (Carga de metadatos).

   b. Pegue el valor de **XML de metadatos de federación** que ha copiado de Azure Portal en el campo **XML Metadata** (Metadatos XML).

   c. A continuación, haga clic en **Save**(Guardar).

### <a name="create-teamzskill-test-user"></a>Creación de un usuario de prueba en TeamzSkill

En esta sección, va a crear un usuario llamado B.Simon en TeamzSkill. TeamzSkill admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en TeamzSkill, se crea uno nuevo cuando se intenta acceder a TeamzSkill.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

#### <a name="sp-initiated"></a>Iniciado por SP:

- Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de TeamzSkill, donde puede iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de TeamzSkill e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

- Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de TeamzSkill para la que configuró el inicio de sesión único.

También puede usar el Panel de acceso de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de TeamzSkill en el Panel de acceso, si está configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para iniciar el flujo de inicio de sesión y, si está configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de TeamzSkill para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado TeamzSkill, puede aplicar el control de sesión, que protege a su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
