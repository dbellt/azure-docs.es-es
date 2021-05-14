---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cornerstone Single Sign-On | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Cornerstone Single Sign-On.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/25/2021
ms.author: jeedes
ms.openlocfilehash: 28e77ecafc32f5b71ff2141aadb00ab484e0067a
ms.sourcegitcommit: 12f15775e64e7a10a5daebcc52154370f3e6fa0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108001959"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cornerstone-single-sign-on"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Cornerstone Single Sign-On

En este tutorial, aprenderá a integrar Cornerstone Single Sign-On con Azure Active Directory (Azure AD). Al integrar Cornerstone Single Sign-On con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Cornerstone Single Sign-On.
* Permitir que los usuarios inicien sesión automáticamente en Cornerstone Single Sign-On con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Cornerstone Single Sign-On.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Cornerstone Single Sign-On admite el inicio de sesión único habilitado por **SP**.
* Cornerstone Single Sign-On admite el [aprovisionamiento automático de usuarios](cornerstone-ondemand-provisioning-tutorial.md).
* Si va a integrar uno o varios productos de esta lista en particular, debe usar la aplicación Cornerstone OnDemand Single Sign-On desde la galería.

    Ofrecemos soluciones para:

    1. Administración del aprendizaje (LMS)
    2. Administración del rendimiento (EPM)
    3. Planeamiento de la sucesión
    4. Contratación (ATS)
    5. Empresa extendida
    6. Human Resources
    7. Contenido del empleado

## <a name="adding-cornerstone-single-sign-on-from-the-gallery"></a>Incorporación de Cornerstone Single Sign-On desde la galería

Para configurar la integración de Cornerstone Single Sign-On en Azure AD, debe agregar la aplicación desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Cornerstone Single Sign-On** en el cuadro de búsqueda.
1. Seleccione **Cornerstone Single Sign-On** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-cornerstone-single-sign-on"></a>Configuración y prueba del inicio de sesión único de Azure AD para Cornerstone Single Sign-On

Configure y pruebe el inicio de sesión único de Azure AD con Cornerstone Single Sign-On mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Cornerstone Single Sign-On.

Para configurar y probar el inicio de sesión único de Azure AD con Cornerstone Single Sign-On, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Cornestone Single Sign-On](#configure-cornerstone-single-sign-on-sso)** , para establecer los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Cornerstone Single Sign-On](#create-cornerstone-single-sign-on-test-user)** , para tener un homólogo de B.Simon en Cornerstone Single Sign-on vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Cornerstone Single Sign-On**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<PORTAL_NAME>.csod.com`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<PORTAL_NAME>.csod.com/samldefault.aspx?ouid=<OUID>`

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<PORTAL_NAME>.csod.com/samldefault.aspx?ouid=<OUID>`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la URL de respuesta, el identificador y la URL de inicio de sesión reales. Para obtener los valores, póngase en contacto con el [equipo de soporte técnico al cliente de Cornerstone Single Sign-On](mailto:moreinfo@csod.com). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

4. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

6. En la sección **Configurar Cornerstone AWS Single Sign-On**, copie las URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Cornerstone Single Sign-On mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Cornerstone Single Sign-On**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-cornerstone-single-sign-on-sso"></a>Configuración de Cornerstone Single Sign-On SSO

Para configurar el inicio de sesión único en **Cornerstone Single Sign-On**, es preciso enviar el **certificado (Base64)** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de Cornerstone Single Sign-On](mailto:moreinfo@csod.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-cornerstone-single-sign-on-test-user"></a>Creación de un usuario de prueba de Cornerstone Single Sign-On

El objetivo de esta sección es crear un usuario llamado B.Simon en Cornerstone Single Sign-On. Cornerstone Single Sign-On admite el aprovisionamiento automático de usuarios, que está habilitado de forma predeterminada. [Aquí](./cornerstone-ondemand-provisioning-tutorial.md) puede encontrar más información sobre cómo configurar el aprovisionamiento automático de usuarios.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Cornerstone Single Sign-On Sign, desde donde puede comenzar el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de Cornerstone Single Sign-On y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Cornerstone Single Sign-On en Aplicaciones, se le redirigirá a la URL de inicio de sesión de Cornerstone Single Sign-On. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Cornerstone Single Sign-On, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
