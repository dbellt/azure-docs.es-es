---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Rescana | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Rescana.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/29/2021
ms.author: jeedes
ms.openlocfilehash: 3a91a6bf2380d0a50a8a7992fa8175ed6caca2dc
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776208"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-rescana"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Rescana

En este tutorial aprenderá a integrar Rescana con Azure Active Directory (Azure AD). Si integra Rescana con Azure AD, podrá hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Rescana.
* Permitir que los usuarios inicien sesión automáticamente en Rescana con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción que permita el inicio de sesión único (SSO) en Rescana.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Rescana permite utilizar el inicio de sesión único con **SP e IDP**.
* Rescana admite el aprovisionamiento de usuarios **Just-In-Time**.

## <a name="adding-rescana-from-the-gallery"></a>Incorporación de Rescana desde la galería

Para poder configurar la integración de Rescana en Azure AD, debe agregar esta aplicación a la lista de aplicaciones SaaS administradas desde la galería.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Rescana** en el cuadro de búsqueda.
1. Seleccione **Rescana** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-sso-for-rescana"></a>Configuración y comprobación del inicio de sesión único de Azure AD con Rescana

Configure y pruebe el inicio de sesión único de Azure AD con Rescana utilizando un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es preciso vincular un usuario de Azure AD y el usuario de Rescana correspondiente.

Para configurar y probar el inicio de sesión único de Azure AD con Rescana, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Rescana](#configure-rescana-sso)** : para configurar el inicio de sesión único en lado de la aplicación.
    1. **[Creación de un usuario de prueba de Rescana](#create-rescana-test-user)** : para tener un homólogo de B.Simon en Rescana que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Rescana**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, escriba los valores de los siguientes campos:

    a. En el cuadro de texto **Identificador**, escriba una de las siguientes direcciones URL:

    | Identificador |
    |-------------|
    | `https://pl.rescana.com/saml/metadata.xml` |
    | `https://portal.rescana.com/saml/metadata.xml` |
    | `https://qa.rescana.com/saml/metadata.xml` |
    |

    b. En el cuadro de texto **URL de respuesta**, escriba una de las siguientes direcciones URL:

    | URL de respuesta |
    |-------------|
    | `https://pl.rescana.com/authorization-code/callback` |
    | `https://portal.rescana.com/authorization-code/callback` |
    | `https://qa.rescana.com/authorization-code/callback` |
    |


1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    a. En el cuadro de texto **URL de inicio de sesión**, escriba una de las siguientes direcciones URL:
    
    | URL de inicio de sesión |
    |---------------|
    | `https://pl.rescana.com/authorization-code/callback` |
    | `https://portal.rescana.com/authorization-code/callback` |
    | `https://qa.rescana.com/authorization-code/callback` |
    |

    b. En el cuadro de texto **Estado de la retransmisión**, escriba un valor con el siguiente patrón:`<INSTANCE_ID>`

    > [!NOTE]
    > Este valor no es real. Sustituya este valor por el estado de retransmisión real. Póngase en contacto con el [servicio de soporte técnico de Rescana](mailto:ops@rescana.com) para obtener el valor. También puede hacer referencia a los patrones que se muestran en la sección **Configuración básica de SAML** de Azure Portal.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Configurar Rescana**, copie las direcciones URL apropiadas conforme a sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Rescana utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Rescana**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-rescana-sso"></a>Configuración del inicio de sesión único en Rescana

Para configurar el inicio de sesión único en **Rescana**, es preciso enviar el **certificado (Base 64)** descargado y las direcciones URL correspondientes que haya copiado de Azure Portal al [servicio de soporte técnico de Rescana](mailto:ops@rescana.com). Dicho equipo lo configura para establecer la conexión de SSO de SAML correctamente en ambos lados.

### <a name="create-rescana-test-user"></a>Creación de un usuario de prueba de Rescana

En esta sección, va a crear un usuario llamado B.Simon en Rescana. Rescana admite el aprovisionamiento Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si el usuario no existe en Rescana, se creará cuando intente acceder a esta aplicación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Haga clic en **Probar esta aplicación** en Azure Portal. De este modo, accederá automáticamente a la dirección URL de inicio de sesión de Rescana, donde podrá comenzar el flujo de inicio de sesión.  

* Acceda directamente a la URL de inicio de sesión de Rescana y ponga en marcha el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* En Azure Portal, haga clic en **Probar esta aplicación**. Al hacerlo, debería iniciar sesión automáticamente en la instancia de Rescana en la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Cuando en Aplicaciones haga clic en el icono de Rescana, si seleccionó el modo SP en la configuración, debería acceder automáticamente a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión. Por el contrario, si seleccionó el modo IDP en la configuración, debería iniciar sesión en la instancia de Rescana en la que configuró el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Rescana, podrá aplicar el control de sesión, que protege la información confidencial de la organización en tiempo real de posibles filtraciones. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


