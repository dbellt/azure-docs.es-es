---
title: Autohospedaje del portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre cómo autohospedar el portal para desarrolladores de API Management.
author: dlepow
ms.author: apimpm
ms.date: 04/15/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: e63a329a10699102802af6d544ab55aa19513f17
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741389"
---
# <a name="self-host-the-api-management-developer-portal"></a>Autohospedaje del portal para desarrolladores de API Management

En este tutorial se describe cómo autohospedar el [portal para desarrolladores de API Management](api-management-howto-developer-portal.md). El autohospedaje le ofrece flexibilidad para ampliar el portal para desarrolladores con lógica y widgets personalizados que personalizan dinámicamente las páginas en tiempo de ejecución. Puede autohospedar varios portales para la instancia de API Management, con características diferentes. Cuando se autohospeda un portal, usted se convierte en su administrador y será responsable de sus actualizaciones. 

En los pasos siguientes se muestra cómo configurar el entorno de desarrollo local, realizar cambios en el portal para desarrolladores, y publicarlos e implementarlos en una cuenta de almacenamiento de Azure.

Si ya ha cargado o modificado archivos multimedia en el portal administrado, consulte [Pasaje de administrado a autohospedado](#move-from-managed-to-self-hosted-developer-portal), más adelante en este artículo.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>Requisitos previos

Para configurar un entorno de desarrollo local, tiene que tener:

- Una instancia de servicio de API Management. Si no tiene una, consulte [Inicio rápido: Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
- Una cuenta de almacenamiento de Azure con la [característica de sitios web estáticos](../storage/blobs/storage-blob-static-website.md) habilitada. Vea [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md).
- GIT en la máquina. Para instalarlo, sigua [este tutorial de GIT](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
- Node.js (versión LTS, `v10.15.0` o posterior) y npm en la máquina. Consulte [Descarga e instalación de Node.js y npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm).
- CLI de Azure. Siga los [pasos de instalación de la CLI de Azure](/cli/azure/install-azure-cli-windows).

## <a name="step-1-set-up-local-environment"></a>Paso 1: Configuración del entorno local

Para configurar el entorno local, tendrá que clonar el repositorio, cambiar a la versión más reciente del portal para desarrolladores e instalar paquetes npm.

1. Clone el repositorio [api-management-developer-portal](https://github.com/Azure/api-management-developer-portal.git) desde GitHub:

    ```console
    git clone https://github.com/Azure/api-management-developer-portal.git
    ```
1. Vaya a la copia local del repositorio:

    ```console
    cd api-management-developer-portal
    ```

1. Extraiga la versión más reciente del portal.

    Antes de ejecutar el código siguiente, compruebe la etiqueta de versión actual en la [sección Releases del repositorio](https://github.com/Azure/api-management-developer-portal/releases) y reemplace el valor `<current-release-tag>` por la etiqueta de versión más reciente.
    
    ```console
    git checkout <current-release-tag>
    ```

1. Instale los paquetes npm disponibles:

    ```console
    npm install
    ```

> [!TIP]
> Use siempre la [versión más reciente del portal](https://github.com/Azure/api-management-developer-portal/releases) y mantenga actualizado el portal bifurcado. Los ingenieros de software usan la rama `master` de este repositorio con fines de desarrollo diario. Tiene versiones inestables del software.

## <a name="step-2-configure-json-files-static-website-and-cors-settings"></a>Paso 2: Configuración de archivos JSON, sitio web estático y opciones de CORS

El portal para desarrolladores requiere la API REST de API Management para administrar el contenido.

### <a name="configdesignjson-file"></a>Archivo config.design.json

Vaya a la carpeta `src` y abra el archivo `config.design.json`.

```json
{
  "environment": "development",
  "managementApiUrl": "https://<service-name>.management.azure-api.net",
  "managementApiAccessToken": "SharedAccessSignature ...",
  "backendUrl": "https://<service-name>.developer.azure-api.net",
  "useHipCaptcha": false
}
```

Configure el archivo:

1. En el valor `managementApiUrl`, reemplace `<service-name>` por el nombre de la instancia de API Management. Si configuró un [dominio personalizado](configure-custom-domain.md), úselo en su lugar (por ejemplo, `https://management.contoso.com`).

    ```json
    {
    ...
    "managementApiUrl": "https://contoso-api.management.azure-api.net"
    ...
    ``` 

1. [Cree manualmente un token de SAS](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-authentication#ManuallyCreateToken) para habilitar el acceso directo de la API REST a la instancia de API Management.

1. Copie el token generado y péguelo como valor `managementApiAccessToken`.

1. En el valor `backendUrl`, reemplace `<service-name>` por el nombre de la instancia de API Management. Si configuró un [dominio personalizado](configure-custom-domain.md), úselo en su lugar (por ejemplo, `https://portal.contoso.com`).

    ```json
    {
    ...
    "backendUrl": "https://contoso-api.developer.azure-api.net"
    ...
    ```

1. Si quiere habilitar CAPTCHA en el portal para desarrolladores, consulte [Habilitación de CAPTCHA](#enable-captcha).

### <a name="configpublishjson-file"></a>Archivo config.publish.json

Vaya a la carpeta `src` y abra el archivo `config.publish.json`.

```json
{
  "environment": "publishing",
  "managementApiUrl": "https://<service-name>.management.azure-api.net",
  "managementApiAccessToken": "SharedAccessSignature...",
  "useHipCaptcha": false
}
```

Configure el archivo:

1. Copie los valores `managementApiUrl` y `managementApiAccessToken` del archivo de configuración anterior y péguelos.

1. Si quiere habilitar CAPTCHA en el portal para desarrolladores, consulte [Habilitación de CAPTCHA](#enable-captcha).

### <a name="configruntimejson-file"></a>Archivo config.runtime.json

Vaya a la carpeta `src` y abra el archivo `config.runtime.json`.

```json
{
  "environment": "runtime",
  "managementApiUrl": "https://<service-name>.management.azure-api.net",
  "backendUrl": "https://<service-name>.developer.azure-api.net"
}
```

Configure el archivo:

1. Copie el valor `managementApiUrl` del archivo de configuración anterior y péguelo.

1. En el valor `backendUrl`, reemplace `<service-name>` por el nombre de la instancia de API Management. Si configuró un [dominio personalizado](configure-custom-domain.md), úselo en su lugar (por ejemplo, `https://portal.contoso.com`).

    ```json
    {
    ...
    "backendUrl": "https://contoso-api.developer.azure-api.net"
    ...
    ```

### <a name="configure-the-static-website"></a>Configuración del sitio web estático

Para configurar la característica **Sitio web estático** en la cuenta de almacenamiento, proporcione las rutas a las páginas de índice y error:

1. Vaya a la cuenta de almacenamiento en Azure Portal y seleccione **Sitio web estático** en el menú de la izquierda.

1. En la página **Sitio web estático**, seleccione **Habilitado**.

1. En el campo **Nombre del documento de índice**, escriba *index.html*.

1. En el campo **Ruta de acceso del documento de error**, escriba *404/index.html*.

1. Seleccione **Guardar**.

### <a name="configure-the-cors-settings"></a>Configuración de las opciones de CORS

Configure las opciones de uso compartido de recursos entre orígenes (CORS):

1. Vaya a la cuenta de almacenamiento en Azure Portal y seleccione **CORS** en el menú de la izquierda.

1. En la pestaña **Blob service**, configure las siguientes reglas:

    | Regla | Valor |
    | ---- | ----- |
    | Orígenes permitidos | * |
    | Métodos permitidos | Seleccione todos los verbos HTTP. |
    | Encabezados permitidos | * |
    | Encabezados expuestos | * |
    | Antigüedad máxima | 0 |

1. Seleccione **Guardar**.

## <a name="step-3-run-the-portal"></a>Paso 3: Ejecución del portal

Ahora puede compilar y ejecutar una instancia local del portal en el modo de desarrollo. En el modo de desarrollo, todas las optimizaciones están desactivadas y los mapas de origen están activados.

Ejecute el siguiente comando:

```console
npm start
```

Después de un breve período, el explorador predeterminado se abre automáticamente con la instancia local del portal para desarrolladores. La dirección predeterminada es `http://localhost:8080`, pero el puerto puede cambiar si `8080` ya está ocupado. Todo cambio en el código base del proyecto desencadenará una recompilación y actualizará la ventana del explorador.

## <a name="step-4-edit-through-the-visual-editor"></a>Paso 4: Edición mediante el editor visual

Use el editor visual para llevar a cabo estas tareas:

- Personalizar el portal
- Crear contenido
- Organizar la estructura del sitio web
- Estilizar el aspecto

Consulte [Tutorial: Acceso y personalización del portal para desarrolladores](api-management-howto-developer-portal-customize.md). Trata los aspectos básicos de la interfaz de usuario administrativa y enumera los cambios recomendados en el contenido predeterminado. Guarde todos los cambios en el entorno local y presione **CTRL+C** para cerrar.

## <a name="step-5-publish-locally"></a>Paso 5: Publicación local

Los datos del portal se originan en forma de objetos de tipo seguro. El siguiente comando los traduce en archivos estáticos y coloca la salida en el directorio `./dist/website`:

```console
npm run publish
```

## <a name="step-6-upload-static-files-to-a-blob"></a>Paso 6: Carga de archivos estáticos en un blob

Use la CLI de Azure para cargar los archivos estáticos generados localmente en un blob y asegúrese de que los visitantes pueden acceder a ellos:

1. Abra el símbolo del sistema de Windows, PowerShell u otro shell de comandos.

1. Ejecute el siguiente comando de la CLI de Azure.
   
    Reemplace `<account-connection-string>` por la cadena de conexión de la cuenta de almacenamiento. Puede obtenerla de la sección **Claves de acceso** de la cuenta de almacenamiento.

    ```azurecli
    az storage blob upload-batch --source dist/website \
        --destination '$web' \
        --connection-string <account-connection-string>
    ```


## <a name="step-7-go-to-your-website"></a>Paso 7: Acceso al sitio web

El sitio web ahora está activo con el nombre de host especificado en las propiedades de Azure Storage (**Punto de conexión principal** en **Sitios web estáticos**).

## <a name="step-8-change-api-management-notification-templates"></a>Paso 8: Cambio de las plantillas de notificación de API Management

Reemplace la dirección URL del portal para desarrolladores en las plantillas de notificación de API Management para que apunten al portal autohospedado. Consulte [Configuración de notificaciones y plantillas de correo electrónico en Azure API Management](api-management-howto-configure-notifications.md).

En concreto, lleve a cabo los siguientes cambios en las plantillas predeterminadas:

> [!NOTE]
> Los valores en las siguientes secciones **Actualizado** suponen que el portal se hospeda en **https:\//portal.contoso.com/** . 

### <a name="email-change-confirmation"></a>Confirmación de cambio de correo electrónico

Actualice la dirección URL del portal para desarrolladores en la plantilla de notificación **Confirmación de cambio de correo electrónico**:

**Contenido original**

```html
<a id="confirmUrl" href="$ConfirmUrl" style="text-decoration:none">
  <strong>$ConfirmUrl</strong></a>
```

**Updated**

```html
<a id="confirmUrl" href="https://portal.contoso.com/signup?$ConfirmQuery" style="text-decoration:none">
  <strong>https://portal.contoso.com/signup?$ConfirmQuery</strong></a>
```

### <a name="new-developer-account-confirmation"></a>Confirmación de nueva cuenta de desarrollador

Actualice la dirección URL del portal para desarrolladores en la plantilla de notificación **Confirmación de nueva cuenta de desarrollador**:

**Contenido original**

```html
<a id="confirmUrl" href="$ConfirmUrl" style="text-decoration:none">
  <strong>$ConfirmUrl</strong></a>
```

**Updated**

```html
<a id="confirmUrl" href="https://portal.contoso.com/signup?$ConfirmQuery" style="text-decoration:none">
  <strong>https://portal.contoso.com/signup?$ConfirmQuery</strong></a>
```

### <a name="invite-user"></a>Invitación a un usuario

Actualice la dirección URL del portal para desarrolladores en la plantilla de notificación **Invitar usuario**:

**Contenido original**

```html
<a href="$ConfirmUrl">$ConfirmUrl</a>
```

**Updated**

```html
<a href="https://portal.contoso.com/confirm-v2/identities/basic/invite?$ConfirmQuery">https://portal.contoso.com/confirm-v2/identities/basic/invite?$ConfirmQuery</a>
```

### <a name="new-subscription-activated"></a>Nueva suscripción activada

Actualice la dirección URL del portal para desarrolladores en la plantilla de notificación **Nueva suscripción activada**:

**Contenido original**

```html
Thank you for subscribing to the <a href="http://$DevPortalUrl/products/$ProdId"><strong>$ProdName</strong></a> and welcome to the $OrganizationName developer community. We are delighted to have you as part of the team and are looking forward to the amazing applications you will build using our API!
```

**Updated**

```html
Thank you for subscribing to the <a href="https://portal.contoso.com/product#product=$ProdId"><strong>$ProdName</strong></a> and welcome to the $OrganizationName developer community. We are delighted to have you as part of the team and are looking forward to the amazing applications you will build using our API!
```

**Contenido original**

```html
Visit the developer <a href="http://$DevPortalUrl/developer">profile area</a> to manage your subscription and subscription keys
```

**Updated**

```html
Visit the developer <a href="https://portal.contoso.com/profile">profile area</a> to manage your subscription and subscription keys
```

**Contenido original**

```html
<a href="http://$DevPortalUrl/docs/services?product=$ProdId">Learn about the API</a>
```

**Updated**

```html
<a href="https://portal.contoso.com/product#product=$ProdId">Learn about the API</a>
```

**Contenido original**

```html
<p style="font-size:12pt;font-family:'Segoe UI'">
  <strong>
    <a href="http://$DevPortalUrl/applications">Feature your app in the app gallery</a>
  </strong>
</p>
<p style="font-size:12pt;font-family:'Segoe UI'">You can publish your application on our gallery for increased visibility to potential new users.</p>
<p style="font-size:12pt;font-family:'Segoe UI'">
  <strong>
    <a href="http://$DevPortalUrl/issues">Stay in touch</a>
  </strong>
</p>
<p style="font-size:12pt;font-family:'Segoe UI'">
      If you have an issue, a question, a suggestion, a request, or if you just want to tell us something, go to the <a href="http://$DevPortalUrl/issues">Issues</a> page on the developer portal and create a new topic.
</p>
```

**Updated**

```html
<!--Remove the entire block of HTML code above.-->
```

### <a name="password-change-confirmation"></a>Confirmación de cambio de contraseña

Actualice la dirección URL del portal para desarrolladores en la plantilla de notificación **Confirmación de cambio de contraseña**:

**Contenido original**

```html
<a href="$DevPortalUrl">$DevPortalUrl</a>
```

**Updated**

```html
<a href="https://portal.contoso.com/confirm-password?$ConfirmQuery">https://portal.contoso.com/confirm-password?$ConfirmQuery</a>
```

### <a name="all-templates"></a>Todas las plantillas

Actualice la dirección URL del portal para desarrolladores en cualquier plantilla que tenga un vínculo en el pie de página:

**Contenido original**

```html
<a href="$DevPortalUrl">$DevPortalUrl</a>
```

**Updated**

```html
<a href="https://portal.contoso.com/">https://portal.contoso.com/</a>
```

## <a name="move-from-managed-to-self-hosted-developer-portal"></a>Pasaje del portal para desarrolladores administrado a autohospedado

Con el tiempo, es posible que los requisitos empresariales cambien. Puede acabar en una situación en la que la versión administrada del portal para desarrolladores de API Management ya no satisfaga sus necesidades. Por ejemplo, un nuevo requisito puede obligarle a crear un widget personalizado que se integre en un proveedor de datos externo. A diferencia de la versión administrada, la versión autohospedada del portal ofrece total flexibilidad y extensibilidad.

### <a name="transition-process"></a>Proceso de transición

Puede realizar la transición de la versión administrada a una versión autohospedada dentro de la misma instancia de servicio de API Management. El proceso conserva las modificaciones que ha hecho en la versión administrada del portal. Asegúrese de hacer una copia de seguridad del contenido del portal de antemano. Puede encontrar el script de copia de seguridad en la carpeta `scripts` en el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) del portal para desarrolladores de API Management.

El proceso de conversión es casi idéntico a la configuración de un portal autohospedado genérico, como se muestra en los pasos anteriores de este artículo. Hay una excepción en el paso de configuración. La cuenta de almacenamiento del archivo `config.design.json` debe ser la misma que la cuenta de almacenamiento de la versión administrada del portal. Consulte [Tutorial: Uso de una identidad asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage con las credenciales de SAS](../active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-storage-sas.md#get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls) para obtener instrucciones sobre cómo recuperar la dirección URL de SAS.

> [!TIP]
> Se recomienda usar una cuenta de almacenamiento independiente en el archivo `config.publish.json`. Este enfoque le da más control y simplifica la administración del servicio de hospedaje del portal.

## <a name="enable-captcha"></a>Habilitación de CAPTCHA

Al configurar el portal autohospedado, es posible que haya deshabilitado CAPTCHA desde el valor `useHipCaptcha`. La comunicación con CAPTCHA se produce a través de un punto de conexión, lo que permite que el uso compartido de recursos entre orígenes (CORS) solo tenga lugar para el nombre de host del portal para desarrolladores administrado. Si el portal para desarrolladores está autohospedado, usa un nombre de host diferente, y CAPTCHA no permitirá la comunicación.

### <a name="update-the-json-config-files"></a>Actualización de los archivos de configuración JSON

Para habilitar CAPTCHA en el portal autohospedado:

1. Asigne un dominio personalizado (por ejemplo, `api.contoso.com`) al punto de conexión del **portal para desarrolladores** del servicio API Management.

    Este dominio se aplica a la versión administrada del portal y al punto de conexión de CAPTCHA. Para conocer los pasos, consulte [Configuración de un nombre de dominio personalizado para la instancia de Azure API Management](configure-custom-domain.md).

1. Vaya a la carpeta `src` del [entorno local](#step-1-set-up-local-environment) del portal autohospedado.

1. Actualice los archivos de configuración `json`:

    | Archivo | Valor nuevo | Nota |
    | ---- | --------- | ---- |
    | `config.design.json`| `"backendUrl": "https://<custom-domain>"` | Reemplace `<custom-domain>` por el dominio personalizado que configuró en el primer paso. |
    |  | `"useHipCaptcha": true` | Cambie el valor a `true`. |
    | `config.publish.json`| `"backendUrl": "https://<custom-domain>"` | Reemplace `<custom-domain>` por el dominio personalizado que configuró en el primer paso. |
    |  | `"useHipCaptcha": true` | Cambie el valor a `true`. |
    | `config.runtime.json` | `"backendUrl": "https://<custom-domain>"` | Reemplace `<custom-domain>` por el dominio personalizado que configuró en el primer paso. |

1. [Publique](#step-5-publish-locally) el portal

1. [Cargue](#step-6-upload-static-files-to-a-blob) y hospede el portal recién publicado.

1. Exponga el portal autohospedado a través de un dominio personalizado.

Los niveles primero y segundo del dominio del portal deben coincidir con el dominio configurado en el primer paso. Por ejemplo, `portal.contoso.com`. Los pasos exactos dependen de la plataforma de hospedaje que elija. Si ha usado una cuenta de almacenamiento de Azure, puede consultar [Asignación de un dominio personalizado a un punto de conexión de Azure Blob Storage](../storage/blobs/storage-custom-domain-name.md) para obtener instrucciones.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [enfoques alternativos al autohospedaje](developer-portal-alternative-processes-self-host.md)
