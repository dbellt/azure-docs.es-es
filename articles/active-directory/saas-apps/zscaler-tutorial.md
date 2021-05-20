---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Zscaler.
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
ms.openlocfilehash: c0b824b35c1cbeeda9639f321200b367d0152e0e
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803159"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-zscaler"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Zscaler

En este tutorial, aprenderá a integrar Zscaler con Azure Active Directory (Azure AD). Al integrar Zscaler con Azure AD, puede hacer lo siguiente:

- Controlar en Azure AD quién tiene acceso a Zscaler.
- Permitir que los usuarios inicien sesión automáticamente en Zscaler con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Suscripción habilitada para el inicio de sesión único (SSO) en Zscaler.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Zscaler admite el inicio de sesión único iniciado por **SP**
- Zscaler admite el aprovisionamiento de usuarios **Just-In-Time**

## <a name="adding-zscaler-from-the-gallery"></a>Incorporación de Zscaler desde la galería

Para configurar la integración de Zscaler en Azure AD, deberá agregar Zscaler desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler** en el cuadro de búsqueda.
1. Seleccione **Zscaler** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-zscaler"></a>Configuración y prueba del inicio de sesión único de Azure AD en Zscaler

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler mediante una usuaria de prueba llamada **B. Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Zscaler](#configure-zscaler-sso)** , para configurar los valores de Inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Zscaler](#create-zscaler-test-user)** , para tener un homólogo de B. Simon en Zscaler vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Zscaler**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<companyname>.zscaler.net`

   > [!NOTE]
   > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico al cliente de Zscaler](https://www.zscaler.com/company/contact) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Zscaler espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

   ![Captura de pantalla que muestra User Attributes (Atributos de usuario) con el icono de edición seleccionado.](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Zscaler espera que se devuelvan algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:

   | Nombre     | Atributo de origen   |
   | -------- | ------------------ |
   | memberOf | user.assignedroles |

   a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

   b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

   c. Deje **Espacio de nombres** en blanco.

   d. Seleccione **Atributo** como origen.

   e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

   f. Haga clic en **Save**(Guardar).

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para saber cómo configurar el valor de Role en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Zscaler**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección va a permitir que B.Simon acceda a Zscaler mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Zscaler**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si ha configurado los roles tal y como se explica en el apartado anterior, puede seleccionarlo en la lista desplegable **Seleccionar un rol**.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-zscaler-sso"></a>Configuración del inicio de sesión único de Zscaler

1. Para automatizar la configuración en Zscaler, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

   ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

1. Después de agregar la extensión al explorador, haga clic en **Configurar Zscaler** para ir a la aplicación Zscaler. Desde allí, proporcione las credenciales de administrador para iniciar sesión en Zscaler. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

   ![Configuración del inicio de sesión único](common/setup-sso.png)

1. Si quiere configurar Zscaler manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Zscaler como administrador y haga lo siguiente:

1. Vaya a **Administración > Autenticación > Configuración de autenticación** y realice los siguientes pasos:

   ![Captura de pantalla que muestra el sitio de Zscaler One con los pasos descritos.](./media/zscaler-tutorial/ic800206.png "Administración")

   a. En Tipo de autenticación, elija **SAML**.

   b. Haga clic en **Configurar SAML**.

1. En la ventana **Editar SAML**, realice los pasos siguientes y haga clic en Guardar.

   ![Administración de usuarios y autenticación](./media/zscaler-tutorial/ic800208.png "Manage Users & Authentication")

   a. En el cuadro de texto **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

   b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

   c. Haga clic en **Cargar** para cargar el certificado de firma de SAML de Azure que ha descargado desde Azure Portal en el **Certificado SSL público**.

   d. Alterne **Habilitar aprovisionamiento automático de SAML**.

   e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para atributos displayName.

   f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para atributos memberOf.

   g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para atributos department.

   h. Haga clic en **Save**(Guardar).

1. En la página del cuadro de diálogo **Configurar autenticación de usuario** , realice los pasos siguientes:

   ![Captura de pantalla que muestra el cuadro de diálogo Configure User Authentication (Configurar autenticación de usuario) con la opción Activate (Activar) seleccionada.](./media/zscaler-tutorial/ic800207.png)

   a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

   b. Haga clic en **Activar**.

## <a name="configuring-proxy-settings&quot;></a>Configuración de los valores de proxy

### <a name=&quot;to-configure-the-proxy-settings-in-internet-explorer&quot;></a>Para definir la configuración de proxy en Internet Explorer

1. Inicie **Internet Explorer**.

1. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el diálogo **Opciones de Internet**.

   ![Opciones de Internet](./media/zscaler-tutorial/ic769492.png &quot;Opciones de Internet")

1. Haga clic en la pestaña **Conexiones** .

   ![Conexiones](./media/zscaler-tutorial/ic769493.png "Conexiones")

1. Haga clic en **Configuración de LAN** para abrir el diálogo **Configuración de LAN**.

1. En la sección del servidor proxy, lleve a cabo estos pasos:

   ![Servidor proxy](./media/zscaler-tutorial/ic769494.png "Servidor proxy")

   a. Seleccione **Usar un servidor proxy para la LAN**.

   b. En el cuadro de texto Dirección, escriba **gateway.zscaler.net**.

   c. En el cuadro de texto Puerto, escriba **80**.

   d. Seleccione **No usar servidor proxy para direcciones locales**.

   e. Haga clic en **Aceptar** para cerrar el diálogo **Configuración de red de área local (LAN)** .

1. Haga clic en **Aceptar** para cerrar el diálogo **Opciones de Internet**.

### <a name="create-zscaler-test-user"></a>Creación de un usuario de prueba de Zscaler

En esta sección, se crea un usuario llamado Britta Simon en Zscaler. Zscaler admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Zscaler, se crea uno después de la autenticación.

> [!Note]
> Si tiene que crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico de Zscaler](https://www.zscaler.com/company/contact).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Zscaler, donde puede iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Zscaler e inicie el flujo de inicio de sesión desde allí.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Zscaler en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Zscaler, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
