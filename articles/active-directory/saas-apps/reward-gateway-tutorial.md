---
title: 'Tutorial: Integración de Azure Active Directory con Reward Gateway | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Reward Gateway.
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
ms.openlocfilehash: 9d7c018319003ef9671d9403c2e17642ec70a5d4
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110476426"
---
# <a name="tutorial-azure-active-directory-integration-with-reward-gateway"></a>Tutorial: Integración de Azure Active Directory con Reward Gateway

En este tutorial, aprenderá a integrar Reward Gateway con Azure Active Directory (Azure AD). Al integrar Reward Gateway con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Reward Gateway.
* Permitir que los usuarios inicien sesión automáticamente en Reward Gateway con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Reward Gateway, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Reward Gateway.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Reward Gateway admite el inicio de sesión único iniciado por **IDP**.

## <a name="add-reward-gateway-from-the-gallery"></a>Adición de Reward Gateway desde la galería

Para configurar la integración de Reward Gateway en Azure AD, deberá agregar Reward Gateway desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Reward Gateway** en el cuadro de búsqueda.
1. Seleccione **Reward Gateway** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-reward-gateway"></a>Configuración y prueba del inicio de sesión único de Azure AD para Reward Gateway

Configure y pruebe el inicio de sesión único de Azure AD con Reward Gateway mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso vincular un usuario de Azure AD y el usuario de Reward Gateway correspondiente.

Para configurar y probar el inicio de sesión único de Azure AD con Reward Gateway, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Reward Gateway](#configure-reward-gateway-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba en Reward Gateway](#create-reward-gateway-test-user)** : el objetivo es tener un homólogo de B.Simon en Reward Gateway vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Reward Gateway**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    | URL del identificador |
    |---|
    | `https://<COMPANY_NAME>.rewardgateway.com` |
    | `https://<COMPANY_NAME>.rewardgateway.co.uk/` |
    | `https://<COMPANY_NAME>.rewardgateway.co.nz/` |
    | `https://<COMPANY_NAME>.rewardgateway.com.au/`|
    |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:

    | URL de respuesta |
    |---|
    | `https://<COMPANY_NAME>.rewardgateway.com/Authentication/EndLogin?idp=<Unique Id>` |
    | `https://<COMPANY_NAME>.rewardgateway.co.uk/Authentication/EndLogin?idp=<Unique Id>` |
    | `https://<COMPANY_NAME>.rewardgateway.co.nz/Authentication/EndLogin?idp=<Unique Id>` |
    | `https://<COMPANY_NAME>.rewardgateway.com.au/Authentication/EndLogin?idp=<Unique Id>` |
    |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Para obtener estos valores empiece por configurar una integración en el portal de Reward Manager. Encontrará más información en https://success.rewardgateway.com/hc/en-us/articles/360038650573-Microsoft-Azure-for-Authentication.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Reward Gateway** (Configurar Reward Gateway), copie las direcciones URL que necesite.

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

En esta sección va a permitir que B.Simon acceda a Reward Gateway mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Reward Gateway**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-reward-gateway-sso"></a>Configuración del inicio de sesión único de Reward Gateway

Para configurar el inicio de sesión único en **Reward Gateway**, empiece por configurar una integración en el portal de Reward Manager. Utilice los metadatos descargados para obtener el certificado de firma y cárguelos durante la configuración. Encontrará más información en https://success.rewardgateway.com/hc/en-us/articles/360038650573-Microsoft-Azure-for-Authentication.

### <a name="create-reward-gateway-test-user"></a>Creación del usuario de prueba en Reward Gateway

En esta sección, creará un usuario denominado Britta Simon en Reward Gateway. Trabaje con el [equipo de soporte técnico de Reward Gateway](mailto:clientsupport@rewardgateway.com) para agregar los usuarios a la plataforma de Reward Gateway. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Reward Gateway para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Reward Gateway en Aplicaciones, debería iniciar sesión automáticamente en la versión de Reward Gateway para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Reward Gateway, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
