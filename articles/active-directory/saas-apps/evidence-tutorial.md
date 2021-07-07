---
title: 'Tutorial: Integración de Azure Active Directory con Evidence.com | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Evidence.com.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/17/2021
ms.author: jeedes
ms.openlocfilehash: f3bdc931572b154b94534a198a3da3d0f649a709
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089135"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-evidencecom"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Evidence.com

En este tutorial, obtendrá información sobre cómo integrar Evidence.com con Azure Active Directory (Azure AD). Al integrar Evidence.com con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Evidence.com.
* Permitir a los usuarios iniciar sesión automáticamente en Evidence.com con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Evidence.com.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Evidence.com admite el inicio de sesión único iniciado por **SP**.

## <a name="add-evidencecom-from-the-gallery"></a>Incorporación de Evidence.com desde la galería

Para configurar la integración de Evidence.com en Azure AD, será preciso que agregue Evidence.com desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Evidence.com** en el cuadro de búsqueda.
1. Seleccione **Evidence.com** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-evidencecom"></a>Configuración y prueba del inicio de sesión único de Azure AD para Evidence.com

Configure y pruebe el inicio de sesión único de Azure AD con Evidence.com con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Evidence.com.

Para configurar y probar el inicio de sesión único de Azure AD con Evidence.com, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Evidence.com](#configure-evidencecom-sso)** , para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Evidence.com](#create-evidencecom-test-user)** , para tener un homólogo de B.Simon en Evidence.com que esté vinculado a la representación de ella en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Evidence.com**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<yourtenant>.evidence.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<yourtenant>.evidence.com`

    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<your tenant>.evidence.com/?class=UIX&proc=Login`.
    
    > [!NOTE]
    > Estos valores no son reales. Actualícelos con la dirección URL de respuesta, el identificador y la dirección URL de inicio de sesión reales. Póngase en contacto con el [equipo de soporte técnico al cliente de Evidence.com](https://communities.taser.com/support/SupportContactUs?typ=LE) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Evidence.com** (Configurar Evidence.com), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Evidence.com mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Evidence.com**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-evidencecom-sso"></a>Configuración del inicio de sesión único de Evidence.com

1. En una ventana de explorador web diferente, inicie sesión como administrador en su inquilino de Evidence.com y vaya a la pestaña **Administrador**.

2. Haga clic en **Agency Single Sign On** (Inicio de sesión único de agencia).

3. Seleccione **Inicio de sesión único basado en SAML**.

4. Copie los valores de **Identificador de Azure AD**, **Dirección URL de inicio de sesión** y **Dirección URL de cierre de sesión** mostrados en Azure Portal y en los campos correspondientes de Evidence.com.

5. Abra el archivo descargado de certificado (Base64) en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de seguridad**. 

6. Guarde la configuración en Evidence.com.

### <a name="create-evidencecom-test-user"></a>Creación de un usuario de prueba de Evidence.com

Para que los usuarios de Azure AD puedan iniciar sesión, se les debe aprovisionar para el acceso dentro de la aplicación Evidence.com. En esta sección, se describe cómo crear cuentas de usuario de Azure AD en Evidence.com.

**Para aprovisionar una cuenta de usuario en Evidence.com:**

1. En una ventana del explorador web, inicie sesión en el sitio de la compañía en Evidence.com como administrador.

2. Vaya a pestaña **Admin** (Administrador).

3. Haga clic en **Add User**(Agregar usuario).

4. Haga clic en el botón **Agregar**.

5. El valor de **Email Address** (Dirección de correo electrónico) del usuario agregado debe coincidir con el nombre de usuario en Azure AD de los usuarios a quienes desea conceder acceso. Si el nombre de usuario y la dirección de correo electrónico no usan el mismo valor en su organización, puede usar la sección **Evidence.com > Atributos > Inicio de sesión único** de Azure Portal para hacer que el identificador de nombre enviado a Evidence.com sea la dirección de correo electrónico.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Evidence.com, donde puede comenzar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Evidence.com e comience el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Evidence.com en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Evidence.com. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Evidence.com, puede aplicar el control de sesión, que protege la exfiltración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
