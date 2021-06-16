---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Clebex | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Clebex.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/22/2021
ms.author: jeedes
ms.openlocfilehash: b0c73206d69f8902fdd9ef9306a35b3fccacc8ec
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111888337"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-clebex"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Clebex

En este tutorial, aprenderá a integrar Clebex con Azure Active Directory (Azure AD). Al integrar Clebex con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Clebex.
* Permitir que los usuarios inicien sesión automáticamente en Clebex con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Clebex.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Clebex admite el inicio de sesión único iniciado por **SP**.

* Clebex admite el aprovisionamiento de usuarios **Just-In-Time**.


## <a name="adding-clebex-from-the-gallery"></a>Incorporación de Clebex desde la galería

Para configurar la integración de Clebex en Azure AD, tendrá que agregar Clebex desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Clebex** en el cuadro de búsqueda.
1. Seleccione **Clebex** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-clebex"></a>Configuración y prueba del SSO de Azure AD para Clebex

Configure y pruebe el inicio de sesión único de Azure AD con Clebex mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Clebex.

Para configurar y probar el inicio de sesión único de Azure AD con Clebex, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Clebex](#configure-clebex-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Clebex](#create-clebex-test-user)** : para tener un homólogo de B.Simon en Clebex que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Clebex**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una dirección URL con el siguiente patrón: `https://<CustomerName>.domain.extention/<ID>`

    b. En el cuadro de texto **URL de respuesta**, escriba una dirección URL con el siguiente patrón: `https://<CustomerName>.domain.extention/<ID>`

    c. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<CustomerName>.domain.extention/<ID>`

    > [!NOTE]
    > Estos valores no son reales. Actualice estos valores con los valores reales de Identificador, URL de respuesta y URL de inicio de sesión. Póngase en contacto con el [equipo de soporte técnico al cliente de Clebex](mailto:support@clebex.net) para obtener estos valores. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Clebex**, copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Clebex mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Clebex**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-clebex-sso"></a>Configuración del inicio de sesión único de Clebex

1. Inicie sesión en el sitio web de Clebex como administrador.

1. Vaya a COMPANY ADMIN -> **Connectors** -> **Single Sign On (SSO)** (Administrador de empresa > Conectores > Inicio de sesión único [SSO]) y haga clic en **select** (Seleccionar).

    ![Captura de pantalla para seleccionar el tipo de conector.](./media/clebex-tutorial/single-sign-on.png)

1. En CONNECTORS (Conectores), seleccione **Azure SSO** y realice los pasos siguientes en la sección EDIT-CONNECTOR (Editar conector).

    ![Captura de pantalla para seleccionar conectores y la configuración.](./media/clebex-tutorial/azure-sso.png)

    a. Copie el valor de **IDENTIFIER** (Identificador) y péguelo en el cuadro de texto **Identificador** de la sección **Configuración básica de SAML** en Azure Portal.

    b. Copie el valor de **REPLY URL** (Dirección URL de respuesta) y péguelo en el cuadro de texto **Dirección URL de respuesta**, en la sección **Configuración básica de SAML** de Azure Portal.

    c. En el cuadro de texto **ENTITY ID** (Identificador de entidad), pegue el valor de **Identificador de Azure AD** que copió de Azure Portal.

    d. En el cuadro de texto **SAML**, pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    e. Abra en el Bloc de notas el archivo del **certificado (Base 64)** que descargó de Azure Portal, copie el contenido y, luego, péguelo en el cuadro de texto **CERTIFICATE** (Certificado).

    f. Haga clic en **SAVE-CHANGES** (Guardar cambios).

### <a name="create-clebex-test-user"></a>Creación de un usuario de prueba de Clebex

En esta sección, se creará el usuario B.Simon en Clebex. Clebex admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de manera predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe aún en Clebex, se crea uno nuevo después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Clebex, desde donde puede comenzar el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Clebex y ponga en marcha el flujo de inicio de sesión desde allí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Clebex en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de Clebex. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Clebex puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).