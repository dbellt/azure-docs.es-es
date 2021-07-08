---
title: 'Tutorial: Integración de Azure Active Directory con Mind Tools Toolkit | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Mind Tools Toolkit.
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
ms.openlocfilehash: d394410ab552cd9e5687decd5951c5ae5583a57e
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111568403"
---
# <a name="tutorial-azure-active-directory-integration-with-mind-tools-toolkit"></a>Tutorial: Integración de Azure Active Directory con Mind Tools Toolkit

En este tutorial, obtendrá información sobre cómo integrar Mind Tools Toolkit con Azure Active Directory (Azure AD). Al integrar Mind Tools Toolkit en Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Mind Tools Toolkit.
* Permitir que los usuarios inicien sesión automáticamente en Mind Tools Toolkit (inicio de sesión único) con sus cuentas de Azure AD.
* Administrar sus cuentas en una ubicación central: Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Mind Tools Toolkit, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Mind Tools Toolkit.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Mind Tools Toolkit admite el inicio de sesión único iniciado por SP.
* Mind Tools Toolkit admite el aprovisionamiento de usuarios Just-In-Time.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-mind-tools-toolkit-from-the-gallery"></a>Adición de Mind Tools Toolkit desde la galería

Para configurar la integración de Mind Tools Toolkit en Azure AD, será preciso que agregue Mind Tools Toolkit desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Mind Tools Toolkit** en el cuadro de búsqueda.
1. Seleccione **Mind Tools Toolkit** en los resultados de la búsqueda y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-mind-tools-toolkit"></a>Configuración y prueba del inicio de sesión único de Azure AD para Mind Tools Toolkit

Configure y pruebe el inicio de sesión único de Azure AD con Mind Tools Toolkit usando una usuaria de prueba llamada **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Mind Tools Toolkit.

Para configurar y probar el inicio de sesión único de Azure AD con Mind Tools Toolkit, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Mind Tools Toolkit](#configure-mind-tools-toolkit-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Mind Tools Toolkit](#create-mind-tools-toolkit-test-user)** : para tener un homólogo de B.Simon en Mind Tools Toolkit que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Mind Tools Toolkit**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, en el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el patrón `https://app.goodpractice.net/#/<subscriptionUrl>/s/<LOCATION_ID>`.

    > [!NOTE]
    > El valor de **URL de inicio de sesión** no es real. Actualícelo con la dirección URL de inicio de sesión real. Pónganse en contacto con el [equipo de soporte técnico de Mind Tools Toolkit](mailto:support@goodpractice.com) para obtener este valor.

1. En la página **Configuración del inicio de sesión único con SAML**, vaya a la sección **Certificado de firma de SAML**. A la derecha de **XML de metadatos de federación**, seleccione **Descargar** para descargar el texto XML y guardarlo en el equipo. El contenido XML depende de las opciones que seleccione.

    ![Sección Certificado de firma de SAML, con Descargar resaltado junto a XML de metadatos de federación](common/metadataxml.png)

1. En la sección **Configurar Mind Tools Toolkit**, copie cualquiera de las siguientes direcciones URL según sus necesidades.

    ![Sección Configurar Mind Tools Toolkit, con las direcciones URL de configuración resaltadas](common/copy-configuration-urls.png)

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

En esta sección va a permitir que B.Simon acceda a Mind Tools Toolkit mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Mind Tools Toolkit**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-mind-tools-toolkit-sso"></a>Configuración del inicio de sesión único de Mind Tools Toolkit

Para configurar el inicio de sesión único en **Mind Tools Toolkit**, es preciso enviar el texto **XML de metadatos de federación** descargado y las direcciones URL copiadas previamente de Azure Portal al [equipo de soporte técnico de Mind Tools Toolkit](mailto:support@goodpractice.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-mind-tools-toolkit-test-user"></a>Creación de un usuario de prueba de Mind Tools Toolkit

En esta sección, se crea un usuario llamado B.Simon en Mind Tools Toolkit. Mind Tools Toolkit admite el aprovisionamiento de usuarios Just-In-Time, el cual está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Mind Tools Toolkit, se crea después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Mind Tools Toolkit, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Mind Tools Toolkit e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Cuando haga clic en el icono de Mind Tools Toolkit en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Mind Tools Toolkit. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Mind Tools Toolkit, puede aplicar el control de sesión, que protege frente a la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
