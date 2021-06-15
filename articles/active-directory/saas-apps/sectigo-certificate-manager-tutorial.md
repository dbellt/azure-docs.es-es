---
title: 'Tutorial: Integración de Azure Active Directory con Sectigo Certificate Manager | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Sectigo Certificate Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/04/2021
ms.author: jeedes
ms.openlocfilehash: 05070571a4c8a1b673ff145a9449c9e744718026
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891435"
---
# <a name="tutorial-azure-active-directory-integration-with-sectigo-certificate-manager"></a>Tutorial: Integración de Azure Active Directory con Sectigo Certificate Manager

En este tutorial, aprenderá a integrar Sectigo Certificate Manager en Azure Active Directory (Azure AD). Al integrar Sectigo Certificate Manager en Azure AD, puede hacer lo siguiente:

* Controle en Azure AD quién tiene acceso a Sectigo Certificate Manager.
* Permita que los usuarios inicien sesión automáticamente en Sectigo Certificate Manager con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la integración de Azure AD con Sectigo Certificate Manager, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción de Azure AD, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una cuenta de Sectigo Certificate Manager.

> [!NOTE]
> Sectigo ejecuta varias instancias de Sectigo Certificate Manager. La instancia principal de Sectigo Certificate Manager es **https:\//cert-manager.com** y esta es la dirección URL que se usa en este tutorial.  Si su cuenta se encuentra en otra instancia, debe ajustar las direcciones URL en consecuencia.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba e integrar Sectigo Certificate Manager con Azure AD.

Sectigo Certificate Manager admite las características siguientes:

* **Inicio de sesión único iniciado por SP**.
* **Inicio de sesión único iniciado por IDP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-sectigo-certificate-manager-in-the-azure-portal"></a>Adición de Sectigo Certificate Manager en Azure Portal

Para configurar la integración de Sectigo Certificate Manager en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Sectigo Certificate Manager** en el cuadro de búsqueda.
1. Seleccione **Sectigo Certificate Manager** en el panel de resultados y, a continuación, agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-sectigo-certificate-manager"></a>Configuración y prueba del inicio de sesión único de Azure AD para Sectigo Certificate Manager

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con Sectigo Certificate Manager mediante un usuario de prueba llamado **B. Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Sectigo Certificate Manager.

Para configurar y probar el inicio de sesión único de Azure AD con Sectigo Certificate Manager, realice los siguientes pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único de Sectigo Certificate Manager](#configure-sectigo-certificate-manager-sso)** : para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Sectigo Certificate Manager](#create-sectigo-certificate-manager-test-user)** : para tener un homólogo de B. Simon en Sectigo Certificate Manager que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Sectigo Certificate Manager**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, siga estos pasos:

    1. En el cuadro de texto **Identificador (id. de entidad)** , para la instancia principal de Sectigo Certificate Manager, escriba **https:\//cert-manager.com/shibboleth**.

    1. En el cuadro de texto **Dirección URL de respuesta**, para la instancia principal de Sectigo Certificate Manager, escriba **https:\//cert-manager.com/Shibboleth.sso/SAML2/POST**.
        
    > [!NOTE]
    > Aunque en general la **dirección URL de inicio de sesión** es obligatoria para el **modo Iniciado por SP**, no es necesario para iniciar sesión en Sectigo Certificate Manager.        

1. Opcionalmente, en la sección **Configuración básica de SAML**, para configurar el **modo iniciado por IDP** y para que la opción **Probar** funcione, siga estos pasos:

    1. Seleccione **Establecer direcciones URL adicionales**.

    1. En el cuadro de texto **Estado de la retransmisión**, escriba su dirección URL específica de cliente de Sectigo Certificate Manager. Para la instancia principal de Sectigo Certificate Manager, escriba **https:\//cert-manager.com/customer/\<customerURI\>/idp**.

1. En la sección **Atributos y notificaciones de usuario**, siga estos pasos:

    1. Elimine todas las **notificaciones adicionales**.
    
    1. Seleccione **Agregar nueva notificación** y agregue las cuatro notificaciones siguientes:
    
        | Nombre | Espacio de nombres | Source | Atributo de origen | Descripción |
        | --- | --- | --- | --- | --- |
        | eduPersonPrincipalName | empty | Atributo | user.userprincipalname | Debe coincidir con el campo **IdP Person ID** de Sectigo Certificate Manager para administradores. |
        | mail | empty | Atributo | user.mail | Requerido |
        | givenName | empty | Atributo | user.givenname | Opcional |
        | sn | empty | Atributo | user.surname | Opcional |

       ![Sectigo Certificate Manager: agregar cuatro nuevas notificaciones](media/sectigo-certificate-manager-tutorial/additional-claims.png)

1. En la sección **Certificado de firma de SAML**, al lado de **XML de metadatos de federación**, seleccione **Descargar**. Guarde el archivo XML en el equipo.

    ![Opción de descarga del XML de metadatos de federación](common/metadataxml.png)

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

En esta sección, se habilitará a B. Simon para que use el inicio de sesión único de Azure concediéndole acceso a Sectigo Certificate Manager.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Sectigo Certificate Manager**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-sectigo-certificate-manager-sso"></a>Configuración del inicio de sesión único en Sectigo Certificate Manager

Para configurar el inicio de sesión único en Sectigo Certificate Manager, envíe el archivo XML de metadatos de federación descargado al [equipo de soporte técnico de Sectigo Certificate Manager](https://sectigo.com/support). El equipo de soporte técnico de Sectigo Certificate Manager usa la información que le envíe para asegurarse de que la conexión de inicio de sesión único de SAML está configurada correctamente en ambos lados.

### <a name="create-sectigo-certificate-manager-test-user"></a>Creación de un usuario de prueba de Sectigo Certificate Manager

En esta sección, se crea un usuario llamado Britta Simon en Sectigo Certificate Manager. Trabaje con el [equipo de soporte técnico de Sectigo Certificate Manager](https://sectigo.com/support) para agregar a los usuarios en la plataforma de Sectigo Certificate Manager. Los usuarios se tienen que crear y activar antes de usar el inicio de sesión único.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD.

#### <a name="test-from-sectigo-certificate-manager-sp-initiated-single-sign-on"></a>Prueba desde Sectigo Certificate Manager (inicio de sesión único iniciado por SP)

Vaya a la dirección URL específica de cliente para la instancia principal de Sectigo Certificate Manager, https:\//cert-manager.com/customer/\<customerURI\>/ y seleccione el botón que hay debajo de **Or Sign In With** (O bien, inicie sesión con).  Si se ha configurado correctamente, iniciará sesión automáticamente en Sectigo Certificate Manager.

#### <a name="test-from-azure-single-sign-on-configuration-idp-initiated-single-sign-on"></a>Prueba desde la configuración de inicio de sesión único de Azure (inicio de sesión único iniciado por IDP)

En el panel de integración de la aplicación **Sectigo Certificate Manager**, seleccione **Inicio de sesión único** y seleccione el botón **Probar**.  Si se ha configurado correctamente, iniciará sesión automáticamente en Sectigo Certificate Manager.

#### <a name="test-by-using-the-my-apps-portal-idp-initiated-single-sign-on"></a>Prueba mediante el portal Aplicaciones (inicio de sesión único iniciado por IDP)

Seleccione **Sectigo Certificate Manager** en el portal Aplicaciones.  Si se ha configurado correctamente, iniciará sesión automáticamente en Sectigo Certificate Manager. Para más información acerca del portal Aplicaciones, consulte [Acceso y uso de aplicaciones en el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado Sectigo Certificate Manager, puede aplicar el control de sesión, que protege su organización, en tiempo real, frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).