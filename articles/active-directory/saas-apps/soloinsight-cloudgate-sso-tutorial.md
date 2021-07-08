---
title: 'Tutorial: Integración de Azure Active Directory con Soloinsight CloudGate SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Soloinsight CloudGate SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/24/2021
ms.author: jeedes
ms.openlocfilehash: 7c52a14ce1115be6efcaf99a29f9ee7ca1836c71
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110452714"
---
# <a name="tutorial-integrate-soloinsight-cloudgate-sso-with-azure-active-directory"></a>Tutorial: Integración de Soloinsight-CloudGate SSO con Azure Active Directory

En este tutorial, aprenderá a integrar Soloinsight-CloudGate SSO con Azure Active Directory (Azure AD). Al integrar Soloinsight-CloudGate SSO con Azure AD, puede:

* Controlar desde Azure AD quién tiene acceso a Soloinsight-CloudGate SSO.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Soloinsight-CloudGate SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único en Soloinsight-CloudGate SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Soloinsight-CloudGate SSO permite utilizar el inicio de sesión único iniciado por **SP**.

## <a name="add-soloinsight-cloudgate-sso-from-the-gallery"></a>Incorporación de Soloinsight-CloudGate SSO desde la galería

Para configurar la integración de Soloinsight-CloudGate SSO en Azure AD, tiene que agregar Soloinsight-CloudGate SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En el cuadro de búsqueda de la sección **Agregar desde la galería**, escriba **Soloinsight-CloudGate SSO**.
1. Seleccione **Soloinsight-CloudGate SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-soloinsight-cloudgate-sso"></a>Configuración y prueba de inicio de sesión único de Azure AD para Soloinsight-CloudGate SSO

Configure y pruebe el inicio de sesión único de Azure AD con Soloinsight-CloudGate SSO mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vinculación entre un usuario de Azure AD y el usuario relacionado de Soloinsight-CloudGate SSO.

Para configurar el inicio de sesión único de Azure AD con Soloinsight-CloudGate SSO, realice los pasos siguientes:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Soloinsight-CloudGate SSO](#configure-soloinsight-cloudgate-sso)** : para configurar el inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Soloinsight-CloudGate SSO](#create-soloinsight-cloudgate-sso-test-user)** : para tener un homólogo de B.Simon en Soloinsight-CloudGate SSO que esté vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

### <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de la aplicación **Soloinsight-CloudGate SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, seleccione **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la página **Configuración básica de SAML**, especifique los valores de los siguientes campos:

    1. En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.sigateway.com/login`

    1. En el cuadro de texto **Identificador (id. de entidad)** , escriba una dirección URL con el siguiente patrón: `https://<SUBDOMAIN>.sigateway.com/process/sso`

   > [!NOTE]
   > Estos valores no son reales. Actualice estos valores con la dirección URL de inicio de sesión real y el identificador que se explica más adelante en la sección **Configuración del inicio de sesión único de Soloinsight CloudGate SSO** del tutorial.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

   ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Soloinsight-CloudGate SSO** (Configurar Soloinsight-CloudGate SSO), copie las direcciones URL adecuadas según sus necesidades.

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

En esta sección, va a permitir que B.Simon acceda a Soloinsight-CloudGate SSO mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Soloinsight CloudGate SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-soloinsight-cloudgate-sso"></a>Configuración de Soloinsight-CloudGate SSO

1. Para automatizar la configuración de Soloinsight-CloudGate SSO, debe instalar la **extensión de inicio de sesión seguro de Mis aplicaciones** en el explorador. Para ello, haga clic en **Instalar la extensión**.

    ![Extensión Mis aplicaciones](common/install-myappssecure-extension.png)

2. Cuando haya agregado la extensión al explorador, haga clic en **Setup Soloinsight-CloudGate SSO** (Configurar Soloinsight-CloudGate SSO) y accederá automáticamente a la aplicación Soloinsight-CloudGate SSO. Una vez allí, escriba las credenciales de administrador para iniciar sesión en Soloinsight-CloudGate SSO. La extensión de explorador configurará automáticamente la aplicación y automatizará los pasos 3 a 8.

    ![Configuración](common/setup-sso.png)

3. Si quiere configurar Soloinsight-CloudGate SSO manualmente, abra una nueva ventana del explorador web, inicie sesión como administrador en el sitio de empresa de Soloinsight-CloudGate SSO y siga estos pasos:

4. Para obtener los valores que se van a pegar en Azure Portal al realizar la configuración básica de SAML, inicie sesión en el portal web de CloudGate con sus credenciales y acceda a la configuración de inicio de sesión único, que puede encontrarse en la siguiente ruta de acceso **Home > Administration > System settings > General** (Inicio > Administración > Configuración del sistema > General).

    ![Configuración del inicio de sesión único en CloudGate](./media/soloinsight-cloudgate-sso-tutorial/main-settings.png)

5. **Dirección URL del consumidor de SAML**

    * Copie los vínculos disponibles en los campos **Saml Consumer URL** (Dirección URL del consumidor de SAML) y **Redirect URL** (Dirección URL de redirección) y péguelos en la sección **Configuración básica de SAML** de Azure Portal, en los campos **Identificador (id. de entidad)** y **Dirección URL de respuesta**, respectivamente.

        ![SAMLIdentifier](./media/soloinsight-cloudgate-sso-tutorial/identifier.png)

6. **Certificado de firma de SAML**

    * Vaya al origen del archivo de certificado (Base64) que descargó de las listas de certificados de firma de SAML de Azure Portal y haga clic en él con el botón derecho. Elija la opción **Edit with Notepad++** (Editar con Notepad++) de la lista. 

        ![SAMLcertificate](./media/soloinsight-cloudgate-sso-tutorial/certificate-file.png)

    * Copie el contenido del archivo de certificado (Base64) de Notepad ++.

        ![Copia del certificado](./media/soloinsight-cloudgate-sso-tutorial/certificate-copy.png)

    * Pegue el contenido del campo **Certificate** (Certificado) de la configuración de inicio de sesión único del portal web de CloudGate y haga clic en el botón Save (Guardar).

        ![Portal de certificados](./media/soloinsight-cloudgate-sso-tutorial/certificate-portal.png)

7. **Grupo predeterminado**

    * Seleccione **Business Admin** (Administrador de empresa) en la lista desplegable de la opción **Default Group** (Grupo predeterminado) del portal web de CloudGate.

        ![Grupo predeterminado](./media/soloinsight-cloudgate-sso-tutorial/default-group.png)

8. **Dirección URL de inicio de sesión e identificador de AD**

    * El valor de **Dirección URL de inicio de sesión** que copió de **Set up Soloinsight-CloudGate SSO** (Configurar Soloinsight-CloudGate SSO) en Azure Portal debe especificarse en la sección de configuración de inicio de sesión único del portal web de CloudGate.

    * Pegue el vínculo de **Dirección URL de inicio de sesión** de Azure Portal en el campo **AD Login URL** (Dirección URL de inicio de sesión de AD) del portal web de CloudGate.

    * Pegue el vínculo de **Identificador de Azure AD** de Azure Portal en el campo **AD Identifier** (Identificador de AD) del portal web de CloudGate.

        ![Inicio de sesión en AD](./media/soloinsight-cloudgate-sso-tutorial/ad-login.png)

### <a name="create-soloinsight-cloudgate-sso-test-user"></a>Creación de un usuario de prueba de Soloinsight CloudGate SSO

Para crear un usuario de prueba, seleccione **Employees** (Empleados) en el menú principal del portal web de CloudGate y rellene el formulario Add New employee (Agregar nuevo empleado). El nivel de autoridad que se va a asignar al usuario de prueba es **Business Admin** (Administrador de empresa). Haga clic en **Create** (Crear) cuando haya rellenado todos los campos obligatorios.

![Prueba de empleado](./media/soloinsight-cloudgate-sso-tutorial/employee-test.png)

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la URL de inicio de sesión de Soloinsight-CloudGate SSO, donde puede poner en marcha el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Soloinsight-CloudGate SSO y ponga en marcha el flujo de inicio de sesión desde ahí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Soloinsight-CloudGate SSO en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de dicha aplicación. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Pasos siguientes

Una vez haya configurado Soloinsight-CloudGate SSO, puede aplicar el control de sesión, que protege a la organización en tiempo real frente a la filtración e infiltración de información confidencial. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
