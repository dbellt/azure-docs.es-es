---
title: 'Tutorial: Integración de Azure Active Directory con Miro | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Miro.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/26/2021
ms.author: jeedes
ms.openlocfilehash: 6066dc8709fe376f99babcc3dd1360e4bf571ef2
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110655350"
---
# <a name="tutorial-integrate-miro-with-azure-active-directory"></a>Tutorial: Integración de Miro con Azure Active Directory

En este tutorial, obtendrá información sobre cómo integrar Miro con Azure Active Directory (Azure AD). En help.miro.com se puede encontrar otra versión de este tutorial. Al integrar Miro con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Miro.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Miro con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Suscripción habilitada para el inicio de sesión único (SSO) en Miro.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba. 
* Miro admite el inicio de sesión único iniciado por **SP e IDP**, así como el aprovisionamiento de usuarios **Just In Time**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-miro-from-the-gallery"></a>Adición de Miro desde la galería

Para configurar la integración de Miro en Azure AD, es preciso agregar dicha solución desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Miro** en el cuadro de búsqueda.
1. Seleccione **Miro** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-miro"></a>Configuración y prueba del inicio de sesión único de Azure AD para Miro

Configure y pruebe el inicio de sesión único de Azure AD con Miro utilizando un usuario de prueba llamado **B. Simon**. Para que el SSO funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Miro.

Para configurar y probar el inicio de sesión único de Azure AD con Miro, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Miro](#configure-miro-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba de Miro](#create-miro-test-user)** : para tener un homólogo de B.Simon en Miro que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de la aplicación **Miro**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, si desea configurar la aplicación en modo iniciado por **IDP**, realice el siguiente paso:

    En el cuadro de texto **Identificador**, escriba la dirección URL `https://miro.com/`

5. Si desea configurar la aplicación en modo iniciado por **SP** en el cuadro de texto **URL de inicio de sesión**, escriba la dirección URL: `https://miro.com/sso/login/`

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo. Lo necesitará para configurar el inicio de sesión único en Miro.

   ![Vínculo de descarga del certificado](common/certificatebase64.png "Vínculo de descarga del certificado")

1. En la sección **Configurar Miro**, copie la dirección URL de inicio de sesión. Lo necesitará para configurar el inicio de sesión único en Miro.

   ![Copiar dirección URL de inicio de sesión](./media/miro-tutorial/login.png "Copiar dirección URL de inicio de sesión")

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

En esta sección, va a permitir que B. Simon acceda a Miro utilizando el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Miro**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

* Otra posibilidad es ir a **Propiedades** de la aplicación y desactivar **Asignación de usuarios necesaria**
![Deshabilitar requisito de asignación](./media/miro-tutorial/properties.png "Deshabilitar requisito de asignación")

## <a name="configure-miro-sso"></a>Configuración del inicio de sesión único de Miro

Para configurar el inicio de sesión único en Miro, use el certificado que descargó anteriormente y la dirección URL de inicio de sesión que copió anteriormente. En la configuración de la cuenta de Miro, vaya a la sección **Seguridad** active **Habilitar SSO/SAML**. 

1. Pegue la dirección URL de inicio de sesión en el campo **SAML Sign-in URL** (Dirección URL de inicio de sesión de SAML).
1. Abra el archivo de certificado con un editor de texto y copie la secuencia de certificados. Pegue la secuencia en el campo **Key x509 Certificate (Certificado x509 clave)** .
![Configuración de Miro](./media/miro-tutorial/security.png "Configuración de Miro")

1. En el campo **Domains** (Dominios), escriba la dirección de dominio, haga clic en **Add** (Agregar) y siga el procedimiento de comprobación. Repita la operación en las restantes direcciones de dominio si tiene alguna. La característica de inicio de sesión único de Miro funcionará para los usuarios finales que los dominios están en la lista. 
![Dominio](./media/miro-tutorial/add-domain.png "Dominio")

1. Decida si va a usar el aprovisionamiento Just-In-Time (extracción de los usuarios en la suscripción durante su registro en Miro) y haga clic en **Save** (Guardar) para completar la configuración del inicio de sesión único en Miro.
![Aprovisionamiento Just-In-Time](./media/miro-tutorial/save-configuration.png "Aprovisionamiento Just-In-Time") 

### <a name="create-miro-test-user"></a>Creación de un usuario de prueba de Miro

En esta sección, se crea un usuario llamado B. Simon en Miro. Miro admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si un usuario no existe en Miro, se crea otro después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones mediante el usuario de prueba B. Simon. 

#### <a name="sp-initiated"></a>Iniciado por SP:

* Acceda directamente a la dirección URL de inicio de sesión de Miro e inicie el flujo de inicio de sesión desde allí.

#### <a name="idp-initiated"></a>Iniciado por IDP:

* Haga clic en **Probar esta aplicación** en Azure Portal y elija iniciar sesión como B. Simon. Debería iniciarse sesión automáticamente en la suscripción de Miro para la que configuró el inicio de sesión único. 

También puede usar Aplicaciones de Microsoft para probar la aplicación en cualquier modo. Al hacer clic en el icono de Miro en Aplicaciones, si se ha configurado en modo SP, se le redirigirá a la página de inicio de sesión de la aplicación para comenzar el flujo de inicio de sesión; y si se ha configurado en modo IDP, se debería iniciar sesión automáticamente en la instancia de Miro para la que ha configurado el inicio de sesión único. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).
