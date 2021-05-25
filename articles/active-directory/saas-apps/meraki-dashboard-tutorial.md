---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Meraki Dashboard | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Meraki Dashboard.
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
ms.openlocfilehash: 87ab7156e92f46630e9eed005c7384d883b0caa0
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804833"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-meraki-dashboard"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Meraki Dashboard

En este tutorial, aprenderá a integrar Meraki Dashboard con Azure Active Directory (Azure AD). Al integrar Meraki Dashboard con Azure AD, puede:

- Controlar en Azure AD quién tiene acceso a Meraki Dashboard.
- Permitir que los usuarios inicien sesión automáticamente en Meraki Dashboard con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción habilitada para el inicio de sesión único (SSO) en Meraki Dashboard.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- El panel de Meraki Dashboard admite el inicio de sesión único iniciado por **IDP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-meraki-dashboard-from-the-gallery"></a>Incorporación de Meraki Dashboard desde la galería

Para configurar la integración de Meraki Dashboard en Azure AD, deberá agregar Meraki Dashboard desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Meraki Dashboard** en el cuadro de búsqueda.
1. Seleccione **Meraki Dashboard** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-meraki-dashboard"></a>Configuración y prueba del inicio de sesión único de Azure AD para Meraki Dashboard

Configure y pruebe el inicio de sesión único de Azure AD con Meraki Dashboard con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Meraki Dashboard.

Para configurar y probar el inicio de sesión único de Azure AD con Meraki Dashboard, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Meraki Dashboard](#configure-meraki-dashboard-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Meraki Dashboard](#create-meraki-dashboard-test-user)** , para tener un homólogo de B.Simon en Meraki Dashboard que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Meraki Dashboard**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

   En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://n27.meraki.com/saml/login/m9ZEgb/< UNIQUE ID >`.

   > [!NOTE]
   > El valor de dirección URL de respuesta no es real. Actualice este valor con la dirección URL de respuesta real, que se explica más adelante en el tutorial.

1. Haga clic en el botón **Save** (Guardar).

1. La aplicación Meraki Dashboard espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

   ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Meraki Dashboard espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | Nombre                                                    | Atributo de origen       |
   | ------------------------------------------------------- | ---------------------- |
   | `https://dashboard.meraki.com/saml/attributes/username` | user.userprincipalname |
   | `https://dashboard.meraki.com/saml/attributes/role`     | user.assignedroles     |

   > [!NOTE]
   > Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la sección **Certificado de firma de SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **Certificado de firma de SAML**.

   ![Edición del certificado de firma de SAML](common/edit-certificate.png)

1. En la sección **Certificado de firma de SAML**, copie el valor de **Huella digital** y guárdelo en el equipo. Este valor debe convertirse para que incluya dos puntos para que el panel de Meraki lo entienda. Por ejemplo, si la huella digital de Azure es `C2569F50A4AAEDBB8E` deberá cambiarla a `C2:56:9F:50:A4:AA:ED:BB:8E` para usarla más adelante en el panel de Meraki.

   ![Copia del valor de la huella digital](common/copy-thumbprint.png)

1. En la sección **Configurar Meraki Dashboard**, copie el valor de la dirección URL de cierre de sesión y guárdelo en el equipo.

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

En esta sección, va a permitir que B.Simon acceda a Meraki Dashboard mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Meraki Dashboard**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".

   ![rol de usuario](./media/meraki-dashboard-tutorial/user-role.png)

   > [!NOTE]
   > La opción **Seleccione un rol** se deshabilitará y el rol predeterminado es USUARIO para el usuario seleccionado.

1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-meraki-dashboard-sso"></a>Configuración del inicio de sesión único de Meraki Dashboard

1. Para automatizar la configuración en Meraki Dashboard, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

   ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Meraki Dashboard** para ir a la aplicación. Desde ahí, especifique las credenciales de administrador para iniciar sesión en Meraki Dashboard. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 7.

   ![Configuración](common/setup-sso.png)

3. Si desea configurar manualmente Meraki Dashboard, abra otra ventana del explorador web e inicie sesión como administrador en el sitio web de Meraki Dashboard.

4. Vaya a **Organization** (Organización)  -> **Configuration** (Configuración).

   ![Pestaña Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure-1.png)

5. En Autenticación, cambie **SAML SSO** (SSO de SAML) a **SAML SSO enabled** (SSO de SAML habilitado).

   ![Autenticación de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure-2.png)

6. Haga clic en **Add a SAML IdP** (Agregar un IdP de SAML).

   ![Agregación de un IdP de SAML en Meraki Dashboard](./media/meraki-dashboard-tutorial/configure-3.png)

7. Pegue el valor de **huella digital** convertido, que ha copiado de Azure Portal y convertido al formato especificado, tal y como se menciona en el paso 9 de la sección anterior, en el cuadro de texto **Huella digital de SHA1 de certificado X.590**. A continuación, haga clic en **Save**(Guardar). Después de guardar, se mostrará la dirección URL del consumidor. Copie el valor de Consumer URL (URL de cliente) y péguelo en el cuadro de texto **URL de respuesta** de la sección **Configuración básica de SAML** en Azure Portal.

   ![Configuración de Meraki Dashboard](./media/meraki-dashboard-tutorial/configure-4.png)

### <a name="create-meraki-dashboard-test-user"></a>Creación de un usuario de prueba de Meraki Dashboard

1. En otra ventana del explorador web, inicie sesión en el sitio web de Meraki Dashboard como administrador.

1. Vaya a **Organization** (Organización)  -> **Administrators** (Administradores).

   ![Administradores de Meraki Dashboard](./media/meraki-dashboard-tutorial/user-1.png)

1. En la sección roles de administrador de SAML, haga clic en el botón **Add SAML role** (Agregar rol de SAML).

   ![Botón Add SAML role (Agregar rol de SAML) de Meraki Dashboard](./media/meraki-dashboard-tutorial/user-2.png)

1. Escriba el rol **meraki_full_admin**, marque **Organization access** (Acceso de la organización) como **Full** (Completo) y haga clic en **Create role** (Crear rol). Repita el proceso para **meraki_readonly_admin** y, esta vez, marque **Organization access** (Acceso de la organización) como **Read-only** (Solo lectura).

   ![Creación de un usuario de Meraki Dashboard](./media/meraki-dashboard-tutorial/user-3.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en Probar esta aplicación en Azure Portal. Se debería iniciar sesión automáticamente en la instancia de Meraki Dashboard para la que ha configurado el inicio de sesión único.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Meraki Dashboard en Mis aplicaciones, se debería iniciar sesión automáticamente en la versión de Meraki Dashboard para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Meraki Dashboard, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
