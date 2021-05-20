---
title: 'Tutorial: Integración de Azure Active Directory con Zscaler Beta | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler Beta.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2020
ms.author: jeedes
ms.openlocfilehash: 080217c511470422d563b64f7609d71a4f9a6594
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108802745"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-beta"></a>Tutorial: Integración de Azure Active Directory con Zscaler Beta

En este tutorial, aprenderá a integrar Zscaler Beta con Azure Active Directory (Azure AD).
Al integrar Zscaler Beta con Azure AD, puede hacer lo siguiente:

- Controlar en Azure AD quién tiene acceso a Zscaler Beta.
- Permitir que los usuarios inicien sesión automáticamente en Zscaler Beta con sus cuentas de Azure AD. Este control de acceso se llama inicio de sesión único (SSO).
- Administrar las cuentas en una ubicación central mediante Azure Portal.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Zscaler Beta, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción de Zscaler Beta que use el inicio de sesión único.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Zscaler Beta admite el inicio de sesión único iniciado por **SP**.
- Zscaler Beta admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-zscaler-beta-from-the-gallery"></a>Incorporación de Zscaler Beta desde la galería

Para configurar la integración de Zscaler Beta en Azure AD, deberá agregar Zscaler Beta desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler Beta** en el cuadro de búsqueda.
1. Seleccione **Zscaler Beta** en el panel de resultados y, luego, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-zscaler-beta"></a>Configuración y prueba del inicio de sesión único de Azure AD para Zscaler Beta

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler Beta mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler Beta.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler Beta, haga lo siguiente:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Zscaler Beta](#configure-zscaler-beta-sso)** : para configurar el inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Zscaler Beta](#create-zscaler-beta-test-user)** : para tener un homólogo de Britta Simon en Zscaler Beta que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Zscaler Beta**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL con la que los usuarios inician sesión en la aplicación Zscaler Beta.

   > [!NOTE]
   > Este valor no es real. Actualice el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de Zscaler Beta](https://www.zscaler.com/company/contact) para obtener este valor.

1. La aplicación Zscaler Beta espera las aserciones de SAML en un formato específico. Debe agregar asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados. Seleccione **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

   ![Cuadro de diálogo Atributos de usuario](common/edit-attribute.png)

1. La aplicación Zscaler Beta espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice estos pasos para agregar el atributo del token de SAML como se muestra en la tabla siguiente.

   | Nombre     | Atributo de origen   |
   | -------- | ------------------ |
   | memberOf | user.assignedroles |

   a. Seleccione **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

   b. En el cuadro **Nombre**, escriba el nombre de atributo que se muestra para la fila.

   c. Deje el cuadro **Espacio de nombres** en blanco.

   d. En **Origen**, seleccione **Atributo**.

   e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

   f. Seleccione **Aceptar**.

   g. Seleccione **Guardar**.

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para saber cómo configurar el valor de Role en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, seleccione **Descargar** para descargar el **Certificado (base 64)**. Guárdelo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Zscaler Beta**, copie las direcciones URL adecuadas según sus necesidades:

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

En esta sección va a permitir que B.Simon acceda a Zscaler Beta mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Zscaler Beta**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-zscaler-beta-sso"></a>Configuración del inicio de sesión único en Zscaler Beta

1. Para automatizar la configuración de Zscaler Beta, instale la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, seleccione **Instale la extensión**.

   ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, al seleccionar **Configurar Zscaler Beta** se abre la aplicación Zscaler Beta. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Zscaler Beta. La extensión del explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

   ![Configuración](common/setup-sso.png)

3. Para configurar Zscaler Beta manualmente, abra una nueva ventana del explorador web. Inicie sesión como administrador en el sitio de la empresa de Zscaler Beta y siga estos pasos.

4. Vaya a **Administración** > **Autenticación** > **Configuración de autenticación** y siga estos pasos.

   ![Administración](./media/zscaler-beta-tutorial/ic800206.png "Administración")

   a. En **Tipo de autenticación**, seleccione **SAML**.

   b. Seleccione **Configurar SAML**.

5. En la ventana **Editar SAML**, siga estos pasos: ![Administración de usuarios y autenticación](./media/zscaler-beta-tutorial/ic800208.png "Manage Users & Authentication")

   a. En el cuadro **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que copió de Azure Portal.

   b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

   c. En el cuadro de texto **Certificado SSL público**, seleccione **Cargar** para cargar el certificado de firma de SAML que descargó de Azure Portal.

   d. Seleccione **Habilitar aprovisionamiento automático de SAML**.

   e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para los atributos displayName.

   f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para los atributos memberOf.

   g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para los atributos department.

   h. Seleccione **Guardar**.

6. En la página del cuadro de diálogo **Configurar autenticación de usuario**, siga estos pasos:

   ![Menú de activación y botón Activar](./media/zscaler-beta-tutorial/ic800207.png)

   a. Mantenga el puntero sobre el menú **Activación** situado en la parte inferior izquierda.

   b. Seleccione **Activar**.

## <a name="configure-proxy-settings&quot;></a>Configuración de los valores de proxy

Para definir la configuración de proxy en Internet Explorer, siga estos pasos.

1. Inicie **Internet Explorer**.

2. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el cuadro de diálogo **Opciones de Internet**.

   ![Cuadro de diálogo Opciones de Internet](./media/zscaler-beta-tutorial/ic769492.png &quot;Opciones de Internet")

3. Seleccione la pestaña **Conexiones**.

   ![Pestaña Connections (Conexiones)](./media/zscaler-beta-tutorial/ic769493.png "Conexiones")

4. Seleccione **Configuración de LAN** para abrir el cuadro de diálogo **Configuración de red de área local (LAN)**.

5. En la sección **Servidor proxy**, siga estos pasos:

   ![Sección del servidor proxy](./media/zscaler-beta-tutorial/ic769494.png "Servidor proxy")

   a. Seleccione la casilla de verificación **Usar un servidor proxy para la LAN**.

   b. En el cuadro de texto **Dirección**, escriba **gateway.Zscaler Beta.net**.

   c. En **Puerto**, escriba **80**.

   d. Seleccione la casilla de verificación **No usar servidor proxy para direcciones locales**.

   e. Seleccione **Aceptar** para cerrar el cuadro de diálogo **Configuración de red de área local (LAN)**.

6. Seleccione **Aceptar** para cerrar el cuadro de diálogo **Opciones de Internet**.

### <a name="create-zscaler-beta-test-user"></a>Creación de un usuario de prueba de Zscaler Beta

En esta sección, se crea el usuario Britta Simon en Zscaler Beta. Zscaler Beta admite el **aprovisionamiento de usuarios Just-In-Time**, que está habilitado de forma predeterminada. No hay nada para hacer en esta sección. Si un usuario deja de existir en Zscaler Beta, se crea uno después de la autenticación.

> [!Note]
> Para crear un usuario de forma manual, póngase en contacto con el [equipo de soporte técnico de Zscaler Beta](https://www.zscaler.com/company/contact).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Zscaler Beta, donde puede iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Zscaler Beta e inicie el flujo de inicio de sesión desde allí.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Zscaler Beta en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Zscaler Beta, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
