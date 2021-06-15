---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Leadfamly | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Leadfamly.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/05/2021
ms.author: jeedes
ms.openlocfilehash: 9ab8d0ca9c6c44510f8cdf94bdbd43aadd8da2ad
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111889065"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-leadfamly"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Leadfamly

En este tutorial aprenderá a integrar Leadfamly con Azure Active Directory (Azure AD). Al integrar Leadfamly con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Leadfamly.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Leadfamly con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Leadfamly.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Leadfamly admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-leadfamly-from-the-gallery"></a>Adición de Leadfamly desde la galería

Para configurar la integración de Leadfamly en Azure AD, deberá agregar Leadfamly desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Leadfamly** en el cuadro de búsqueda.
1. Seleccione **Leadfamly** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-leadfamly"></a>Configuración y prueba del inicio de sesión único de Azure AD para Leadfamly

Configure y pruebe el inicio de sesión único de Azure AD con Leadfamly mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Leadfamly.

Para configurar y probar el inicio de sesión único de Azure AD con Leadfamly, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Leadfamly](#configure-leadfamly-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Leadfamly](#create-leadfamly-test-user)** : para tener un homólogo de B.Simon en Leadfamly vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación de **Leadfamly**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://appv2.leadfamly.com/saml-sso/<INSTANCE ID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Leadfamly](mailto:support@leadfamly.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Leadfamly**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Leadfamly mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Leadfamly**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-leadfamly-sso"></a>Configuración del inicio de sesión único en Leadfamly

1. Inicie sesión como administrador en el sitio de la compañía de Leadfamly.

2. Vaya a **Account** ->**Customer information** ->**SAML SSO** (Cuenta > Información del cliente > SSO de SAML).

![Cuenta](./media/leadfamly-tutorial/configuration.png "Cuenta") 

3. Habilite el **SSO de SMAL**, seleccione el proveedor **Azure AD** en la lista desplegable y realice los siguientes pasos.

![Información](./media/leadfamly-tutorial/account.png "Información") 

  a. Copie el valor de **Identifier** (Identificador) y péguelo en el cuadro de texto URL del **identificador** de la sección **Configuración básica de SAML** en Azure Portal.

  b. Copie el valor de la **dirección URL de respuesta** y péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

  c. Copie el valor de **Sign on URL** (URL de inicio de sesión) y péguelo en el cuadro de texto **URL de inicio de sesión** de la sección **Configuración básica de SAML** de Azure Portal.

  d. Abra el archivo **XML de metadatos de federación** descargado de Azure Portal en el Bloc de notas y cargue el contenido en **XML de metadatos de federación**.

  e. Haga clic en **Guardar**.

### <a name="create-leadfamly-test-user"></a>Creación de un usuario de prueba de Leadfamly

1. En otra ventana del explorador web, inicie sesión en el sitio web de Leadfamly como administrador.

2. Vaya a **Account** -> **Users** -> **Invite user** (Cuenta > Usuarios > Invitar usuario).

![Sección Usuarios](./media/leadfamly-tutorial/users.png "Sección Usuarios") 

3. Rellene los valores obligatorios en los campos siguientes y haga clic en **Save** (Guardar).

![Modificación de usuarios](./media/leadfamly-tutorial/modify-user.png "Modificación de usuarios") 

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Leadfamly, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Leadfamly e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Leadfamly en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Leadfamly, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).