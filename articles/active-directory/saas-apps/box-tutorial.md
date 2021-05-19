---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Box | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Box.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2021
ms.author: jeedes
ms.openlocfilehash: 79153734beb9ddc8bb43c1c4928844c5d4bd18e7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108740004"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-box"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Box

En este tutorial, obtendrá información sobre cómo integrar Box con Azure Active Directory (Azure AD). Al integrar Box con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Box.
* Permitir que los usuarios inicien sesión automáticamente en Box con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Box.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Box admite el inicio de sesión único iniciado por **SP**
* Box admite el [aprovisionamiento y desaprovisionamiento **automático** de usuarios](./box-userprovisioning-tutorial.md) (recomendado).
* Box admite el aprovisionamiento de usuarios **Just-In-Time**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-box-from-the-gallery"></a>Adición de Box desde la galería

Para configurar la integración de Box en Azure AD, es preciso agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Box** en el cuadro de búsqueda.
1. Seleccione **Box** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-box"></a>Configuración y prueba del inicio de sesión único de Azure AD para Box

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con Box mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Box.

Para configurar y probar el inicio de sesión único de Azure AD con Box, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Box](#configure-box-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Box](#create-box-test-user)** , para tener un homólogo de B. Simon en Box vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Box**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.account.box.com`

    b. En el cuadro de texto **Identificador (Id. de entidad)** , escriba una dirección URL: `box.net`

    c. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://sso.services.box.net/sp/ACS.saml2`

    > [!NOTE]
    > El valor de la dirección URL de inicio de sesión no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de atención al cliente de Box](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Box espera las aserciones de SAML en un formato específico, por lo que es necesario agregar asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla le muestra un ejemplo de esto. El valor predeterminado de **Identificador de usuario único** es **user.userprincipalname**, pero Box espera que este valor se asigne a la dirección de correo electrónico del usuario. Para ello, puede usar el atributo **user.mail** de la lista o usar el valor de atributo correspondiente en función de la configuración de su organización.

    ![imagen](common/default-attributes.png)


1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

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

En esta sección, va a permitir que B. Simon acceda a Box mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Box**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-box-sso"></a>Configuración del inicio de sesión único de Box

1. Para automatizar la configuración en Box, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Set up Box** (Configurar Box) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en Box. La extensión de explorador configurará automáticamente la aplicación y automatizará el paso 3.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Box manualmente, abra otra ventana del explorador web e inicie sesión como administrador en el sitio web de Box y siga el procedimiento de [Configuración del inicio de sesión único por su cuenta](https://community.box.com/t5/How-to-Guides-for-Admins/Setting-Up-Single-Sign-On-SSO-for-your-Enterprise/ta-p/1263#ssoonyourown).

> [!NOTE]
> Si no puede configurar la configuración de SSO para su cuenta de Box, deberá enviar el **XML de metadatos de federación** descargado al [equipo de soporte técnico de Box](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-box-test-user"></a>Creación de un usuario de prueba de Box

En esta sección, se crea un usuario llamado a Britta Simon en Box. Box admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Box, se crea uno nuevo después de la autenticación.

> [!NOTE]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Box](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* En Azure Portal, seleccione **Probar esta aplicación**. Esta acción le redirigirá a la dirección URL de inicio de sesión de Box, donde puede comenzar el flujo de inicio de sesión.

* Vaya directamente a la dirección URL de inicio de sesión de Box e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Box en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

### <a name="push-an-azure-group-to-box"></a>Inserción de un grupo de Azure en Box

Puede insertar un grupo de Azure en Box y sincronizar ese grupo. Azure inserta grupos en Box mediante una integración de nivel de API.

1. En **Usuarios y grupos**, busque el grupo que quiere asignar a Box.
1. En **Aprovisionamiento**, asegúrese de que la opción **Synchronize Azure Active Directory Groups to Box** (Sincronizar Azure Active Directory con Box) esté seleccionada. Esta configuración sincroniza los grupos que asignó en el paso anterior. Estos grupos pueden tardar algún tiempo en insertarse desde Azure.

> [!NOTE]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Box](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Box, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
