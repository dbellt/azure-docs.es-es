---
title: 'Tutorial: Creación de flujos de usuario y directivas personalizadas (Azure Active Directory B2C)'
description: Siga este tutorial para aprender a crear flujos de usuario y directivas personalizadas en Azure Portal para habilitar el registro, el inicio de sesión y la edición de perfiles de usuario para las aplicaciones en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 04/26/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: fa1250189c5ba11426ac993bed8fde398ae37ea2
ms.sourcegitcommit: 12f15775e64e7a10a5daebcc52154370f3e6fa0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108001473"
---
# <a name="tutorial-create-user-flows-in-azure-active-directory-b2c"></a>Tutorial: Creación de flujos de usuario en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

En las aplicaciones, puede que tenga flujos de usuario que permitan a los usuarios registrarse, iniciar sesión o administrar su perfil. Puede crear varios flujos de usuario de diferentes tipos en un inquilino de Azure Active Directory B2C (Azure AD B2C) y usarlos en las aplicaciones según sea necesario. Los flujos de usuario se pueden volver a usar en todas las aplicaciones.

::: zone pivot="b2c-user-flow"
Un flujo de usuario permite determinar cómo interactúan los usuarios con su aplicación cuando realizan acciones como iniciar sesión, registrarse, editar un perfil o restablecer una contraseña. En este artículo aprenderá a:
::: zone-end

::: zone pivot="b2c-custom-policy"
Las [directivas personalizadas](custom-policy-overview.md) son archivos de configuración que definen el comportamiento del inquilino de Azure Active Directory B2C (Azure AD B2C). En este artículo aprenderá a:
::: zone-end

> [!div class="checklist"]
> * Creación de un flujo de usuario de registro e inicio de sesión
> * Habilitar el autoservicio de restablecimiento de contraseña
> * Creación de un flujo de usuario de edición de perfil

::: zone pivot="b2c-user-flow"
> [!IMPORTANT]
> Hemos cambiado la manera en que hacemos referencia a las versiones del flujo de usuario. Anteriormente, se ofrecían las versiones V1 (para entornos de producción) y V1.1 y V2 (preliminares). Ahora hemos consolidado los flujos de usuario en dos versiones: flujos de usuario **recomendados**, con las características más recientes, y flujos de usuario **estándar (heredado)** . En la nube pública, todos los flujos de usuario de las versiones preliminares heredadas (V1.1 y V2) se encuentran en un proceso de entrada en desuso que finalizará el **1 de agosto de 2021**. Para más información, consulte [Versiones de flujos de usuario de Azure Active Directory B2C](user-flow-versions.md). *Estos cambios solo se aplican a la nube pública de Azure. Otros entornos seguirán usando el [control de versiones de flujos de usuario heredado](user-flow-versions-legacy.md).* 
::: zone-end

## <a name="prerequisites"></a>Requisitos previos

::: zone pivot="b2c-user-flow"
- Si todavía no tiene uno, [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
- [Registre la aplicación](tutorial-register-applications.md) en el inquilino que ha creado para que pueda comunicarse con Azure AD B2C.
::: zone-end

::: zone pivot="b2c-custom-policy"

- Si todavía no tiene uno, [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
- [Registre la aplicación](tutorial-register-applications.md) en el inquilino que ha creado para que pueda comunicarse con Azure AD B2C.
- [Cree una aplicación de Facebook](identity-provider-facebook.md#create-a-facebook-application). Omita los requisitos previos y el restablecimiento de los pasos descritos en el artículo [Configuración del registro y el inicio de sesión con una cuenta de Facebook](identity-provider-facebook.md). Aunque no se requiere una aplicación de Facebook para el uso de directivas personalizadas, se usa en este tutorial para demostrar la habilitación del inicio de sesión social en una directiva personalizada.

::: zone-end

::: zone pivot="b2c-user-flow"
## <a name="create-a-sign-up-and-sign-in-user-flow"></a>Creación de un flujo de usuario de registro e inicio de sesión

Este flujo de usuario de registro y de inicio de sesión controla las experiencias de registro y de inicio de sesión del cliente con una sola configuración. A los usuarios se les lleva por el proceso correcto según el contexto.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

    ![Inquilino de B2C, panel de directorio y suscripción, Azure Portal](./media/tutorial-create-user-flows/directory-subscription-pane.png)

1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Flujos de usuario** y **Nuevo flujo de usuario**.

    ![Página Flujos de usuario del portal con el botón Nuevo flujo de usuario resaltado](./media/tutorial-create-user-flows/signup-signin-user-flow.png)

1. En la página **Crear un flujo de usuario**, seleccione el flujo de usuario **Registrarse e iniciar sesión**.

    ![Página Selección de un flujo de usuario con las opciones Registrarse e Iniciar sesión resaltadas](./media/tutorial-create-user-flows/select-user-flow-type.png)

1. En **Seleccione una versión**, elija **Recomendada** y, luego, seleccione **Crear**. [Más información](user-flow-versions.md) sobre las versiones del flujo de usuario.

    ![Página Creación de flujo de usuario en Azure Portal con las propiedades resaltadas](./media/tutorial-create-user-flows/select-version.png)

1. Escriba un **nombre** para el flujo de usuario. Por ejemplo, *signupsignin1*.
1. En **Proveedores de identidades**, seleccione **Registro por correo electrónico**.
1. En **Atributos y notificaciones de usuario**, elija los atributos y las notificaciones que desea recopilar y enviar al usuario durante el registro. Por ejemplo, seleccione **Mostrar más** y elija los atributos y las notificaciones de **País o región**, **Nombre para mostrar** y **Código postal**. Haga clic en **OK**.

    ![Página de selección de atributos y notificaciones con tres notificaciones seleccionadas](./media/tutorial-create-user-flows/signup-signin-attributes.png)

1. Haga clic en **Crear** para agregar el flujo de usuario. El prefijo *B2C_1* se anexa automáticamente al nombre.

### <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Seleccione el flujo de usuario que ha creado para abrir su página de información general y, luego, **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *webapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Haga clic en **Ejecutar flujo de usuario** y, a continuación, seleccione **Registrarse ahora**.

    ![Página Ejecutar flujo de usuario del portal con el botón Ejecutar flujo de usuario resaltado](./media/tutorial-create-user-flows/signup-signin-run-now.PNG)

1. Escriba una dirección de correo electrónico válida, haga clic en **Enviar código de verificación**, escriba el código de verificación que reciba y seleccione **Comprobar código**.
1. Escriba la contraseña nueva y confírmela.
1. Seleccione su país y región, escriba el nombre que desee que aparezca, escriba un código postal y haga clic en **Crear**. El token se devuelve al `https://jwt.ms` y debe mostrarse.
1. Ahora puede ejecutar el flujo de usuario nuevo y debe ser capaz de iniciar sesión con la cuenta que ha creado. El token devuelto incluye las notificaciones que seleccionó para país/región, nombre y código postal.

> [!NOTE]
> Actualmente, la experiencia "Ejecutar flujo de usuario" no es compatible con el tipo de dirección URL de respuesta de SPA que usa el flujo de código de autorización. Para usar la experiencia "Ejecutar flujo de usuario" con estos tipos de aplicaciones, registre una dirección URL de respuesta de tipo "Web" y habilite el flujo implícito tal como se describe [aquí](tutorial-register-spa.md).

## <a name="enable-self-service-password-reset"></a>Habilitar el autoservicio de restablecimiento de contraseña

Para habilitar el [autoservicio de restablecimiento de contraseña](add-password-reset-policy.md) para el flujo de usuario de registro o inicio de sesión:

1. Seleccione el flujo de usuario de registro o de inicio de sesión que creó.
1. En el menú de la izquierda, en **Configuración**, seleccione **Propiedades**.
1. En **Complejidad de la contraseña**, seleccione **Autoservicio de restablecimiento de contraseña**.
1. Seleccione **Guardar**.

### <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Seleccione el flujo de usuario que ha creado para abrir su página de información general y, luego, **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *webapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione **Ejecutar flujo de usuario**.
1. En la página de registro o inicio de sesión, seleccione **¿Olvidó la contraseña?**
1. Compruebe la dirección de correo electrónico de la cuenta que creó anteriormente y, luego seleccione **Continuar**.
1. Ahora tiene la oportunidad de cambiar la contraseña para el usuario. Cambie la contraseña y seleccione **Continuar**. El token se devuelve al `https://jwt.ms` y debe mostrarse.

## <a name="create-a-profile-editing-user-flow"></a>Creación de un flujo de usuario de edición de perfil

Si desea permitir que los usuarios editen sus perfiles en la aplicación, debe usar un flujo de usuario de edición de perfiles.

1. En el menú de la página de información general del inquilino de Azure AD B2C, seleccione **Flujos de usuario** y **Nuevo flujo de usuario**.
1. En la página **Crear un flujo de usuario**, seleccione el flujo de usuario **Edición de perfiles**. 
1. En **Seleccione una versión**, elija **Recomendada** y, luego, seleccione **Crear**.
1. Escriba un **nombre** para el flujo de usuario. Por ejemplo, *profileediting1*.
1. En **Proveedores de identidades**, seleccione **Inicio de sesión de cuenta local**.
2. En **Atributos de usuario**, elija los atributos que desee que el cliente pueda modificar en su perfil. Por ejemplo, seleccione **Mostrar más** y elija los atributos y las notificaciones de **Nombre para mostrar** y **Puesto**. Haga clic en **OK**.
3. Haga clic en **Crear** para agregar el flujo de usuario. El prefijo *B2C_1* se anexa automáticamente al nombre.

### <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Seleccione el flujo de usuario que ha creado para abrir su página de información general y, luego, **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *webapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Haga clic en **Ejecutar flujo de usuario** y, a continuación, inicie sesión con la cuenta que creó anteriormente.
1. Ahora tiene la oportunidad de cambiar el nombre para mostrar y el puesto del usuario. Haga clic en **Continuar**. El token se devuelve al `https://jwt.ms` y debe mostrarse.
::: zone-end

::: zone pivot="b2c-custom-policy"
> [!TIP]
> En este artículo se explica cómo configurar el inquilino manualmente. Puede automatizar todo el proceso con este artículo. La automatización implementará el [paquete de inicio SocialAndLocalAccountsWithMFA](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack) de Azure AD B2C, que proporcionará los recorridos de registro e inicio de sesión, restablecimiento de contraseña y edición de perfil. Para automatizar el tutorial siguiente, visite la [aplicación de configuración de IEF](https://aka.ms/iefsetup) y siga las instrucciones.


## <a name="add-signing-and-encryption-keys"></a>Agregar claves de firma y cifrado

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En la página de información general, en **Directivas**, seleccione **Identity Experience Framework**.

### <a name="create-the-signing-key"></a>Crear la clave de firma

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Generate`.
1. En **Nombre**, escriba `TokenSigningKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
1. En **Tipo de clave**, seleccione **RSA**.
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

### <a name="create-the-encryption-key"></a>Crear la clave de cifrado

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Generate`.
1. En **Nombre**, escriba `TokenEncryptionKeyContainer`. Es posible que se agregue automáticamente el prefijo `B2C_1A`_.
1. En **Tipo de clave**, seleccione **RSA**.
1. En **Uso de la clave**, seleccione **Cifrado**.
1. Seleccione **Crear**.

### <a name="create-the-facebook-key"></a>Crear la clave de Facebook

Agregue el [secreto de la aplicación](identity-provider-facebook.md) de la aplicación de Facebook como una clave de directiva. Puede usar el secreto de aplicación de la aplicación que creó como parte de los requisitos previos de este artículo.

1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Manual`.
1. En **Nombre**, escriba `FacebookSecret`. Es posible que se agregue automáticamente el prefijo `B2C_1A_`.
1. En **Secreto**, escriba el *secreto de aplicación* de la aplicación de Facebook de developers.facebook.com. Este valor es el secreto, no el identificador de la aplicación.
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

## <a name="register-identity-experience-framework-applications"></a>Registro de las aplicaciones del marco de experiencia de identidad

Azure AD B2C requiere que registre dos aplicaciones que se usen para registrar usuarios e iniciar su sesión con cuentas locales: *IdentityExperienceFramework* (una API web) y *ProxyIdentityExperienceFramework* (una aplicación nativa) con permiso delegado de la aplicación IdentityExperienceFramework. Los usuarios pueden registrarse con una dirección de correo electrónico o un nombre de usuario y una contraseña para acceder a las aplicaciones registradas en el inquilino, lo que crea una "cuenta local". Las cuentas locales solo existen en el inquilino de Azure AD B2C.

Solo tiene que registrar estas dos aplicaciones en el inquilino de Azure AD B2C una vez.

### <a name="register-the-identityexperienceframework-application"></a>Registrar la aplicación IdentityExperienceFramework

Para registrar una aplicación en el inquilino de Azure AD B2C, puede usar la experiencia **Registros de aplicaciones**.

1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. En **Nombre**, escriba `IdentityExperienceFramework`.
1. En **Tipos de cuenta admitidos**, seleccione **Solo las cuentas de este directorio organizativo**.
1. En **URI de redirección**, seleccione **Web** y, a continuación, escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com`, donde `your-tenant-name` es el nombre de dominio del inquilino de Azure AD B2C.
1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

A continuación, exponga la API agregando un ámbito:

1. En el menú de la izquierda, en **Administrar**, seleccione **Exponer una API**.
1. Seleccione **agregar un ámbito** y, a continuación, **Guardar y continuar** para aceptar el URI de identificador de aplicación predeterminado.
1. Escriba los valores siguientes para crear un ámbito que permita la ejecución de la directiva personalizada en el inquilino de Azure AD B2C:
    * **Nombre de ámbito**: `user_impersonation`
    * **Nombre para mostrar del consentimiento del administrador**: `Access IdentityExperienceFramework`
    * **Descripción del consentimiento del administrador**: `Allow the application to access IdentityExperienceFramework on behalf of the signed-in user.`
1. Seleccione **Agregar ámbito**.

* * *

### <a name="register-the-proxyidentityexperienceframework-application"></a>Registrar la aplicación ProxyIdentityExperienceFramework

1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. En **Nombre**, escriba `ProxyIdentityExperienceFramework`.
1. En **Tipos de cuenta admitidos**, seleccione **Solo las cuentas de este directorio organizativo**.
1. En **URI de redirección**, use la lista desplegable para seleccionar **Cliente público o nativo (móvil y escritorio)** .
1. En **URI de redirección**, escriba `myapp://auth`.
1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.
1. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.

A continuación, especifique que la aplicación se debe tratar como un cliente público:

1. En el menú izquierdo, en **Administrar**, seleccione **Autenticación**.
1. En **Configuración avanzada**, en la sección **Permitir flujos de clientes públicos**, establezca **Habilite los siguientes flujos móviles y de escritorio** en **Sí**. Asegúrese de que **"allowPublicClient": true** esté establecido en el manifiesto de aplicación. 
1. Seleccione **Guardar**.

Ahora, conceda permisos al ámbito de la API que expuso anteriormente en el registro de *IdentityExperienceFramework*:

1. En el menú de la izquierda, en **Administrar**, seleccione **Permisos de API**.
1. En **Permisos configurados**, seleccione **Agregar un permiso**.
1. Seleccione la pestaña **Mis API** y, después, seleccione la aplicación **IdentityExperienceFramework**.
1. En **Permiso**, seleccione el ámbito **user_impersonation** que definió anteriormente.
1. Seleccione **Agregar permisos**. Como se indicó, espere unos minutos antes de continuar con el paso siguiente.
1. Seleccione **Conceder consentimiento de administrador para (el nombre de inquilino)** .
1. Seleccione la cuenta de administrador que tiene actualmente la sesión iniciada o inicie sesión con una cuenta en el inquilino de Azure AD B2C que tenga asignado al menos el rol *Administrador de aplicaciones en la nube*.
1. Seleccione **Aceptar**.
1. Seleccione **Actualizar** y luego compruebe que "Concedido para..." aparece en **Estado** para los ámbitos offline_access, openid y user_impersonation. Los permisos pueden tardar unos minutos en propagarse.

* * *

## <a name="custom-policy-starter-pack"></a>Paquete de inicio de directivas personalizadas

Las directivas personalizadas son un conjunto de archivos XML que se cargan en el inquilino de Azure AD B2C para definir perfiles técnicos y recorridos de usuario. Proporcionamos paquetes de inicio con varias directivas predefinidas para que pueda empezar a trabajar rápidamente. Cada uno de estos paquetes de inicio contiene el menor número de perfiles técnicos y recorridos del usuario necesarios para lograr los escenarios descritos:

- **LocalAccounts**: habilita el uso solo de cuentas locales.
- **SocialAccounts**: habilita el uso solo de cuentas sociales (o federadas).
- **SocialAndLocalAccounts**: habilita el uso de cuentas locales y sociales.
- **SocialAndLocalAccountsWithMFA**: habilita opciones sociales, locales y de autenticación multifactor.

Cada paquete de inicio contiene lo siguiente:

- **Archivo base**: se requieren algunas modificaciones en el archivo base. Ejemplo: *TrustFrameworkBase.xml*
- **Archivo de extensión**: este archivo es donde se hace la mayoría de los cambios de configuración. Ejemplo: *TrustFrameworkExtensions.xml*
- **Archivos de usuario de confianza**: archivos específicos de la tarea a los que llama la aplicación. Ejemplos: *SignUpOrSignin.xml*, *ProfileEdit.xml*, *PasswordReset.xml*.

En este artículo, editará los archivos de directivas personalizadas XML del paquete de inicio **SocialAndLocalAccounts**. Si necesita un editor XML, [pruebe Visual Studio Code](https://code.visualstudio.com/download), un editor multiplataforma ligero.

### <a name="get-the-starter-pack"></a>Obtención del paquete de inicio

Obtenga los paquetes de inicio de directivas personalizadas en GitHub y, luego, actualice los archivos XML del paquete de inicio SocialAndLocalAccounts con el nombre del inquilino de Azure AD B2C.

1. [Descargue el archivo .zip](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) o clone el repositorio:

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
    ```

1. En todos los archivos del directorio **SocialAndLocalAccounts**, reemplace la cadena `yourtenant` por el nombre de su inquilino de Azure AD B2C.

    Por ejemplo, si el nombre del inquilino de B2C es *contosotenant*, todas las instancias de `yourtenant.onmicrosoft.com` se convierten en `contosotenant.onmicrosoft.com`.

### <a name="add-application-ids-to-the-custom-policy"></a>Agregar identificadores de aplicación a la directiva personalizada

Agregue los identificadores de aplicación al archivo de extensiones *TrustFrameworkExtensions.xml*.

1. Abra `SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`** y busque el elemento `<TechnicalProfile Id="login-NonInteractive">`.
1. Reemplace ambas instancias de `IdentityExperienceFrameworkAppId` por el identificador de la aplicación IdentityExperienceFramework que creó antes.
1. Reemplace ambas instancias de `ProxyIdentityExperienceFrameworkAppId` por el identificador de la aplicación ProxyIdentityExperienceFramework que creó antes.
1. Guarde el archivo.

## <a name="upload-the-policies"></a>Cargar las directivas

1. Seleccione el elemento de menú **Identity Experience Framework** en el inquilino de B2C en Azure Portal.
1. Seleccione **Cargar directiva personalizada**.
1. En este orden, cargue los archivos de directiva:
    1. *TrustFrameworkBase.xml*
    1. *TrustFrameworkExtensions.xml*
    1. *SignUpOrSignin.xml*
    1. *ProfileEdit.xml*
    1. *PasswordReset.xml*

A medida que cargue los archivos, Azure agregará el prefijo `B2C_1A_` a cada uno.

> [!TIP]
> Si el editor XML admite la validación, valide los archivos con el archivo de esquema XML `TrustFrameworkPolicy_0.3.0.0.xsd` que se encuentra en el directorio raíz del paquete de inicio. La validación del esquema XML identifica los errores antes de realizar la carga.

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. En **Directivas personalizadas**, seleccione **B2C_1A_signup_signin**.
1. Para **Seleccionar aplicación** en la página Información general de la directiva personalizada, seleccione la aplicación web denominada *webapp1* que ha registrado antes.
1. Asegúrese de que la **URL de respuesta** sea `https://jwt.ms`.
1. Seleccione **Ejecutar ahora**.
1. Regístrese con una dirección de correo electrónico.
1. Vuelva a seleccionar **Ejecutar ahora**.
1. Inicie sesión con la misma cuenta para confirmar que tiene una configuración correcta.

## <a name="add-facebook-as-an-identity-provider"></a>Incorporación de Facebook como proveedor de identidades

Como se mencionó en [Requisitos previos](#prerequisites), *no* se requiere Facebook para el uso de directivas personalizadas, pero aquí se usa para mostrar cómo se puede habilitar el inicio de sesión social federado en una directiva personalizada.

1. En el archivo `SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`** , sustituya el valor de `client_id` por el identificador de aplicación de Facebook:

   ```xml
   <TechnicalProfile Id="Facebook-OAUTH">
     <Metadata>
     <!--Replace the value of client_id in this technical profile with the Facebook app ID"-->
       <Item Key="client_id">00000000000000</Item>
   ```

1. Cargue el archivo *TrustFrameworkExtensions.xml* en el inquilino.
1. En **Directivas personalizadas**, seleccione **B2C_1A_signup_signin**.
1. Seleccione **Ejecutar ahora** y elija Facebook para iniciar sesión con Facebook y probar la directiva personalizada.

::: zone-end
## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Creación de un flujo de usuario de registro e inicio de sesión
> * Creación de un flujo de usuario de edición de perfil
> * Creación de un flujo de usuario de restablecimiento de contraseña

A continuación, obtenga información sobre cómo usar Azure AD B2C para iniciar sesión y registrar usuarios en una aplicación. Siga la aplicación web de ASP.NET vinculada a continuación o vaya a otra aplicación de la tabla de contenido en **Autenticación de usuarios**.

> [!div class="nextstepaction"]
> [Tutorial: Habilitación de la autenticación en una aplicación web mediante Azure Active Directory B2C >](tutorial-web-app-dotnet.md)
