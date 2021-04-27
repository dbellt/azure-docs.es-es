---
title: 'Tutorial: integración de Azure Active Directory con M-Files | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y M-Files.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/24/2021
ms.author: jeedes
ms.openlocfilehash: 680380722746522a0eb3fe6518452472be952075
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107482848"
---
# <a name="tutorial-azure-active-directory-integration-with-m-files"></a>Tutorial: integración de Azure Active Directory con M-Files

En este tutorial, aprenderá a integrar M-Files con Azure Active Directory (Azure AD). Al integrar M-Files con Azure AD, puede hacer lo siguiente:

* Determinar en Azure AD quién tiene acceso a M-Files.
* Permitir a los usuarios iniciar sesión automáticamente en M-Files con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en M-Files.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* M-Files admite el inicio de sesión único puesto en marcha por **SP**.

## <a name="add-m-files-from-the-gallery"></a>Incorporación de M-Files desde la galería

Para configurar la integración de M-Files en Azure AD, es preciso agregar M-Files desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **M-Files** en el cuadro de búsqueda.
1. Seleccione **M-files** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-m-files"></a>Configuración y prueba del inicio de sesión único de Azure AD para M-Files

Configure y pruebe el inicio de sesión único de Azure AD con M-files utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de M-Files.

Para configurar y probar el inicio de sesión único de Azure AD con M-Files, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en M-Files](#configure-m-files-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en M-Files](#create-m-files-test-user)** , para tener un homólogo de B.Simon en M-Files que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **M-Files**, busque la sección **Administrar** y seleccione el **inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<tenantname>.cloudvault.m-files.com/authentication/MFiles.AuthenticationProviders.Core/sso`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<tenantname>.cloudvault.m-files.com`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de M-Files](mailto:support@m-files.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Set up M-Files** (Configurar M-Files), copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a M-Files mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **M-Files**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-m-files-sso"></a>Configuración del inicio de sesión único de M-Files

1. Para configurar SSO en su aplicación, póngase en contacto con el [equipo de soporte técnico de M-Files](mailto:support@m-files.com) y proporciónele el archivo de metadatos descargado.
   
    >[!NOTE]
    >Siga estos pasos si desea configurar SSO para la aplicación de escritorio de archivo M-Files. Si solo desea configurar SSO para la versión de web de M-Files, no se necesitan pasos adicionales.  

1. Siga estos pasos para configurar la aplicación de escritorio M-Files para habilitar SSO con Azure AD. Para descargar M-Files, vaya a la página de [descarga de M-Files](https://www.m-files.com/customers/product-downloads/download-update-links/).

1. Abra la ventana **M-Files Desktop Settings** (Configuración de escritorio de M-Files). A continuación, haga clic en **Agregar**.
   
    ![Captura de pantalla que muestra la ventana M-Files Desktop Settings (Configuración de escritorio de M-Files) donde puede seleccionar la opción Add (Agregar).](./media/m-files-tutorial/settings.png)

1. En la ventana **Document Vault Connection Properties** (Propiedades de conexión del almacén de documentos), siga estos pasos:
   
    ![Captura de pantalla que muestra la ventana Document Vault Connection Properties (Propiedades de conexión del almacén de documentos), donde puede especificar los valores descritos.](./media/m-files-tutorial/general.png)  

    En la sección Server (Servidor), escriba los valores como se indica a continuación:  

    a. En **Name** (Nombre), escriba `<tenant-name>.cloudvault.m-files.com`. 
 
    b. En **Port Number** (Número de puerto), escriba **4466**. 

    c. En **Protocol** (Protocolo), seleccione **HTTPS**. 

    d. En el campo **Authentication** (Autenticación), seleccione **Specific Windows user** (Usuario específico de Windows). Luego, se le solicitará una página de inicio de sesión. Escriba sus credenciales de Azure AD. 

    e. En **Vault on Server** (Almacén en servidor),  seleccione el almacén correspondiente en el servidor.
 
    f. Haga clic en **OK**.

### <a name="create-m-files-test-user"></a>Creación de un usuario de prueba en M-Files

El objetivo de esta sección es crear un usuario de prueba llamado Britta Simon en M-Files. Póngase en contacto con el [equipo de soporte técnico de M-Files](mailto:support@m-files.com) para agregar los usuarios en M-Files.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de M-Files, desde donde puede poner en marcha el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de M-Files y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de M-Files en Aplicaciones, se le redirigirá a la URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado M-Files, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
