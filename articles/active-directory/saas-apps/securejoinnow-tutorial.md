---
title: 'Tutorial: integración de Azure Active Directory con SecureW2 JoinNow Connector | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y SecureW2 JoinNow Connector.
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
ms.openlocfilehash: f000cf2b58bb9f9b3c2077dbad490e6a4ff3da43
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109735915"
---
# <a name="tutorial-integrate-securew2-joinnow-connector-with-azure-active-directory"></a>Tutorial: Integración de SecureW2 JoinNow Connector con Azure Active Directory

En este tutorial, aprenderá a integrar SecureW2 JoinNow Connector con Azure Active Directory (Azure AD). Al integrar SecureW2 JoinNow Connector con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a SecureW2 JoinNow Connector
* Permitir que los usuarios inicien sesión automáticamente en SecureW2 JoinNow Connector con sus cuentas de Azure AD
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en SecureW2 JoinNow Connector.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* SecureW2 JoinNow Connector admite el inicio de sesión único iniciado por **SP**.

## <a name="add-securew2-joinnow-connector-from-the-gallery"></a>Adición de SecureW2 JoinNow Connector desde la galería

Para configurar la integración de SecureW2 JoinNow Connector en Azure AD, deberá agregar SecureW2 JoinNow Connector desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **SecureW2 JoinNow Connector** en el cuadro de búsqueda.
1. Seleccione **SecureW2 JoinNow Connector** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-securew2-joinnow-connector"></a>Configuración y prueba del inicio de sesión único de Azure AD para SecureW2 JoinNow Connector

Configure y pruebe el inicio de sesión único de Azure AD con SecureW2 JoinNow Connector mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de SecureW2 JoinNow Connector.

Para configurar y probar el inicio de sesión único de Azure AD con SecureW2 JoinNow Connector, siga este procedimiento:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de SecureW2 JoinNow Connector](#configure-securew2-joinnow-connector-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de SecureW2 JoinNow Connector](#create-securew2-joinnow-connector-test-user)** , para tener un homólogo de B.Simon en SecureW2 JoinNow Connector que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **SecureW2 JoinNow Connector**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<organization-identifier>-auth.securew2.com/auth/saml/SSO`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<organization-identifier>-auth.securew2.com/auth/saml`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con la dirección URL y el identificador reales de inicio de sesión. Póngase en contacto con el [equipo de atención al cliente de SecureW2 JoinNow Connector](mailto:support@securew2.com) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos** y seleccione **Descargar** para descargar el certificado y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar SecureW2 JoinNow Connector**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, permitirá que B.Simon acceda a SecureW2 JoinNow Connector mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **SecureW2 JoinNow Connector**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-securew2-joinnow-connector-sso"></a>Configuración del inicio de sesión único de SecureW2 JoinNow Connector

Para configurar el inicio de sesión único en **SecureW2 JoinNow Connector**, es preciso enviar el **XML de metadatos** descargado y las direcciones URL correspondientes copiadas de Azure Portal al [equipo de soporte técnico de SecureW2 JoinNow Connector](mailto:support@securew2.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-securew2-joinnow-connector-test-user"></a>Creación de un usuario de prueba de SecureW2 JoinNow Connector

En esta sección, creará el usuario Britta Simon en SecureW2 JoinNow Connector. Trabaje con el [equipo de soporte técnico de SecureW2 JoinNow Connector](mailto:support@securew2.com) para agregar los usuarios a la plataforma de SecureW2 JoinNow Connector. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de SecureW2 JoinNow Connector, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de SecureW2 JoinNow Connector e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de SecureW2 JoinNow Connector en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de la aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado SecureW2 JoinNow Connector, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).