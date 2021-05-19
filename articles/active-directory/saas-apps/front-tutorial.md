---
title: 'Tutorial: Integración de Azure Active Directory con Front | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Front.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2021
ms.author: jeedes
ms.openlocfilehash: 2b07317445ec5992b07a280e107546d6e430d3ca
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734055"
---
# <a name="tutorial-azure-active-directory-integration-with-front"></a>Tutorial: integración de Azure Active Directory con Front

En este tutorial, obtendrá información sobre cómo integrar Front con Azure Active Directory (Azure AD). Al integrar Front con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Front.
* Permitir que los usuarios inicien sesión automáticamente en Front con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Front.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Front admite el inicio de sesión único iniciado por **IDP**.

## <a name="adding-front-from-the-gallery"></a>Adición de Front desde la galería

Para configurar la integración de Front en Azure AD, deberá agregar Front desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Front** en el cuadro de búsqueda.
1. Seleccione **Front** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-front"></a>Configuración y prueba del inicio de sesión único de Azure AD en Front

Configure y pruebe el inicio de sesión único de Azure AD con Front mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Front.

Para configurar y probar el inicio de sesión único de Azure AD con Front, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Front](#configure-front-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación del usuario de prueba de Front](#create-front-test-user)** : para tener un homólogo de B.Simon en Front vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Front**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<companyname>.frontapp.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.frontapp.com/sso/saml/callback`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico para clientes de Front](mailto:support@frontapp.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Set up Front** (Configurar Front), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a Front mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Front**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.


## <a name="configure-front-sso"></a>Configuración del inicio de sesión único en Front

1. Inicie sesión en el sitio web de Front como administrador.

2. Vaya a la **configuración y** seleccione las **preferencias**.

3. Realice los pasos siguientes en la página **Company preferences** (Preferencias de la compañía).
   
    ![Captura de pantalla que muestra la sección "Company preferences" (Preferencias de la compañía) con el vínculo "Single Sign On" (Inicio de sesión único) seleccionado.](./media/front-tutorial/single-sign-on.png)

    a. Haga clic en **Single Sign On** (Inicio de sesión único) en el panel de navegación izquierdo.

    b. Seleccione **SAML** en la lista desplegable de **Single Sign On** (Inicio de sesión único).

    c. En el cuadro de texto **Entry Point** (Punto de entrada), pegue el valor de **Dirección URL de inicio de sesión** que copió de Azure Portal.

    d. Seleccione el tipo **Requested authentication context** (Contexto de autenticación solicitado) como **Disabled** (Deshabilitado).

    e. Abra el archivo descargado de **certificado (Base64)** en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de firma**.

7. En la sección **Service provider settings** (configuración del proveedor de servicios), siga estos pasos:

    ![Configuración del inicio de sesión único en la aplicación](./media/front-tutorial/service-provider.png)

    a. Copie el valor de **Entity ID** (Identificador de entidad) y péguelo en el cuadro de texto **Identificador**, que se encuentra en la sección **Dominio y direcciones URL de Front** de Azure Portal.

    b. Copie el valor de **ACS URL** y péguelo en el cuadro de texto **URL de respuesta**, que se encuentra en la sección **Dominio y direcciones URL de Front** de Azure Portal.
    
8. Haga clic en el botón **Guardar** .


### <a name="create-front-test-user"></a>Creación del usuario de prueba de Front

En esta sección, creará un usuario llamado Britta Simon en Front. Trabaje con el [equipo de soporte técnico al cliente de Front](mailto:support@frontapp.com) para agregar los usuarios a la plataforma de Front. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Front para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Front en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Front para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Front, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).