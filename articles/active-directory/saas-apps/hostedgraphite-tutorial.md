---
title: 'Tutorial: Integración de Azure Active Directory con Hosted Graphite | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Hosted Graphite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/11/2021
ms.author: jeedes
ms.openlocfilehash: a3848f29f9f8b12981b84a5ac025f0a73e6f21f1
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063049"
---
# <a name="tutorial-azure-active-directory-integration-with-hosted-graphite"></a>Tutorial: Integración de Azure Active Directory con Hosted Graphite

En este tutorial, aprenderá a integrar Hosted Graphite con Azure Active Directory (Azure AD). Cuando integra Hosted Graphite con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Hosted Graphite.
* Permitir que los usuarios inicien sesión automáticamente en Hosted Graphite con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Hosted Graphite.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Hosted Graphite admite el inicio de sesión único iniciado por **SP e IDP**.
* Hosted Graphite admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-hosted-graphite-from-the-gallery"></a>Adición de Hosted Graphite desde la galería

Para configurar la integración de Hosted Graphite en Azure AD, deberá agregar Hosted Graphite desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Hosted Graphite** en el cuadro de búsqueda.
1. Seleccione **Hosted Graphite** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-hosted-graphite"></a>Configuración y prueba del inicio de sesión único de Azure AD para Hosted Graphite

Configure y pruebe el inicio de sesión único de Azure AD con Hosted Graphite mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Hosted Graphite.

Para configurar y probar el inicio de sesión único de Azure AD con Hosted Graphite, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Hosted Graphite](#configure-hosted-graphite-sso)** : para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Hosted Graphite](#create-hosted-graphite-test-user)** : para tener un homólogo de B.Simon en Hosted Graphite que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Hosted Graphite**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://www.hostedgraphite.com/metadata/<USER_ID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://www.hostedgraphite.com/complete/saml/<USER_ID>`

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.hostedgraphite.com/login/saml/<USER_ID>/`

    > [!NOTE]
    > Tenga en cuenta que estos no son valores reales. Tiene que actualizarlos con el identificador, la dirección URL de respuesta y la URL de inicio de sesión reales. Para obtener estos valores, puede ir a Acceso -> Configuración de SAML en la aplicación o ponerse en contacto con el [equipo de soporte técnico de Hosted Graphite](mailto:help@hostedgraphite.com).

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Configurar Hosted Graphite**, copie las direcciones URL adecuadas según sus requisitos.

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

En esta sección, va a conceder a B.Simon acceso a Hosted Graphite mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Hosted Graphite**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-hosted-graphite-sso"></a>Configuración del inicio de sesión único de Hosted Graphite

1. Inicie la sesión en el inquilino de Hosted Graphite como administrador.

2. Vaya a la **página de configuración de SAML** en la barra lateral (**Acceso -> Configuración de SAML**).

    ![Captura de pantalla que muestra el menú Access (Acceso) con SAML Setup (Configuración de SAML) seleccionado.](./media/hostedgraphite-tutorial/setup.png)

3. Confirme que estas direcciones URL coinciden con la configuración realizada en la sección **Configuración básica de SAML** de Azure Portal.

    ![Captura de pantalla que muestra Configuración básica de SAML.](./media/hostedgraphite-tutorial/configuration.png)

4. En los cuadros de texto **Entity or Issuer ID** (Identificador de entidad o de emisor) y **SSO Login URL** (Dirección URL de inicio de sesión de SSO), pegue el valor de **Identificador de Azure AD** y **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    ![Captura de pantalla que muestra entradas para el proveedor de identidades.](./media/hostedgraphite-tutorial/integration.png)

5. Seleccione **Read-only** (Solo lectura) en **Default User Role** (Rol de usuario predeterminado).

    ![Captura de pantalla que muestra el rol de usuario predeterminado, que es Read-only (Solo lectura).](./media/hostedgraphite-tutorial/role.png)

6. Abra el certificado codificado en base 64 descargado de Azure Portal en el Bloc de notas, copie su contenido en el Portapapeles y luego péguelo en el cuadro de texto **Certificado X.509**.

    ![Captura de pantalla que muestra el certificado X punto 509.](./media/hostedgraphite-tutorial/certificate.png)

7. Haga clic en el botón **Guardar** .

### <a name="create-hosted-graphite-test-user"></a>Creación de un usuario de prueba de Hosted Graphite

En esta sección se creará un usuario llamado Britta Simon en Hosted Graphite. Hosted Graphite admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe aún en Hosted Graphite, se crea uno nuevo después de la autenticación.

> [!NOTE]
> Si necesita crear un usuario manualmente, es preciso que se ponga contacto con el [equipo de soporte técnico de Hosted Graphite](<mailto:help@hostedgraphite.com>).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Hosted Graphite, desde donde puede poner en marcha el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de Hosted Graphite e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de Hosted Graphite para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Hosted Graphite en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Hosted Graphite para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado Hosted Graphite, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
