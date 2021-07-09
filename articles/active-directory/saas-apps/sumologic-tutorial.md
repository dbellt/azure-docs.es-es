---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SumoLogic | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SumoLogic.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/14/2021
ms.author: jeedes
ms.openlocfilehash: 16b54db4ed05e5e9133dcb210ed1d8dac97595fc
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110090969"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sumologic"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con SumoLogic

En este tutorial, aprenderá cómo integrar SumoLogic con Azure Active Directory (Azure AD). Al integrar SumoLogic con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SumoLogic.
* Permitir a los usuarios iniciar sesión automáticamente en SumoLogic con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en SumoLogic.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SumoLogic admite el inicio de sesión único iniciado por **IDP**.

## <a name="add-sumologic-from-the-gallery"></a>Incorporación de SumoLogic desde la galería

Para configurar la integración de SumoLogic en Azure AD, deberá agregar SumoLogic desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SumoLogic** en el cuadro de búsqueda.
1. Seleccione **SumoLogic** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-sumologic"></a>Configuración y prueba del inicio de sesión único de Azure AD para SumoLogic

Configure y pruebe el inicio de sesión único de Azure AD con SumoLogic mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SumoLogic.

Para configurar y probar el inicio de sesión único de Azure AD con SumoLogic, lleve a cabo los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en SumoLogic](#configure-sumologic-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SumoLogic](#create-sumologic-test-user)** : para tener un homólogo de B.Simon en SumoLogic que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **SumoLogic**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar inicio de sesión único con SAML** realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con uno de los siguientes patrones:

    | URL del identificador |
    |---|
    | `https://service.sumologic.com`|
    | `https://<tenantname>.us2.sumologic.com`|
    | `https://<tenantname>.us4.sumologic.com`|
    | `https://<tenantname>.eu.sumologic.com`|
    | `https://<tenantname>.jp.sumologic.com`|
    | `https://<tenantname>.de.sumologic.com`|
    | `https://<tenantname>.ca.sumologic.com`|
    |

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con uno de los siguientes patrones:

    | URL de respuesta |
    |---|
    | `https://service.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.us2.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.us4.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.eu.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.jp.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.de.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.ca.sumologic.com/sumo/saml/consume/<tenantname>` |
    | `https://service.au.sumologic.com/sumo/saml/consume/<tenantname>` |
    |

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Póngase en contacto con el [equipo de soporte al cliente de SumoLogic](https://www.sumologic.com/contact-us/) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación SumoLogic espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación SumoLogic espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

    |  Nombre | Atributo de origen |
    | ---------------| --------------- |
    | Nombre | user.givenname |
    | Apellidos | user.surname |
    | Roles | user.assignedroles |

    > [!NOTE]
    > Haga clic [aquí](../develop/active-directory-enterprise-app-role-management.md) para saber cómo configurar el valor de **Role** en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar SumoLogic**, copie las direcciones URL adecuadas en función de sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a SumoLogic mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SumoLogic**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sumologic-sso"></a>Configuración del inicio de sesión único de SumoLogic

1. En otra ventana del explorador web, inicie sesión en su sitio de la compañía de SumoLogic como administrador.

1. Vaya a **Administrar** -> **Seguridad**.

    ![Administración](./media/sumologic-tutorial/security.png "Administrar")

1. Haga clic en **SAML**.

    ![Configuración de seguridad global](./media/sumologic-tutorial/settings.png "Configuración de seguridad global")

1. En la lista **Seleccionar una configuración o crear una nueva**, seleccione **Azure AD** y, después, haga clic en **Configurar**.

    ![Captura de pantalla que muestra Configure SAML 2.0 (Configurar S A M L 2.0), donde puede seleccionar Azure A D.](./media/sumologic-tutorial/configure.png "Configurar SAML 2.0")

1. En el cuadro de diálogo **Configurar SAML 2.0** , realice los pasos siguientes:

    ![Captura de pantalla que muestra el cuadro de diálogo Configure SAML 2.0 (Configurar S A M L 2.0), donde puede especificar los valores descritos.](./media/sumologic-tutorial/configuration.png "Configurar SAML 2.0")

    a. En el cuadro de texto **Nombre de configuración**, escriba **Azure AD**.

    b. Seleccione **Modo de depuración**.

    c. En el cuadro de texto **Issuer** (Emisor), pegue el valor de **Identificador Azure AD** que ha copiado de Azure Portal.

    d. En el cuadro de texto **Authentication request URL** (Dirección URL de solicitud de autenticación), pegue el valor del campo **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. Abra el certificado codificado en base 64 en el Bloc de notas, copie su contenido en el Portapapeles y luego pegue todo el certificado en el cuadro de texto **Certificado X.509** .

    f. Como **Atributo de correo electrónico**, seleccione **Usar SAML Subject**.  

    g. Seleccione **Configuración de inicio de sesión iniciada por el SP**.

    h. En el cuadro de texto de la **ruta de acceso de inicio de sesión**, escriba **Azure** y haga clic en **Guardar**.

### <a name="create-sumologic-test-user"></a>Creación de un usuario de prueba de SumoLogic

Para permitir que los usuarios de Azure AD inicien sesión en SumoLogic, deben aprovisionarse a SumoLogic. En el caso de SumoLogic, el aprovisionamiento es una tarea manual.

**Para aprovisionar una cuenta de usuario, realice estos pasos:**

1. Inicie sesión en su inquilino de **SumoLogic**.

1. Vaya a **Administrar \> usuarios**.

    ![Captura de pantalla que muestra la opción Usuarios seleccionada en el menú Administrar.](./media/sumologic-tutorial/user.png "Usuarios")

1. Haga clic en **Agregar**.

    ![Captura de pantalla que muestra el botón para Agregar para Usuarios.](./media/sumologic-tutorial/add-user.png "Usuarios")

1. En el cuadro de diálogo **Nuevo usuario** , realice los pasos siguientes:

    ![New User](./media/sumologic-tutorial/new-account.png "Nuevo usuario") (Usuario nuevo)

    a. Escriba la información relacionada de la cuenta de Azure AD que quiere aprovisionar en los cuadros de texto **First Name** (Nombre), **Last Name** (Apellidos) y **Email** (Correo electrónico).
  
    b. Seleccione un rol.
  
    c. Como **Estado**, seleccione **Activo**.
  
    d. Haga clic en **Save**(Guardar).

> [!NOTE]
> Puede usar cualquier otra API o herramienta de creación de cuentas de usuario de SumoLogic ofrecida por SumoLogic para aprovisionar cuentas de usuario de Azure AD.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de SumoLogic para la que configuró el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de SumoLogic en el panel Mis aplicaciones, debería iniciar sesión automáticamente en la versión de SumoLogic para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SumoLogic, puede aplicar el control de sesión, que protege la exfiltración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
