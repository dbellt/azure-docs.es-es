---
title: 'Tutorial: Integración de Azure Active Directory con Symantec Web Security Service (WSS) | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Symantec Web Security Service (WSS).
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
ms.openlocfilehash: af7d126bfdc9ff8edf6b498747fab9c7f497a0f4
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484853"
---
# <a name="tutorial-azure-active-directory-integration-with-symantec-web-security-service-wss"></a>Tutorial: Integración de Azure Active Directory con Symantec Web Security Service (WSS)

En este tutorial aprenderá cómo integrar su cuenta de Symantec Web Security Service (WSS) con su cuenta de Azure Active Directory (Azure AD) para que WSS pueda autenticar un usuario final que se aprovisiona en Azure AD mediante la autenticación de SAML y aplicar reglas de directiva de nivel de usuario o grupo.

La integración de Symantec Web Security Service (WSS) con Azure AD proporciona las siguientes ventajas:

- Administrar todos los usuarios finales y los grupos que usan la cuenta de WSS desde el portal de Azure AD.

- Permitir que los usuarios finales se autentiquen en WSS con sus credenciales de Azure AD.

- Habilitar la aplicación de reglas de directiva de nivel de grupo y usuario definidas en la cuenta de WSS.

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Symantec Web Security Service (WSS).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Symantec Web Security Service (WSS) admite inicio de sesión único iniciado por **IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-symantec-web-security-service-wss-from-the-gallery"></a>Adición de Symantec Web Security Service (WSS) desde la galería

Para configurar la integración de Symantec Web Security Service (WSS) en Azure AD, es preciso agregar Symantec Web Security Service (WSS) desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Symantec Web Security Service (WSS)** en el cuadro de búsqueda.
1. Seleccione **Symantec Web Security Service (WSS)** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-symantec-web-security-service-wss"></a>Configuración y prueba del inicio de sesión único de Azure AD para Symantec Web Security Service (WSS)

Configure y pruebe el inicio de sesión único de Azure AD con Symantec Web Security Service (WSS) mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vínculo entre un usuario de Azure AD y el usuario correspondiente de Symantec Web Security Service (WSS).

Para configurar el inicio de sesión único de Azure AD con Symantec Web Security Service (WSS), siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Symantec Web Security Service (WSS)](#configure-symantec-web-security-service-wss-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Symantec Web Security Service (WSS)](#create-symantec-web-security-service-wss-test-user)** : para tener un homólogo de B.Simon en Symantec Web Security Service (WSS) vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Symantec Web Security Service (WSS)** , busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En el cuadro de diálogo **Configuración básica de SAML**, realice los siguientes pasos:

    a. En el cuadro de texto **Identificador**, escriba la dirección URL: `https://saml.threatpulse.net:8443/saml/saml_realm`

    b. En el cuadro de texto **URL de respuesta**, escriba la dirección URL: `https://saml.threatpulse.net:8443/saml/saml_realm/bcsamlpost`

    > [!NOTE]
    > Póngase en contacto con el [equipo de soporte técnico de Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si los valores para **Identificador** y **URL de respuesta** no funcionasen por algún motivo. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

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

En esta sección va a permitir que B.Simon acceda a Symantec Web Security Service (WSS) mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Symantec Web Security Service (WSS)**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-symantec-web-security-service-wss-sso"></a>Configuración del inicio de sesión único en Symantec Web Security Service (WSS)

Para configurar el inicio de sesión único en Symantec Web Security Service (WSS), consulte la documentación en línea de WSS. El archivo **XML de metadatos de federación** descargado se debe importar en el portal de WSS. Póngase en contacto con el [equipo de soporte técnico de Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si necesita ayuda con la configuración en el portal de WSS.

### <a name="create-symantec-web-security-service-wss-test-user"></a>Creación de un usuario de prueba de Symantec Web Security Service (WSS)

En esta sección, creará un usuario llamado Britta Simon en Symantec Web Security Service (WSS). El nombre de usuario final correspondiente se puede crear manualmente en el portal de WSS o puede esperar a que los usuarios y grupos que se han aprovisionado en Azure AD se sincronicen con el portal de WSS transcurridos unos minutos (unos 15 minutos). Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único. La dirección IP pública del equipo del usuario final, que se utilizará para examinar sitios web, también debe aprovisionarse en el portal de Symantec Web Security Service (WSS).

> [!NOTE]
> Haga clic [aquí](https://www.bing.com/search?q=my+ip+address&qs=AS&pq=my+ip+a&sc=8-7&cvid=29A720C95C78488CA3F9A6BA0B3F98C5&FORM=QBLH&sp=1) para obtener la dirección IP pública de su máquina.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

* Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Symantec Web Security Service (WSS) para la que configurara el inicio de sesión único.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Symantec Web Security Service (WSS) en Aplicaciones, se debería iniciar sesión automáticamente en la instancia de Symantec Web Security Service (WSS) para la que configurara el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada Symantec Web Security Service (WSS), podrá aplicar el control de sesión, que protege la información confidencial de la organización de la filtración y la infiltración en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
