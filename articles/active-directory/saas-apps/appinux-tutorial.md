---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Appinux | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Appinux.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2020
ms.author: jeedes
ms.openlocfilehash: 344d2317a0a8370c962ff28dfb5dc75e42dfeb76
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802079"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-appinux"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Appinux

En este tutorial, aprenderá a integrar Appinux con Azure Active Directory (Azure AD). Al integrar Appinux con Azure AD, puede hacer lo siguiente:

- Controlar en Azure AD quién tiene acceso a Appinux.
- Permitir que los usuarios inicien sesión automáticamente en Appinux con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Suscripción habilitada para el inicio de sesión único (SSO) en Appinux.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Appinux admite el inicio de sesión único iniciado por **SP**

- Appinux admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-appinux-from-the-gallery"></a>Adición de Appinux desde la galería

Para configurar la integración de Appinux en Azure AD, será preciso que agregue Appinux desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Appinux** en el cuadro de búsqueda.
1. Seleccione **Appinux** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-appinux"></a>Configuración y prueba del inicio de sesión único de Azure AD para Appinux

Configure y pruebe el inicio de sesión único de Azure AD con Appinux mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Appinux.

Para configurar y probar el inicio de sesión único de Azure AD con Appinux, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Appinux](#configure-appinux-sso)**, para configurar los valores de Inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Appinux](#create-appinux-test-user)**, para tener un homólogo de B.Simon en Appinux que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Appinux**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<Appinux_SUBDOMAIN>.appinux.com`

   b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<Appinux_SUBDOMAIN>.appinux.com/simplesaml/module.php/saml/sp/metadata.php/default-sp`

   > [!NOTE]
   > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte de cliente de Appinux](https://support.appinux.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. Appinux espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo Atributos de usuario.

   ![imagen](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Appinux espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | **Nombre**         | **Espacio de nombres**                                                  | **Atributo de origen**                         |
   | ---------------- | -------------------------------------------------------------- | -------------------------------------------- |
   | `givenname`      | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`        | `user.givenname`                             |
   | `surname`        | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`        | `user.surname`                               |
   | `emailaddress`   | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`        | `user.mail`                                  |
   | `name`           | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`        | `user.userprincipalname`                     |
   | `UserType`       | `http://bcv.appinux.com/claims`                                | `Provide the value as per your organization` |
   | `Tag`            | `http://appinux.com/Tag`                                       | `Provide the value as per your organization` |
   | `Role`           | `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` | `user.assignedroles`                         |
   | `email`          | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/email`  | `user.mail`                                  |
   | `wanshort`       | `http://appinux.com/windowsaccountname2`                       | `extractmailprefix([userprincipalname])`     |
   | `nameidentifier` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`        | `user.employeeid`                            |

   > [!NOTE]
   > Appinux espera roles para los usuarios asignados a la aplicación. Configure estos roles en Azure AD para que se puedan asignar los roles correspondientes a los usuarios. Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Appinux**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Appinux utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Appinux**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-appinux-sso"></a>Configuración del inicio de sesión único de Appinux

Para configurar el inicio de sesión único en **Appinux**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Appinux](https://support.appinux.com/). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-appinux-test-user"></a>Creación de un usuario de prueba de Appinux

En esta sección, se crea un usuario llamado Britta Simon en Appinux. Appinux admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Appinux, se crea uno después de la autenticación.

> [!Note]
> Si tiene que crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Appinux](https://support.appinux.com).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Appinux, donde puede iniciar el flujo de inicio de sesión.

- Acceda directamente a la dirección URL de inicio de sesión de Appinux y comience el flujo de inicio de sesión desde ahí.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Appinux en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Appinux. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Appinux, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
