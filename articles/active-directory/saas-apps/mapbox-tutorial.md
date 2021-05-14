---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Mapbox | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Mapbox.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/16/2020
ms.author: jeedes
ms.openlocfilehash: 3863e4a6a6429a5ce8d7558d97901b0c0e523dbb
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108801719"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-mapbox"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Mapbox

En este tutorial, aprenderá a integrar Mapbox con Azure Active Directory (Azure AD). Al integrar Mapbox con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Mapbox.
* Permitir que los usuarios inicien sesión automáticamente en Mapbox con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).


## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Mapbox.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Mapbox admite el SSO iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-mapbox-from-the-gallery"></a>Incorporación de Mapbox desde la galería

Para configurar la integración de Mapbox en Azure AD, tiene que agregar Mapbox desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Add from the gallery** (Agregar desde la galería), escriba **Mapbox** en el cuadro de búsqueda.
1. Seleccione **Mapbox** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-mapbox"></a>Configuración y prueba del inicio de sesión único de Azure AD para Mapbox

Configure y pruebe el inicio de sesión único de Azure AD con Mapbox mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una vinculación entre un usuario de Azure AD y el usuario correspondiente de Mapbox.

Para configurar y probar el inicio de sesión único de Azure AD con Mapbox, sita estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Mapbox](#configure-mapbox-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Mapbox](#create-mapbox-test-user)** , para tener un usuario correspondiente a B.Simon en Mapbox que esté vinculado a su representación de usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Mapbox**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, la aplicación está preconfigurada y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón **Guardar**.

1. La aplicación Mapbox espera las aserciones de SAML en un formato específico, por lo que es necesario agregar asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Mapbox espera que se devuelvan algunos atributos más en la respuesta de SAML que se muestran a continuación. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre   |  Atributo de origen|
    | -----|--------- |
    | rol | user.assignedroles |
    | | |

    > [!NOTE]
    > Para aprender a configurar roles en Azure AD, consulte [este vínculo](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui).

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (sin procesar)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

1. En la sección **Set up Mapbox** (Configurar Mapbox), copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección, va a permitir que B.Simon acceda a Mapbox mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Mapbox**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-mapbox-sso"></a>Configuración del inicio de sesión único de MapBox

1. Abra una nueva ventana del explorador web e inicie sesión en Mapbox como administrador.

1. Haga clic en la pestaña **Settings** (Configuración).

    ![Pestaña de configuración de Mapbox](./media/mapbox-tutorial/configure1.png)

1. Haga clic en la pestaña **Security** (Seguridad) en el panel de navegación izquierdo.

    ![Pestaña de seguridad de Mapbox](./media/mapbox-tutorial/configure2.png)

1. Haga clic en **Edit single sign-on** (Editar inicio de sesión único).

    ![Edición de inicio de sesión único de Mapbox](./media/mapbox-tutorial/configure3.png)

1. Desplácese hasta el **Step 3: Setup SAML single sign-on for Mapbox** (Paso 3: Configuración del inicio de sesión único de SAML para MapBox) y realice los pasos siguientes:

    ![Configuración de MapBox](./media/mapbox-tutorial/configure4.png)

    1. En el cuadro de texto **Idp Sign-on URL** (URL de inicio de sesión de Idp), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    1. En el cuadro de texto **Issuer ID** (Id. del emisor), pegue el valor de **Identificador de Azure AD** que ha copiado de Azure Portal.

    1. Abra el archivo **Certificado(Base64)** que descargó de Azure Portal en el Bloc de notas, copie el contenido y, luego, péguelo en el cuadro de texto **Certificado X.509**.

    1. Haga clic en **Save single sign-on settings** (Guardar configuración de inicio de sesión único).

### <a name="create-mapbox-test-user"></a>Creación de un usuario de prueba de MapBox

En esta sección, creará un usuario llamado Britta Simon en Mapbox. Trabaje con el [equipo de soporte técnico de Mapbox](mailto:help@mapbox.com) para agregar los usuarios a la plataforma de Mapbox. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Mapbox para la que ha configurado el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Mapbox en Mis aplicaciones, debería iniciar sesión automáticamente en la aplicación Mapbox para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Mapbox, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
