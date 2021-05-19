---
title: 'Tutorial: Integración de Azure Active Directory con Everbridge | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Everbridge.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/10/2021
ms.author: jeedes
ms.openlocfilehash: 74ea971a16d1c16be17f16cc5345e162fd974f43
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737202"
---
# <a name="tutorial-azure-active-directory-integration-with-everbridge"></a>Tutorial: Integración de Azure Active Directory con Everbridge

En este tutorial, aprenderá a integrar Everbridge con Azure Active Directory (Azure AD).
Al integrar Everbridge con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Everbridge.
* Permitir que los usuarios inicien sesión automáticamente en Everbridge con sus cuentas de Azure AD. Este control de acceso se llama inicio de sesión único (SSO).
* Administrar las cuentas en una ubicación central mediante Azure Portal.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Everbridge, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción Everbridge que usa el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Everbridge admite el inicio de sesión único basado en IDP.

## <a name="add-everbridge-from-the-gallery"></a>Adición de Everbridge desde la galería

Para configurar la integración de Everbridge en Azure AD, es preciso agregar Everbridge desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Everbridge** en el cuadro de búsqueda.
1. Seleccione **Everbridge** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-everbridge"></a>Configuración y prueba del inicio de sesión único de Azure AD en Everbridge

Configure y pruebe el inicio de sesión único de Azure AD con Everbridge mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Everbridge.

Para configurar el inicio de sesión único de Azure AD con Everbridge, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Everbridge](#configure-everbridge-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Everbridge](#create-everbridge-test-user)** : para tener un homólogo de Britta Simon en Everbridge que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Everbridge**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

    >[!NOTE]
    >Configure la aplicación como portal administrador *o* como portal miembro tanto en Azure Portal como en el portal Everbridge.

4. Para configurar la aplicación **Everbridge** como **portal administrador de Everbridge**, en la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro **Identificador**, escriba una dirección URL que siga el patrón.
    `https://sso.everbridge.net/<API_Name>`

    b. En el cuadro **URL de respuesta**, escriba una dirección que siga el patrón.
    `https://manager.everbridge.net/saml/SSO/<API_Name>/alias/defaultAlias`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de identificador y URL de respuesta. Póngase en contacto con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. Para configurar la aplicación **Everbridge** como **portal miembro de Everbridge**, en la sección **Configuración básica de SAML**, siga estos pasos:

  * Después, realice los siguientes pasos si quiere configurar la aplicación en modo iniciado por IDP:

    a. En el cuadro **Identificador**, escriba una dirección URL con el formato `https://sso.everbridge.net/<API_Name>/<Organization_ID>`.

    b. En el cuadro **URL de respuesta**, escriba una dirección URL con el formato `https://member.everbridge.net/saml/SSO/<API_Name>/<Organization_ID>/alias/defaultAlias`.

   * Seleccione **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por SP:

     a. En el cuadro **URL de inicio de sesión**, escriba una dirección URL con el formato `https://member.everbridge.net/saml/login/<API_Name>/<Organization_ID>/alias/defaultAlias?disco=true`.

     > [!NOTE]
     > Estos valores no son reales. Actualice estos valores con los valores de identificador y de direcciones URL de inicio de sesión y de respuesta reales. Póngase en contacto con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el archivo **XML de metadatos de federación**. Guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

7. En la sección **Configurar Everbridge**, copie las direcciones URL adecuadas según sus necesidades:

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

En esta sección, va a permitir que B.Simon acceda a Everbridge mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Everbridge**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-everbridge-sso"></a>Configuración del inicio de sesión único de Everbridge

Para configurar el inicio de sesión único en **Everbridge** como **portal administrador de Everbridge**, siga estos pasos.
 
1. En otra ventana del explorador web, inicie sesión en Everbridge como administrador.

1. En el menú de la parte superior, seleccione la pestaña **Settings** (Configuración). En **Security** (Security), seleccione **Single Sign-On for Manager Portal** (Inicio de sesión único para Manager Portal).
   
     ![Configurar inicio de sesión único](./media/everbridge-tutorial/settings.png)
   
     a. En el cuadro **Name** (Nombre), escriba el nombre del proveedor de identificador. Un ejemplo es el nombre de su empresa.
   
     b. En el cuadro **API Name** (Nombre de la API), escriba el nombre de la API.
   
     c. Seleccione **Choose File** (Elegir archivo) para cargar el archivo de metadatos que descargó de Azure Portal.
   
     d. En **SAML Identity Location** (Ubicación de identidad de SAML), seleccione **Identity is in the NameIdentifier element of the Subject statement** (La identidad está en el elemento NameIdentifier de la instrucción de asunto).
   
     e. En el cuadro **Identity provider Login URL** (URL de inicio de sesión del proveedor de identidades), pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.
   
     f. En **Service Provider Initiated Request Binding** (Vinculación de solicitud iniciada del proveedor de servicios), seleccione **HTTP Redirect** (Redirección HTTP).

     g. Seleccione **Guardar**.

## <a name="configure-everbridge-as-everbridge-member-portal-sso"></a>Configuración de Everbridge con el inicio de sesión único en el portal de miembros de Everbridge

Para configurar el inicio de sesión único en **Everbridge** como **portal miembro de Everbridge**, tiene que enviar el **archivo XML de metadatos de federación** descargado al [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-everbridge-test-user"></a>Creación de un usuario de prueba de Everbridge

En esta sección, creará un usuario de prueba llamado Britta Simon en Everbridge. Para agregar usuarios en la plataforma Everbridge, colabore con el [equipo de soporte técnico de Everbridge](mailto:support@everbridge.com). Los usuarios se tienen que crear y activar en Everbridge antes de usar el inicio de sesión único. 

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal. Se iniciará la sesión automáticamente en la instancia de Everbridge para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Everbridge en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Everbridge para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que ha configurado Everbridge, puede aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).