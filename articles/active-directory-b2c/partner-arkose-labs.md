---
title: Tutorial para configurar Azure Active Directory B2C con Arkose Labs
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con Arkose Labs para identificar una cuenta peligrosa y fraudulenta
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/18/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: c10a39b050fa66192f762ba642b4c8ac2e080250
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258150"
---
# <a name="tutorial-configure-arkose-labs-with-azure-active-directory-b2c"></a>Tutorial: Configuración de Arkose Labs con Azure Active Directory B2C

En este tutorial de ejemplo, aprenderá a integrar la autenticación de Azure Active Directory (AD) B2C con [Arkose Labs](https://www.arkoselabs.com/). Arkose Labs ayuda a las organizaciones contra ataques de bots, ataques de adquisición de cuentas y aperturas de cuentas fraudulentas.  

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- Una cuenta de [Arkose Labs](https://www.arkoselabs.com/book-a-demo/).

## <a name="scenario-description"></a>Descripción del escenario

La integración de Arkose Labs incluye los siguientes componentes:

- **Arkose Labs**: un servicio de fraude y abuso para la protección contra bots y otro abuso automatizado.

- **Flujo de usuario de registro de Azure AD B2C**: la experiencia de registro que utilizará el servicio Arkose Labs. Usará JavaScript y HTML personalizados, así como conectores de API para integrarse con el servicio de Arkose Labs.

- **Funciones de Azure**: punto de conexión de la API hospedado por el usuario que funciona con la característica de conectores de API. Esta API es responsable de realizar la validación del lado del servidor del token de sesión de Arkose Labs.

En el diagrama siguiente se describe cómo Arkose Labs se integra con Azure AD B2C.

![Imagen que muestra el diagrama de arquitectura de Arkose Labs](media/partner-arkose-labs/arkose-labs-architecture-diagram.png)

| Paso  | Descripción |
|---|---|
|1     | Un usuario se registra y crea una cuenta. Cuando el usuario selecciona Enviar, aparece un desafío de cumplimiento de Arkose Labs.         |
|2     |  Una vez que el usuario completa el desafío, Azure AD B2C envía el estado a Arkose Labs para generar un token. |
|3     |  Arkose Labs genera un token y lo envía de vuelta a Azure AD B2C.   |
|4     |  Azure AD B2C llama a una API web intermedia para pasar el formulario de registro.      |
|5     |  La API web intermedia envía el formulario de registro a Arkose Lab para la comprobación de tokens.    |
|6     | Arkose Lab procesa y envía los resultados de la comprobación de nuevo a la API web intermedia.|
|7     | La API web intermedia envía el resultado correcto o incorrecto del desafío a Azure AD B2C. |
|8     | Si el desafío se completa correctamente, se envía un formulario de registro a Azure AD B2C, y Azure AD B2C completa la autenticación.|

## <a name="onboard-with-arkose-labs"></a>Incorporación con Arkose Labs

1. Póngase en contacto con [Arkose](https://www.arkoselabs.com/book-a-demo/) y cree una cuenta.

2. Una vez creada la cuenta, vaya a https://dashboard.arkoselabs.com/login.  

3. En el panel, vaya a la configuración del sitio para buscar la clave pública y la clave privada. Esta información se necesitará más adelante para configurar Azure AD B2C. Los valores de las claves públicas y privadas se conocen como `ARKOSE_PUBLIC_KEY` y `ARKOSE_PRIVATE_KEY` en el [código de ejemplo](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose).

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

### <a name="part-1--create-a-arkosesessiontoken-custom-attribute"></a>Parte 1: Creación de un atributo personalizado ArkoseSessionToken

Para crear un atributo personalizado, siga estos pasos.  

1. Vaya a **Azure Portal** > **Azure AD B2C**.

2. Seleccione **Atributos de usuario**

3. Seleccione **Agregar**.

4. Escriba **ArkoseSessionToken** como nombre de atributo.

5. Seleccione **Crear**

Más información sobre los [atributos personalizados](./user-flow-custom-attributes.md?pivots=b2c-user-flow).

### <a name="part-2---create-a-user-flow"></a>Parte 2: Creación de un flujo de usuario

El flujo de usuario puede ser para el **registro** y el **inicio de sesión** o solo para el **registro**. El flujo de usuario de Arkose Labs solo se mostrará durante el registro.

1. Consulte las [instrucciones](./tutorial-create-user-flows.md) para crear un flujo de usuario. Si usa un flujo de usuario existente, debe ser del tipo de versión **recomendado (siguiente versión preliminar)** .

2. En la configuración del flujo de usuario, vaya a **Atributos de usuario** y seleccione la notificaciones de **ArkoseSessionToken**.

![Imagen que muestra cómo seleccionar atributos personalizados](media/partner-arkose-labs/select-custom-attribute.png)

### <a name="part-3---configure-custom-html-javascript-and-page-layouts"></a>Parte 3: Configuración de diseños de página, HTML y JavaScript personalizados

Vaya al [script HTML](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose/blob/main/Assets/selfAsserted.html) proporcionado. El archivo contiene una plantilla HTML con etiquetas `<script>` de JavaScript que hará tres cosas:

1. Cargue el script de Arkose Labs, que representa el widget de Arkose Labs y realiza la validación de Arkose Labs del lado cliente.

2. Oculte el elemento de entrada y la etiqueta `extension_ArkoseSessionToken`, correspondientes al atributo personalizado `ArkoseSessionToken`, de la interfaz de usuario que se muestra al usuario.

3. Cuando un usuario completa el desafío de Arkose Labs, este comprueba la respuesta del usuario y genera un token. La devolución de llamada `arkoseCallback` en JavaScript personalizado establece el valor de `extension_ArkoseSessionToken` en el valor del token generado. Este valor se enviará al punto de conexión de la API, tal y como se describe en la sección siguiente.

    Consulte [este artículo](https://arkoselabs.atlassian.net/wiki/spaces/DG/pages/214176229/Standard+Setup) para obtener información sobre la validación del lado cliente de Arkose Labs.

Siga los pasos mencionados para usar el HTML y JavaScript personalizados para el flujo de usuario.

1. Modifique el archivo [selfAsserted.html](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose/blob/main/Assets/selfAsserted.html) para que `<ARKOSE_PUBLIC_KEY>` coincida con el valor generado para la validación del lado cliente y que se use para cargar el script de Arkose Labs para su cuenta.

2. Hospede la página HTML en un punto de conexión web habilitado para el uso compartido de recursos entre orígenes (CORS). [Cree una cuenta de almacenamiento de blobs de Azure](../storage/common/storage-account-create.md?tabs=azure-portal&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) y [configure CORS](/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services).

  >[!NOTE]
  >Si tiene su propio HTML personalizado, copie y pegue los elementos `<script>` en la página HTML.

3. Siga estos pasos para configurar los diseños de página.

   a. En Azure Portal, vaya a **Azure AD B2C**.

   b. Seleccione **Flujos de usuario** y elija el flujo de usuario.

   c. Seleccione **Diseños de página**.

   d. Seleccione el diseño **Página de registro de cuenta local**.

   e. Cambie **Usar contenido de la página personalizada** a **SÍ**.

   f. Pegue el URI donde reside el HTML personalizado en **Usar contenido de la página personalizada**.

   g. Si usa proveedores de identidades sociales, repita los **pasos e** y **f** para el diseño **Página de registro de la cuenta de red social**.

   ![Imagen que muestra los diseños de página](media/partner-arkose-labs/page-layouts.png)

4. En el flujo de usuario, vaya a **Propiedades** y seleccione **Habilitar JavaScript** para aplicar el diseño de página (versión preliminar). Consulte este [artículo](./javascript-and-page-layout.md?pivots=b2c-user-flow) para más información.

### <a name="part-4---create-and-deploy-your-api"></a>Parte 4: Creación e implementación de la API

Instale la [extensión de Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) para Visual Studio Code.

>[!Note]
>En los pasos que se mencionan en esta sección se da por supuesto que está usando Visual Studio Code para implementar la función de Azure. También puede usar Azure Portal, el terminal o el símbolo del sistema o cualquier otro editor de código para implementarla.

#### <a name="run-the-api-locally"></a>Ejecución local de la API

1. Vaya a la extensión de Azure en Visual Studio Code en la barra de navegación izquierda. Seleccione la carpeta del **proyecto local** que representa la función de Azure local.

2. Presione **F5** o use el menú **Depurar** > **Iniciar depuración** para iniciar el depurador y adjuntarlo al host de Azure Functions. Este comando usa automáticamente la configuración de depuración única que la función de Azure ha creado.

3. La extensión de la función de Azure generará automáticamente algunos archivos para el desarrollo local, instalará las dependencias y las herramientas básicas para funciones si no están ya instaladas. Estas herramientas ayudan en la experiencia de depuración.

4. La salida de la herramienta básica para funciones aparece en el panel **Terminal** Visual Studio Code. Una vez iniciado el host, presione **Alt+clic** en la dirección URL local que se muestra en la salida para abrir el explorador y ejecutar la función. En el explorador de Azure Functions, haga clic con el botón derecho en la función para ver la dirección URL de la función hospedada localmente.

Para volver a implementar la instancia local durante las pruebas, repita los pasos 1 a 4.

#### <a name="add-environment-variables"></a>Adición de las variables de entorno

En este ejemplo se protege el punto de conexión de la API web mediante la [autenticación básica de HTTPc](https://tools.ietf.org/html/rfc7617).

El nombre de usuario y la contraseña se almacenan como variables de entorno y no como parte del repositorio. Consulte el archivo [local.settings.json](../azure-functions/functions-run-local.md?tabs=macos%2ccsharp%2cbash#local-settings-file) para obtener más información.

1. Creación de un archivo local.settings.json en la carpeta raíz

2. Copie y pegue el código siguiente en el archivo:

```
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "BASIC_AUTH_USERNAME": "<USERNAME>",
    "BASIC_AUTH_PASSWORD": "<PASSWORD>",
    "ARKOSE_PRIVATE_KEY": "<ARKOSE_PRIVATE_KEY>",
    "B2C_EXTENSIONS_APP_ID": "<B2C_EXTENSIONS_APP_ID>"
  }
}
```
Los valores **BASIC_AUTH_USERNAME** y **BASIC_AUTH_PASSWORD** serán las credenciales usadas para autenticar la llamada API en la función de Azure. Elija los valores deseados.

`<ARKOSE_PRIVATE_KEY>` es el secreto del lado del servidor que generó en el servicio Arkose Labs. Se usa para llamar a la [API de validación del lado del servidor de Arkose Labs](https://arkoselabs.atlassian.net/wiki/spaces/DG/pages/266214758/Server-Side+Instructions) para validar el valor de `ArkoseSessionToken` generado por el front-end.

`<B2C_EXTENSIONS_APP_ID>` es el identificador de la aplicación que usa Azure AD B2C para almacenar los atributos personalizados en el directorio. Para encontrar este identificador de aplicación, vaya a Registros de aplicaciones, busque b2c-extensions-app y copie el identificador de la aplicación (cliente) en el panel de **información general**. Quite los caracteres `-`.

![Imagen que muestra la búsqueda por identificador de aplicación](media/partner-arkose-labs/search-app-id.png)

#### <a name="deploy-the-application-to-the-web"></a>Implementación de la aplicación en la web

1. Siga los pasos que se mencionan en [esta](/azure/javascript/tutorial-vscode-serverless-node-04) guía para implementar la función de Azure en la nube. Copie la dirección URL web del punto de conexión de la función de Azure.

2. Una vez implementada, seleccione la opción **Configuración de carga**. Cargará las variables de entorno en la [configuración de la aplicación](../azure-functions/functions-develop-vs-code.md?tabs=csharp#application-settings-in-azure) del servicio de aplicaciones. Esta configuración de la aplicación también se puede establecer o [administrar mediante Azure Portal.](../azure-functions/functions-how-to-use-azure-function-app-settings.md)

Consulte [este artículo](../azure-functions/functions-develop-vs-code.md?tabs=csharp#republish-project-files) para más información sobre el desarrollo de Visual Studio Code para Azure Functions.

#### <a name="configure-and-enable-the-api-connector"></a>Configuración y habilitación del conector de API

[Cree un conector de API](./add-api-connector.md) y habilítelo para el flujo de usuario. La configuración del conector de API debe ser similar a la siguiente:

![Imagen que muestra cómo configurar el conector de API](media/partner-arkose-labs/configure-api-connector.png)

- **URL del punto de conexión**: es la dirección URL de la función que copió anteriormente mientras implementaba la función de Azure.

- **Nombre de usuario y contraseña**: son el nombre de usuario y la contraseña que definió anteriormente como variables de entorno.

Para habilitar el conector de API, en la configuración del **conector de API** del flujo de usuario, seleccione el conector que se va a invocar en el paso **antes de crear el usuario**. Se invocará la API cuando un usuario seleccione **Crear** en el flujo de registro. La API realizará una validación del lado del servidor del valor `ArkoseSessionToken`, que se estableció mediante la devolución de llamada del widget Arkose `arkoseCallback`.

![Imagen que muestra la habilitación del conector de API](media/partner-arkose-labs/enable-api-connector.png)

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Flujos de usuario**.

2. Seleccione el flujo de usuario que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. Aplicación: seleccione la aplicación registrada (el ejemplo es JWT).

   b. URL de respuesta: seleccione la dirección URL de redireccionamiento.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Cerrar sesión

6. Recorra el flujo de inicio de sesión.  

7. Aparecerá un rompecabezas de Arkose Labs después de seleccionar **Continuar**.

## <a name="additional-resources"></a>Recursos adicionales

- [Códigos de ejemplo](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose) para el flujo de usuario de registro de Azure AD B2C

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy)