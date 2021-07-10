---
title: 'Tutorial: Integración de Azure Active Directory con Panopto | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Panopto.
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
ms.openlocfilehash: ce999b7cf06ae9b9a9c9fe591b631d8bb0d14caa
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111569938"
---
# <a name="tutorial-azure-active-directory-integration-with-panopto"></a>Tutorial: Integración de Azure Active Directory con Panopto

En este tutorial, aprenderá a integrar Panopto con Azure Active Directory (Azure AD). La integración de Panopto con Azure AD permite:

* Controlar en Azure AD quién tiene acceso a Panopto.
* Habilitar que los usuarios inicien sesión automáticamente en Panopto con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Panopto.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Panopto admite el inicio de sesión único iniciado por **SP**.

* Panopto admite el aprovisionamiento de usuarios **Just-In-Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-panopto-from-the-gallery"></a>Adición de Panopto desde la galería

Para configurar la integración de Panopto en Azure AD, es preciso agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Panopto** en el cuadro de búsqueda.
1. Seleccione **Panopto** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-panopto"></a>Configuración y prueba del inicio de sesión único de Azure AD para Panopto

Configure y pruebe el inicio de sesión único de Azure AD con Panopto mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Panopto.

Para configurar y probar el inicio de sesión único de Azure AD con Panopto, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Panopto](#configure-panopto-sso)** : para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Panopto](#create-panopto-test-user)** : para tener un homólogo de B. Simon en Panopto que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Panopto**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<TENANT_NAME>.panopto.com`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico para clientes de Panopto](mailto:support@panopto.com) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up Panopto** (Configurar Panopto), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B. Simon acceda a Panopto mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Panopto**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-panopto-sso"></a>Configuración del inicio de sesión único en Panopto

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Panopto como administrador.

2. En la barra de herramientas de la izquierda, haga clic en **Sistema** y, a continuación, haga clic en **Proveedores de identidades**.
   
    ![Sistema](./media/panopto-tutorial/toolbar.png "Sistema")

3. Haga clic en **Agregar proveedor**.
   
    ![Proveedores de identidades](./media/panopto-tutorial/provider.png "Proveedores de identidades")
   
4. En la sección de del proveedor SAML, lleve a cabo estos pasos:
   
    ![Configuración de SaaS](./media/panopto-tutorial/configuration.png "Configuración de SaaS")
    
    a. En la lista **Tipo de proveedor**, seleccione **SAML20**.    
    
    b. En el cuadro de texto **Nombre de instancia**, escriba un nombre para la instancia.

    c. En el cuadro de texto **Descripción detallada**, escriba una descripción detallada.
    
    d. En el cuadro de texto **Bounce Page Url** (Dirección URL de la página de rebote), pegue el valor de la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    f. Abra el certificado codificado en base 64 descargado de Azure Portal, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Public Key** (Clave pública).

5. Haga clic en **Save**(Guardar).

### <a name="create-panopto-test-user"></a>Creación de un usuario de prueba en Panopto

En esta sección, se crea un usuario llamado Britta Simon en Panopto. Panopto admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Panopto, se crea después de la autenticación.

>[!NOTE]
>Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Panopto ofrecida por Panopto para aprovisionar cuentas de usuario de Azure AD.
>

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de Panopto, donde podrá iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Panopto e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Panopto en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Tras configurar Panopto, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
