---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler B2B User Portal | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler B2B User Portal.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/08/2021
ms.author: jeedes
ms.openlocfilehash: b3b44505cd96640e3ef1e1b1766ffb194413b450
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111896115"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-zscaler-b2b-user-portal"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler B2B User Portal

En este tutorial aprenderá a integrar Zscaler B2B User Portal con Azure Active Directory (Azure AD). Al integrar Zscaler B2B User Portal con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Zscaler B2B User Portal.
* Permitir que los usuarios inicien sesión automáticamente en Zscaler B2B User Portal con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Zscaler B2B User Portal.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Zscaler B2B User Portal admite el inicio de sesión único iniciado por **IDP**.

* Zscaler B2B User Portal admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-zscaler-b2b-user-portal-from-the-gallery"></a>Incorporación de Zscaler B2B User Portal desde la galería

Para configurar la integración de Zscaler B2B User Portal en Azure AD, deberá agregar Zscaler B2B User Portal desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler B2B User Portal** en el cuadro de búsqueda.
1. Seleccione **Zscaler B2B User Portal** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-zscaler-b2b-user-portal"></a>Configuración y prueba del inicio de sesión único de Azure AD para Zscaler B2B User Portal

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler B2B User Portal mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler B2B User Portal.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler B2B User Portal, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Zscaler B2B User Portal](#configure-zscaler-b2b-user-portal-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Zscaler B2B User Portal](#create-zscaler-b2b-user-portal-test-user)** , para tener un homólogo de B.Simon en Zscaler B2B User Portal vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Zscaler B2B User Portal**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://samlsp.private.zscaler.com/auth/metadata/<UniqueID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://samlsp.private.zscaler.com/auth/login?domain=EXAMPLE`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Zscaler B2B User Portal](https://help.zscaler.com/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Zscaler B2B User Portal**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a Zscaler B2B User Portal mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Zscaler B2B User Portal**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-zscaler-b2b-user-portal-sso"></a>Configuración del inicio de sesión único de Zscaler B2B User Portal

1. Abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Zscaler B2B User Portal como administrador y lleve a cabo los siguientes pasos:

1. En la parte izquierda del menú, haga clic en **Administration** (Administración) y vaya a la sección **AUTHENTICATION** (Autenticación) y haga clic en **IdP Configuration** (Configuración de IdP).

    ![Administrador de Zscaler Private Access Administrator](./media/zscaler-b2b-user-tutorial/tutorial-zscaler-private-access-administration.png)

1. En la esquina superior derecha, haga clic en **Add IdP Configuration** (Agregar configuración de IdP). 

    ![IDP de Administrador de Zscaler Private Access](./media/zscaler-b2b-user-tutorial/tutorial-zscaler-private-access-idp.png)

1. En la página **Add IdP Configuration** (Agregar configuración de IdP), realice estos pasos:
 
    ![Selección de Administrador de Zscaler Private Access](./media/zscaler-b2b-user-tutorial/tutorial-zscaler-private-access-select.png)

    a. Haga clic en **Select File** (Seleccionar archivo) para cargar el archivo de metadatos descargado de Azure AD en el campo **IdP Metadata File Upload** (Carga de archivo de metadatos de IdP).

    b. Lee los **metadatos de IdP** de Azure AD y rellena la información de todos los campos como se indica a continuación.

    ![Configuración de Administrador de Zscaler Private Access](./media/zscaler-b2b-user-tutorial/config.png)

    c. Seleccione el dominio en el campo **Domains** (Dominios).
    
    d. Haga clic en **Save**(Guardar).

### <a name="create-zscaler-b2b-user-portal-test-user"></a>Creación de un usuario de prueba de Zscaler B2B User Portal

En esta sección, se crea una usuaria llamada Britta Simon en Zscaler B2B User Portal. Zscaler B2B User Portal admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe ya en Zscaler B2B User Portal, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Zscaler B2B User Portal para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Zscaler B2B User Portal en Aplicaciones, debería iniciar sesión automáticamente en la versión de Zscaler B2B User Portal para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Zscaler B2B User Portal, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
