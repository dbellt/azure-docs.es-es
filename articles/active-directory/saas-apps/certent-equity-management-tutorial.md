---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Certent Equity Management | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Certent Equity Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/16/2020
ms.author: jeedes
ms.openlocfilehash: 0ebebd323c683036a472106894cb0e42ba460996
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "108807546"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-certent-equity-management"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Certent Equity Management

En este tutorial, aprenderá a integrar Certent Equity Management con Azure Active Directory (Azure AD). Al integrar Certent Equity Management con Azure AD, puede:

- Controlar en Azure AD quién tiene acceso a Certent Equity Management.
- Permitir que los usuarios inicien sesión automáticamente en Certent Equity Management con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción de Certent Equity Management habilitada para el inicio de sesión único (SSO).

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Certent Equity Management admite el SSO iniciado por **IDP**

## <a name="adding-certent-equity-management-from-the-gallery"></a>Adición de Certent Equity Management desde la galería

Para configurar la integración de Certent Equity Management en Azure AD, es preciso agregar Certent Equity Management desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Certent Equity Management** en el cuadro de búsqueda.
1. Seleccione **Certent Equity Management** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-certent-equity-management"></a>Configuración y prueba del inicio de sesión único de Azure AD para Certent Equity Management

Configure y pruebe el inicio de sesión único de Azure AD con Certent Equity Management con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Certent Equity Management.

Para configurar y probar el inicio de sesión único de Azure AD con Certent Equity Management, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Certent Equity Management](#configure-certent-equity-management-sso)**: para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Certent Equity Management](#create-certent-equity-management-test-user)**: para tener un homólogo de B.Simon en Certent Equity Management que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Certent Equity Management**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configurar el inicio de sesión único con SAML**, escriba los valores de los siguientes campos:

   a. En el cuadro de texto **Identificador**, escriba una dirección URL con el patrón siguiente: `https://<SUBDOMAIN>.certent.com/sys/sso/saml/acs.aspx`

   b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.certent.com/sys/sso/saml/acs.aspx`

   > [!NOTE]
   > Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Estos valores no son reales. Actualice estos valores con el identificador y la URL de respuesta reales. Para obtener estos valores, póngase en contacto con el analista de integración de Certent asignado por el Customer Success Manager. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Certent Equity Management espera las aserciones de SAML en un formato concreto, lo que requiere la agregación de asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados.

   ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Certent Equity Management espera que se devuelvan algunos atributos más, que se muestran a continuación, en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.

   | Nombre    | Atributo de origen       |
   | ------- | ---------------------- |
   | COMPANY | user.companyname       |
   | USER    | user.userprincipalname |
   | ROLE    | user.assignedroles     |

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para saber cómo configurar el valor de **Role** en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **XML de metadatos de federación** y seleccione **Descargar** para descargar el certificado y guardarlo en su equipo.

   ![Vínculo de descarga del certificado](common/metadataxml.png)

1. En la sección **Configurar Certent Equity Management**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a conceder a B.Simon acceso a Certent Equity Management mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Certent Equity Management**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-certent-equity-management-sso"></a>Configuración del inicio de sesión único de Certent Equity Management

Para configurar el inicio de sesión único en **Certent Equity Management**, es preciso enviar el **XML de metadatos de federación** descargado y las direcciones URL apropiadas copiadas de Azure Portal al analista de integración de Certent asignado por el Customer Success Manager. Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-certent-equity-management-test-user"></a>Creación de un usuario de prueba de Certent Equity Management

En esta sección, se crea el usuario Britta Simon en Certent Equity Management. Trabaje con el analista de integración de Certent asignado por Customer Success Manager para agregar los usuarios a la plataforma de Certent Equity Management. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Certent Equity Management para la que ha configurado el inicio de sesión único.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Certent Equity Management en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Certent Equity Management para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Certent Equity Management, puede aplicar el control de sesión, que protege la filtración e infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
