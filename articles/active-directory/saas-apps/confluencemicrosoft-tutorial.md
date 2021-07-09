---
title: 'Tutorial: Integración del inicio de sesión único de Azure Active Directory con el inicio de sesión único de SAML en Confluence de Microsoft | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Confluence SAML SSO by Microsoft.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2021
ms.author: jeedes
ms.openlocfilehash: 77e98560ee483f55fc4a808a8c888f20af983a90
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110478588"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-confluence-saml-sso-by-microsoft"></a>Tutorial: Integración del inicio de sesión único de Azure Active Directory con Confluence SAML SSO by Microsoft

En este tutorial, obtendrá información sobre cómo integrar Confluence SAML SSO by Microsoft con Azure Active Directory (Azure AD). Al integrar Confluence SAML SSO by Microsoft con Azure AD, puede:

* Controlar en Azure AD quién tiene acceso a Confluence SAML SSO by Microsoft.
* Permitir que los usuarios inicien sesión automáticamente en Confluence SAML SSO by Microsoft con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).


## <a name="description"></a>Descripción:

Use la cuenta de Microsoft Azure Active Directory con el servidor Confluence de Atlassian para habilitar el inicio de sesión único. De esta forma, todos los usuarios de la organización pueden usar las credenciales de Azure AD para iniciar sesión en la aplicación Confluence. Este complemento usa SAML 2.0 para la federación.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Confluence SAML SSO by Microsoft, se necesitan los siguientes elementos:

- Una suscripción de Azure AD
- Aplicación de servidor de Confluence instalada en un servidor Windows de 64 bits (de manera local o en la infraestructura de IaaS en la nube)
- El servidor de Confluence es compatible con HTTPS
- Tenga en cuenta que las versiones admitidas para el complemento Confluence se mencionan en la sección siguiente.
- El servidor de Confluence es accesible en Internet, especialmente a la página de inicio de sesión de Azure AD para la autenticación, y debe poder recibir el token de Azure AD
- Las credenciales de administrador se configuran en Confluence
- WebSudo está deshabilitado en Confluence
- Usuario de prueba creado en la aplicación de servidor de Confluence

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción de Confluence. Pruebe primero la integración en el entorno de desarrollo o de ensayo de la aplicación y, después, use el entorno de producción.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

Para empezar, necesita los siguientes elementos:

* No use el entorno de producción, salvo que sea necesario.
* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único de Confluence SAML SSO by Microsoft.

## <a name="supported-versions-of-confluence"></a>Versiones compatibles de Confluence

En la actualidad se admiten las siguientes versiones de Confluence:

- Confluence: 5.0 a 5.10
- Confluence: 6.0.1 a 6.15.9
- Confluence: 7.0.1 a 7.10.0

> [!NOTE]
> Tenga en cuenta que nuestro complemento de Confluence también funciona con Ubuntu 16.04.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Confluence SAML SSO by Microsoft admite el inicio de sesión único iniciado por **SP**.

## <a name="adding-confluence-saml-sso-by-microsoft-from-the-gallery"></a>Agregar Confluence SAML SSO by Microsoft desde la galería

Para configurar la integración de Confluence SAML SSO by Microsoft en Azure AD, deberá agregar Confluence SAML SSO by Microsoft desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Confluence SAML SSO by Microsoft** en el cuadro de búsqueda.
1. Seleccione **Confluence SAML SSO by Microsoft** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-confluence-saml-sso-by-microsoft"></a>Configuración y prueba del inicio de sesión único de Azure AD para Confluence SAML SSO by Microsoft

Configure y pruebe el inicio de sesión único de Azure AD con Confluence SAML SSO by Microsoft mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Confluence SAML SSO by Microsoft.

Para configurar el inicio de sesión único de Azure AD con Confluence SAML SSO by Microsoft, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Confluence SAML SSO by Microsoft](#configure-confluence-saml-sso-by-microsoft-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Confluence SAML SSO by Microsoft](#create-confluence-saml-sso-by-microsoft-test-user)** , para tener un homólogo de B. Simon en Confluence SAML SSO by Microsoft que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Confluence SAML SSO by Microsoft**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN:PORT>/`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN:PORT>/plugins/servlet/saml/auth`
    
    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<DOMAIN:PORT>/plugins/servlet/saml/auth`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. En caso de que sea una dirección URL con nombre, el puerto es opcional. Estos valores se reciben durante la configuración del complemento de Confluence, que se explica más adelante en el tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

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

En esta sección va a permitir que B. Simon acceda a SAML SSO by Microsoft mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Confluence SAML SSO by Microsoft**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-confluence-saml-sso-by-microsoft-sso"></a>Configuración del inicio de sesión único de Confluence SAML SSO by Microsoft

1. En otra ventana del explorador web, inicie sesión en la instancia de Confluence como administrador.

1. Mantenga el mouse encima del icono de engranaje y haga clic en **Complementos**.

    ![Captura de pantalla que muestra el icono de engranaje seleccionado y la opción "Complementos" resaltada en el menú desplegable.](./media/confluencemicrosoft-tutorial/add-on-1.png)

1. Descargue el complemento del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=56503). Cargue manualmente el complemento proporcionado por Microsoft mediante el menú **Cargar complemento**. La descarga del complemento está contemplada en el [Acuerdo de servicio de Microsoft](https://www.microsoft.com/servicesagreement/).

    ![Captura de pantalla que muestra la página "Administrar complementos" con la acción "Cargar complemento" seleccionada.](./media/confluencemicrosoft-tutorial/add-on-12.png)

1. Para ejecutar el escenario de proxy inverso de Confluence o el escenario del equilibrador de carga, realice los pasos siguientes:

    > [!NOTE]
    > En primer lugar, debe configurar el servidor con las siguientes instrucciones y después instalar el complemento.

    a. Agregue el siguiente atributo en el puerto del **conector** del archivo **server.xml** de la aplicación de servidor JIRA.

    `scheme="https" proxyName="<subdomain.domain.com>" proxyPort="<proxy_port>" secure="true"`

    ![Captura de pantalla que muestra el archivo "server.xml" con el atributo agregado al puerto del conector.](./media/confluencemicrosoft-tutorial/reverse-proxy-1.png)

    b. Cambie la **URL base** en **Configuración del sistema** en función del proxy o equilibrador de carga.

    ![Captura de pantalla que muestra la página "Administración - Configuración" con la opción "URL base" resaltada.](./media/confluencemicrosoft-tutorial/reverse-proxy-2.png)

1. Una vez instalado el complemento, aparece en la sección de complementos **Instalados por el usuario** de **Administrar complemento**. Haga clic en **Configurar** para configurar el nuevo complemento.

    ![Captura de pantalla que muestra la sección "Instalados por el usuario" con el botón "Configurar" resaltado.](./media/confluencemicrosoft-tutorial/add-on-15.png)

1. Siga estos pasos en la página de configuración:

    ![Captura de pantalla que muestra la página de configuración de inicio de sesión único.](./media/confluencemicrosoft-tutorial/add-on-53.png)

    > [!TIP]
    > Asegúrese de que hay un solo certificado asignado a la aplicación, de forma que no se produzca ningún error en la resolución de los metadatos. Si hay varios certificados, el administrador recibe un error después de resolver los metadatos.

    1. En el cuadro de texto **URL de metadatos**, pegue la **dirección URL de metadatos de federación de aplicación** que copió en Azure Portal y haga clic en el botón **Resolver**. Se lee la dirección URL de metadatos de IdP y se rellena toda la información de campos.

    1. Copie los valores **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión**, y péguelos en los cuadros de texto **Identificador, Dirección URL de respuesta y Dirección URL de inicio de sesión** respectivamente en la sección **Configuración básica de SAML** de Azure Portal.

    1. En **Nombre del botón de inicio de sesión** escriba el nombre del botón que la organización quiere que los usuarios vean en la pantalla de inicio de sesión.

    1. En **Descripción del botón de inicio de sesión** escriba el nombre del botón que la organización quiere que los usuarios vean en la pantalla de inicio de sesión.

    1. En **SAML User ID Locations** (Ubicaciones de Id. de usuario de SAML), seleccione **User ID is in the NameIdentifier element of the Subject statement** (El Id. de usuario está en el elemento NameIdentifier de la instrucción Subject) o **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute).  Este identificador debe ser el identificador de usuario de Confluence. Si el identificador de usuario no coincide, el sistema no permitirá que los usuarios inicien sesión. 

       > [!Note]
       > La ubicación del Id. de usuario de SAML predeterminada es el identificador de nombre. Puede cambiarlo a una opción de atributo y escribir el nombre de atributo adecuado.

    1. Si selecciona la opción **User ID is in an Attribute element** (El Id. de usuario está en un elemento Attribute), escriba el nombre del atributo cuando se espera el id. de usuario en el cuadro de texto **Nombre del atributo**. 

    1. Si se usa el dominio federado (por ejemplo, ADFS, etc.) con Azure AD, haga clic en la opción **Habilitar detección de dominio principal** y configure el **nombre de dominio**.

    1. En **Nombre de dominio**, escriba el nombre del dominio en el caso de inicios de sesión basados en ADFS.

    1. Active **Enable Single Sign out** (Habilitar cierre de sesión único) si quiere que se cierre la sesión de Azure AD cuando un usuario cierre la sesión de Confluence. 

    1. Habilite la casilla **Force Azure Login** (Forzar inicio de sesión en Azure) solo si desea conectarse con las credenciales de Azure AD.

       > [!Note]
       > Para habilitar el formulario de inicio de sesión predeterminado para el inicio de sesión de administrador en la página de inicio de sesión esté habilitada la opción de forzar inicio de sesión en Azure, agregue el parámetro de consulta a la dirección URL del explorador.
       > `https://<DOMAIN:PORT>/login.action?force_azure_login=false`

    1. Haga clic en el botón **Save** (Guardar) para guardar la configuración.

       > [!NOTE]
       > Para más información acerca de la instalación y la solución de problemas, visite [MS Confluence SSO Connector Admin Guide](./ms-confluence-jira-plugin-adminguide.md) (Guía de administración del conector SSO de MS Confluence). También hay una sección de [preguntas frecuentes](./ms-confluence-jira-plugin-adminguide.md) que puede servirle de ayuda.

### <a name="create-confluence-saml-sso-by-microsoft-test-user"></a>Creación de un usuario de prueba de Confluence SAML SSO by Microsoft

Para permitir que los usuarios de Azure AD inicien sesión en el servidor local de Confluence, se deben aprovisionar en Confluence SAML SSO by Microsoft. Para Confluence SAML SSO by Microsoft, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión como administrador en el servidor local de Confluence.

1. Mantenga el mouse encima del icono de engranaje y haga clic en **Administración de usuarios**.

    ![Agregar empleado](./media/confluencemicrosoft-tutorial/user-1.png)

1. En la sección Usuarios, haga clic en la pestaña **Agregar usuarios**. En la página del cuadro de diálogo **Agregar un usuario**, realice los siguientes pasos:

    ![Captura de pantalla que muestra la página de administración de Confluence con la pestaña "Agregar usuarios" seleccionada e información escrita en "Agregar un usuario".](./media/confluencemicrosoft-tutorial/user-2.png)

    a. En el cuadro de texto **Nombre de usuario**, escriba el correo electrónico de un usuario, por ejemplo, B. Simon.

    b. En el cuadro de texto **Nombre completo**, escriba el nombre completo de un usuario; por ejemplo, B. Simon.

    c. En el cuadro de texto **Correo electrónico**, escriba la dirección de correo electrónico de un usuario, por ejemplo, B.Simon@contoso.com.

    d. En el cuadro de texto **Contraseña**, escriba la contraseña de B. Simon.

    e. Haga clic en **Confirmar contraseña** y vuelva a escribir la contraseña.

    f. Haga clic en el botón **Agregar**.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Confluence SAML SSO by Microsoft, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Confluence SAML SSO by Microsoft e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Confluence SAML SSO by Microsoft en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión único de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Confluence SAML SSO by Microsoft, puede aplicar el control de sesión, que protege a la organización frente a la filtración e infiltración de información confidencial en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
