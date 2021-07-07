---
title: 'Tutorial: Integración de Azure Active Directory con 123FormBuilder SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y 123FormBuilder SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: jeedes
ms.openlocfilehash: 524cd127e4ee05134d45e67bb97d544d4fdfb325
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451700"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-123formbuilder-sso"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con 123FormBuilder SSO

En este tutorial aprenderá a integrar 123FormBuilder SSO con Azure Active Directory (Azure AD). Al integrar 123FormBuilder SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a 123FormBuilder SSO.
* Permitir que los usuarios inicien sesión automáticamente en 123FormBuilder SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en 123FormBuilder SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* 123FormBuilder SSO admite el inicio de sesión único iniciado por **SP e IDP**.
* 123FormBuilder SSO admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-123formbuilder-sso-from-the-gallery"></a>Incorporación de 123FormBuilder SSO desde la galería

Para configurar la integración de 123FormBuilder SSO en Azure AD, es preciso agregar 123FormBuilder SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **123FormBuilder SSO** en el cuadro de búsqueda.
1. Seleccione **123FormBuilder SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-123formbuilder-sso"></a>Configuración y prueba de SSO de Azure AD para 123FormBuilder SSO

Configure y pruebe el inicio de sesión único de Azure AD con 123FormBuilder SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de 123FormBuilder SSO.

Para configurar y probar el SSO de Azure AD con 123FormBuilder SSO, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de 123FormBuilder SSO](#configure-123formbuilder-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de 123FormBuilder SSO](#create-123formbuilder-sso-test-user)** , para tener un homólogo de B.Simon en 123FormBuilder SSO vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **123FormBuilder SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://www.123formbuilder.com/saml/azure_ad/<TENANT_ID>/metadata`


    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.123formbuilder.com/saml/azure_ad/<TENANT_ID>/acs`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.123formbuilder.com/saml/azure_ad/<TENANT_ID>/sso`

    > [!NOTE]
    > Estos valores no son reales. Debe actualizar este valor con las direcciones URL y el identificador reales, como se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar 123FormBuilder SSO**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a 123FormBuilder SSO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **123FormBuilder SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-123formbuilder-sso"></a>Configuración del inicio de sesión único de 123FormBuilder SSO

1. Para configurar el inicio de sesión único en **123FormBuilder SSO**, vaya a [https://www.123formbuilder.com/form-2709121/](https://www.123formbuilder.com/form-2709121/) y siga estos pasos:

    ![Captura de pantalla que muestra la pantalla de configuración SSO SAML - Identity Provider (SSO de SAML: proveedor de identidades).](./media/123formbuilder-tutorial/submit.png) 

    a. En el cuadro de texto **Email** (Correo electrónico), escriba el correo electrónico del usuario, en el ejemplo `B.Simon@Contoso.com`.

    b. Haga clic en **Upload** (Cargar) y vaya al archivo XML de metadatos que ha descargado de Azure Portal.

    c. Haga clic en **ENVIAR FORMULARIO**.

2. En **Microsoft Azure AD - Inicio de sesión único - Configurar las opciones de la aplicación**, realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/123formbuilder-tutorial/configuration.png)

    a. Si desea configurar la aplicación en el **modo iniciado por IDP**, copie el valor de **IDENTIFIER** (IDENTIFICADOR) de la instancia y péguelo en el cuadro de texto **Identificador** en la sección **Configuración básica de SAML** en Azure Portal.

    b. Si desea configurar la aplicación en el **modo iniciado por IDP**, copie el valor de **REPLY URL** (URL DE RESPUESTA) de la instancia y péguelo en el cuadro de texto **URL de respuesta** en la sección **Configuración básica de SAML**  en Azure Portal.

    c. Si desea configurar la aplicación en el **modo iniciado por SP**, copie el valor de **SIGN ON URL** (URL DE INICIO DE SESIÓN) de la instancia y péguelo en el cuadro de texto **URL de inicio de sesión** en la sección **Configuración básica de SAML**  en Azure Portal.

### <a name="create-123formbuilder-sso-test-user"></a>Creación de un usuario de prueba de 123FormBuilder SSO

En esta sección, se crea un usuario llamado Britta Simon en 123FormBuilder SSO. 123FormBuilder SSO admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe ya en 123FormBuilder SSO, se crea después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de 123FormBuilder SSO, donde puede poner en marcha el flujo de inicio de sesión.  

* Acceda directamente a la dirección URL de inicio de sesión de 123FormBuilder SSO y ponga en marcha el flujo de inicio de sesión desde ahí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal y debería iniciar sesión automáticamente en la instancia de 123FormBuilder SSO para la que configuró el SSO. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de 123FormBuilder SSO en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de 123FormBuilder SSO para la que ha configurado el SSO. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado 123FormBuilder SSO, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
