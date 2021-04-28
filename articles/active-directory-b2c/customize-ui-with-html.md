---
title: Personalización de la interfaz de usuario con plantillas HTML
titleSuffix: Azure AD B2C
description: Obtenga información acerca de cómo personalizar la interfaz de usuario con plantillas HTML en las aplicaciones que usan Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/19/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 8f9f6dc1abd08c5e53f3d44a8f6ec1b3e20786ed
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107717448"
---
# <a name="customize-the-user-interface-with-html-templates-in-azure-active-directory-b2c"></a>Personalización de la interfaz de usuario con plantillas HTML en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

La personalización de marca y la personalización de la interfaz de usuario que Azure Active Directory B2C (Azure AD B2C) muestra a los clientes ayuda a proporcionar una experiencia de usuario sin problemas en la aplicación. Estas experiencias incluyen las opciones de registro, inicio de sesión, edición de perfiles y restablecimiento de contraseñas. En este artículo se presentan los métodos de personalización de la interfaz de usuario (UI). 

> [!TIP]
> Si solo quiere modificar el logotipo del banner, la imagen de fondo y el color de fondo de las páginas de flujo de usuario, puede probar la característica [Personalización de marca de empresa](customize-ui.md).

## <a name="custom-html-and-css-overview"></a>Información general sobre HTML y CSS personalizados

Azure AD B2C ejecuta código en el explorador del cliente con un enfoque denominado [Uso compartido de recursos entre orígenes (CORS)](https://www.w3.org/TR/cors/). En tiempo de ejecución, el contenido se carga desde una dirección URL especificada en un flujo de usuario o directiva personalizada. Cada página de la experiencia del usuario carga su contenido desde la dirección URL que especifique para esa página. Cuando el contenido se carga desde la dirección URL se combina con un fragmento HTML insertado en Azure AD B2C y, a continuación, se muestra la página al cliente.

![Margen de contenido de la página personalizada](./media/customize-ui-with-html/html-content-merging.png)

### <a name="custom-html-page-content"></a>Contenido de la página HTML personalizada

Cree una página HTML con su propia personalización de marca para servir el contenido de la página personalizada. Esta página puede ser una página `*.html` estática, o una página dinámica como .NET, Node.js o PHP.

El contenido de la página personalizada puede contener cualquier elemento HTML, incluidos CSS y JavaScript, pero no puede incluir elementos no seguros como iframes. El único elemento necesario es un elemento div con el `id` establecido en `api`, como este `<div id="api"></div>` dentro de la página HTML.

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Product Brand Name</title>
</head>
<body>
    <div id="api"></div>
</body>
</html>
```

#### <a name="customize-the-default-azure-ad-b2c-pages"></a>Personalización de las páginas predeterminadas de Azure AD B2C

En lugar de crear el contenido de la página personalizada desde el principio, puede personalizar el contenido de la página predeterminada de Azure AD B2C.

En la tabla siguiente se muestra el contenido de la página predeterminada proporcionado por Azure AD B2C. Descargue los archivos y úselos como punto de partida para crear sus propias páginas personalizadas.

| Página predeterminada | Descripción | Id. de definición de contenido<br/>(solo directiva personalizada) |
|:-----------------------|:--------|-------------|
| [exception.html](https://login.microsoftonline.com/static/tenant/default/exception.cshtml) | **Página de error**. Esta página se muestra cuando se produce una excepción o un error. | *api.error* |
| [selfasserted.html](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) |  **Página autoafirmada**. Use este archivo como contenido de la página predeterminada para una página de registro en una cuenta social, una página de registro en una cuenta local, una página de inicio de sesión en una cuenta local, el restablecimiento de contraseña y mucho más. El formulario puede contener varios controles de entrada, como un cuadro de entrada de texto, un cuadro de entrada de contraseña, un botón de radio, cuadros desplegables de selección única y casillas de selección múltiple. | *api.localaccountsignin*, *api.localaccountsignup*, *api.localaccountpasswordreset*, *api.selfasserted* |
| [multifactor-1.0.0.html](https://login.microsoftonline.com/static/tenant/default/multifactor-1.0.0.cshtml) | **Página de autenticación multifactor**. Esta página permite a los usuarios verificar sus números de teléfono (mediante texto o voz) durante el registro o el inicio de sesión. | *api.phonefactor* |
| [updateprofile.html](https://login.microsoftonline.com/static/tenant/default/updateProfile.cshtml) | **Página de actualización de perfil**. Esta página contiene un formulario que los usuarios pueden usar para actualizar su perfil. Esta página es similar a la página de registro en una cuenta social, excepto por los campos de entrada de contraseña. | *api.selfasserted.profileupdate* |
| [unified.html](https://login.microsoftonline.com/static/tenant/default/unified.cshtml) | **Página de inicio de sesión o registro unificada**. Esta página controla los procesos de registro y de inicio de sesión del usuario. Los usuarios pueden utilizar proveedores de identidades de empresa, proveedores de identidades sociales como Facebook y Google+ o cuentas locales. | *api.signuporsignin* |

## <a name="hosting-the-page-content"></a>Hospedaje del contenido de la página

Al utilizar sus propios archivos HTML y CSS para personalizar la interfaz de usuario, hospede el contenido de la interfaz de usuario en cualquier punto de conexión HTTPS disponible públicamente que admita CORS. Por ejemplo, [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md), [Azure App Service](../app-service/index.yml), servidores web, redes CDN, AWS S3 o sistemas de uso compartido de archivos.

## <a name="guidelines-for-using-custom-page-content"></a>Directrices para usar el contenido de la página personalizada

- Use una dirección URL absoluta al incluir recursos externos como archivos multimedia, CSS y JavaScript en el archivo HTML.
- Con la [versión de diseño de página](page-layout.md) 1.2.0 y posterior, puede agregar el atributo `data-preload="true"` en las etiquetas HTML para controlar el orden de carga de CSS y JavaScript. Con `data-preload="true"`, la página se construye antes de que se muestre al usuario. Este atributo ayuda a evitar que la página "parpadee" al cargar previamente el archivo CSS, sin que el HTML sin estilo se muestre al usuario. El siguiente fragmento de código HTML muestra el uso de la etiqueta `data-preload`.
  ```HTML
  <link href="https://path-to-your-file/sample.css" rel="stylesheet" type="text/css" data-preload="true"/>
  ```
- Se recomienda comenzar con el contenido de la página predeterminada y compilar sobre él.
- Puede [incluir JavaScript](javascript-and-page-layout.md) en el contenido personalizado.
- Estas son las versiones de explorador admitidas:
  - Internet Explorer 11 y 10 y Microsoft Edge
  - Compatibilidad limitada con Internet Explorer 9 y 8.
  - Google Chrome 42.0 y versiones posteriores
  - Mozilla Firefox 38.0 y versiones posteriores
  - Safari para iOS y macOS, versión 12 y posteriores
- Debido a las restricciones de seguridad, Azure AD B2C no es compatible con los elementos HTML `frame`, `iframe` o `form`.

## <a name="localize-content"></a>Localización del contenido

Puede localizar el contenido HTML si habilita la [personalización de idioma](language-customization.md) en su inquilino de Azure AD B2C. Al habilitar esta característica, Azure AD B2C puede reenviar el parámetro `ui_locales` de OpenID Connect al punto de conexión. El servidor de contenido puede usar este parámetro para proporcionar páginas HTML específicas del idioma.

Como alternativa, puede extraer contenido de distintos lugares, según la configuración regional que use. En el punto de conexión habilitado para CORS, puede configurar una estructura de carpetas para hospedar contenido de idiomas concretos. Se llamará al adecuado si usa el valor de carácter comodín `{Culture:RFC5646}`.

Por ejemplo, el URI de la página personalizada podría ser similar al siguiente:

```http
https://contoso.blob.core.windows.net/{Culture:RFC5646}/myHTML/unified.html
```

Puede cargar la página en francés mediante la extracción de contenido de:

```http
https://contoso.blob.core.windows.net/fr/myHTML/unified.html
```

## <a name="custom-page-content-walkthrough"></a>Tutorial sobre el contenido de la página personalizada

A continuación se muestra una información general acerca del proceso:

1. Prepare una ubicación para hospedar el contenido de la página personalizada (un punto de conexión HTTPS con acceso público y habilitado para CORS).
1. Descargue y personalice un archivo de contenido de la página predeterminada, por ejemplo `unified.html`.
1. Publique el contenido de la página personalizada en el punto de conexión HTTPS disponible públicamente.
1. Establecimiento de uso compartido de recursos entre orígenes (CORS) para la aplicación web.
1. Apunte la directiva al URI de contenido de la directiva personalizada.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

### <a name="1-create-your-html-content"></a>1. Creación del contenido HTML

Puede crear contenido de la página personalizada con el nombre de la marca del producto en el título.

1. Copie el siguiente fragmento de código HTML. Se trata de código HTML5 con formato correcto con un elemento vacío llamado *\<div id="api"\>\</div\>* situado en las etiquetas *\<body\>* . Este elemento indica dónde se va a insertar el contenido de Azure AD B2C.

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My Product Brand Name</title>
   </head>
   <body>
       <div id="api"></div>
   </body>
   </html>
   ```

1. Pegue el fragmento de código copiado en un editor de texto.
1. Use CSS para aplicar estilo a los elementos de la interfaz de usuario que inserta Azure AD B2C en la página. En el ejemplo siguiente se muestra un archivo CSS sencillo que también incluye la configuración de los elementos HTML insertados en el registro:

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center;
    }
    .entry {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center;
    }
    .create {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

1.  Guarde el archivo como *customize-ui.html*.

> [!NOTE]
> Si se usa login.microsoftonline.com, los elementos de formulario HTML se quitarán debido a las restricciones de seguridad. Si quiere usar elementos de formulario HTML en el contenido HTML personalizado, [utilice b2clogin.com](b2clogin.md).

### <a name="2-create-an-azure-blob-storage-account"></a>2. Creación de una cuenta de Almacenamiento de blobs de Azure

En este artículo se usa Azure Blob Storage para hospedar el contenido. Puede elegir hospedar el contenido en un servidor web, pero deberá [habilitar CORS en el servidor web](https://enable-cors.org/server.html).

Para hospedar el contenido HTML en Blob Storage, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú **central**, seleccione **Nuevo** > **Almacenamiento** > **Cuenta de almacenamiento**.
1. Seleccione una **Suscripción** para la cuenta de almacenamiento.
1. Cree un **Grupo de recursos** o seleccione uno existente.
1. Escriba un **Nombre** único para la cuenta de almacenamiento.
1. Seleccione la **Ubicación geográfica** de la cuenta de almacenamiento.
1. El **Modelo de implementación** puede permanecer como **Resource Manager**.
1. El **Rendimiento** puede permanecer como **Estándar**.
1. Cambie **Tipo de cuenta** a **Almacenamiento de blobs**.
1. La **Replicación** puede permanecer como **RA-GRS**.
1. El **Nivel de acceso** puede permanecer **Activo**.
1. Haga clic en **Revisar y crear** para crear la cuenta de almacenamiento.
    Espere a que termine la implementación y la página de la **cuenta de almacenamiento** se abrirá automáticamente.

#### <a name="21-create-a-container"></a>2.1 Creación de un contenedor

Para crear un contenedor público en Blob Storage, realice los siguientes pasos:

1. En **Blob service** en el menú izquierdo, seleccione **Blobs**.
1. Seleccione **+ Contenedor**.
1. En **Nombre**, escriba *root*. El nombre puede ser el que quiera, por ejemplo *contoso*; sin embargo, usamos *root* en este ejemplo por motivos de simplicidad.
1. Para **Nivel de acceso público**, seleccione **Blob** y, luego, **Aceptar**.
1. Seleccione **root** para abrir el contenedor nuevo.

#### <a name="22-upload-your-custom-page-content-files"></a>2.2 Carga de los archivos de contenido de la página personalizada

1. Seleccione **Cargar**.
1. Haga clic en el icono de carpeta junto a **Seleccionar un archivo**.
1. Vaya a **customize-ui.html**, que creó anteriormente en la sección Personalización de la interfaz de usuario de la página, y selecciónelo.
1. Si quiere cargar en una subcarpeta, expanda **Avanzadas** y escriba un nombre de la carpeta en **Cargar en carpeta**.
1. Seleccione **Cargar**.
1. Seleccione el blob **customize-ui.html** que cargó.
1. A la derecha del cuadro de texto **URL**, seleccione el icono **Copiar al Portapapeles** para copiar la dirección URL en el Portapapeles.
1. En el explorador web, vaya a la dirección URL que copió para verificar que puede acceder al blob que cargó. Si no puede acceder, por ejemplo, si hay un error `ResourceNotFound`, asegúrese de que el tipo de acceso del contenedor esté establecido en **blob**.

### <a name="3-configure-cors"></a>3. Configuración de CORS

Para configurar Blob Storage para el uso compartido de recursos entre orígenes (CORS), realice los siguientes pasos:

1. En el menú, seleccione **CORS**.
1. En **Orígenes permitidos**, escriba `https://your-tenant-name.b2clogin.com`. Reemplace `your-tenant-name` por el nombre del inquilino de Azure AD B2C. Por ejemplo, `https://fabrikam.b2clogin.com`. Al escribir su nombre de inquilino, use solo minúsculas.
1. En **Métodos permitidos**, seleccione `GET` y `OPTIONS`.
1. En **Encabezados permitidos**, escriba un asterisco (*).
1. En **Encabezados expuestos**, escriba un asterisco (*).
1. Para **Antigüedad máxima**, introduzca 200.
1. Seleccione **Guardar**.

#### <a name="31-test-cors"></a>3.1 Prueba de CORS

Para comprobar que está listo, realice los siguientes pasos:

1. Repita el paso de configuración de CORS. En **Orígenes permitidos**, escriba `https://www.test-cors.org`.
1. Vaya a [www.test-cors.org](https://www.test-cors.org/). 
1. En el cuadro de **URL remotas**, pegue la dirección URL del archivo HTML. Por ejemplo: `https://your-account.blob.core.windows.net/root/azure-ad-b2c/unified.html`
1. Seleccione **Enviar solicitud**.
    El resultado debe ser `XHR status: 200`. 
    Si recibe un error, asegúrese de que la configuración de CORS sea correcta. Puede que también deba borrar la caché del explorador o abrir una sesión de navegación privada, para ello, presione Ctrl + Mayús + P.

::: zone pivot="b2c-user-flow"

### <a name="4-update-the-user-flow"></a>4. Actualización del flujo de usuario

1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario** y, luego, elija el flujo de usuario *B2C_1_signupsignin1*.
1. Seleccione **Diseños de página** y, a continuación, en **Página unificada de inicio de sesión o de registro**, haga clic en **Sí** para **Usar contenido de la página personalizada**.
1. En **URI de página personalizado**, escriba el identificador URI del archivo *custom-ui.html* que anotó anteriormente.
1. En la parte superior de la página, seleccione **Guardar**.

### <a name="5-test-the-user-flow"></a>5. Prueba del flujo de usuario

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario** y seleccione el flujo *B2C_1_signupsignin1*.
1. En la parte superior de la página, haga clic en **Ejecutar flujo de usuario**.
1. Haga clic en el botón **Ejecutar flujo de usuario**.

Debería ver una página similar al ejemplo siguiente con los elementos centrados basándose en el archivo CSS que ha creado:

![Explorador web que muestra la página de registro o inicio de sesión con elementos de la interfaz de usuario personalizados](./media/customize-ui-with-html/run-now.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

### <a name="4-modify-the-extensions-file"></a>4. Modificación del archivo de extensiones

Para configurar la personalización de la interfaz de usuario, copie **ContentDefinition** y sus elementos secundarios del archivo base en el archivo de extensiones.

1. Abra el archivo base de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkBase.xml`**</em>. Este archivo base es uno de los archivos de directivas incluidos en el paquete de inicio de directivas personalizadas, que debería haber obtenido en el requisito previo, [Introducción a las directivas personalizadas en Azure Active Directory B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy).
1. Busque y copie todo el contenido del elemento **ContentDefinitions**.
1. Abra el archivo de extensión. Por ejemplo, *TrustFrameworkExtensions.xml*. Busque el elemento **BuildingBlocks**. Si el elemento no existe, agréguelo.
1. Pegue todo el contenido del elemento **ContentDefinitions** que ha copiado como elemento secundario del elemento **BuildingBlocks**.
1. Busque el elemento **ContentDefinition** que contenga `Id="api.signuporsignin"` en el XML que ha copiado.
1. Cambie el valor de **LoadUri** a la dirección URL del archivo HTML que cargó en el almacenamiento. Por ejemplo, `https://your-storage-account.blob.core.windows.net/your-container/customize-ui.html`.

    La directiva personalizada debería ser similar al fragmento de código siguiente:

    ```xml
    <BuildingBlocks>
      <ContentDefinitions>
        <ContentDefinition Id="api.signuporsignin">
          <LoadUri>https://your-storage-account.blob.core.windows.net/your-container/customize-ui.html</LoadUri>
          <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
          <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.0.0</DataUri>
          <Metadata>
            <Item Key="DisplayName">Signin and Signup</Item>
          </Metadata>
        </ContentDefinition>
      </ContentDefinitions>
    </BuildingBlocks>
    ```

1. Guarde el archivo de extensiones.

### <a name="5-upload-and-test-your-updated-custom-policy"></a>5. Carga y prueba de la directiva personalizada actualizada

#### <a name="51-upload-the-custom-policy"></a>5.1 Carga de la directiva personalizada

1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Cargar directiva personalizada**.
1. Cargue el archivo de extensiones que cambió anteriormente.

#### <a name="52-test-the-custom-policy-by-using-run-now"></a>5.2 Prueba de la directiva personalizada con **Ejecutar ahora**

1. Seleccione la directiva cargada y, a continuación, **Ejecutar ahora**.
1. Debería poder registrarse con una dirección de correo electrónico.

## <a name="configure-dynamic-custom-page-content-uri"></a>Configuración del URI de contenido de página personalizada dinámica

Con las directivas personalizadas de Azure AD B2C, puede enviar un parámetro de la ruta de la dirección URL o una cadena de consulta. Al pasar dicho parámetro al punto de conexión HTML, puede cambiar de forma dinámica el contenido de la página. Por ejemplo, puede cambiar la imagen de fondo en la página de inicio de sesión o de registro de Azure AD B2C en función de un parámetro que se pasa desde la aplicación web o dispositivo móvil. El parámetro puede ser cualquier [solucionador de notificaciones](claim-resolver-overview.md), como el identificador de la aplicación, el identificador de idioma o el parámetro de cadena de consulta personalizada, como `campaignId`.

### <a name="sending-query-string-parameters"></a>Envío de parámetros de cadena de consulta

Para enviar parámetros de cadena de consulta, en la [directiva del usuario de confianza](relyingparty.md), agregue un elemento `ContentDefinitionParameters` tal como se muestra a continuación.

```xml
<RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <UserJourneyBehaviors>
    <ContentDefinitionParameters>
        <Parameter Name="campaignId">{OAUTH-KV:campaignId}</Parameter>
        <Parameter Name="lang">{Culture:LanguageName}</Parameter>
        <Parameter Name="appId">{OIDC:ClientId}</Parameter>
    </ContentDefinitionParameters>
    </UserJourneyBehaviors>
    ...
</RelyingParty>
```

En la definición de contenido, cambie el valor de `LoadUri` a `https://<app_name>.azurewebsites.net/home/unified`. La directiva personalizada `ContentDefinition` debería ser similar al fragmento de código siguiente:

```xml
<ContentDefinition Id="api.signuporsignin">
  <LoadUri>https://<app_name>.azurewebsites.net/home/unified</LoadUri>
  ...
</ContentDefinition>
```

Cuando Azure AD B2C carga la página, realiza una llamada al punto de conexión del servidor web:

```http
https://<app_name>.azurewebsites.net/home/unified?campaignId=123&lang=fr&appId=f893d6d3-3b6d-480d-a330-1707bf80ebea
```

### <a name="dynamic-page-content-uri"></a>URI de contenido de página dinámica

Puede extraer contenido de distintos lugares, según los parámetros utilizados. En el punto de conexión habilitado para CORS, configure una estructura de carpetas para hospedar contenido. Por ejemplo, puede organizar el contenido en la estructura siguiente. Raíz *carpeta/carpeta por idioma/sus archivos html*. Por ejemplo, el URI de la página personalizada podría ser similar al siguiente:

```xml
<ContentDefinition Id="api.signuporsignin">
  <LoadUri>https://contoso.blob.core.windows.net/{Culture:LanguageName}/myHTML/unified.html</LoadUri>
  ...
</ContentDefinition>
```

Azure AD B2C envía el código ISO de dos letras para el idioma, `fr` para francés:

```http
https://contoso.blob.core.windows.net/fr/myHTML/unified.html
```

::: zone-end

## <a name="sample-templates"></a>Plantillas de ejemplo

Aquí encontrará plantillas de ejemplo de personalización de interfaz de usuario:

```bash
git clone https://github.com/azure-ad-b2c/html-templates
```

El proyecto contiene las plantillas siguientes:
- [Azul océano](https://github.com/azure-ad-b2c/html-templates/tree/main/templates/AzureBlue)
- [Gris pizarra](https://github.com/azure-ad-b2c/html-templates/tree/main/templates/MSA)
- [Clásico](https://github.com/azure-ad-b2c/html-templates/tree/main/templates/classic)
- [Recursos de plantilla](https://github.com/azure-ad-b2c/html-templates/tree/main/templates/src)

Para usar el ejemplo:

1. Clone el repositorio en la máquina local. Seleccione una carpeta de plantilla, `/AzureBlue`, `/MSA` o `/classic`.
1. Cargue todos los archivos de la carpeta de plantilla y la carpeta `/src` en Blob Storage, como se describe en las secciones anteriores.
1. A continuación, abra cada archivo `\*.html` en la carpeta de plantillas. A continuación, reemplace todas las instancias de direcciones URL `https://login.microsoftonline.com` por la dirección URL que cargó en el paso 2. Por ejemplo:
    
    De:
    ```html
    https://login.microsoftonline.com/templates/src/fonts/segoeui.WOFF
    ```

    A:
    ```html
    https://your-storage-account.blob.core.windows.net/your-container/templates/src/fonts/segoeui.WOFF
    ```
    
1. Guarde los archivos `\*.html` y cárguelos en Blob Storage.
1. Ahora modifique la directiva, que apunta al archivo HTML, como se mencionó anteriormente.
1. Si ve que faltan las fuentes, imágenes o CSS, compruebe las referencias en la directiva de extensiones y los archivos `\*.html`.

## <a name="use-company-branding-assets-in-custom-html"></a>Uso de recursos de personalización de marca de empresa en HTML personalizado

Para usar los recursos de [personalización de marca de la empresa](customize-ui.md#configure-company-branding) en un HTML personalizado, agregue las siguientes etiquetas fuera de la etiqueta `<div id="api">`. El origen de la imagen se reemplaza con el de la imagen de fondo y el logotipo del banner.

```HTML
<img data-tenant-branding-background="true" />
<img data-tenant-branding-logo="true" alt="Company Logo" />
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo habilitar el [código JavaScript de cliente](javascript-and-page-layout.md).
