---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con FAX.PLUS | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y FAX.PLUS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/19/2021
ms.author: jeedes
ms.openlocfilehash: 88cc75570c6c41bad875c87210fdf1bcc793075f
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111897089"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-faxplus"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con FAX.PLUS

En este tutorial aprenderá a integrar FAX.PLUS con Azure Active Directory (Azure AD). Si integra FAX.PLUS con Azure AD, podrá hacer lo siguiente:

* Controlar desde Azure AD quién tiene acceso a FAX.PLUS.
* Permitir que los usuarios inicien sesión automáticamente en FAX.PLUS con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción que permita utilizar el inicio de sesión único (SSO) de FAX.PLUS.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* FAX.PLUS permite utilizar el inicio de sesión único con **SP e IDP**.
* FAX.PLUS admite el aprovisionamiento de usuarios **Just In Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-faxplus-from-the-gallery"></a>Incorporación de FAX.PLUS desde la galería

Para configurar la integración de FAX.PLUS en Azure AD, es preciso agregar esta aplicación a la lista de aplicaciones SaaS administradas desde la galería.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **FAX.PLUS** en el cuadro de búsqueda.
1. Seleccione **FAX.PLUS** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-faxplus"></a>Configuración y comprobación del inicio de sesión único de Azure AD con FAX.PLUS

Configure y pruebe el inicio de sesión único de Azure AD con FAX.PLUS utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso vincular un usuario de Azure AD con el usuario de FAX.PLUS correspondiente.

Para configurar y probar el inicio de sesión único de Azure AD con FAX.PLUS, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en FAX.PLUS](#configure-faxplus-sso)** : para configurar el inicio de sesión único en el lado de la aplicación.
    1. **[Creación de un usuario de prueba de FAX.PLUS](#create-faxplus-test-user)** : para tener un homólogo de B.Simon en FAX.PLUS que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **FAX.PLUS**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://www.fax.plus/login`

1. Haga clic en **Save**(Guardar).

1. La aplicación FAX.PLUS espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación FAX.PLUS espera que, en la respuesta de SAML, se devuelvan algunos atributos más, que se muestran a continuación. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | --------------- | --------- |
    | firstname  | user.givenname |
    | lastname   | user.surname   |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar FAX.PLUS**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon utilice el inicio de sesión único de Azure concediéndole acceso a FAX.PLUS.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **FAX.PLUS**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-faxplus-sso"></a>Configuración del inicio de sesión único en FAX.PLUS

1. Para automatizar la configuración en FAX.PLUS, debe instalar la **extensión del navegador de inicio de sesión seguro de Aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Set up FAX.PLUS** (Configurar FAX.PLUS) para ir a la aplicación del mismo nombre. En ella, escriba las credenciales de administrador para iniciar sesión en FAX.PLUS. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 5.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar FAX.PLUS manualmente, en otra ventana del explorador web, inicie sesión en su sitio empresarial de FAX.PLUS como administrador.

2. Vaya a la sección **Security** (Seguridad) del perfil de administrador y desplácese hacia abajo hasta **Advanced** (Opciones avanzadas).

3. En el panel **Configuration** (Configuración), haga clic en el botón **Activate Single Sign-On** (Activar inicio de sesión único) y siga estos pasos.
    
    ![Cuenta](./media/fax.plus-tutorial/configuration.png "Cuenta") 

    a. En el cuadro de texto **Identificador de entidad**, pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    b. En el cuadro de texto **Single Sign On URL** (Dirección URL de inicio de sesión único), pegue el valor de la **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra en el Bloc de notas el archivo del **certificado (Base64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **Certificate X.509** (Certificado).

    d. Si desea utilizar el inicio de sesión único, active la casilla **Only Allow SSO Login for Admin User** (Permitir inicio de sesión único solo del usuario administrador). 

    e. Haga clic en **Confirmar**.

### <a name="create-faxplus-test-user"></a>Creación de un usuario de prueba de FAX.PLUS

En esta sección, se crea un usuario llamado Britta Simon en FAX.PLUS. FAX.PLUS admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de manera predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en FAX.PLUS, se creará después de realizar la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. De este modo, accederá automáticamente a la dirección URL de inicio de sesión de FAX.PLUS, donde podrá comenzar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de FAX.PLUS y comience el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* En Azure Portal, haga clic en **Probar esta aplicación**. Al hacerlo, debería iniciar sesión automáticamente en la instancia de FAX.PLUS en la que haya configurado el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Si hace clic en el icono de FAX.PLUS de Aplicaciones y seleccionó el modo SP en la configuración, accederá automáticamente a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión. Por el contrario, si seleccionó el modo IDP, debería iniciar sesión automáticamente en la instancia de FAX.PLUS en la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado FAX.PLUS, puede aplicar el control de sesión, que protege la información confidencial de la organización en tiempo real de posibles filtraciones. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).