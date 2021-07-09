---
title: 'Tutorial: Integración de Azure Active Directory con Deputy | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Deputy.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: jeedes
ms.openlocfilehash: a72900f33347370e193df9ed040df4ff9e51528f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063608"
---
# <a name="tutorial-azure-active-directory-integration-with-deputy"></a>Tutorial: Integración de Azure Active Directory con Deputy

En este tutorial, aprenderá a integrar Deputy con Azure Active Directory (Azure AD). Al integrar Deputy con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Deputy.
* Permitir que los usuarios inicien sesión automáticamente en Deputy con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Deputy.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Deputy admite el inicio de sesión único iniciado por **SP e IDP**.
* Deputy admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-deputy-from-the-gallery"></a>Incorporación de Deputy desde la galería

Para configurar la integración de Deputy en Azure AD, es preciso agregar Deputy desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Deputy** en el cuadro de búsqueda.
1. Seleccione **Deputy** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-deputy"></a>Configuración y prueba del SSO de Azure AD para Deputy

Configure y pruebe el inicio de sesión único de Azure AD con Deputy mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario correspondiente de Deputy.

Para configurar y probar el inicio de sesión único de Azure AD con Deputy, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Deputy](#configure-deputy-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de usuario de prueba de Deputy](#create-deputy-test-user)** , para tener un homólogo de B.Simon en Deputy que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Deputy**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en el modo iniciado por **IDP** siga estos pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    ```http
    https://<subdomain>.<region>.au.deputy.com
    https://<subdomain>.<region>.ent-au.deputy.com
    https://<subdomain>.<region>.na.deputy.com
    https://<subdomain>.<region>.ent-na.deputy.com
    https://<subdomain>.<region>.eu.deputy.com
    https://<subdomain>.<region>.ent-eu.deputy.com
    https://<subdomain>.<region>.as.deputy.com
    https://<subdomain>.<region>.ent-as.deputy.com
    https://<subdomain>.<region>.la.deputy.com
    https://<subdomain>.<region>.ent-la.deputy.com
    https://<subdomain>.<region>.af.deputy.com
    https://<subdomain>.<region>.ent-af.deputy.com
    https://<subdomain>.<region>.an.deputy.com
    https://<subdomain>.<region>.ent-an.deputy.com
    https://<subdomain>.<region>.deputy.com
    ```

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:
    
    ```http
    https://<subdomain>.<region>.au.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-au.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.na.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-na.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.eu.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-eu.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.as.deputy.com/exec/devapp/samlacs.
    https://<subdomain>.<region>.ent-as.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.la.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-la.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.af.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-af.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.an.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.ent-an.deputy.com/exec/devapp/samlacs
    https://<subdomain>.<region>.deputy.com/exec/devapp/samlacs
    ```

5. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<your-subdomain>.<region>.deputy.com`

    >[!NOTE]
    > El sufijo de la región de Deputy es opcional o debe usar uno de los siguientes: au | na | eu |as |la |af |an |ent-au |ent-na |ent-eu |ent-as | ent-la | ent-af | ent-an

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico de Deputy](https://www.deputy.com/call-centers-customer-support-scheduling-software) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Deputy espera las aserciones de SAML en un formato específico, lo que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Deputy espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre |  Atributo de origen|
    | -------------- | --------- |
    | Nombre | user.givenname|
    | Apellido | user.surname |

6. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

7. En la sección **Set up Deputy** (Configurar Deputy), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a Deputy.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Deputy**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-deputy-sso"></a>Configuración del inicio de sesión único de Deputy

1. Inicie sesión en su cuenta de Deputy como administrador.

1. En la esquina superior derecha, haga clic en su cuenta y seleccione **Business settings** (Configuración empresarial).

    ![Captura de pantalla de la configuración empresarial](./media/deputy-tutorial/business-settings.png)

1. A continuación, en la pestaña **General**, haga clic en **Configuración de inicio de sesión único**.

    ![Captura de pantalla de la configuración de inicio de sesión](./media/deputy-tutorial/general.png)

2. En la página **Configuración de inicio de sesión único** realice los pasos siguientes.

    ![Configurar inicio de sesión único](./media/deputy-tutorial/configuration.png)
    
    a. Haga clic en **Habilitar el inicio de sesión único**.
   
    b. En el cuadro de texto **URL de inicio de sesión del proveedor de identidades**, pegue el valor de **Dirección URL de inicio de sesión**  que ha copiado de Azure Portal.

    c. En el cuadro de texto **Emisor de proveedor de identidades**, pegue el valor de **Identificador (id. de entidad)** que ha copiado en Azure Portal.

    d. Abra el **Certificado (Base64)** que descargó de Azure Portal en el Bloc de notas, copie el contenido y péguelo en el cuadro de texto **Certificado X.509**.

    e. Habilite **Single sign-on login required** (Inicio de sesión único necesario) si desea iniciar sesión con SSO.
   
    f. Habilite **Just-in-time provisioning (Aprovisionamiento Just-In-Time)** y, en los campos **Nombre** y **Apellidos**, asigne los nombres de los atributos que ha configurado en la sección **Atributos y notificaciones de usuario**, como `First name` y `Last name`.

    g. Haga clic en **Apply changes** (Aplicar cambios).

### <a name="create-deputy-test-user"></a>Creación de usuario de prueba de Deputy

En esta sección, se creará el usuario B.Simon en Deputy. Deputy admite el aprovisionamiento de usuarios Just-In-Time, habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Deputy, se crea otro después de la autenticación.

#### <a name="to-add-the-user-manually-perform-the-following-steps"></a>Para agregar manualmente el usuario, siga los pasos siguientes:

1. Inicie sesión en su sitio de la compañía de Deputy como administrador.

2. En la parte superior del panel de navegación, haga clic en **Contactos**.

3. Haga clic en el botón **Agregar personas** y en **Add Single Person** (Agregar una sola persona).
   
    ![Add People (Agregar personas)](./media/deputy-tutorial/create-user-1.png "Agregar personas")

4. Realice los pasos siguientes en la pestaña **General** para agregar un usuario.
   
    ![New User](./media/deputy-tutorial/create-user-2.png "Nuevo usuario") (Usuario nuevo)

    a. En los cuadros de texto **Nombre** y **Apellidos**, rellene los campos como **Britta** y **Simon**.
   
    b. En el cuadro de texto **Trabaja en**, escriba el nombre de la empresa.
   
    c. Haga clic en el botón **Guardar** .

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de TeamzSkill, donde puede iniciar el flujo de inicio de sesión.  

* Vaya directamente a la dirección URL de inicio de sesión de TeamzSkill e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal; debería iniciar sesión automáticamente en la instancia de TeamzSkill para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de TeamzSkill en Mis aplicaciones, si está configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión y, si está configurado en modo IDP, debería iniciar sesión automáticamente en la instancia de TeamzSkill para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Deputy, puede aplicar el control de sesión, que protege la exfiltración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
