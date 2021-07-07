---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con HRworks Single Sign-On | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y HRworks Single Sign-On.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/26/2021
ms.author: jeedes
ms.openlocfilehash: 5de617feb34db1ada91423d8f3cb3ed235f15151
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111570101"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-hrworks-single-sign-on"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con HRworks Single Sign-On

En este tutorial, aprenderá a integrar HRworks Single Sign-On con Azure Active Directory (Azure AD). Al integrar HRworks Single Sign-On con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a HRworks Single Sign-On.
* Permitir que los usuarios inicien sesión automáticamente en HRworks Single Sign-On con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en HRworks Single Sign-On.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* HRworks Single Sign-On admite el inicio de sesión único iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-hrworks-single-sign-on-from-the-gallery"></a>Incorporación de HRworks Single Sign-On desde la galería

Para configurar la integración de HRworks Single Sign-On en Azure AD, será preciso que agregue HRworks Single Sign-On desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **HRworks Single Sign-On** en el cuadro de búsqueda.
1. Seleccione **HRworks Single Sign-On** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-hrworks-single-sign-on"></a>Configuración y prueba del inicio de sesión único de Azure AD para HRworks Single Sign-On

Configure y pruebe el inicio de sesión único de Azure AD con HRworks Single Sign-On mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de HRworks Single Sign-On.

Para configurar y probar el inicio de sesión único de Azure AD con HRworks Single Sign-On, siga estos siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de HRworks Single Sign-On](#configure-hrworks-single-sign-on-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de HRworks Single Sign-On](#create-hrworks-single-sign-on-test-user)** , para tener un homólogo de B.Simon en HRworks Single Sign-On que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **HRworks Single Sign-On**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://login.hrworks.de/?companyId=<COMPANY_ID>&directssologin=true`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de HRworks Single Sign-On](mailto:nadja.sommerfeld@hrworks.de) para obtener este valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Set up HRworks Single Sign-On** (Configurar HRworks Single Sign-On), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a HRworks Single Sign-On utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **HRworks Single Sign-On**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-hrworks-single-sign-on-sso"></a>Configuración del inicio de sesión único de HRworks Single Sign-On

1. Para automatizar la configuración en HRworks Single Sign-On, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instale la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Set up HRworks Single Sign-On** (Configurar HRworks Single Sign-On) para ir a esta aplicación. En ella, proporcione las credenciales de administrador para iniciar sesión en HRworks Single Sign-On. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 y 4.

    ![Configuración](common/setup-sso.png)

1. Si quiere configurar HRworks Single Sign-On manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de HRworks Single Sign-On como administrador y haga lo siguiente:

1. Haga clic en **Administrador** > **Datos básicos** > **Seguridad** > **Inicio de sesión único** en el lado izquierdo de la barra de menús y realice los pasos siguientes:

    ![Configurar inicio de sesión único](./media/hrworks-single-sign-on-tutorial/configure.png)

    a. Seleccione la casilla **Utilizar inicio de sesión único (SSO)** .

    b. Seleccione **Metadatos XML** como **método de entrada de metadatos**.

    c. Seleccione **Individual NameID identifier** (Identificador de NameID individual) como **Value for NameID** (Valor para NameID).

    d. En el Bloc de notas, abra el archivo XML de metadatos que descargó de Azure Portal, copie el contenido y luego péguelo en el cuadro de texto **Metadatos**.

    e. Haga clic en **Save**(Guardar).

### <a name="create-hrworks-single-sign-on-test-user"></a>Creación de un usuario de prueba de HRworks Single Sign-On

Para permitir que los usuarios de Azure AD inicien sesión en HRworks Single Sign-On, se deben aprovisionar en HRworks Single Sign-On. En HRworks Single Sign-On, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en HRworks Single Sign-On como administrador.

1. Haga clic en **Administrator** (Administrador) > **Persons** (Personas) > **Persons** (Personas) > **New person** (Nueva persona) en el lado izquierdo de la barra de menús.

     ![Captura de pantalla que muestra la página de H R Works con las opciones Persons (Personas) y New person (Nueva persona) seleccionadas.](./media/hrworks-single-sign-on-tutorial/persons.png)

1. En el menú emergente, haga clic en **Next** (Siguiente).

    ![Captura de pantalla que muestra una lista de países que puede elegir para la persona.](./media/hrworks-single-sign-on-tutorial/new-person.png)

1. En el elemento emergente **Create new person with country for legal terms** (Crear nueva persona con un país para condiciones legales), rellene los detalles correspondientes **First name** (Nombre), **Last name** (Apellidos) y haga clic en **Create** (Crear).

    ![Captura de pantalla que muestra cuadros de texto en los que puede especificar el nombre y el apellido de la persona.](./media/hrworks-single-sign-on-tutorial/create-person.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de HRworks Single Sign-On, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de HRworks Single Sign-On e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de HRworks Single Sign-On en Aplicaciones, se le redirigirá a la URL de inicio de sesión de HRworks Single Sign-On. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado HRworks Single Sign-On, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
