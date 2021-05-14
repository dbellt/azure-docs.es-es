---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Google Cloud (G Suite) Connector | Microsoft Docs'
description: Obtenga información sobre cómo configurar el inicio de sesión único entre Azure Active Directory y Google Cloud (G Suite) Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/11/2021
ms.author: jeedes
ms.openlocfilehash: 04d93913711dc8f03e35ac811b46158dfd038c61
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108750704"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-google-cloud-g-suite-connector"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Google Cloud (G Suite) Connector

En este tutorial, aprenderá a integrar Google Cloud (G Suite) Connector con Azure Active Directory (Azure AD). Al integrar Google Cloud (G Suite) Connector con Azure AD, puede hacer lo siguiente:

* En Azure AD se puede controlar quién tiene acceso a Google Cloud (G Suite) Connector.
* Permitir que los usuarios inicien sesión automáticamente en Google Cloud (G Suite) Connector con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD.
- Una suscripción habilitada para el inicio de sesión único en Google Cloud (G Suite) Connector.
- Una suscripción de Google Apps o Google Cloud Platform

> [!NOTE]
> Para probar los pasos de este tutorial, no se recomienda el uso de un entorno de producción. Este documento se creó con la nueva experiencia de inicio de sesión del usuario. Si aún usa la anterior, el programa de instalación parecerá diferente. Puede habilitar la nueva experiencia en la configuración de inicio de sesión único de la aplicación G Suite. Vaya a las **aplicaciones empresariales de Azure AD**, seleccione **Google Cloud (G Suite) Connector**, seleccione **Inicio de sesión único** y, luego, haga clic en **Probar la nueva experiencia**.

Para probar los pasos de este tutorial, debe seguir estas recomendaciones:

- No use el entorno de producción, salvo que sea necesario.
- Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

1. **P: ¿Es esta integración compatible con la integración del inicio de sesión único de Google Cloud Platform con Azure AD?**

    A. Sí. Google Cloud Platform y Google Apps comparten la misma plataforma de autenticación. Por ello, para la integración de GCP debe configurar el inicio de sesión único con Google Apps.

2. **P: ¿Son los Chromebooks y otros dispositivos Chrome compatibles con el inicio de sesión único de Azure AD?**
  
    A. Sí, los usuarios pueden iniciar sesión en sus dispositivos Chromebook con sus credenciales de Azure AD. Consulte este [artículo de soporte técnico de Google Cloud (G Suite) Connector](https://support.google.com/chrome/a/answer/6060880) para más información sobre por qué puede que se pidan las credenciales a los usuarios dos veces.

3. **P: Si se habilita el inicio de sesión único, ¿podrán usar los usuarios sus credenciales de Azure AD para iniciar sesión en cualquier producto de Google, como Google Classroom, GMail, Google Drive, YouTube, etc.?**

    A. Sí, en función de [qué aplicaciones de Google Cloud (G Suite) Connector](https://support.google.com/a/answer/182442?hl=en&ref_topic=1227583) decida habilitar o deshabilitar para su organización.

4. **P: ¿Puedo habilitar el inicio de sesión único solo para un subconjunto de mis usuarios de Google Cloud (G Suite) Connector?**

    A. No; si activa el inicio de sesión único, es necesario inmediatamente que todos los usuarios de Google Cloud (G Suite) Connector se autentiquen con sus credenciales de Azure AD. Dado que Google Cloud (G Suite) Connector no admite tener varios proveedores de identidades, el proveedor de identidades para su entorno de Google Cloud (G Suite) Connector puede ser Azure AD o Google, pero no ambos al mismo tiempo.

5. **P: Si un usuario inicia sesión a través de Windows, ¿se autentica automáticamente en Google Cloud (G Suite) Connector sin que se le pida una contraseña?**

    A. Hay dos opciones para habilitar este escenario. En primer lugar, los usuarios podrían iniciar sesión en dispositivos Windows 10 a través de [Azure Active Directory Join](../devices/overview.md). Como alternativa, los usuarios podrían iniciar sesión en dispositivos Windows que están unidos a un dominio en un entorno Active Directory local que se ha habilitado para el inicio de sesión único en Azure AD a través de una implementación de los [Servicios de federación de Active Directory (AD FS)](../hybrid/plan-connect-user-signin.md) . Ambas opciones requieren que realice los pasos del tutorial siguiente para permitir el inicio de sesión único entre Azure AD y Google Cloud (G Suite) Connector.

6. **P: ¿Qué debo hacer cuando aparece un mensaje de error de correo electrónico no válido?**

    A. En esta configuración, se necesita el atributo de correo electrónico para que los usuarios puedan iniciar sesión. Este atributo no se puede establecer manualmente.

    El atributo de correo electrónico se rellena automáticamente para cualquier usuario con una licencia válida de Exchange. Si el usuario no está habilitado para correo electrónico, se recibirá este error ya que la aplicación debe obtener este atributo para proporcionar acceso.

    Puede ir a portal.office.com con una cuenta de administrador, hacer clic en Centro de administración < Facturación < Suscripciones, seleccionar su suscripción a Microsoft 365 y, a continuación, hacer clic en Asignar a usuarios, seleccionar los usuarios para los que desee comprobar su suscripción y, en el panel derecho, hacer clic en Editar licencias.

    Una vez asignada la licencia de Microsoft 365, puede tardar algunos minutos en aplicarse. Después de eso, el atributo user.mail se rellenará automáticamente y el problema debería resolverse.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Google Cloud (G Suite) Connector admite el inicio de sesión único iniciado por **SP**.

* Google Cloud (G Suite) Connector admite el [aprovisionamiento de usuarios **automático**](./g-suite-provisioning-tutorial.md)

## <a name="adding-google-cloud-g-suite-connector-from-the-gallery"></a>Incorporación de Google Cloud (G Suite) Connector desde la galería

Para configurar la integración de Google Cloud (G Suite) Connector en Azure AD, deberá agregar Google Cloud (G Suite) Connector desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Google Cloud (G Suite) Connector** en el cuadro de búsqueda.
1. Seleccione **Google Cloud (G Suite) Connector** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-google-cloud-g-suite-connector"></a>Configuración y prueba del inicio de sesión único de Azure AD para Google Cloud (G Suite) Connector

Configure y pruebe el inicio de sesión único de Azure AD con Google Cloud (G Suite) Connector mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Google Cloud (G Suite) Connector.

Para configurar y probar el inicio de sesión único de Azure AD con Google Cloud (G Suite) Connector, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Google Cloud (G Suite) Connector](#configure-google-cloud-g-suite-connector-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Google Cloud (G Suite) Connector](#create-google-cloud-g-suite-connector-test-user)** , para tener un homólogo de B.Simon en Google Cloud (G Suite) Connector que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Google Cloud (G Suite) Connector**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar **Gmail**, realice los pasos siguientes:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.google.com/a/<yourdomain.com>/ServiceLogin?continue=https://mail.google.com`.

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:

    ```http
    google.com/a/<yourdomain.com>
    google.com
    https://google.com
    https://google.com/a/<yourdomain.com>
    ```

    c. En el cuadro de texto **URL de respuesta** , escriba una dirección URL con el siguiente patrón: 

    ```http
    https://www.google.com/acs
    https://www.google.com/a/<yourdomain.com>/acs
    ```

1. En la sección **Configuración básica de SAML**, si desea configurar **Google Cloud Platform**, realice los pasos siguientes:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://www.google.com/a/<yourdomain.com>/ServiceLogin?continue=https://console.cloud.google.com`.

    b. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón:
    
    ```http
    google.com/a/<yourdomain.com>
    google.com
    https://google.com
    https://google.com/a/<yourdomain.com>
    ```
    
    c. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: . 
    
    ```http
    https://www.google.com/acs
    https://www.google.com/a/<yourdomain.com>/acs
    ```

    > [!NOTE]
    > Estos valores no son reales. Debe actualizarlos con la dirección URL y el identificador reales de inicio de sesión. Google Cloud (G Suite) Connector no proporciona el valor de identificador de entidad/identificador en la configuración del inicio de sesión único, por lo que si desactiva la opción **domain specific issuer** (emisor de dominio específico), el valor del identificador será `google.com`. Si activa la opción **domain specific issuer** (emisor de dominio específico), la opción será `google.com/a/<yourdomainname.com>`. Para activar o desactivar la opción **domain specific issuer** (emisor de dominio específico) debe ir a la sección **Configuración del inicio de sesión único de Google Cloud (G Suite) Connector** que se explica más adelante en el tutorial. Para más información, póngase en contacto con el [equipo de soporte de cliente de Google Cloud (G Suite) Connector](https://www.google.com/contact/).

1. La aplicación Google Cloud (G Suite) Connector espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de pantalla le muestra un ejemplo de esto. El valor predeterminado de **Identificador de usuario único** es **user.userprincipalname**, pero Google Cloud (G Suite) Connector espera que este valor se asigne a la dirección de correo electrónico del usuario. Para ello, puede usar el atributo **user.mail** de la lista o usar el valor de atributo correspondiente en función de la configuración de su organización.

    ![imagen](common/default-attributes.png)


1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Google Cloud (G Suite) Connector**, copie las direcciones URL adecuadas según sus necesidades.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

    ```Logout URL
    https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0
    ```

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

En esta sección, permitirá que B.Simon acceda a Google Cloud (G Suite) Connector mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Google Cloud (G Suite) Connector**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-google-cloud-g-suite-connector-sso"></a>Configuración del inicio de sesión único de Google Cloud (G Suite) Connector

1. Abra una nueva pestaña en el explorador e inicie sesión en la [consola de administración de Google Cloud (G Suite) Connector](https://admin.google.com/) con su cuenta de administrador.

2. Haga clic en **Seguridad**. Si no ve el vínculo, puede estar oculto debajo del menú **Más controles** en la parte inferior de la pantalla.

    ![Haga clic en Seguridad.][10]

3. En la página **Seguridad**, haga clic en **Configurar inicio de sesión único (SSO)** .

    ![Haga clic en SSO.][11]

4. Realice los cambios de configuración siguientes:

    ![Configuración de SSO][12]

    a. Seleccione **Configurar SSO con un proveedor de identidades de terceros**.

    b. En el campo **Dirección URL de la página de inicio de sesión** de Google Cloud (G Suite) Connector, pegue el valor de la **URL de inicio de sesión** que copió de Azure Portal.

    c. En el campo **Dirección URL de la página de cierre de sesión** de Google Cloud (G Suite) Connector, pegue el valor de la **URL de inicio de sesión** que copió de Azure Portal.

    > [!NOTE]
    > Google Cloud (G Suite) se basa en el protocolo de cierre de sesión de SAML. Por lo tanto, en el campo **Dirección URL de la página de cierre de sesión** debe usar la dirección URL de cierre de sesión de SAML; es decir, la dirección URL de inicio de sesión como valor para el mismo.

    d. En Google Cloud (G Suite) Connector, para el **certificado de verificación**, cargue el certificado que descargó de Azure Portal.   

    e. Active o desactive la opción **Use a domain specific issuer** (Usar un emisor de dominio específico) según la nota mencionada en la sección **Configuración básica de SAML** anterior en Azure AD.

    f. En el campo **Cambiar URL de contraseña** de Google Cloud (G Suite) Connector, pegue el valor de **Cambiar URL de contraseña** que copió de Azure Portal.

    g. Haga clic en **Save**(Guardar).

### <a name="create-google-cloud-g-suite-connector-test-user"></a>Creación de un usuario de prueba de Google Cloud (G Suite) Connector

El objetivo de esta sección es [crear un usuario en Google Cloud (G Suite) Connector](https://support.google.com/a/answer/33310?hl=en) llamado B.Simon. Una vez creado manualmente el usuario en Google Cloud (G Suite) Connector, este podrá iniciar sesión con sus credenciales de inicio de sesión de Microsoft 365.

Google Cloud (G Suite) Connector también admite el aprovisionamiento de usuarios automático. Para configurar el aprovisionamiento automático de usuarios, primero debe [configurar Google Cloud (G Suite) Connector para el aprovisionamiento automático de usuarios](./g-suite-provisioning-tutorial.md).

> [!NOTE]
> Asegúrese de que el usuario ya existe en Google Cloud (G Suite) Connector si el aprovisionamiento en Azure AD no se ha activado antes de probar el inicio de sesión único.

> [!NOTE]
> Si necesita crear manualmente un usuario, póngase en contacto con el [equipo de soporte técnico de Google](https://www.google.com/contact/).

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Google Cloud (G Suite) Connector, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de Google Cloud (G Suite) Connector e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Google Cloud (G Suite) Connector en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Google Cloud (G Suite) Connector, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).

<!--Image references-->

[10]: ./media/google-apps-tutorial/gapps-security.png
[11]: ./media/google-apps-tutorial/security-gapps.png
[12]: ./media/google-apps-tutorial/gapps-sso-config.png
