---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Single Sign-on for Skytap | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Single Sign-on for Skytap.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/28/2021
ms.author: jeedes
ms.openlocfilehash: 4afda5105cf00faeb818eef71f9a645ce0ff8bf7
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111557583"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-single-sign-on-for-skytap"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Single Sign-on for Skytap

En este tutorial, aprenderá a integrar Single Sign-on for Skytap con Azure Active Directory (Azure AD). Al integrar Single Sign-on for Skytap con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Single Sign-on for Skytap.
* Permitir que los usuarios inicien sesión automáticamente en Single Sign-on for Skytap con sus cuentas de Azure AD.
* Puede administrar sus cuentas en una ubicación central, Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Single Sign-on for Skytap.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Single Sign-on for Skytap admite el inicio de sesión único iniciado por SP e IDP.

## <a name="add-single-sign-on-for-skytap-from-the-gallery"></a>Adición de Single Sign-on for Skytap desde la galería

Para configurar la integración de Single Sign-on for Skytap en Azure AD, es preciso que agregue Single Sign-on for Skytap desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Single Sign-on for Skytap** en el cuadro de búsqueda.
1. Seleccione **Single Sign-on for Skytap** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-single-sign-on-for-skytap"></a>Configuración y prueba del inicio de sesión único de Azure AD para Single Sign-on for Skytap

Configure y pruebe el inicio de sesión único de Azure AD con Single Sign-on for Skytap mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Single Sign-on for Skytap.

Estos son los pasos generales para configurar y probar el inicio de sesión único de Azure AD con Single Sign-on for Skytap:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Single Sign-on for Skytap](#configure-single-sign-on-for-skytap-sso)** para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Single Sign-on for Skytap](#create-single-sign-on-for-skytap-test-user)** para tener un homólogo de B.Simon en Single Sign-on for Skytap. Este homólogo está vinculado a la representación del usuario en Azure AD.
1. **[Comprobación del inicio de sesión único](#test-sso)** , para verificar que la configuración funciona correctamente.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación de **Single Sign-on for Skytap**, busque la sección **Administrar** y seleccione **inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)
   
1. En la sección **Configuración básica de SAML**, si quiere configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `http://pingone.com/<custom EntityID>`.

    b. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://sso.connect.pingidentity.com/sso/sp/ACS.saml2`

1. Opcionalmente, puede seleccionar **Establecer direcciones URL adicionales** y seguir estos pasos si desea configurar la aplicación en el modo iniciado por **SP**:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://sso.connect.pingidentity.com/sso/sp/initsso?saasid=<saasid>&idpid=<idpid>`.

    
    b. En el cuadro de texto **Estado de la retransmisión**, escriba una dirección URL que siga este patrón: `https://pingone.com/1.0/<custom ID>`.

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con el identificador real, la dirección URL de respuesta, la dirección URL de inicio de sesión y el estado de la retransmisión. Póngase en contacto con el [equipo de soporte técnico al cliente de Single Sign-on for Skytap](mailto:support@skytap.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configuración del inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación**. Seleccione **Descargar** para descargar el archivo de metadatos y guárdelo en el equipo.

    ![Captura de pantalla del vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Single Sign-on for Skytap**, copie las direcciones URL adecuadas según sus necesidades.

    ![Captura de pantalla de la copia de las direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En Azure Portal, en el panel izquierdo, seleccione **Azure Active Directory** > **Usuarios** > **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Seleccione la casilla **Mostrar contraseña** y, después, anote el valor que se muestra en el campo **Contraseña**.
   1. Seleccione **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección va a permitir que B.Simon acceda a Single Sign-on for Skytap mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Single Sign-on for Skytap**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-single-sign-on-for-skytap-sso"></a>Configuración del inicio de sesión único en Single Sign-on for Skytap

Para configurar el inicio de sesión único en Single Sign-on for Skytap, es preciso enviar el contenido de **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Single Sign-on for Skytap](mailto:support@skytap.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-single-sign-on-for-skytap-test-user"></a>Creación de un usuario de prueba en Single Sign-on for Skytap

En esta sección, va a crear un usuario llamado B.Simon en Single Sign-on for Skytap. Trabaje con el [equipo de soporte técnico al cliente de Single Sign-on for Skytap](mailto:support@skytap.com) para agregar los usuarios a la plataforma de Single Sign-on for Skytap. No puede usar el inicio de sesión único hasta que cree y active los usuarios.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Single Sign-on for Skytap, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Single Sign-on for Skytap e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Single Sign-on for Skytap para la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Single Sign-on for Skytap en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Single Sign-on for Skytap para la que configuró el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Single Sign-on for Skytap, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
