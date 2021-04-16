---
title: 'Tutorial: Integración de Azure Active Directory con Greenhouse | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Greenhouse.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/26/2021
ms.author: jeedes
ms.openlocfilehash: 6a14b16e34faa827228594bf6d4f0bd9ed48cf72
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221762"
---
# <a name="tutorial-azure-active-directory-integration-with-greenhouse"></a>Tutorial: integración de Azure Active Directory con Greenhouse

En este tutorial, obtendrá información sobre cómo integrar Greenhouse con Azure Active Directory (Azure AD). Al integrar Greenhouse con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Greenhouse.
* Permitir que los usuarios inicien sesión automáticamente en Greenhouse con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Greenhouse.

> [!NOTE] 
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública. 

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Greenhouse admite el inicio de sesión único iniciado por **SP e IDP**.

## <a name="adding-greenhouse-from-the-gallery"></a>Agregación de Greenhouse desde la galería

Para configurar la integración de Greenhouse en Azure AD, deberá agregar Greenhouse desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Greenhouse** en el cuadro de búsqueda.
1. Seleccione **Greenhouse** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-greenhouse"></a>Configuración y prueba del inicio de sesión único de Azure AD para Greenhouse

Configure y pruebe el inicio de sesión único de Azure AD con Greenhouse mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Greenhouse.

Para configurar y probar el inicio de sesión único de Azure AD con Greenhouse, es preciso completar los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Greenhouse](#configure-greenhouse-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Greenhouse](#create-greenhouse-test-user)**: para tener un homólogo de Britta Simon en Greenhouse que esté vinculado a su representación en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Greenhouse**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

    ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<COMPANYNAME>.greenhouse.io`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:
    
    | URL de respuesta|
    | -------------- |
    | `https://<COMPANYNAME>.greenhouse.io/users/saml/consume` |
    | `https://app.greenhouse.io/<ENTITY ID>/users/saml/consume` |
    |

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<COMPANYNAME>.greenhouse.io`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y las direcciones URL de inicio de sesión y de respuesta reales. Póngase en contacto con el [equipo de soporte de cliente de Greenhouse](https://www.greenhouse.io/contact) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar Greenhouse**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Greenhouse mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Greenhouse**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-greenhouse-sso"></a>Configuración del inicio de sesión único de Greenhouse

1. En otra ventana del explorador web, inicie sesión en el sitio web de Greenhouse como administrador.

1. Vaya a **Configure > Dev Center > Single Sign-On** (Configurar > Centro de desarrollo > Inicio de sesión único).

    ![Captura de pantalla de la página de inicio de sesión único](./media/greenhouse-tutorial/configure.png)

1. En la página **Single sign-on** (Inicio de sesión único) realice los pasos siguientes.

    ![Captura de pantalla de la página de configuración del inicio de sesión único](./media/greenhouse-tutorial/sso-page.png)

    a. Copie el valor de **SSO Assertion Consumer URL** (Dirección URL del consumidor de aserciones de SSO) y péguelo en el cuadro de texto **URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    b. En el cuadro de texto **Entity ID/Issuer** (Identificador de entidad/Emisor), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    c. En el cuadro de texto **Single Sign On URL** (Dirección URL de inicio de sesión único), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    d. Abra el archivo **XML de metadatos de federación** descargado de Azure Portal en el Bloc de notas, copie el contenido del XML de metadatos y péguelo en el cuadro de texto **IdP Certificate Fingerprint** (Huella digital del certificado del IdP).

    e. Seleccione el valor de **Name Identifier Format** (Formato de identificador de nombre) en la lista desplegable.

    f. Haga clic en **Begin Testing** (Comenzar pruebas).

    >[!NOTE]
    >También puede cargar el archivo **XML de metadatos de federación** haciendo clic en la opción **Choose File** (Elegir archivo).

### <a name="create-greenhouse-test-user"></a>Creación de un usuario de prueba de Greenhouse

Para permitir que los usuarios de Azure AD inicien sesión en Greenhouse, deben aprovisionarse en Greenhouse. En el caso de Greenhouse, el aprovisionamiento es una tarea manual.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Greenhouse que proporcione Greenhouse para aprovisionar cuentas de usuario de Azure AD. 

**Para aprovisionar cuentas de usuario, realice estos pasos:**

1. Inicie sesión en el sitio de la compañía de **Greenhouse** como administrador.

2. Vaya a **Configure > Users > New Users** (Configurar > Usuarios > Nuevos usuarios).
   
    ![Usuarios](./media/greenhouse-tutorial/create-user-1.png "Usuarios")

4. En la sección **Agregar nuevos usuarios**, realice los siguientes pasos:
   
    ![Agregar nuevo usuario](./media/greenhouse-tutorial/create-user-2.png "Agregar nuevo usuario")

    a. En el cuadro de texto **Escribir correos electrónicos del usuario**, escriba la dirección de correo electrónico de una cuenta válida de Azure Active Directory que quiera aprovisionar.

    b. Haga clic en **Save**(Guardar).    
   
      >[!NOTE]
      >El titular de la cuenta de Azure Active Directory recibirá un mensaje de correo electrónico con un vínculo para confirmar la cuenta antes de que se active.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Greenhouse, donde podrá iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Greenhouse e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal y debería iniciar sesión automáticamente en la instancia de Greenhouse para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Greenhouse de Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Greenhouse para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).



## <a name="next-steps"></a>Pasos siguientes

Una vez configurada la aplicación Greenhouse, podrá aplicar el control de sesión, que protege a su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).