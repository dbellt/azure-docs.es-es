---
title: 'Tutorial: integración de Azure Active Directory con TOPdesk - Public | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y TOPdesk - Public.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/08/2021
ms.author: jeedes
ms.openlocfilehash: 15ec0af4e576c33d20208c234bd34cdfd8f65087
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108207616"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---public"></a>Tutorial: Integración de Azure Active Directory con TOPdesk - Public

En este tutorial, obtendrá información sobre cómo integrar TOPdesk - Public con Azure Active Directory (Azure AD). Al integrar TOPdesk - Public con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a TOPdesk - Public.
* Permitir que los usuarios puedan iniciar sesión automáticamente en TOPdesk - Public con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en TOPdesk - Public.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* TOPdesk - Public admite el inicio de sesión único iniciado por **SP**.

## <a name="add-topdesk---public-from-the-gallery"></a>Incorporación de TOPdesk - Public desde la galería

Para configurar la integración de TOPdesk - Public en Azure AD, deberá agregar TOPdesk - Public desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **TOPdesk - Public** en el cuadro de búsqueda.
1. En el panel de resultados, seleccione **TOPdesk - Public** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-topdesk---public"></a>Configuración y prueba del inicio de sesión único de Azure AD SSO para TOPdesk - Public

Configure y pruebe el inicio de sesión único de Azure AD con TOPdesk - Public mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de TOPdesk - Public.

Para configurar y probar el inicio de sesión único de Azure AD con TOPdesk - Public, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de TOPdesk - Public](#configure-topdesk---public-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de TOPdesk - Public](#create-topdesk---public-test-user)** : para tener un homólogo de B.Simon en TOPdesk - Public que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **TOPdesk - Public**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4.  En la sección **Configuración básica de SAML**, si tiene el **archivo de metadatos del proveedor de servicios**, lleve a cabo los siguientes pasos:

    >[!NOTE]
    >Obtendrá el **archivo de metadatos del proveedor de servicios**, como se explica más adelante en la sección **Configuración del inicio de sesión único de TOPdesk - Public** del tutorial.

    a. Haga clic en **Cargar el archivo de metadatos**.
    
    ![Carga del archivo de metadatos](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Elección del archivo de metadatos](common/browse-upload-metadata.png)

    c. Una vez que se haya cargado correctamente el archivo de metadatos, el valor de **Identificador** y **URL de respuesta** se rellena automáticamente en la sección Configuración básica de SAML.

    d. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net`

    e. En el cuadro de texto **URL del identificador**, rellene la dirección URL de metadatos de TOPdesk que puede recuperar de la configuración de esta aplicación. Debe usar el patrón siguiente: `https://<companyname>.topdesk.net/saml-metadata/<identifier>`
    
    f. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.topdesk.net/tas/public/login/verify`.
    
    > [!NOTE] 
    > Si los valores de **Identificador** y de **Dirección URL de respuesta** no se rellenan automáticamente, debe escribirlos manualmente. En Identificador, siga el patrón como se mencionó anteriormente y obtenga el valor de Dirección URL de respuesta en la sección **Configuración del inicio de sesión único de TOPdesk - Public**, que se explica más adelante en el tutorial. El valor de **Dirección URL de inicio de sesión** no es real de modo que debe actualizarla con esta. Póngase en contacto con el [equipo de soporte técnico de TOPdesk - Public](https://my.topdesk.com/) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

6. En la sección **Configurar TOPdesk - Public**, copie las direcciones URL que necesite.

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

En esta sección, va a permitir que B.Simon acceda a TOPdesk - Public mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **TOPdesk - Public**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-topdesk---public-sso"></a>Configuración del inicio de sesión único de TOPdesk - Public

1. Inicie sesión en el sitio **TOPdesk - Public** de su empresa como administrador.

2. En el menú **TOPdesk**, haga clic en **Configuración**.
   
    ![Configuración](./media/topdesk-public-tutorial/menu.png "Configuración")

3. Haga clic en **Configuración de inicio de sesión**.
   
    ![Login Settings (Configuración de inicio de sesión)](./media/topdesk-public-tutorial/login.png "Login Settings")

4. Expanda el menú **Configuración de inicio de sesión** menú y haga clic en **General**.
   
    ![Configuración general](./media/topdesk-public-tutorial/general.png "Configuración general")

5. En la sección **Público** de la sección de configuración **Inicio de sesión SAML**, realice los pasos siguientes:
   
    ![Technical Settings (Configuración técnica)](./media/topdesk-public-tutorial/public.png "Configuración técnica")
   
    a. Haga clic en **Descargar** para descargar el archivo de metadatos públicos y guárdelo en el equipo.
   
    b. Abra el archivo de metadatos descargado y luego busque el nodo **AssertionConsumerService**.

    ![AssertionConsumerService](./media/topdesk-public-tutorial/service.png "AssertionConsumerService")
   
    c. Copia el valor de **AssertionConsumerService**, péguelo en el cuadro de texto **Dirección URL de respuesta** de la sección **Configuración básica de SAML**.      
   
6. Lleve a cabo los siguientes pasos para crear un archivo de certificado:
    
    ![Certificate](./media/topdesk-public-tutorial/certificate-file.png "Certificado")
    
    a. Abra el archivo de metadatos descargado de Azure Portal.
    
    b. Expanda el nodo **RoleDescriptor** con **xsi:type** de **fed:ApplicationServiceType**.
    
    c. Copie el valor del nodo **X509Certificate**.
    
    d. Guarde en un archivo el valor **X509Certificate** copiado localmente en el equipo.

7. En la sección **Público**, haga clic en **Agregar**.
    
    ![SAML Login (Inicio de sesión de SAML)](./media/topdesk-public-tutorial/add.png "Inicio de sesión SAML")

8. En la página del cuadro de diálogo **Asistente para la configuración SAML**, lleve a cabo los pasos siguientes:
    
    ![SAML Configuration Assistant (Asistente de configuración de SAML)](./media/topdesk-public-tutorial/configuration.png "SAML configuration assistant")
    
    a. Para cargar el archivo de metadatos descargado de Azure Portal, en **Federation Metadata** (Metadatos de federación) haga clic en **Browse** (Examinar).

    b. Para cargar el archivo de certificado, en **Certificado (RSA)**, haga clic en **Examinar**.

    c. Para cargar el archivo de logotipo que obtuvo del equipo de soporte técnico de TOPdesk, en **Icono del logotipo**, haga clic en **Examinar**.

    d. En el cuadro de texto **User name attribute** (Atributo de nombre de usuario), escriba `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    e. En el cuadro de texto **Nombre para mostrar**, escriba un nombre para la configuración.

    f. Haga clic en **Save**(Guardar).

### <a name="create-topdesk---public-test-user"></a>Creación de un usuario de prueba de TOPdesk - Public

Para permitir que los usuarios de Azure AD inicien sesión en TOPdesk - Public, deben aprovisionarse en TOPdesk - Public. En el caso de TOPdesk - Public, el aprovisionamiento es una tarea manual.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Siga estos pasos para configurar el aprovisionamiento de usuario:

1. Inicie sesión en su sitio de la compañía de **TOPdesk - Public** como administrador.

2. En el menú de la parte superior, haga clic en **TOPdesk \> New \> Support Files \> Person** (TOPdesk > Nuevo > Archivos de soporte > Persona).
   
    ![Person](./media/topdesk-public-tutorial/files.png "Persona")

3. En el cuadro de diálogo Nueva persona, realice los pasos siguientes:
   
    ![Nueva persona](./media/topdesk-public-tutorial/new.png "Nueva persona")
   
    a. Haga clic en la pestaña General.

    b. En el cuadro de texto **Apellidos**, escriba los apellidos del usuario, en este caso, Simon.
 
    c. Seleccione un **Sitio** para la cuenta.
 
    d. Haga clic en **Save**(Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de TOPdesk - Public ofrecida por TOPdesk - Public para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de TOPdesk - Public, desde donde podrá iniciar el flujo de inicio de sesión. 

* Vaya directamente a la URL de inicio de sesión de TOPdesk - Public y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de TOPdesk - Public en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de TOPdesk - Public. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que se ha configurado TOPdesk - Public, puede aplicar el control de sesión, que protege su organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).