---
title: 'Tutorial: Integración de Azure Active Directory con 10,000ft Plans | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y 10,000ft Plans.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/03/2021
ms.author: jeedes
ms.openlocfilehash: fda0f8cfccef8d6928ca33a7e1932a7371f4d4cf
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111956590"
---
# <a name="tutorial-azure-active-directory-integration-with-10000ft-plans"></a>Tutorial: integración de Azure Active Directory con 10,000ft Plans

En este tutorial, aprenderá a integrar 10,000ft Plans con Azure Active Directory (Azure AD). Al integrar 10,000ft Plans con Azure AD, puede hacer lo siguiente:

* En Azure AD, puede controlar quién tiene acceso a 10,000ft Plans.
* Permitir que los usuarios inicien sesión automáticamente en 10,000ft Plans con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con 10,000ft Plans, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en 10,000ft Plans.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* 10,000ft Plans admite el inicio de sesión único iniciado por **SP**.
* 10,000ft Plans admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="add-10000ft-plans-from-the-gallery"></a>Incorporación de 10,000ft Plans desde la galería

Para configurar la integración de 10,000ft Plans en Azure AD, será preciso que agregue 10,000ft Plans desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **10,000ft Plans** en el cuadro de búsqueda.
1. Seleccione **10,000ft Plans** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-10000ft-plans"></a>Configuración y prueba del SSO de Azure AD para 10,000ft Plans

Configure y pruebe el inicio de sesión único de Azure AD con 10,000ft Plans mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de 10,000ft Plans.

Para configurar y probar el inicio de sesión único de Azure AD con 10,000ft Plans, complete los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en 10,000ft Plans](#configure-10000ft-plans-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de 10,000ft Plans](#create-10000ft-plans-test-user)** : para tener un homólogo de B.Simon en 10,000ft Plans que esté vinculado a su representación en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **10,000ft Plans**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, siga estos pasos:

    a. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://app.10000ft.com`

    b. En el cuadro de texto **Identificador (id. de entidad)** , escriba la dirección URL: `https://app.10000ft.com/saml/metadata`.

    > [!NOTE]
    > El valor de **Identificador** es distinto si tiene un dominio personalizado. Para obtener este valor, póngase en contacto con el [equipo de soporte técnico de 10,000ft Plans](https://www.10000ft.com/plans/support). También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **certificado (sin procesar)** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/certificateraw.png)

6. En la sección **Set up 10,000ft Plans** (Configurar 10,000ft Plans), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, habilitará a B.Simon para que use el inicio de sesión único de Azure concediéndole acceso a 10,000ft Plans.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **10,000ft Plans**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-10000ft-plans-sso"></a>Configuración del inicio de sesión único de 10,000ft Plans

Para configurar el inicio de sesión único en **10,000ft Plans**, es preciso enviar el **certificado (sin procesar)** descargado y las direcciones URL apropiadas copiadas de Azure Portal al [equipo de soporte técnico de 10,000ft Plans](https://www.10000ft.com/plans/support). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-10000ft-plans-test-user"></a>Creación de un usuario de prueba de 10,000ft Plans

En esta sección, se crea un usuario llamado Britta Simon en 10,000ft Plans. 10,000ft Plans admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario aún no existe en 10,000ft Plans, se crea uno nuevo después de la autenticación.

> [!NOTE]
> Si necesita crear manualmente un usuario, es preciso que se ponga contacto con el [equipo de soporte técnico de 10,000ft Plans](https://www.10000ft.com/plans/support).

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esto le redirigirá a la dirección URL de inicio de sesión de 10,000ft Plans, donde puede iniciar el flujo de inicio de sesión. 

* Vaya directamente a la dirección URL de inicio de sesión de 10,000ft Plans e inicie el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Cuando haga clic en el icono de 10 000ft Plans en Aplicaciones, se le redirigirá a la URL de inicio de sesión de 10 000ft Plans. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado 10,000ft Plans, puede aplicar el control de sesión, que protege contra la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).