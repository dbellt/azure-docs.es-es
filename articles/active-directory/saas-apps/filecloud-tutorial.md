---
title: 'Tutorial: Integración de Azure Active Directory con FileCloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y FileCloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2021
ms.author: jeedes
ms.openlocfilehash: b1f0056dd34cabe18e135a1caa04d4ed07194568
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111984373"
---
# <a name="tutorial-azure-active-directory-integration-with-filecloud"></a>Tutorial: integración de Azure Active Directory con FileCloud

En este tutorial, aprenderá a integrar FileCloud con Azure Active Directory (Azure AD). Al integrar FileCloud con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a FileCloud.
* Permitir que los usuarios inicien sesión automáticamente en FileCloud con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en FileCloud.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* FileCloud admite el inicio de sesión único iniciado por **SP**.

* FileCloud admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-filecloud-from-the-gallery"></a>Adición de FileCloud desde la galería

Para configurar la integración de FileCloud en Azure AD, será preciso que agregue FileCloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **FileCloud** en el cuadro de búsqueda.
1. Seleccione **FileCloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-filecloud"></a>Configuración y prueba del inicio de sesión único de Azure AD en FileCloud

Configure y pruebe el inicio de sesión único de Azure AD con FileCloud mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de FileCloud.

Para configurar y probar el inicio de sesión único de Azure AD con FileCloud, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en FileCloud](#configure-filecloud-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de FileCloud](#create-filecloud-test-user)** : para tener un homólogo de B.Simon en FileCloud que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **FileCloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.filecloudonline.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.filecloudonline.com/simplesaml/module.php/saml/sp/metadata.php/default-sp`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de FileCloud](mailto:support@codelathe.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up FileCloud** (Configurar FileCloud), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Firstbird mediante el inicio de sesión único de FileCloud.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **FileCloud**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-filecloud-sso"></a>Configuración del inicio de sesión único de FileCloud

1. En otra ventana del explorador web, inicie sesión en el inquilino de FileCloud como administrador.

2. En la barra de navegación de la izquierda, haga clic en **Settings**(Configuración). 
   
    ![Captura de pantalla que muestra la opción "Settings" (Configuración) resaltada en el panel de navegación izquierdo.](./media/filecloud-tutorial/setting.png)

3. Haga clic en la ficha **SSO** de la sección de configuración. 
   
    ![Captura de pantalla que muestra la sección "Settings" (Configuración) con la pestaña "S S O" seleccionada.](./media/filecloud-tutorial/tab.png)

4. Seleccione **SAML** como **Default SSO Type** (Tipo de SSO predeterminado) en el panel **Configuración de inicio de sesión único (SSO)** .
   
    ![Captura de pantalla que muestra el panel "Single Sign On (S S O) Settings" (Configuración de inicio de sesión único [S S O]) con la opción "S A M L" seleccionada.](./media/filecloud-tutorial/panel.png)

5. En el cuadro de texto **IdP End Point URL** (URL de punto de conexión de IdP), pegue el valor de **Identificador de Azure AD**, que ha copiado de Azure Portal.

    ![Captura de pantalla que muestra la sección "S A M L Settings" (Configuración de S A M L) con la opción "I d P End Point U R L" (U R L de punto de conexión de I d P) resaltada.](./media/filecloud-tutorial/identifier.png)

6. Abra el archivo de metadatos descargado en el Bloc de notas, copie el contenido del mismo en el Portapapeles y luego péguelo en el cuadro de texto **IdP Meta Data** (Metadatos de IdP) en el panel **Configuración de SAML**.

    ![Configuración del inicio de sesión único en la aplicación](./media/filecloud-tutorial/metadata.png)

7. Haga clic en el botón **Guardar** .

### <a name="create-filecloud-test-user"></a>Creación de un usuario de prueba de FileCloud

En esta sección se creará un usuario llamado Britta Simon en FileCloud. FileCloud admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en FileCloud, se crea uno nuevo después de la autenticación.

>[!NOTE]
>Si necesita crear manualmente un usuario, es preciso que se ponga en contacto con el [equipo de soporte técnico de FileCloud](mailto:support@codelathe.com).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de FileCloud, donde podrá comenzar el flujo de inicio de sesión. 

* Acceda directamente a la URL de inicio de sesión de FileCloud y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de FileCloud en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado FileCloud, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).