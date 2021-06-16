---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TAP App Security | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TAP App Security.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/22/2021
ms.author: jeedes
ms.openlocfilehash: 5f382f0b8403179dec1300d732350a5454a1a323
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111887211"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-tap-app-security"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con TAP App Security

En este tutorial, aprenderá a integrar TAP App Security con Azure Active Directory (Azure AD). Al integrar TAP App Security con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a TAP App Security.
* Permitir que los usuarios inicien sesión automáticamente en TAP App Security con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en TAP App Security.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TAP App Security admite el inicio de sesión único iniciado por **SP**.

* TAP App Security admite el aprovisionamiento de usuarios **Just-in-Time**.

## <a name="adding-tap-app-security-from-the-gallery"></a>Adición de TAP App Security desde la galería

Para configurar la integración de TAP App Security en Azure AD, debe agregar TAP App Security desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TAP App Security** en el cuadro de búsqueda.
1. Seleccione **TAP App Security** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-tap-app-security"></a>Configuración y prueba del inicio de sesión único de Azure AD para TAP App Security

Configure y pruebe el inicio de sesión único de Azure AD con TAP App Security con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de TAP App Security.

Para configurar y probar el inicio de sesión único de Azure AD con TAP App Security, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de TAP App Security](#configure-tap-app-security-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de TAP App Security](#create-tap-app-security-test-user)** : para tener un homólogo de B.Simon en TAP App Security que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **TAP App Security**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://app.tapappsecurity.com/<CUSTOMER_DOMAIN_WITHOUT_EXTENSION>/metadata`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://app.tapappsecurity.com/<CUSTOMER_DOMAIN_WITHOUT_EXTENSION>/acs`

    c. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://webapp.tapappsecurity.com/`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico al cliente de TAP App Security](mailto:support@tapappsecurity.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación TAP App Security espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación TAP App Security espera que se pasen algunos atributos más en la respuesta de SAML que se muestran a continuación. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | --------------- | --------- |
    | email | user.mail |
    | first | user.givenname |
    | login | user.userprincipalname |
    | last | user.surname |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar TAP App Security**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección va a permitir que B.Simon acceda a TAP App Security mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TAP App Security**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-tap-app-security-sso"></a>Configuración del inicio de sesión único de TAP App Security

1. Inicie sesión en el sitio web de TAP App Security como administrador.

1. Haga clic en **Single Sign-On** (Inicio de sesión único) en la barra de navegación izquierda y seleccione **Active Directory**.

    ![Captura de pantalla del inicio de sesión único.](./media/tap-app-security-tutorial/active-directory.png)

1. Haga clic en el botón **Integrate Active Directory app** (Integrar aplicación de Active Directory). A continuación, escriba el dominio de su organización y haga clic en el botón **Save** (Guardar).

    ![Captura de pantalla de Active Directory.](./media/tap-app-security-tutorial/domain.png)

1. Haga clic en el icono de engranaje como se muestra a continuación.

    ![Captura de pantalla del icono.](./media/tap-app-security-tutorial/gear-icon.png)

1. Realice los pasos siguientes en la página **Active Directory integration** (Integración de Active Directory).

    ![Captura de pantalla de la integración.](./media/tap-app-security-tutorial/configuration.png)

    a. Copie el valor de **Reply URL (Assertion Consumer Service URL)** (Dirección URL de respuesta [URL del Servicio de consumidor de aserciones]) y péguelo en el cuadro **URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    b. Copie el valor de **Identifier (Entity ID)** (Identificador [id. de entidad]) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    c. Haga clic en **Choose File** (Elegir archivo) y cargue el archivo **XML de metadatos de federación** descargado de Azure Portal.

    d. Haga clic en **Save**(Guardar).

### <a name="create-tap-app-security-test-user"></a>Creación de un usuario de prueba de TAP App Security

En esta sección, se crea un usuario llamado B.Simon en TAP App Security. TAP App Security admite el aprovisionamiento Just-In-Time, que está habilitado de manera predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe aún en TAP App Security, se crea uno nuevo cuando se intenta acceder a TAP App Security.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de TAP App Security, desde donde podrá comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de TAP App Security e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de TAP App Security en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de TAP App Security. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado TAP App Security, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).