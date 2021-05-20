---
title: 'Tutorial: integración de Azure Active Directory con ZScaler ZSCloud | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y ZScaler ZSCloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/21/2020
ms.author: jeedes
ms.openlocfilehash: 74c65a28688906b4c9cfecf786f5862e31143ead
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804347"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-zscloud"></a>Tutorial: integración de Azure Active Directory con ZScaler ZSCloud

En este tutorial, aprenderá a integrar Zscaler ZSCloud con Azure Active Directory (Azure AD). Al integrar Zscaler ZSCloud con Azure AD, puede hacer lo siguiente:

- Controlar en Azure AD quién tiene acceso a Zscaler ZSCloud.
- Permitir que los usuarios inicien sesión automáticamente en Zscaler ZSCloud con sus cuentas de Azure AD.
- Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con ZScaler ZSCloud, necesita los siguientes elementos:

- Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción habilitada para el inicio de sesión único en Zscaler ZSCloud.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

- Zscaler ZSCloud admite el inicio de sesión único iniciado por **SP**.

- Zscaler ZSCloud admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-zscaler-zscloud-from-the-gallery"></a>Incorporación de ZScaler ZSCloud desde la galería

Para configurar la integración de ZScaler ZSCloud en Azure AD, será preciso agregar ZScaler ZSCloud desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Zscaler ZSCloud** en el cuadro de búsqueda.
1. Seleccione **Zscaler ZSCloud** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-zscaler-zscloud"></a>Configuración y prueba del inicio de sesión único de Azure AD para Zscaler ZSCloud

Configure y pruebe el inicio de sesión único de Azure AD con Zscaler ZSCloud mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Zscaler ZSCloud.

Para configurar y probar el inicio de sesión único de Azure AD con Zscaler ZSCloud, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
   1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
   1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Zscaler ZSCloud](#configure-zscaler-zscloud-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
   1. **[Creación de un usuario de prueba de Zscaler ZSCloud](#create-zscaler-zscloud-test-user)** , para tener un homólogo de B.Simon en Zscaler ZSCloud vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Zscaler ZSCloud**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

   En el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL con la que los usuarios inician sesión en la aplicación ZScaler ZSCloud.

   > [!NOTE]
   > Tiene que actualizar el valor con la dirección URL de inicio de sesión real. Póngase en contacto con el [equipo de soporte técnico de ZScaler ZSCloud](https://help.zscaler.com/) para obtenerlo. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. La aplicación Zscaler ZSCloud espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token SAML. La siguiente captura de muestra la lista de atributos predeterminados. Haga clic en el icono **Editar** para abrir el cuadro de diálogo **Atributos de usuario**.

   ![Captura de pantalla que muestra User Attributes (Atributos de usuario) con el icono de edición seleccionado.](common/edit-attribute.png)

1. Además de lo anterior, la aplicación Zscaler ZSCloud espera que se usen algunos atributos más en la respuesta de SAML. En la sección **Notificaciones del usuario** del cuadro de diálogo **Atributos de usuario**, realice los siguientes pasos para agregar el atributo Token SAML como se muestra en la tabla siguientes:

   | Nombre     | Atributo de origen   |
   | -------- | ------------------ |
   | memberOf | user.assignedroles |

   a. Haga clic en **Agregar nueva notificación** para abrir el cuadro de diálogo **Administrar las notificaciones del usuario**.

   ![Captura de pantalla que muestra User claims (Reclamaciones de usuario) con la opción Add new claim (Agregar nueva reclamación).](common/new-save-attribute.png)

   ![Captura de pantalla que muestra Manage user claims (Administrar reclamaciones de usuario) para escribir los valores que se describen.](common/new-attribute-details.png)

   b. En el cuadro de texto **Nombre**, escriba el nombre que se muestra para la fila.

   c. Deje **Espacio de nombres** en blanco.

   d. Seleccione **Atributo** como origen.

   e. En la lista **Atributo de origen**, escriba el valor de atributo que se muestra para esa fila.

   f. Haga clic en **Save**(Guardar).

   > [!NOTE]
   > Haga clic [aquí](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui) para saber cómo configurar el valor de Role en Azure AD.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (Base64)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Zscaler ZSCloud**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a Britta Simon para que use el inicio de sesión único de Azure concediéndole acceso a ZScaler ZSCloud.

1. En Azure Portal, seleccione **Aplicaciones empresariales**, **Todas las aplicaciones** y, luego, **Zscaler ZSCloud**.
2. En la lista de aplicaciones, seleccione **ZScaler ZSCloud**.
3. En el menú de la izquierda, seleccione **Usuarios y grupos**.
4. Haga clic en el botón **Agregar usuario** y, después, seleccione **Usuarios y grupos** en el cuadro de diálogo **Agregar asignación**.
5. En el cuadro de diálogo **Usuarios y grupos**, seleccione el usuario, como **Britta Simon**, en la lista y luego haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.

   ![Captura de pantalla que muestra el cuadro de diálogo Usuarios y grupos, donde puede seleccionar un usuario.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_users.png)

6. En el cuadro de diálogo **Seleccionar rol**, elija el rol de usuario adecuado de la lista y, luego, haga clic en el botón **Seleccionar** situado en la parte inferior de la pantalla.

   ![Captura de pantalla que muestra el cuadro de diálogo Seleccionar rol, donde puede elegir un rol de usuario.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_roles.png)

7. En el cuadro de diálogo **Agregar asignación**, seleccione el botón **Asignar**.

   ![Captura de pantalla que muestra el cuadro de diálogo Agregar asignación, donde puede seleccionar Asignar.](./media/zscaler-zscloud-tutorial/tutorial_zscalerzscloud_assign.png)

   > [!NOTE]
   > El rol de acceso predeterminado no se admite porque interrumpirá el aprovisionamiento, por lo que no se puede seleccionar al asignar el usuario.

## <a name="configure-zscaler-zscloud-sso"></a>Configuración del inicio de sesión único en Zscaler ZSCloud

1. Para automatizar la configuración en Zscaler ZSCloud, debe instalar la **extensión del explorador de inicio de sesión seguro de Mis aplicaciones**. Para ello, haga clic en **Instalar la extensión**.

   ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Después de agregar la extensión al explorador, haga clic en **Configurar Zscaler ZSCloud** para ir a la aplicación Zscaler ZSCloud. Desde allí, proporcione las credenciales del administrador para iniciar sesión en Zscaler ZSCloud. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 6.

   ![Configuración del inicio de sesión único](common/setup-sso.png)

3. Si quiere configurar Zscaler ZSCloud manualmente, abra una nueva ventana del explorador web, inicie sesión en el sitio de empresa de Zscaler ZSCloud como administrador y haga lo siguiente:

4. Vaya a **Administración > Autenticación > Configuración de autenticación** y realice los siguientes pasos:

   ![Captura de pantalla que muestra el sitio de Zscaler con los pasos descritos.](./media/zscaler-zscloud-tutorial/ic800206.png "Administración")

   a. En Tipo de autenticación, elija **SAML**.

   b. Haga clic en **Configurar SAML**.

5. En la ventana **Editar SAML**, realice los pasos siguientes y haga clic en Guardar.  

   ![Administración de usuarios y autenticación](./media/zscaler-zscloud-tutorial/ic800208.png "Manage Users & Authentication")

   a. En el cuadro de texto **Dirección URL del portal de SAML**, pegue la **dirección URL de inicio de sesión** que ha copiado de Azure Portal.

   b. En el cuadro de texto **Atributo de nombre de inicio de sesión**, escriba **NameID**.

   c. Haga clic en **Cargar** para cargar el certificado de firma de SAML de Azure que ha descargado de Azure Portal en **Certificado SSL público**.

   d. Alterne **Habilitar aprovisionamiento automático de SAML**.

   e. En el cuadro de texto **Atributo de nombre para mostrar del usuario**, escriba **displayName** si desea habilitar el aprovisionamiento automático de SAML para atributos displayName.

   f. En el cuadro de texto **Atributo de nombre del grupo**, escriba **memberOf** si desea habilitar el aprovisionamiento automático de SAML para atributos memberOf.

   g. En el cuadro de texto **Atributo de nombre del departamento**, escriba **department** si desea habilitar el aprovisionamiento automático de SAML para atributos department.

   h. Haga clic en **Save**(Guardar).

6. En la página del cuadro de diálogo **Configurar autenticación de usuario** , realice los pasos siguientes:

   ![Captura de pantalla que muestra el cuadro de diálogo Configure User Authentication (Configurar autenticación de usuario) con la opción Activate (Activar) seleccionada.](./media/zscaler-zscloud-tutorial/ic800207.png)

   a. Mantenga el puntero sobre el menú **Activación** situado cerca de la parte inferior izquierda.

   b. Haga clic en **Activar**.

## <a name="configuring-proxy-settings&quot;></a>Configuración de los valores de proxy

### <a name=&quot;to-configure-the-proxy-settings-in-internet-explorer&quot;></a>Para definir la configuración de proxy en Internet Explorer

1. Inicie **Internet Explorer**.

2. Seleccione **Opciones de Internet** en el menú **Herramientas** para abrir el diálogo **Opciones de Internet**.

   ![Opciones de Internet](./media/zscaler-zscloud-tutorial/ic769492.png &quot;Opciones de Internet")

3. Haga clic en la pestaña **Conexiones** .

   ![Conexiones](./media/zscaler-zscloud-tutorial/ic769493.png "Conexiones")

4. Haga clic en **Configuración de LAN** para abrir el diálogo **Configuración de LAN**.

5. En la sección del servidor proxy, lleve a cabo estos pasos:

   ![Servidor proxy](./media/zscaler-zscloud-tutorial/ic769494.png "Servidor proxy")

   a. Seleccione **Usar un servidor proxy para la LAN**.

   b. En el cuadro de texto Dirección, escriba **gateway.Zscaler ZSCloud.net**.

   c. En el cuadro de texto Puerto, escriba **80**.

   d. Seleccione **No usar servidor proxy para direcciones locales**.

   e. Haga clic en **Aceptar** para cerrar el diálogo **Configuración de red de área local (LAN)** .

6. Haga clic en **Aceptar** para cerrar el diálogo **Opciones de Internet**.

### <a name="create-zscaler-zscloud-test-user"></a>Creación de un usuario de prueba de Zscaler ZSCloud

En esta sección, se crea un usuario llamado Britta Simon en Zscaler ZSCloud. Zscaler ZSCloud admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario deja de existir en Zscaler ZSCloud, se crea uno después de la autenticación.

> [!Note]
> Si tiene que crear un usuario manualmente, póngase en contacto con el [equipo de soporte técnico de Zscaler ZSCloud](https://help.zscaler.com/).

### <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones.

- Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Zscaler ZSCloud, donde podrá iniciar el flujo de inicio de sesión.

- Vaya directamente a la dirección URL de inicio de sesión de Zscaler ZSCloud e inicie el flujo de inicio de sesión desde allí.

- Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Zscaler ZSCloud en Mis aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurada la aplicación Zscaler ZSCloud, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
