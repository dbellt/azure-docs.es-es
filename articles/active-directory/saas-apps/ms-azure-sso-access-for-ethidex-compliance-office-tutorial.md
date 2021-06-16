---
title: 'Tutorial: Integración de inicio de sesión único (SSO) de Azure Active Directory con MS Azure SSO Access for Ethidex Compliance Office™ | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y MS Azure SSO Access for Ethidex Compliance Office™.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2021
ms.author: jeedes
ms.openlocfilehash: 42180a293ce6e720c54979257fcec7d36e996731
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111893235"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ms-azure-sso-access-for-ethidex-compliance-office"></a>Tutorial: Integración de inicio de sesión único (SSO) de Azure Active Directory con MS Azure SSO Access for Ethidex Compliance Office™

En este tutorial, aprenderá a integrar MS Azure SSO Access for Ethidex Compliance Office™ con Azure Active Directory (Azure AD). Al integrar MS Azure SSO Access for Ethidex Compliance Office™ con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a MS Azure SSO Access for Ethidex Compliance Office™.
* Permitir que los usuarios puedan iniciar sesión automáticamente en MS Azure SSO Access for Ethidex Compliance Office™ con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).


## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en MS Azure SSO Access for Ethidex Compliance Office™.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* MS Azure SSO Access for Ethidex Compliance Office™ admite el inicio de sesión único iniciado por **IDP**.

## <a name="adding-ms-azure-sso-access-for-ethidex-compliance-office-from-the-gallery"></a>Adición de MS Azure SSO Access for Ethidex Compliance Office™ desde la galería

Para configurar la integración de MS Azure SSO Access for Ethidex Compliance Office™ en Azure AD, deberá agregar MS Azure SSO Access for Ethidex Compliance Office™ desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **MS Azure SSO Access for Ethidex Compliance Office™** en el cuadro de búsqueda.
1. Seleccione **MS Azure SSO Access for Ethidex Compliance Office™** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-ms-azure-sso-access-for-ethidex-compliance-office"></a>Configuración y prueba del inicio de sesión único de Azure AD para MS Azure SSO Access for Ethidex Compliance Office™

Configure y pruebe el inicio de sesión único de Azure AD con MS Azure SSO Access for Ethidex Compliance Office™ utilizando un usuario de prueba llamado **B.Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de MS Azure SSO Access for Ethidex Compliance Office™.

Para configurar y probar el inicio de sesión único de Azure AD con MS Azure SSO Access for Ethidex Compliance Office™, es preciso seguir estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de MS Azure SSO Access for Ethidex Compliance Office](#configure-ms-azure-sso-access-for-ethidex-compliance-office-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de MS Azure SSO Access for Ethidex Compliance Office](#create-ms-azure-sso-access-for-ethidex-compliance-office-test-user)** , para tener un homólogo de B.Simon en MS Azure SSO Access for Ethidex Compliance Office™ que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **MS Azure SSO Access for Ethidex Compliance Office™**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `com.ethidex.prod.<CLIENTID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.ethidex.com/saml2/sp/acs/<CLIENTID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación MS Azure SSO Access for Ethidex Compliance Office™ espera que las aserciones de SAML estén en un formato específico, así que es necesario agregar asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de pantalla muestra la lista de atributos predeterminados, donde **nameidentifier** se asigna con **user.userprincipalname**. La aplicación MS Azure SSO Access for Ethidex Compliance Office™ espera que **nameidentifier** se asigne con **user.mail**, por lo que debe editar la asignación de atributos haciendo clic en el icono **Editar** y cambiar dicha asignación.

    ![imagen](common/edit-attribute.png)

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar MS Azure SSO Access for Ethidex Compliance Office™**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a MS Azure SSO Access for Ethidex Compliance Office™ utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **MS Azure SSO Access for Ethidex Compliance Office™**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-ms-azure-sso-access-for-ethidex-compliance-office-sso"></a>Configuración del inicio de sesión único de MS Azure SSO Access for Ethidex Compliance Office

Para configurar el inicio de sesión único en **MS Azure SSO Access for Ethidex Compliance Office™**, es preciso enviar el **certificado (Base64)** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-ms-azure-sso-access-for-ethidex-compliance-office-test-user"></a>Creación de un usuario de prueba de MS Azure SSO Access for Ethidex Compliance Office

En esta sección, creará un usuario llamado B.Simon en MS Azure SSO Access for Ethidex Compliance Office™. Trabaje con el [equipo de soporte técnico de MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com) para agregar los usuarios a la plataforma de MS Azure SSO Access for Ethidex Compliance Office™. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* En Azure Portal, haga clic en Probar esta aplicación. Al hacerlo, debería iniciar sesión automáticamente en la instancia de Ethidex Compliance Office™ en la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono Ethidex Compliance Office™ en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Ethidex Compliance Office™ en la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Ethidex Compliance Office™, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).