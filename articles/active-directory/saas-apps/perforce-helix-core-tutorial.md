---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Perforce Helix Core - Helix Authentication Service | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Perforce Helix Core - Helix Authentication Service.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2021
ms.author: jeedes
ms.openlocfilehash: c91e9090df14180a0605e3709792fae9c2f8a505
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110471216"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-perforce-helix-core---helix-authentication-service"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Perforce Helix Core - Helix Authentication Service

En este tutorial, aprenderá a integrar Perforce Helix Core - Helix Authentication Service con Azure Active Directory (Azure AD). Al integrar Perforce Helix Core - Helix Authentication Service con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Perforce Helix Core - Helix Authentication Service.
* Permitir que los usuarios inicien sesión automáticamente en Perforce Helix Core - Helix Authentication Service con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Perforce Helix Core - Helix Authentication Service.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Perforce Helix Core - Helix Authentication Service admite el inicio de sesión único iniciado por **SP**.

## <a name="add-perforce-helix-core---helix-authentication-service-from-the-gallery"></a>Adición de Perforce Helix Core - Helix Authentication Service desde la galería

Para configurar la integración de Perforce Helix Core - Helix Authentication Service en Azure AD, tiene que agregar Perforce Helix Core - Helix Authentication Service desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Perforce Helix Core - Helix Authentication Service** en el cuadro de búsqueda.
1. Seleccione **Perforce Helix Core - Helix Authentication Service** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-perforce-helix-core---helix-authentication-service"></a>Configuración y prueba del inicio de sesión único de Azure AD para Perforce Helix Core - Helix Authentication Service

Configure y pruebe el inicio de sesión único de Azure AD con Perforce Helix Core - Helix Authentication Service mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una vinculación entre un usuario de Azure AD y el usuario relacionado de Perforce Helix Core - Helix Authentication Service.

Para configurar y probar el inicio de sesión único de Azure AD con Perforce Helix Core - Helix Authentication Service, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Perforce Helix Core - Helix Authentication Service](#configure-perforce-helix-core---helix-authentication-service-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Perforce Helix Core - Helix Authentication Service](#create-perforce-helix-core---helix-authentication-service-test-user)** , para tener un usuario equivalente a B.Simon en Perforce Helix Core - Helix Authentication Service que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Perforce Helix Core - Helix Authentication Service**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<HELIX-AUTH-SERVICE>.<CUSTOMER_HOSTNAME>.com/`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<HELIX-AUTH-SERVICE>.<CUSTOMER_HOSTNAME>.com/saml`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<HELIX-AUTH-SERVICE>.<CUSTOMER_HOSTNAME>.com/saml/sso`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte técnico al cliente de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

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

En esta sección va a permitir que B.Simon acceda a Perforce Helix Core - Helix Authentication Service mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Perforce Helix Core - Helix Authentication Service**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-perforce-helix-core---helix-authentication-service-sso"></a>Configuración del inicio de sesión único de Perforce Helix Core - Helix Authentication Service

Para configurar el inicio de sesión único en **Perforce Helix Core - Helix Authentication Service**, debe enviar el valor de **Dirección URL de metadatos de federación de aplicación** al [equipo de soporte técnico de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-perforce-helix-core---helix-authentication-service-test-user"></a>Creación de un usuario de prueba de Perforce Helix Core - Helix Authentication Service

En esta sección, creará un usuario llamado B.Simon en Perforce Helix Core - Helix Authentication Service. Colabore con el [equipo de soporte técnico de Perforce Helix Core - Helix Authentication Service](mailto:support@perforce.com) para agregar los usuarios a la plataforma de Perforce Helix Core - Helix Authentication Service. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Perforce Helix Core - Helix Authentication Service, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Perforce Helix Core - Helix Authentication Service e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Helix Core - Helix Authentication Service en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Perforce Helix Core - Helix Authentication Service. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Perforce Helix Core - Helix Authentication Service, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración y la infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
