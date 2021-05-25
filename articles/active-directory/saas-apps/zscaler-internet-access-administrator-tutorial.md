---
title: 'Tutorial: Integración de Azure Active Directory con Zscaler Internet Access Administrator | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Internet Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2021
ms.author: jeedes
ms.openlocfilehash: f94aee8fd609870e2ac0e49721e8d7c21892428b
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804293"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-internet-access-administrator"></a>Tutorial: Integración de Azure Active Directory con Zscaler Internet Access Administrator

En este tutorial, aprenderá a integrar Zscaler Internet Access Administrator con Azure Active Directory (Azure AD). Al integrar Zscaler Internet Access Administrator con Azure AD, puede:

- Controlar en Azure AD quién tiene acceso a Zscaler Internet Access Administrator.
- Permitir que los usuarios inicien sesión automáticamente en Zscaler Internet Access Administrator con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción que permita utilizar el inicio de sesión único (SSO) en Zscaler Internet Access Administrator.

> [!NOTE]
> Esta integración también está disponible para usarse desde el entorno de la nube del gobierno de EE. UU de Azure AD. Puede encontrar esta aplicación en la galería de aplicaciones de la nube del gobierno de EE. UU. de Azure AD y configurarla de la misma manera que en la nube pública.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Zscaler Internet Access Administrator permite utilizar el inicio de sesión único con **IDP**.

## <a name="add-zscaler-internet-access-administrator-from-the-gallery"></a>Incorporación de Zscaler Internet Access Administrator desde la galería

Para configurar la integración de Zscaler Internet Access Administrator con Azure AD, debe agregar Zscaler Internet Access Administrator desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler Internet Access Administrator** en el cuadro de búsqueda.
1. Seleccione **Zscaler Internet Access Administrator** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-zscaler-internet-access-administrator"></a>Configuración y prueba del inicio de sesión único de Azure AD en Zscaler Internet Access Administrator

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler Internet Access Administrator con un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler Internet Access Administrator.

Para configurar el inicio de sesión único en Azure AD con Zscaler Internet Access Administrator, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Zscaler Internet Access Administrator](#configure-zscaler-internet-access-administrator-sso)** : para configurar el inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Zscaler Internet Access Administrator](#create-zscaler-internet-access-administrator-test-user)** : para tener un homólogo de Britta Simon en Zscaler Internet Access Administrator que esté vinculado a la representación del usuario en Azure AD.
3. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Zscaler Internet Access Administrator**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   a. En el cuadro de texto **Identificador**, escriba una de las siguientes direcciones URL en función de sus necesidades:

   | Identificador                       |
   | -------------------------------- |
   | `https://admin.zscaler.net`      |
   | `https://admin.zscalerone.net`   |
   | `https://admin.zscalertwo.net`   |
   | `https://admin.zscalerthree.net` |
   | `https://admin.zscloud.net`      |
   | `https://admin.zscalerbeta.net`  |

   b. En el cuadro de texto **URL de respuesta**, escriba una de las siguientes direcciones URL en función de sus necesidades:

   | URL de respuesta                                    |
   | -------------------------------------------- |
   | `https://admin.zscaler.net/adminsso.do`      |
   | `https://admin.zscalerone.net/adminsso.do`   |
   | `https://admin.zscalertwo.net/adminsso.do`   |
   | `https://admin.zscalerthree.net/adminsso.do` |
   | `https://admin.zscloud.net/adminsso.do`      |
   | `https://admin.zscalerbeta.net/adminsso.do`  |

1. La aplicación Zscaler Internet Access Administrator espera las aserciones de SAML en un formato específico. Configure las siguientes notificaciones para esta aplicación. Puede administrar los valores de estos atributos en la sección **User Attributes & Claims** (Atributos y notificaciones del usuario) de la página de integración de aplicaciones. En la **página Configurar el inicio de sesión único con SAML**, haga clic en el botón **Editar** para abrir el cuadro de diálogo **User Attributes & Claims** (Atributos y notificaciones del usuario).

   ![Vínculo de atributos](./media/zscaler-internet-access-administrator-tutorial/attributes.png)

1. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, configure el atributo Token SAML como muestra la imagen anterior y realice los siguientes pasos:

   | Nombre | Atributo de origen   |
   | ---- | ------------------ |
   | Role | user.assignedroles |

   a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

   b. En la lista **Atributo de origen**, seleccione el valor del atributo.

   c. Haga clic en **Aceptar**.

   d. Haga clic en **Save**(Guardar).

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para saber cómo configurar el valor de Role en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Zscaler Internet Access Administrator** (Configurar Zscaler Internet Access Administrator), copie las direcciones URL que necesite.

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

En esta sección, permitirá que B.Simon use el inicio de sesión único de Azure para acceder a Zscaler Internet Access Administrator.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Zscaler Internet Access Administrator**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-zscaler-internet-access-administrator-sso"></a>Configuración del inicio de sesión único en Zscaler Internet Access Administrator

1. En otra ventana del explorador web, inicie sesión en la interfaz de usuario de Zscaler Internet Access Administrator.

2. Vaya a **Administración > Administración de administradores**, realice los pasos siguientes y haga clic en Guardar:

   ![Captura de pantalla que muestra Administrator Management (Administración de administradores) con opciones para habilitar la autenticación SAML, cargar el certificado S S L y especificar un emisor.](./media/zscaler-internet-access-administrator-tutorial/management.png "Administración")

   a. Marque **Enable SAML Authentication** (Habilitar autenticación SAML).

   b. Haga clic en **Upload** (Cargar) para cargar el certificado de firma de SAML de Azure que ha descargado desde Azure Portal en el **Certificado SSL público**.

   c. Opcionalmente, para mayor seguridad, agregue los datos del **Emisor** para verificar el emisor de la respuesta SAML.

3. En la interfaz de usuario del administrador, realice los pasos siguientes:

   ![Captura de pantalla que muestra la interfaz de usuario del administrador, donde puede realizar los pasos.](./media/zscaler-internet-access-administrator-tutorial/activation.png)

   a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

   b. Haga clic en **Activar**.

### <a name="create-zscaler-internet-access-administrator-test-user"></a>Creación de un usuario de prueba en Zscaler Internet Access Administrator

El objetivo de esta sección es crear un usuario llamado Britta Simon en Zscaler Internet Access Administrator. Zscaler Internet Access no admite el aprovisionamiento Just-In-Time para el inicio de sesión único del administrador. Debe crear una cuenta de administrador manualmente.
Para instrucciones sobre cómo crear una cuenta de administrador, consulte la documentación de Zscaler:

https://help.zscaler.com/zia/adding-admins

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en Probar esta aplicación en Azure Portal; debería iniciar sesión automáticamente en la instancia de Zscaler Internet Access Administrator para la que configuró el inicio de sesión único.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Zscaler Internet Access Administrator en Aplicaciones, debería iniciar sesión automáticamente en la instancia de Zscaler Internet Access Administrator para la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Zscaler Internet Access Administrator, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
