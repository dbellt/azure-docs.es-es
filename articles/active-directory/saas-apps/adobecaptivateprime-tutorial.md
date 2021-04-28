---
title: 'Tutorial: Integración de Azure Active Directory con Adobe Captivate Prime | Microsoft Docs'
description: Obtenga información para configurar el inicio de sesión único entre Azure Active Directory y Adobe Captivate Prime.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: jeedes
ms.openlocfilehash: 527b2bf9e7f97e0ffffa3d2cab1fae45a6393f49
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125524"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-captivate-prime"></a>Tutorial: Integración de Azure Active Directory con Adobe Captivate Prime

En este tutorial aprenderá a integrar Adobe Captivate Prime con Azure Active Directory (Azure AD). Al integrar Adobe Captivate Prime con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Adobe Captivate Prime.
* Permitir que los usuarios inicien sesión automáticamente en Adobe Captivate Prime con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Adobe Captivate Prime.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Adobe Captivate Prime admite el inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-adobe-captivate-prime-from-the-gallery"></a>Incorporación de Adobe Captivate Prime desde la galería

Para configurar la integración de Adobe Captivate Prime en Azure AD, necesita agregar Adobe Captivate Prime desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Adobe Captivate Prime** en el cuadro de búsqueda.
1. Seleccione **Adobe Captivate Prime** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-adobe-captivate-prime"></a>Configuración y prueba del inicio de sesión único de Azure AD para Adobe Captivate Prime

Configure y pruebe el inicio de sesión único de Azure AD con Adobe Captivate Prime mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Adobe Captivate Prime.

Para configurar y probar el inicio de sesión único de Azure AD con Adobe Captivate Prime, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Adobe Captivate Prime](#configure-adobe-captivate-prime-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Adobe Captivate Prime](#create-adobe-captivate-prime-test-user)** : para tener un homólogo de B.Simon en Adobe Captivate Prime vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Adobe Captivate Prime**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba la dirección URL: `https://captivateprime.adobe.com`

    b. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://captivateprime.adobe.com/saml/SSO`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Adobe Captivate Prime** (Configurar Adobe Captivate Prime), copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

7. Vaya a la pestaña **Propiedades**, copie la **URL de acceso de usuario** y péguela en el Bloc de notas.

    ![Vínculo de acceso de usuario](./media/adobecaptivateprime-tutorial/adobe.png)

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

En esta sección se habilita el acceso a Adobe Captivate Prime de B.Simon mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Adobe Captivate Prime**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-adobe-captivate-prime-sso"></a>Configuración del inicio de sesión único en Adobe Captivate Prime

Para configurar el inicio de sesión único en **Adobe Captivate Prime**, es preciso enviar el **XML de metadatos de federación** descargado, la **dirección URL de acceso de usuario** copiada y las direcciones URL adecuadas copiadas de Azure Portal al [equipo de soporte técnico de Adobe Captivate Prime](mailto:captivateprimesupport@adobe.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-adobe-captivate-prime-test-user"></a>Creación de un usuario de prueba de Adobe Captivate Prime

En esta sección se crea un usuario denominado Britta Simon en Adobe Captivate Prime. Trabaje con el [equipo de soporte técnico de Adobe Captivate Prime](mailto:captivateprimesupport@adobe.com) para agregar los usuarios en la plataforma de Adobe Captivate Prime. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Adobe Captivate Prime para la que configurara el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Adobe Captivate Prime en Aplicaciones debería iniciar sesión automáticamente en la versión de Adobe Captivate Prime para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Adobe Captivate Prime, puede aplicar el control de sesión, que protege la información confidencial de la organización de la infiltración y la filtración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).