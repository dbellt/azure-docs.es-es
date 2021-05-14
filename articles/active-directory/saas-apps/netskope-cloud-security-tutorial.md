---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netskope Administrator Console | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Netskope Administrator Console.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/02/2021
ms.author: jeedes
ms.openlocfilehash: 5414d263f8cf707736f4df98a2abbbd89a94b1b4
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804815"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netskope-administrator-console"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Netskope Administrator Console

En este tutorial aprenderá a integrar Netskope Administrator Console con Azure Active Directory (Azure AD). Al integrar Netskope Administrator Console con Azure AD, puede:

- Controlar en Azure AD quién tiene acceso a Netskope Administrator Console.
- Permitir que los usuarios inicien sesión automáticamente en Netskope Administrator Console con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción habilitada para inicio de sesión único (SSO) en Netskope Administrator Console.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Netskope Administrator Console admite el inicio de sesión único comenzado por **SP e IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-netskope-administrator-console-from-the-gallery"></a>Incorporación de Netskope Administrator Console desde la galería

Para configurar la integración de Netskope Administrator Console en Azure AD, debe agregarla desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Netskope Administrator Console** en el cuadro de búsqueda.
1. Seleccione **Netskope Administrator Console** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-netskope-administrator-console"></a>Configuración y prueba del inicio de sesión único de Azure AD para Netskope Administrator Console

Configure y pruebe el inicio de sesión único de Azure AD con Netskope Administrator Console con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Netskope Administrator Console.

Para configurar y probar el inicio de sesión único de Azure AD con Netskope Administrator Console, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Netskope Administrator Console](#configure-netskope-administrator-console-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba en Netskope Administrator Console](#create-netskope-administrator-console-test-user)** : para tener un homólogo de B.Simon en Netskope Administrator Console vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Netskope Administrator Console**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

   En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<tenant_host_name>/saml/acs`

   > [!NOTE]
   > Este valor no es real. Actualícelo con la dirección URL de respuesta real. Este valor se explica más adelante en el tutorial.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

   En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantname>.goskope.com`

   > [!NOTE]
   > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto [el equipo de soporte técnico al cliente de Netskope Administrator Console](mailto:support@netskope.com) para obtener el valor de la dirección URL de inicio de sesión. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Netskope Administrator Console espera las aserciones de SAML en un formato específico, lo cual requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

   ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Netskope Administrator Console espera que se pasen algunos atributos más en la respuesta de SAML que se muestran a continuación. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | Nombre       | Atributo de origen   |
   | ---------- | ------------------ |
   | admin-role | user.assignedroles |

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para aprender a crear roles en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Netskope Administrator Console** (Configurar Netskope Administrator Console), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Netskope Administrator Console mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Netskope Administrator Console**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-netskope-administrator-console-sso"></a>Configuración del inicio de sesión único en Netskope Administrator Console

1. Abra una nueva pestaña en el explorador e inicie sesión en el sitio de la empresa de Netskope Administrator Console como administrador.

1. Haga clic en la pestaña **Settings** (Configuración) en el panel de navegación izquierdo.

   ![Captura de pantalla que muestra la opción Setting (Configuración) seleccionada en el panel de navegación.](./media/netskope-cloud-security-tutorial/configure-settings.png)

1. Haga clic en la pestaña **Administration** (Administración).

   ![Captura de pantalla que muestra la opción Administration (Administración) seleccionada en Settings (Configuración)](./media/netskope-cloud-security-tutorial/administration.png)

1. Haga clic en **SSO** (Inicio de sesión único).

   ![Captura de pantalla que muestra S S O (Inicio de sesión único) seleccionado en Administration (Administración).](./media/netskope-cloud-security-tutorial/tab.png)

1. En la sección **Network Settings** (Configuración de red), lleve a cabo estos pasos:

   ![Captura de pantalla que muestra la opción Network Settings (Configuración de red), donde puede especificar los valores descritos.](./media/netskope-cloud-security-tutorial/network.png)

   a. Copie el valor de **Assertion Consumer Service URL** (URL del Servicio de consumidor de aserciones) y péguelo en el cuadro de texto **URL de respuesta** de **Configuración básica de SAML** en Azure Portal.

   b. Copie el valor de **Service Provider Entity ID** (Id. de entidad del proveedor de servicios) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

1. Haga clic en **EDITAR CONFIGURACIÓN** en la sección **SSO/SLO Settings** (Configuración de SSO/SLO).

   ![Captura de pantalla de S S O / S L O Settings (Configuración de S S O/S L O) donde puede seleccionar EDITAR CONFIGURACIÓN.](./media/netskope-cloud-security-tutorial/settings.png)

1. En la página emergente **Configuración**, realice los pasos siguientes:

   ![Captura de pantalla que muestra el cuadro de diálogo Configuración, donde puede especificar los valores que se han indicado.](./media/netskope-cloud-security-tutorial/configuration.png)

   a. Seleccione **Enable SSO** (Habilitar SSO).

   b. En el cuadro de texto **IDP URL** (URL de IDP), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

   c. En el cuadro de texto **IDENTIFICADOR DE LA ENTIDAD DE IDP**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

   d. Abra el certificado descargado con codificación Base64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **IDP CERTIFICATE** (CERTIFICADO DE IDP).

   e. Seleccione **Enable SSO** (Habilitar SSO).

   f. En el cuadro de texto **IDP SLO URL** (URL DE CIERRE DE SESIÓN DE IDP), pegue el valor de **URL de cierre de sesión** que ha copiado de Azure Portal.

   g. Haga clic en **ENVIAR**.

### <a name="create-netskope-administrator-console-test-user"></a>Creación de un usuario de prueba en Netskope Administrator Console

1. Abra una nueva pestaña en el explorador e inicie sesión en el sitio de la empresa de Netskope Administrator Console como administrador.

1. Haga clic en la pestaña **Settings** (Configuración) en el panel de navegación izquierdo.

   ![Captura de pantalla de la pestaña Settings (Configuración) seleccionada.](./media/netskope-cloud-security-tutorial/configure-settings.png)

1. Haga clic en la pestaña **Active Platform** (Plataforma activa).

   ![Captura de pantalla que muestra la opción Active Platform (Plataforma activa) seleccionada en Settings (Configuración).](./media/netskope-cloud-security-tutorial/user-1.png)

1. Haga clic en la pestaña **Users** (Usuarios).

   ![Captura de pantalla que muestra la opción Users (Usuarios) seleccionada en Active Platform (Plataforma activa).](./media/netskope-cloud-security-tutorial/add-user.png)

1. Haga clic en **ADD USERS** (AGREGAR USUARIOS).

   ![Captura de pantalla que muestra el cuadro de diálogo Users (Usuarios) donde puede seleccionar ADD USERS (AGREGAR USUARIOS).](./media/netskope-cloud-security-tutorial/user-add.png)

1. Escriba la dirección de correo electrónico del usuario al que desea agregar y haga clic en **ADD** (AGREGAR).

   ![Captura de pantalla que muestra la opción Add Users (Agregar usuarios) donde puede especificar una lista de usuarios.](./media/netskope-cloud-security-tutorial/add-user-popup.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

#### <a name="sp-initiated"></a>Iniciado por SP:

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Netskope Administrator Console, donde puede iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Netskope Administrator Console y comience el flujo de inicio de sesión desde ahí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

- Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Netskope Administrator Console para la que configuró el inicio de sesión único.

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Netskope Administrator Console en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Netskope Administrator Console para la que haya configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Netskope Administrator Console, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
