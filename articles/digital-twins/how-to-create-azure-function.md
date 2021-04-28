---
title: Configuración de una función de Azure para procesar datos
titleSuffix: Azure Digital Twins
description: Consulte cómo crear una función de Azure a la que se pueda acceder y que se desencadene mediante gemelos digitales.
author: baanders
ms.author: baanders
ms.date: 8/27/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5dddaabf47a261f741b3b1cb8d3319d589c4e474
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107480768"
---
# <a name="connect-function-apps-in-azure-for-processing-data"></a>Configuración de aplicaciones de funciones de Azure para procesar datos

Los gemelos digitales se pueden actualizar en función de los datos mediante [rutas de eventos](concepts-route-events.md) con recursos de proceso. Por ejemplo, una función que se haya realizado mediante [Azure Functions](../azure-functions/functions-overview.md) puede actualizar un gemelo digital en respuesta a:
* Datos de telemetría de dispositivos procedentes de Azure IoT Hub.
* Un cambio de propiedad u otros datos procedentes de otro gemelo digital dentro del grafo gemelo.

Este artículo se le mostrará cómo crear una función de Azure para su utilización con Azure Digital Twins. Para crear una función, deberá seguir estos pasos básicos:

1. Cree un proyecto de Azure Functions en Visual Studio.
2. Escriba una función que tenga un desencadenador de [Azure Event Grid](../event-grid/overview.md).
3. Agregue código de autenticación a la función para que pueda acceder a Azure Digital Twins.
4. Publique la aplicación de funciones en Azure.
5. Configure la [seguridad](concepts-security.md) para la aplicación de funciones.

## <a name="prerequisite-set-up-azure-digital-twins"></a>Requisito previo: configuración de Azure Digital Twins

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="create-a-function-app-in-visual-studio"></a>Creación de una aplicación de funciones en Visual Studio

En Visual Studio 2019, selecciona **Archivo** > **Nuevo** > **Proyecto**. Busque la plantilla de **Azure Functions**. Seleccione **Next** (Siguiente).

:::image type="content" source="media/how-to-create-azure-function/create-azure-function-project.png" alt-text="Captura de pantalla de Visual Studio que muestra el cuadro de diálogo Nuevo proyecto. La plantilla de proyecto de Azure Functions está resaltada.":::

Especifique un nombre para la aplicación de funciones y, después, seleccione __Crear__.

:::image type="content" source="media/how-to-create-azure-function/configure-new-project.png" alt-text="Captura de pantalla de Visual Studio que muestra el cuadro de diálogo para configurar un nuevo proyecto. La configuración incluye el nombre del proyecto, la ubicación de almacenamiento, la opción para crear una nueva solución y el nombre de la solución.":::

Seleccione el tipo de aplicación de funciones **Desencadenador de Event Grid** y, después, seleccione __Crear__.

:::image type="content" source="media/how-to-create-azure-function/event-grid-trigger-function.png" alt-text="Captura de pantalla de Visual Studio que muestra el cuadro de diálogo para crear una nueva aplicación de Azure Functions. La opción Desencadenador de Event Grid está resaltada.":::

Después de crear la aplicación de funciones, Visual Studio genera un código de ejemplo en un archivo *Function1.cs* en la carpeta del proyecto. Esta breve función se usa para registrar eventos.

:::image type="content" source="media/how-to-create-azure-function/visual-studio-sample-code.png" alt-text="Captura de pantalla de Visual Studio. Se muestra la ventana del proyecto nuevo. Se muestra el código de una función de ejemplo en un archivo denominado Function1." lightbox="media/how-to-create-azure-function/visual-studio-sample-code.png":::

## <a name="write-a-function-that-has-an-event-grid-trigger"></a>Escritura de una función que tenga un desencadenador de Event Grid

Para escribir una función, agregue un SDK a la aplicación de funciones. La aplicación de funciones interactúa con Azure Digital Twins con el [SDK de Azure Digital Twins para .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client). 

Para usar el SDK, debe incluir los siguientes paquetes en el proyecto. Instale los paquetes mediante el administrador de paquetes NuGet de Visual Studio. O bien, agregue los paquetes mediante `dotnet` en una herramienta de línea de comandos.

* [Azure.DigitalTwins.Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core/)
* [Azure.Identity](https://www.nuget.org/packages/Azure.Identity/)
* [System.Net.Http](https://www.nuget.org/packages/System.Net.Http/)
* [Azure.Core](https://www.nuget.org/packages/Azure.Core/)

A continuación, en el Explorador de soluciones de Visual Studio, abra el archivo _Function1.cs_, que incluye el código de ejemplo. Agregue las siguientes instrucciones `using` para los paquetes.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="Function_dependencies":::

## <a name="add-authentication-code-to-the-function"></a>Incorporación de código de autenticación a la función

Ahora declare las variables de nivel de clase y agregue un código de autenticación que permitirá a la función acceder a Azure Digital Twins. Agregue las variables y el código a la función en el archivo _Function1.cs_.

* **Código para leer la URL del servicio Azure Digital Twins como una variable de entorno.** Se recomienda leer la dirección URL del servicio en una variable de entorno, en lugar de codificarla de forma rígida en la función. Establecerá el valor de esta variable de entorno [más adelante en este artículo](#set-up-security-access-for-the-function-app). Para más información sobre las variables de entorno, consulte [Administración de la aplicación de funciones](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal).

    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="ADT_service_URL":::

* **Una variable estática para contener una instancia de HttpClient.** HttpClient es relativamente costoso de crear, por lo que queremos evitar crearlo para cada invocación de función.

    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="HTTP_client":::

* **Credenciales de identidad administrada.**
    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="ManagedIdentityCredential":::

* **Una variable local _DigitalTwinsClient_.** Agregue una variable dentro de la función para que contenga la instancia del cliente de Azure Digital Twins. *No* haga que esta variable sea estática dentro de la clase.
    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs" id="DigitalTwinsClient":::

* **Una comprobación de NULL para _adtInstanceUrl_.** Agregue una comprobación de NULL y, a continuación, encapsule la lógica de la función en un bloque try/catch para detectar cualquier excepción.

Después de estos cambios, el código de la función tendrá un aspecto similar al del siguiente ejemplo.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/adtIngestFunctionSample.cs":::

Ahora que la aplicación está escrita, puede publicarla en Azure.

## <a name="publish-the-function-app-to-azure"></a>Publicación de la aplicación de funciones en Azure

[!INCLUDE [digital-twins-publish-azure-function.md](../../includes/digital-twins-publish-azure-function.md)]

### <a name="verify-the-publication-of-your-function"></a>Comprobación de la publicación de la función

1. Use sus credenciales para iniciar sesión en [Azure Portal](https://portal.azure.com/).
2. En el cuadro de búsqueda de la parte superior de la ventana, busque el nombre de la aplicación de funciones y selecciónelo.

    :::image type="content" source="media/how-to-create-azure-function/search-function-app.png" alt-text="Captura de pantalla que muestra Azure Portal. En el campo de búsqueda, escriba el nombre de la aplicación de funciones." lightbox="media/how-to-create-azure-function/search-function-app.png":::

3. En la página **Aplicación de funciones** que se abrirá, en el menú de la izquierda, elija **Funciones**. Si la función se ha publicado correctamente, su nombre aparecerá en la lista.

    > [!Note] 
    > Es posible que tenga que esperar unos minutos o actualizar la página un par de veces antes de que se muestre la función en la lista de funciones publicadas.

    :::image type="content" source="media/how-to-create-azure-function/view-published-functions.png" alt-text="Visualización de las funciones publicadas en Azure Portal." lightbox="media/how-to-create-azure-function/view-published-functions.png":::

Para acceder a Azure Digital Twins, la aplicación de funciones necesita una identidad administrada por el sistema con permisos para acceder a la instancia de Azure Digital Twins. Va a realizar esa configuración a continuación.

## <a name="set-up-security-access-for-the-function-app"></a>Configuración del acceso de seguridad para la aplicación de funciones

Puede configurar el acceso de seguridad para la aplicación de funciones mediante la CLI de Azure o Azure Portal. Siga los pasos de su opción preferida.

# <a name="cli"></a>[CLI](#tab/cli)

Ejecute estos comandos en [Azure Cloud Shell](https://shell.azure.com) o en una [instalación local de la CLI de Azure](/cli/azure/install-azure-cli).
Puede usar la identidad administrada por el sistema de la aplicación de funciones para asignarle el rol **Propietario de datos de Azure Digital Twins** para la instancia de Azure Digital Twins. Este role le concede a la aplicación de funciones el permiso en la instancia para realizar actividades del plano de datos. A continuación, permita que la función acceda a la dirección URL de la instancia mediante la definición de una variable de entorno.

### <a name="assign-an-access-role"></a>Asignación de un rol de acceso

[!INCLUDE [digital-twins-permissions-required.md](../../includes/digital-twins-permissions-required.md)]

El esqueleto de la función en los ejemplos anteriores requiere que se le pase un token de portador. Si no se pasa un token de portador, la aplicación de funciones no puede autenticarse con Azure Digital Twins. 

Para asegurarse de que se pase el token de portador, configure los permisos de [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) para que la aplicación de funciones pueda acceder a Azure Digital Twins. Estos permisos se configuran solo una vez para cada aplicación de funciones.


1. Use el siguiente comando para ver los detalles de la identidad administrada por el sistema de la función. Anote el valor del campo `principalId` de la salida.

    ```azurecli-interactive 
    az functionapp identity show -g <your-resource-group> -n <your-App-Service-(function-app)-name> 
    ```

    >[!NOTE]
    > Si el resultado está vacío en lugar de mostrar los detalles de la identidad, cree una nueva identidad administrada por el sistema para la función con este comando:
    > 
    >```azurecli-interactive    
    >az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>  
    >```
    >
    > La salida muestra los detalles de la identidad, incluido el valor de `principalId` requerido para el siguiente paso. 

1. Use el valor `principalId` en el siguiente comando para asignar la identidad de la aplicación de funciones al rol _Propietario de datos de Azure Digital Twins_  de la instancia de Azure Digital Twins.

    ```azurecli-interactive 
    az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Data Owner"
    ```

### <a name="configure-application-settings"></a>Configuración de la aplicación

Permita que la función acceda a la dirección URL de la instancia mediante la definición de una variable de entorno para este fin. Para más información sobre las variables de entorno, consulte [Administración de la aplicación de funciones](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). 

> [!TIP]
> Para crear la dirección URL de la instancia de Azure Digital Twins, agregue *https://* al principio del nombre de host de la instancia. Para ver el nombre de host, junto con todas las propiedades de la instancia, ejecute `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

```azurecli-interactive 
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-host-name>"
```

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Realice los pasos siguientes en [Azure Portal](https://portal.azure.com/).

### <a name="assign-an-access-role"></a>Asignación de un rol de acceso

[!INCLUDE [digital-twins-permissions-required.md](../../includes/digital-twins-permissions-required.md)]

Una identidad administrada asignada por el sistema permite que los recursos de Azure se autentiquen en servicios en la nube (por ejemplo, Azure Key Vault) sin almacenar las credenciales en el código. Después de habilitar la identidad administrada asignada por el sistema, se pueden conceder todos los permisos necesarios a través del control de acceso basado en rol de Azure. 

El ciclo de vida de este tipo de identidad administrada está ligado al ciclo de vida de este recurso. Además, cada recurso puede tener solo una identidad administrada asignada por el sistema.

1. Busque la aplicación de funciones en [Azure Portal](https://portal.azure.com/); para ello, escriba su nombre en el cuadro de búsqueda. Seleccione la aplicación en los resultados. 

    :::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Captura de pantalla de Azure Portal.El nombre de la aplicación de funciones aparece en la barra de búsqueda del portal y el resultado de la búsqueda está resaltado.":::

1. En la página de la aplicación de funciones, en el menú de la izquierda, seleccione __Identidad__ para trabajar con una identidad administrada para la función. En la página __Asignado por el sistema__, compruebe que el __Estado__ está establecido en **Activado**. Si no es así, establézcalo ahora y **Guarde** el cambio.

    :::image type="content" source="media/how-to-create-azure-function/verify-system-managed-identity.png" alt-text="Captura de pantalla de Azure Portal. En la página Identidad de la aplicación de funciones, la opción Estado está establecida en Activado." lightbox="media/how-to-create-azure-function/verify-system-managed-identity.png":::

1. Seleccione __Asignaciones de roles de Azure__.

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-1.png" alt-text="Captura de pantalla del Azure Portal. En la página Identidad de la función de Azure, en Permisos, está resaltado el botón Asignaciones de roles de Azure." lightbox="media/how-to-create-azure-function/add-role-assignment-1.png":::

    Seleccione __+ Agregar asignación de roles (versión preliminar)__ .

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-2.png" alt-text="Captura de pantalla de Azure Portal. En la página Asignaciones de roles de Azure, el botón Agregar asignación de roles está resaltado." lightbox="media/how-to-create-azure-function/add-role-assignment-2.png":::

1. En la página __Agregar asignación de roles (versión preliminar)__ , seleccione los siguientes valores:

    * **Ámbito**: _Grupo de recursos_.
    * **Suscripción**: Seleccione su suscripción a Azure.
    * **Grupo de recursos**: seleccione el grupo de recursos.
    * **Rol**: _Propietario de datos de Azure Digital Twins_.

    Seleccione __Guardar__ para guardar los detalles.

    :::image type="content" source="media/how-to-create-azure-function/add-role-assignment-3.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo agregar una nueva asignación de roles. El cuadro de diálogo muestra los campos Ámbito, Suscripción, Grupo de recursos y Rol.":::

### <a name="configure-application-settings"></a>Configuración de la aplicación

Para hacer que la dirección URL de la instancia de Azure Digital Twins sea accesible para la función, establezca una variable de entorno. La configuración de la aplicación se expone como variables de entorno que permiten acceder a la instancia de Azure Digital Twins. Para más información sobre las variables de entorno, consulte [Administración de la aplicación de funciones](../azure-functions/functions-how-to-use-azure-function-app-settings.md?tabs=portal). 

Para establecer una variable de entorno con la dirección URL de la instancia, busque primero el nombre de host de la instancia: 

1. Busque la instancia en [Azure Portal](https://portal.azure.com). 
1. En el menú de la izquierda, seleccione __Información general__. 
1. Copie el valor de __Nombre de host__.

    :::image type="content" source="media/how-to-create-azure-function/instance-host-name.png" alt-text="Captura de pantalla de Azure Portal. Está resaltado el valor Nombre de host en la página Información general de la instancia.":::

Ahora puede crear una configuración de la aplicación:

1. En la barra de búsqueda del portal, busque la aplicación de funciones y selecciónela en los resultados.

    :::image type="content" source="media/how-to-create-azure-function/portal-search-for-function-app.png" alt-text="Captura de pantalla de Azure Portal. Se está buscando el nombre de la aplicación de funciones en la barra de búsqueda del portal. El resultado de la búsqueda está resaltado.":::

1. En el lado izquierdo, seleccione __Configuración__. Después, en la pestaña __Configuración de la aplicación__, seleccione __+ Nueva configuración de la aplicación__.

    :::image type="content" source="media/how-to-create-azure-function/application-setting.png" alt-text="Captura de pantalla de Azure Portal. En la pestaña Configuración de la aplicación de funciones, el botón para crear una nueva configuración de aplicación está resaltado.":::

1. En la ventana que se abre, use el valor de nombre de host que copió para crear una configuración de la aplicación.
    * **Nombre**: ADT_SERVICE_URL
    * **Valor**: https://{nombre-de-host-de-azure-digital-twins}
    
    Seleccione __Aceptar__ para crear una configuración de la aplicación.
    
    :::image type="content" source="media/how-to-create-azure-function/add-application-setting.png" alt-text="Captura de pantalla de Azure Portal. En la página Agregar o editar configuración de la aplicación, aparecen rellenados los campos Nombre y Valor. El botón Aceptar está resaltado.":::

1. Después de crear la configuración, debería aparecer en la pestaña __Configuración de la aplicación__. Compruebe que **ADT_SERVICE_URL** aparezca en la lista. A continuación, seleccione __Guardar__ para guardar la nueva configuración de la aplicación.

    :::image type="content" source="media/how-to-create-azure-function/application-setting-save-details.png" alt-text="Captura de pantalla de Azure Portal. En la pestaña Configuración de la aplicación, está resaltada la nueva configuración ADT_SERVICE_URL. El botón Guardar también está resaltado.":::

1. Cualquier cambio en la configuración de la aplicación requiere un reinicio de la aplicación; por lo tanto, seleccione __Continuar__ para reiniciar la aplicación cuando se le solicite.

    :::image type="content" source="media/how-to-create-azure-function/save-application-setting.png" alt-text="Captura de pantalla de Azure Portal. Una nota indica que los cambios realizados en la configuración de la aplicación reiniciarán la aplicación. El botón Continuar está resaltado.":::

---

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha configurado una aplicación de funciones de Azure para usarla con Azure Digital Twins. A continuación, consulte cómo crear una función básica para [ingerir datos de IoT Hub en Azure Digital Twins](how-to-ingest-iot-hub-data.md).
