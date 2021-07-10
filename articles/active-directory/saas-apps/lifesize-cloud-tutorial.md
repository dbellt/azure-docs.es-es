---
title: 'Tutorial: Integración de Azure Active Directory con Lifesize Cloud | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Lifesize Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/24/2021
ms.author: jeedes
ms.openlocfilehash: 9baa2ad771bb722f655d2016a5c37b8d87f7da98
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110481246"
---
# <a name="tutorial-azure-active-directory-integration-with-lifesize-cloud"></a>Tutorial: Integración de Azure Active Directory con Lifesize Cloud

En este tutorial, aprenderá a integrar Lifesize Cloud con Azure Active Directory (Azure AD). Al integrar Lifesize Cloud con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Lifesize Cloud.
* Permitir a los usuarios iniciar sesión automáticamente en Lifesize Cloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Lifesize Cloud.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Lifesize Cloud admite el inicio de sesión único iniciado por **SP**.

* Lifesize Cloud admite el aprovisionamiento **automatizado** de usuarios.

## <a name="add-lifesize-cloud-from-the-gallery"></a>Incorporación de Lifesize Cloud desde la galería

Para configurar la integración de Lifesize Cloud en Azure AD, necesita agregar Lifesize Cloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Lifesize Cloud** en el cuadro de búsqueda.
1. Seleccione **Lifesize Cloud** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-lifesize-cloud"></a>Configuración y prueba del inicio de sesión único de Azure AD para Lifesize Cloud

Configure y pruebe el inicio de sesión único de Azure AD con Lifesize Cloud mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Lifesize Cloud.

Para configurar y probar el inicio de sesión único de Azure AD con Lifesize Cloud, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Lifesize Cloud](#configure-lifesize-cloud-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Lifesize Cloud](#create-lifesize-cloud-test-user)** : para tener un homólogo de B.Simon en Lifesize Cloud vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Lifesize Cloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://login.lifesizecloud.com/ls/?acs`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://login.lifesizecloud.com/<COMPANY_NAME>`

    c. Haga clic en **Establecer direcciones URL adicionales**.

    d. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://webapp.lifesizecloud.com/?ent=<IDENTIFIER>`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico de clientes de Lifesize Cloud Client](https://legacy.lifesize.com/en/support) para obtener los valores de la dirección URL de inicio de sesión y del identificador; el valor de estado de retransmisión lo puede obtener de la configuración del inicio de sesión único que se explica más adelante en este tutorial. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Lifesize Cloud** (Configurar Lifesize Cloud), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a Lifesize Cloud mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Lifesize Cloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-lifesize-cloud-sso"></a>Configuración del inicio de sesión único de Lifesize Cloud

1. Para configurar SSO para su aplicación, inicie sesión en la aplicación de Lifesize Cloud con privilegios de administrador.

2. En la esquina superior derecha, haga clic en su nombre y, después, en **Configuración avanzada**.

    ![Captura de pantalla que muestra el elemento de menú Advanced Settings (Configuración avanzada).](./media/lifesize-cloud-tutorial/settings.png)

3. En Configuración avanzada, haga clic en el vínculo **Configuración de SSO**. Se abrirá la página de configuración de SSO de la instancia.

    ![Captura de pantalla que muestra el elemento de menú Advanced Settings (Configuración avanzada), donde puede seleccionar S S O Configuration (Configuración de S S O).](./media/lifesize-cloud-tutorial/configuration.png)

4. Ahora, configure los valores siguientes en la interfaz de configuración de SSO.

    ![Captura de pantalla que muestra la página S S O Configuration (Configuración de S S O) donde puede especificar los valores descritos.](./media/lifesize-cloud-tutorial/values.png)

    a. En el cuadro de texto **Emisor de proveedor de identidades**, pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    b.  En el cuadro de texto **Dirección URL de inicio de sesión**, pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el certificado codificado en base 64 descargado de Azure Portal en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado X.509**.
  
    d. En las asignaciones de SAML Attribute del cuadro de texto First Name (Nombre), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    e. En las asignaciones de SAML Attribute del cuadro de texto **Last Name** (Apellidos), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    f. En las asignaciones de SAML Attribute del cuadro de texto **Email** (Dirección de correo electrónico), escriba el valor `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

5. Para comprobar la configuración, puede hacer clic en el botón **Probar**.

    >[!NOTE]
    >Para realizar una prueba correctamente, necesita completar el asistente para configuración en Azure AD y, además, proporcionar acceso los usuarios o grupos que pueden realizar la prueba.

6. Para habilitar SSO, seleccione el botón **Habilitar SSO**.

7. Ahora, haga clic en el botón **Actualizar** para guardar la configuración. Se generará el valor RelayState. Copie el valor de RelayState, que se genera en el cuadro de texto y péguelo en el cuadro de texto **Estado de la retransmisión** en la sección **Dominio y direcciones URL de Lifesize Cloud**.

### <a name="create-lifesize-cloud-test-user"></a>Creación del usuario de prueba de Lifesize Cloud

En esta sección, creará el usuario Britta Simon en Lifesize Cloud. Lifesize Cloud no admite el aprovisionamiento de usuarios automático. Después de autenticarse correctamente en Azure AD, el usuario se aprovisionará automáticamente en la aplicación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Lifesize Cloud, donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Lifesize Cloud y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Lifesize Cloud en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Lifesize Cloud. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que configure Lifesize Cloud, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
