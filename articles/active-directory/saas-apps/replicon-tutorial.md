---
title: 'Tutorial: Integración de Azure Active Directory con Replicon | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Replicon.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/13/2021
ms.author: jeedes
ms.openlocfilehash: 8ccabdb23f1d06dc45ebfd06117cc8c9a168a8ad
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110064763"
---
# <a name="tutorial-integrate-replicon-with-azure-active-directory"></a>Tutorial: Integración de Replicon con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar Replicon con Azure Active Directory (Azure AD). Al integrar Replicon con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Replicon.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Replicon con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción de Replicon con el inicio de sesión único (SSO) habilitado.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 

* Replicon admite el inicio de sesión único iniciado por **SP**.

## <a name="add-replicon-from-the-gallery"></a>Incorporación de Replicon desde la galería

Para configurar la integración de Replicon en Azure AD, es preciso agregar Replicon desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Replicon** en el cuadro de búsqueda.
1. Seleccione **Replicon** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-replicon"></a>Configuración y prueba del SSO de Azure AD para Replicon

Configure y pruebe el inicio de sesión único de Azure AD con Replicon utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Replicon.

Para configurar y probar el inicio de sesión único de Azure AD con Replicon, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Replicon](#configure-replicon-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Replicon](#create-replicon-test-user)** , para tener un homólogo de B.Simon en Replicon que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Replicon**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://global.replicon.com/!/saml2/<client name>/sp-sso/post`

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://global.replicon.com/!/saml2/<client name>`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://global.replicon.com/!/saml2/<client name>/sso/post`

    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de inicio de sesión, el identificador y la dirección URL de respuesta reales. Póngase en contacto con el [equipo de soporte de cliente de Replicon](https://www.replicon.com/customerzone/contact-support) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. Haga clic en el icono de lápiz para editar la configuración de **Certificado de firma de SAML**.

    ![Algoritmo de firma](common/signing-algorithm.png)

    1. Seleccione **Firmar aserción SAML**  como la **Opción de firma**.

    1. Seleccione **SHA-256** como el **Algoritmo de firma**.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `BrittaSimon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Replicon utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Replicon**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-replicon-sso"></a>Configuración del inicio de sesión único de Replicon

1. En otra ventana del explorador web, inicie sesión en el sitio de la compañía de Replicon como administrador.

2. Lleve a cabo los siguientes pasos para configurar SAML 2.0:

    ![Habilitar la autenticación de SAML](./media/replicon-tutorial/authentication.png "Habilitar autenticación SAML")

    a. Para mostrar el cuadro de diálogo **EnableSAML Authentication2** (Habilitar SAML Authentication2), agregue lo siguiente a la URL después de la clave de su compañía: `/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

    1. Lo siguiente muestra el esquema de la dirección URL completa: `https://na2.replicon.com/<YourCompanyKey>/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

   b. Haga clic en **+** para expandir la sección **v20Configuration**.

   c. Haga clic en **+** para expandir la sección **metaDataConfiguration**.

   d. Seleccione **SHA256** en xmlSignatureAlgorithm.

   e. Haga clic en **Elegir archivo** para seleccionar el archivo XML de metadatos del proveedor de identidades y haga clic en **Enviar**.

### <a name="create-replicon-test-user"></a>Creación de un usuario de prueba de Replicon

El objetivo de esta sección es crear un usuario de prueba llamado B.Simon en Replicon.

**Para crear un usuario manualmente, siga los pasos siguientes:**

1. En una ventana del explorador web, inicie sesión en el sitio de la compañía de Replicon como administrador.

2. Vaya a **Administración \> Usuarios**.

    ![Usuarios](./media/replicon-tutorial/administration.png "Usuarios")

3. Haga clic en **+Agregar usuario**.

    ![Agregar usuario](./media/replicon-tutorial/user.png "Agregar usuario")

4. En la sección **Perfil de usuario** , lleve a cabo estos pasos:

    ![Perfil de usuario](./media/replicon-tutorial/profile.png "Perfil de usuario")

    a. En el cuadro de texto **Nombre de inicio de sesión**, escriba la dirección de correo electrónico del usuario de Azure AD que quiera aprovisionar, como `B.Simon@contoso.com`.

    > [!NOTE]
    > El nombre de inicio de sesión debe coincidir con la dirección de correo electrónico del usuario en Azure AD.

    b. En **Tipo de autenticación,** seleccione **SSO**.

    c. Establezca el identificador de autenticación en el mismo valor del nombre de inicio de sesión (la dirección de correo electrónico de Azure AD del usuario).

    d. En el cuadro de texto **Departamento** , escriba el departamento del usuario.

    e. En **Tipo de empleado**, seleccione **Administrador**.

    f. Haga clic en **Guardar perfil de usuario**.

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de Replicon que proporcione Replicon para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Replicon, donde puede comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Replicon y comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Replicon en Mis aplicaciones, se le redirigirá a la URL de inicio de sesión de Replicon. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Replicon, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la exfiltración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
