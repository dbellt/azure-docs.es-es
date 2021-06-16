---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Harmony Connect | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Check Point Harmony Connect.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/04/2021
ms.author: jeedes
ms.openlocfilehash: 18a919827ec213f5165d8b7d913f0c489d63ee5c
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891849"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-check-point-harmony-connect"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Check Point Harmony Connect | Microsoft Docs

En este tutorial, aprenderá a integrar Check Point Harmony Connect con Azure Active Directory (Azure AD). Al integrar Check Point Harmony Connect con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Check Point Harmony Connect.
* Permitir que los usuarios inicien sesión automáticamente en Check Point Harmony Connect con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para inicio de sesión único (SSO) en Check Point Harmony Connect.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Check Point Harmony Connect admite el inicio de sesión único iniciado por **SP**.
> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.


## <a name="adding-check-point-harmony-connect-from-the-gallery"></a>Adición de Check Point Harmony Connect desde la galería

Para configurar la integración de Check Point Harmony Connect en Azure AD, es preciso agregar Check Point Harmony Connect desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Check Point Harmony Connect** en el cuadro de búsqueda.
1. Seleccione **Check Point Harmony Connect** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-check-point-harmony-connect"></a>Configuración y prueba del inicio de sesión único de Azure AD para Check Point Harmony Connect

Configure y pruebe el inicio de sesión único de Azure AD con Check Point Harmony Connect con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Check Point Harmony Connect.

Para configurar y probar el inicio de sesión único de Azure AD con Check Point Harmony Connect, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Check Point Harmony Connect](#configure-check-point-harmony-connect-sso)** , para configurar los valores del inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Check Point Harmony Connect](#create-check-point-harmony-connect-test-user)** , para tener un homólogo de B.Simon en Check Point Identity Awareness que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Check Point Harmony Connect**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://cloudinfra-gw.portal.checkpoint.com/api/saml/sso`.

1. La aplicación Check Point Harmony Connect espera las aserciones de SAML en un formato específico, lo cual requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Check Point Harmony Connect espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | ---------------- | --------- |
    | groups | user.groups |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Check Point Harmony Connect**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Check Point Harmony Connect mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Check Point Harmony Connect**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-check-point-harmony-connect-sso"></a>Configuración del inicio de sesión único en Check Point Harmony Connect

1. Inicie sesión en el sitio web de Check Point Harmony Connect como administrador.

1. Haga clic en **SETTINGS** (Configuración), vaya a **Identity Provider** (Proveedor de identidades) y haga clic en **CONNECT NOW** (Conectar ahora).

    ![Captura de pantalla de la sección Identity Provider (Proveedor de identidades).](./media/check-point-harmony-connect-tutorial/identity-provider.png)

1. Seleccione **Microsoft Azure AD** como proveedor de identidades y haga clic en **NEXT** (Siguiente).

    ![Captura de pantalla para seleccionar el proveedor de identidades.](./media/check-point-harmony-connect-tutorial/select-identity-provider.png)

1. En la página **Verify Domain** (Comprobar dominio), escriba el dominio de la organización y escriba este registro DNS generado en el servidor DNS como registro TXT y haga clic en **NEXT** (Siguiente).

    ![Captura de pantalla del valor de dominio.](./media/check-point-harmony-connect-tutorial/domain.png)

1. En la página Allow Connectivity (Permitir conectividad), siga estos pasos:

    ![Captura de pantalla de la página Allow Connectivity (Permitir conectividad).](./media/check-point-harmony-connect-tutorial/allow-connectivity.png)

    a. Copie el valor de **ENTITY ID** (Identificador de entidad) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    b. Copie el valor de **REPLY URL** (Dirección URL de respuesta) y péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    c. Haga clic en **SIGUIENTE**.

1. En la página **Configure Metadata** (Configurar metadatos), cargue el **XML de metadatos de federación** que ha descargado de Azure Portal.

1. En la página **CONFIRM IDENTITY PROVIDER** (Confirmar el proveedor de identidades), haga clic en **Add** (Agregar) para completar la configuración.

### <a name="create-check-point-harmony-connect-test-user"></a>Creación de un usuario de prueba de Check Point Harmony Connect

1. Inicie sesión en el sitio web de Check Point Harmony Connect como administrador.

1. Vaya **Policy** -> **Access Control** (Directiva > Control de accesos), cree una **nueva regla** y haga clic en **(+)** para agregar un **nuevo usuario**. 

    ![captura de pantalla de creación de un nuevo usuario.](./media/check-point-harmony-connect-tutorial/add-new-user.png)

1. En la ventana **ADD USER** (AGREGAR USUARIO), escriba el nombre y el nombre de usuario en los cuadros de texto correspondientes y haga clic en **ADD** (AGREGAR).

    ![captura de pantalla de creación de usuario.](./media/check-point-harmony-connect-tutorial/add-user.png)

## <a name="test-sso"></a>Prueba de SSO 

Para probar Check Point Harmony Connect, vaya a su servicio de autenticación y autentíquese mediante la cuenta de prueba que ha creado en la sección **Creación de un usuario de prueba de Azure AD**.

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Check Point Harmony Connect, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).