---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Druva | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Druva.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/02/2021
ms.author: jeedes
ms.openlocfilehash: 858e63272faea2aaf2572a5e97e24ba5858d5bda
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111568454"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-druva"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Druva

En este tutorial, aprenderá a integrar Druva con Azure Active Directory (Azure AD). Al integrar Druva con Azure AD, se puede:

* Controlar en Azure AD quién tiene acceso a Druva.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Druva con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Druva.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Druva admite el inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-druva-from-the-gallery"></a>Adición de Druva desde la galería

Para configurar la integración de Druva en Azure AD, tendrá que agregar Druva desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Druva** en el cuadro de búsqueda.
1. Seleccione **Druva** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-druva"></a>Configuración y prueba del inicio de sesión único de Azure AD para Druva

Configure y pruebe el inicio de sesión único de Azure AD con Druva mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Druva.

Para configurar y probar el inicio de sesión único de Azure AD con Druva, siga los pasos a continuación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Druva](#configure-druva-sso)** , para definir los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Druva](#create-druva-test-user)** , para tener un homólogo de B.Simon en Druva que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Druva**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Identificador (id. de entidad)** , escriba el valor de la cadena: `DCP-login`.
    
    b. En el cuadro de texto **URL de respuesta (URL del Servicio de consumidor de aserciones)** , escriba la siguiente URL: `https://cloud.druva.com/wrsaml/consume`.

1. Haga clic en **Save**(Guardar).

1. La aplicación Druva espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Druva espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    | Nombre | Atributo de origen|
    | ------------------- | -------------------- |
    | emailAddress | user.email |
    | druva_auth_token | Token de SSO generado a partir de la consola de administración de DCP, sin comillas.  Por ejemplo: X-XXXXX-XXXX-S-A-M-P-L-E+TXOXKXEXNX=. Azure agrega comillas automáticamente alrededor del token de autenticación. |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Druva** (Configurar Druva), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Druva mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Druva**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-druva-sso"></a>Configurar el inicio de sesión único de Druva

1. En otra ventana del explorador web, inicie sesión como administrador en el sitio de la compañía en Druva.

1. Haga clic en el logotipo de Druva en la esquina superior izquierda y, a continuación, haga clic en **Druva Cloud Settings** (Configuración de la nube de Druva).

    ![Configuración](./media/druva-tutorial/cloud.png "Configuración")

1. En la pestaña **Inicio de sesión único**, haga clic en **Editar**.

    ![Captura de pantalla que muestra la pestaña "Configuración de acceso - Inicio de sesión único" con el botón "Editar" seleccionado.](./media/druva-tutorial/edit-tab.png "Configuración de inicio de sesión único")

1. En la página **Edit Single Sign-On Settings** (Editar la configuración de inicio de sesión único), siga estos pasos:

    ![Configuración de inicio de sesión único](./media/druva-tutorial/configuration.png "Configuración de inicio de sesión único")

    1. En el cuadro de texto **ID Provider Login URL** (Dirección URL de inicio de sesión del proveedor de identidades), pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    1. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado de proveedor de Id.** .

       > [!NOTE]
       > Para habilitar el inicio de sesión único para administradores, seleccione las casillas **Administrators log into Druva Cloud through SSO provider** (Los administradores inician sesión en Druva en la nube a través del proveedor de SSO) y **Allow failsafe access to Druva Cloud administrators(recommended)** (Permitir el acceso a prueba de fallos a los administradores de Druva en la nube [recomendado]). Druva recomienda habilitar la opción **Failsafe for Administrators** (Seguridad a prueba de fallos para administradores) para que tengan que acceder a la consola de DCP en caso de que se produzcan errores en IdP. También permite a los administradores usar la contraseña de SSO y DCP para acceder a la consola de DCP.

    1. Haga clic en **Save**(Guardar). Esto permite el acceso a Druva Cloud Platform con SSO.

### <a name="create-druva-test-user"></a>Creación de un usuario de prueba en Druva

En esta sección, se crea un usuario llamado B.Simon en Druva. Druva admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Druva, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Druva para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Druva en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Druva para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que configure Druva, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
