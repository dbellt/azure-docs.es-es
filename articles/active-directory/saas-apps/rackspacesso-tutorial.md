---
title: 'Tutorial: Integración de Azure Active Directory con Rackspace SSO | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Rackspace SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/14/2021
ms.author: jeedes
ms.openlocfilehash: 5c0019d53e1f011d5dfd9bf687f938101284ffee
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075329"
---
# <a name="tutorial-azure-active-directory-integration-with-rackspace-sso"></a>Tutorial: Integración de Azure Active Directory con Rackspace SSO

En este tutorial aprenderá a integrar Rackspace SSO con Azure Active Directory (Azure AD). Al integrar Rackspace SSO con Azure AD, puede hacer lo siguiente:

* Controlar en Azure AD quién tiene acceso a Rackspace SSO.
* Permitir que los usuarios puedan iniciar sesión automáticamente en Rackspace SSO con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

## <a name="prerequisites"></a>Requisitos previos

Para configurar la integración de Azure AD con Rackspace SSO, necesita lo siguiente:

* Una suscripción de Azure AD. Si no dispone de un entorno de Azure AD, puede obtener [una cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para SSO en Rackspace SSO.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, puede configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Rackspace SSO admite el SSO iniciado por **SP**.

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="add-rackspace-sso-from-the-gallery"></a>Incorporación de Rackspace SSO desde la galería

Para configurar la integración de Rackspace SSO en Azure AD, tiene que agregar Rackspace SSO desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en Azure Portal con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Rackspace SSO** en el cuadro de búsqueda.
1. Seleccione **Rackspace SSO** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-sso-for-rackspace-sso"></a>Configuración y prueba del SSO de Azure AD para Rackspace SSO

En esta sección, configurará y probará el inicio de sesión único de Azure AD con Rackspace SSO con un usuario de prueba llamado **Britta Simon**.
Si se usa el inicio de sesión único con Rackspace, los usuarios de este se crearán automáticamente la primera vez que inicien sesión en el portal de Rackspace. 

Para configurar y probar el SSO de Azure AD con Rackspace SSO, siga estos pasos:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con Britta Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para permitir que Britta Simon use el inicio de sesión único de Azure AD.
2. **[Configuración del inicio de sesión único de Rackspace SSO](#configure-rackspace-sso-single-sign-on)**: para configurar los valores de Inicio de sesión único en la aplicación.
    1. **[Configuración de la asignación de atributos en el panel de control de Rackspace](#set-up-attribute-mapping-in-the-rackspace-control-panel)**: para asignar roles de Rackspace a usuarios de Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En Azure Portal, en la página de integración de aplicaciones de **Rackspace SSO**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configuración del inicio de sesión único con SAML**, haga clic en el icono de lápiz de **Configuración básica de SAML** para editar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

4. En la sección **Configuración básica de SAML**, cargue el **archivo de metadatos del proveedor de servicios**, que se puede descargar de la [dirección URL](https://login.rackspace.com/federate/sp.xml), y siga estos pasos:

    a. Haga clic en **Cargar el archivo de metadatos**.

    ![Captura de pantalla que muestra la sección Configuración básica de SAML con el vínculo Cargar el archivo de metadatos.](common/upload-metadata.png)

    b. Haga clic en el **logotipo de la carpeta** para seleccionar el archivo de metadatos y luego en **Cargar**.

    ![Captura de pantalla que muestra un cuadro de diálogo en el que puede seleccionar y cargar un archivo.](common/browse-upload-metadata.png)

    c. Una vez que se ha cargado correctamente el archivo de metadatos, las direcciones URL necesarias se rellenan automáticamente.

    d. En el cuadro de texto **Dirección URL de inicio de sesión**, escriba la dirección URL: `https://login.rackspace.com/federate/`

5. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en **Descargar** para descargar el **XML de metadatos de federación** de las opciones proporcionadas según sus requisitos y guárdelo en el equipo.

    ![Vínculo de descarga del certificado](common/metadataxml.png)

Este archivo se cargará en Rackspace para llenar los valores de configuración de Identity Federation requeridos.

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

En esta sección va a permitir que B.Simon acceda a Rackspace SSO mediante el SSO de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Rackspace SSO**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.
1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.
1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que se asigne un rol a los usuarios, puede seleccionarlo en la lista desplegable **Seleccionar un rol**. Si no se ha configurado ningún rol para esta aplicación, verá seleccionado el rol "Acceso predeterminado".
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-rackspace-sso-single-sign-on"></a>Configuración del inicio de sesión único de Rackspace SSO

Para configurar el inicio de sesión único en **Rackspace SSO**:

1. Consulte la documentación de [Add an Identity Provider to the Control Panel](https://developer.rackspace.com/docs/rackspace-federation/gettingstarted/add-idp-cp/) (Agregar un proveedor de identidades al panel de control)
1. En ella se proporcionan los pasos necesarios para:
    1. Crear un proveedor de identidades
    1. Especificar un dominio de correo electrónico que los usuarios utilizarán para identificar la compañía al iniciar sesión.
    1. Cargar el **XML de metadatos de federación** que ha descargado previamente desde el panel de control de Azure.

Así se configurarán los valores básicos de SSO necesarios para la conexión entre Azure y Rackspace.

### <a name="set-up-attribute-mapping-in-the-rackspace-control-panel"></a>Configuración de la asignación de atributos en el panel de control de Rackspace

Rackspace usa una **directiva de asignación de atributos** para asignar roles y grupos de Rackspace a los usuarios de inicio de sesión único. La **directiva de asignación de atributos** convierte las notificaciones SAML de Azure AD en los campos de la configuración de usuario que Rackspace requiere. Se puede encontrar más información en la [documentación de los conceptos básicos de la asignación de atributos](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace. Algunas consideraciones que hay que tener en cuenta:

* Si desea asignar distintos niveles de acceso a Rackspace mediante grupos de Azure AD, deberá habilitar la notificación de grupos en la configuración del inicio de sesión único de **Rackspace SSO** en Azure. La **directiva de asignación de atributos** se usará para que coincidan esos grupos con los roles y grupos de Rackspace deseados:

    ![Configuración de la notificación de grupos](common/sso-groups-claim.png)

* De forma predeterminada, Azure AD envía el identificador de usuario a los grupos de Azure AD en la notificación SAML, en lugar del nombre del grupo. Sin embargo, si se va a sincronizar la instancia local de Active Directory con Azure AD, tiene la opción de enviar los nombres reales de los grupos:

    ![Configuración del nombre de la notificación de grupos](common/sso-groups-claims-names.png)

En el siguiente ejemplo de **directiva de asignación de atributo** se muestra:
1. El establecimiento del nombre de usuario de Rackspace en la notificación SAML `user.name`. Se puede usar cualquier notificación, pero es más común seleccionar un campo que contenga la dirección de correo electrónico del usuario.
1. El establecimiento de los roles `admin` y `billing:admin` de Rackspace en un usuario, para lo que se hace coincidir un grupo de Azure AD con un nombre de grupo o un identificador de usuario de grupo. Se usa una *sustitución* de `"{0}"` en el campo `roles` y se reemplazará por los resultados de la expresiones de reglas `remote`.
1. El uso de la  *sustitución predeterminada*`"{D}"` para que Rackspace pueda recuperar campos de SAML adicionales mediante la búsqueda de notificaciones SAML estándar y conocidas en el intercambio de SAML.

```yaml
---
mapping:
    rules:
    - local:
        user:
          domain: "{D}"
          name: "{At(http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)}"
          email: "{D}"
          roles:
              - "{0}"
          expire: "{D}"
      remote:
          - path: |
              (
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='7269f9a2-aabb-9393-8e6d-282e0f945985') then ('admin', 'billing:admin') else (),
                if (mapping:get-attributes('http://schemas.microsoft.com/ws/2008/06/identity/claims/groups')='MyAzureGroup') then ('admin', 'billing:admin') else ()
              )
            multiValue: true
  version: RAX-1
```
> [!TIP]
> Asegúrese de que utiliza un editor de texto que valide la sintaxis de YAML al editar el archivo de directiva.

Para ver más ejemplos, consulte la [documentación de los documentación de los conceptos básicos de la asignación de atributos](https://developer.rackspace.com/docs/rackspace-federation/appendix/map/) de Rackspace.

## <a name="test-sso"></a>Prueba de SSO

En esta sección, probará la configuración de inicio de sesión único de Azure AD con las siguientes opciones. 

* Haga clic en **Probar esta aplicación** en Azure Portal. Esta acción le redirigirá a la dirección URL de inicio de sesión de Rackspace SSO, donde puede poner en marcha el flujo de inicio de sesión. 

* Acceda directamente a la dirección URL de inicio de sesión de Rackspace SSO y ponga en marcha el flujo de inicio de sesión desde ahí.

* Puede usar Mis aplicaciones de Microsoft. Al hacer clic en el icono de Rackspace SSO en Aplicaciones, se le redirigirá a la dirección URL de inicio de sesión de esa plataforma. Para más información acerca de Aplicaciones, consulte [Inicio de sesión e inicio de aplicaciones desde el portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

También puede usar el botón **Validate** (Validar) de la configuración del inicio de sesión único de **Rackspace SSO**:

   ![Botón Validate (Validar) de SSO](common/sso-validate-sign-on.png)

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado Rackspace SSO, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).
