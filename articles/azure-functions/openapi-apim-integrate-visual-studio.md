---
title: Creación de API sin servidor en Visual Studio usando Azure Functions y API Management
description: Use Visual Studio para crear una función desencadenada por HTTP junto con una definición OpenAPI, lo cual posibilita la integración de Azure API Management para que otras aplicaciones y servicios puedan llamar a su API basada en funciones sin servidor.
ms.topic: tutorial
ms.date: 05/07/2021
ms.openlocfilehash: b8c6afc64f15df3124ab4052568bf36d209d4fde
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384194"
---
# <a name="create-serverless-apis-in-visual-studio-using-azure-functions-and-api-management-integration-preview"></a>Creación de API sin servidor en Visual Studio usando Azure Functions y la integración de API Management (versión preliminar)

Las API de REST a menudo se describen con una definición de OpenAPI. Este archivo contiene información acerca de operaciones en una API y sobre cómo se deben estructurar los datos de la solicitud y la respuesta para dicha API.

En este tutorial aprenderá a:

> [!div class="checklist"]
> * crear un proyecto de función sin servidor en Visual Studio;
> * probar localmente API de función mediante la funcionalidad integrada de OpenAPI;
> * publicar un proyecto en una aplicación de funciones de Azure mediante la integración de API Management; 
> * obtener la clave de acceso para la función y establecerla en API Management;
> * descargar el archivo de definición de OpenAPI.

La función sin servidor que cree proporciona una API con la que puede determinar si resulta rentable una reparación de emergencia en una turbina eólica. Dado que tanto la aplicación de funciones como la instancia de API Management que cree usan planes de consumo, completar este tutorial tiene un coste mínimo para usted.

> [!NOTE]
> La integración de OpenAPI y API Management que se incluye en este artículo se encuentra actualmente en versión preliminar. Este método para exponer una API sin servidor solo es compatible con funciones de la biblioteca de clases de C# (.NET Core 3.1). Los entornos de ejecución de todos los demás lenguajes deben [usar la integración de Azure API Management desde el portal](functions-openapi-definition.md). 

## <a name="prerequisites"></a>Requisitos previos

+ [Visual Studio 2019](https://azure.microsoft.com/downloads/), versión 16.10 o una versión posterior. Asegúrese de seleccionar la carga de trabajo de **desarrollo de Azure** durante la instalación. 

+ Una [suscripción a Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing) que esté activa. Si no la tiene, cree una [cuenta gratuita](https://azure.microsoft.com/free/dotnet/) antes de empezar.

## <a name="create-a-functions-project"></a>Creación de un proyecto de Functions

La plantilla del proyecto de Azure Functions de Visual Studio crea un proyecto que puede publicar en una aplicación de función en Azure. También creará una función desencadenada por HTTP que permite generar el archivo de definición de OpenAPI (anteriormente archivo Swagger).

1. En el menú de Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**.

1. En **Crear un proyecto**, escriba *functions* en el cuadro de búsqueda, elija la plantilla **Azure Functions** y seleccione **Siguiente**.

1. En **Configurar el nuevo proyecto**, rellene el campo **Nombre del proyecto** (por ejemplo, `TurbineRepair`) y seleccione **Crear**. 

1. En la configuración de **Crear aplicación de Azure Functions**, use los valores de la tabla siguiente:

    | Configuración      | Valor  | Descripción                      |
    | ------------ |  ------- |----------------------------------------- |
    | **Versión de .NET** | **.NET Core 3 (LTS)** | Este valor crea un proyecto de función que usa la versión 3.x del entorno de ejecución de Azure Functions. La generación de archivos de OpenAPI solo se admite en la versión 3.x del entorno de ejecución de Functions. |
    | **Plantilla de función** | **Desencadenador HTTP con OpenAPI** | Este valor crea una función desencadenada por una solicitud HTTP, con la opción de generar un archivo de definición de OpenAPI.  |
    | **Cuenta de almacenamiento (AzureWebJobsStorage)**  | **Emulador de Storage** | Puede usar el emulador para el desarrollo local de funciones de desencadenador HTTP. Dado que una aplicación de funciones de Azure necesita una cuenta de almacenamiento, se asigna o se crea una cuando publica su proyecto en Azure. |
    | **Nivel de autorización** | **Función** | Cuando se ejecutan en Azure, los clientes deben proporcionar una clave al acceder al punto de conexión. Para obtener más información sobre las claves y la autorización, consulte [Claves de acceso de función](functions-bindings-http-webhook-trigger.md#authorization-keys). |
    
    ![Configuración de un proyecto de Azure Functions](./media/openapi-apim-integrate-vs/functions-project-settings.png)

1. Seleccione **Crear** para crear el proyecto de función y la función de desencadenador HTTP, con la opción de usar OpenAPI. 

Visual Studio crea un proyecto y una clase de nombre `Function1`, que contiene código reutilizable para el tipo de función de desencadenador HTTP. A continuación, reemplace este código de plantilla de función por su propio código personalizado. 

## <a name="update-the-function-code"></a>Actualización del código de la función

La función usa un desencadenador HTTP que toma dos parámetros:

| Nombre de parámetro | Descripción|
| ---- | ---- |
| *hours* | Tiempo estimado para reparar una turbina, redondeado a la hora completa más cercana. |
| *capacidad* | La capacidad de la turbina, en kilovatios. |

A continuación, la función calcula el coste de la reparación y los ingresos que podría generar la turbina en un período de 24 horas. Los parámetros se proporcionan en la cadena de consulta o en la carga de una solicitud POST. 

En el archivo de proyecto Function1.cs, reemplace el contenido del código de biblioteca de clases generado por el siguiente código:

:::code language="csharp" source="~/functions-openapi-turbine-repair/TurbineRepair/Function1.cs":::

Este código de función devuelve un mensaje de `Yes` o `No` para indicar si una reparación de emergencia es rentable. También devuelve la oportunidad de ingresos que representa la turbina y el costo de repararla.

## <a name="run-and-verify-the-api-locally"></a>Ejecución y comprobación locales de la API

Al ejecutar la función, los puntos de conexión de OpenAPI hacen que resulte fácil probar la función localmente usando una página generada. No es necesario proporcionar claves de acceso de función cuando se ejecuta localmente.

1. Presione F5 para iniciar el proyecto. Cuando el entorno de ejecución de Functions se inicia localmente, se muestra un conjunto de puntos de conexión de OpenAPI y Swagger en la salida, junto con el punto de conexión de la función.  

1. En el explorador, abra el punto de conexión RenderSwaggerUI, que debe tener un aspecto parecido a `http://localhost:7071/api/swagger/ui`. Se representa una página en función de las definiciones de OpenAPI.

1. Seleccione **POST** > **Pruébelo**, especifique valores para `hours` y `capacity`, ya sea como parámetros de consulta o en el cuerpo de la solicitud JSON, y seleccione **Ejecutar**. 

    :::image type="content" source="media/openapi-apim-integrate-vs/swagger-ui-post.png" alt-text="IU de Swagger UI para probar la API TurbineRepair":::

1. Si especifica valores enteros, como 6 para `hours` y 2500 para `capacity`, obtendrá una respuesta JSON similar al siguiente ejemplo:
   
    :::image type="content" source="media/openapi-apim-integrate-vs/swagger-ui-response.png" alt-text="Datos JSON de respuesta de la función TurbineRepair":::

Ahora ya existe una función que determina la rentabilidad de las reparaciones de emergencia. A continuación, publique el proyecto y las definiciones de API en Azure.

## <a name="publish-the-project-to-azure"></a>Publicar el proyecto en Azure

Para poder publicar el proyecto, debe tener una aplicación de funciones en la suscripción de Azure. La publicación de Visual Studio crea una aplicación de funciones la primera vez que publica su proyecto. También puede crear una instancia de API Management que se integra con su aplicación de funciones para exponer la API TurbineRepair.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**, en **Destino**, seleccione **Azure** y, a continuación, seleccione **Siguiente**.

1. En **Destino específico**, elija **Aplicación de funciones de Azure (Windows)** , para crear una aplicación de funciones que se ejecute en Windows, y seleccione **Siguiente**.

1. En **Instancia de la función**, seleccione **+ Crear una nueva función de Azure…** .

    :::image type="content" source="media/openapi-apim-integrate-vs/publish-new-resource.png" alt-text="Creación de una nueva instancia de aplicación de funciones":::

1. Crear una nueva instancia con los valores especificados en la tabla siguiente:

    | Configuración      | Valor  | Descripción                                |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre** | Nombre único globalmente | Nombre que identifica de forma única la nueva aplicación de función. Acepte este nombre o escriba uno nuevo. Los caracteres válidos son `a-z`, `0-9` y `-`. |
    | **Suscripción** | Su suscripción | La suscripción de Azure que se va a usar. Acepte esta suscripción o seleccione una nueva en la lista desplegable. |
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)** | Nombre del grupo de recursos |  Nombre del grupo de recursos en el que se va a crear la aplicación de función. Seleccione un grupo de recursos existente en la lista desplegable o elija la opción **Nuevo** para crear un nuevo grupo de recursos.|
    | **[Tipo de plan](functions-scale.md)** | Consumo | Cuando publique el proyecto en una aplicación de funciones que se ejecute en un [plan Consumo](consumption-plan.md), solo pagará por las ejecuciones de la aplicación. Otros planes de hospedaje suponen costos más elevados. |
    | **Ubicación** | Ubicación del servicio | Elija una **ubicación** en una [región](https://azure.microsoft.com/regions/) próxima a usted o a otros servicios a los que las funciones accedan. |
    | **[Azure Storage](storage-considerations.md)** | Cuenta de almacenamiento de uso general | El entorno de ejecución de Functions necesita una cuenta de Azure Storage. Seleccione **Nueva** para configurar una cuenta de almacenamiento de uso general. También puede elegir una cuenta existente que cumpla los [requisitos de la cuenta de almacenamiento](storage-considerations.md#storage-account-requirements).  |

    :::image type="content" source="media/openapi-apim-integrate-vs/create-function-app-with-storage.png" alt-text="Creación de una aplicación de funciones con Storage":::

1. Seleccione **Crear** para crear una aplicación de funciones y sus recursos relacionados en Azure. El estado de la creación del recurso se muestra en la parte inferior izquierda de la ventana. 

1. En **Instancia de Functions**, asegúrese de que la opción **Ejecutar desde archivo de paquete** esté activada. La aplicación de funciones se implementa con la [implementación de un archivo zip](functions-deployment-technologies.md#zip-deploy) y con el modo de [ejecución desde el paquete](run-functions-from-deployment-package.md) habilitado. Este es el método de implementación recomendado para su proyecto de Functions, ya que proporciona un mejor rendimiento. 

1. Seleccione **Siguiente** y, en la página **API Management**, elija también **+ Crear una API de API Management**.

1.  Para crear una **API en API Management**, use los valores de la siguiente tabla:

    | Configuración      | Valor  | Descripción                                |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre de la API** | TurbineRepair | Nombre de la API. |
    | **Nombre de la suscripción** | Su suscripción | La suscripción de Azure que se va a usar. Acepte esta suscripción o seleccione una nueva en la lista desplegable. |
    | **Grupos de recursos** | Nombre del grupo de recursos | En la lista desplegable, seleccione el mismo grupo de recursos que la aplicación de funciones.   |
    | **Servicio API Management** | Nueva instancia | Seleccione **Nuevo** para crear una instancia de API Management en el nivel sin servidor.   |

    :::image type="content" source="media/openapi-apim-integrate-vs/create-api-management-api.png" alt-text="Creación de una instancia de API Management con una API":::.

1. Seleccione **Crear** para crear la instancia de API Management con la API TurbineRepair a partir de la integración de funciones.

1. Seleccione **Finalizar** y verifique que aparece **Listo para publicar** en la página de publicación. A continuación, seleccione **Publicar** para implementar el paquete con los archivos del proyecto en su nueva aplicación de funciones de Azure. 

    Una vez finalizada la implementación, en la pestaña **Publicar** aparecerá la dirección URL raíz de la aplicación de funciones de Azure. 

## <a name="get-the-function-access-key"></a>Obtención de la clave de acceso de función

1. En la pestaña **Publicar**, seleccione los puntos suspensivos ( **…** ) junto a **Hospedaje** y seleccione **Abrir en Azure Portal**. La aplicación de funciones que ha creado se abrirá en Azure Portal en su navegador predeterminado. 

1. En **Funciones**, seleccione **Funciones** >  **TurbineRepair** y, a continuación, **Claves de función**. 

    :::image type="content" source="media/openapi-apim-integrate-vs/get-function-keys.png" alt-text="Obtención de una clave de acceso para la función TurbineRepair":::

1. En **Claves de función**, seleccione **valor predeterminado** y copie el **valor**. Ya puede establecer esta clave en API Management para que pueda acceder al punto de conexión de la función.

## <a name="configure-api-management"></a>Configuración de API Management

1. En la pestaña **Publicar**, seleccione los puntos suspensivos ( **…** ) junto a **Hospedaje** y seleccione **Abrir API en Azure Portal**. La instancia de API Management que ha creado se abrirá en Azure Portal en su navegador predeterminado. Esta instancia de API Management ya está vinculada a la aplicación de funciones. 

1. En **API**, seleccione **Extensión OpenAPI de Azure Functions** > **Prueba** > **Ejecución POST**; a continuación, en **Directiva de entrada**, seleccione **Agregar directiva**.

    :::image type="content" source="media/openapi-apim-integrate-vs/apim-add-policy.png" alt-text="Incorporación de una directiva de entrada a la API":::

1. En **Agregar directiva de entrada**, elija **Establecer parámetros de consulta**, escriba `code` para **Nombre**, seleccione **+Valor**, pegue la clave de función copiada y seleccione **Guardar**. API Management incluye la clave de función cuando pasa la llamada al punto de conexión de la función. 

Ahora que la clave de función está establecida, puede llamar a la API para comprobar que funciona cuando se hospeda en Azure.

## <a name="verify-the-api-in-azure"></a>Comprobación de la API en Azure

1. En la API, seleccione la pestaña **Prueba** y **Ejecución POST**. A continuación, especifique el siguiente código en **Cuerpo de la solicitud** > **Sin formato** y seleccione **Enviar**:

    ```json
    {
        "hours": "6",
        "capacity": "2500"
    }
    ```

    :::image type="content" source="media/openapi-apim-integrate-vs/api-management-test-function-api.png" alt-text="Página de prueba de OpenAPI en la API de API Management":::

    Como antes, también puede proporcionar los mismos valores que los parámetros de consulta. 

1. Seleccione **Enviar** y, a continuación, consulte la **respuesta HTTP** para comprobar que la API devuelve los mismos resultados.

## <a name="download-the-openapi-definition"></a>Descarga de la definición de OpenAPI

Si la API funciona según lo previsto, puede descargar la definición de OpenAPI.

1. 1. En **API**, seleccione **Extensión OpenAPI de Azure Functions** , los puntos suspensivos ( **…** ) y **Exportar**.
   
   ![Descarga de la definición de OpenAPI](media/openapi-apim-integrate-vs/download-definition.png)

2. Elija el medio de exportación de API, incluidos los archivos OpenAPI en varios formatos. También puede [exportar API de Azure API Management a Power Platform](../api-management/export-api-power-platform.md). 

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si no cree que vaya a necesitar estos recursos en el futuro, puede eliminarlos mediante la eliminación del grupo de recursos.
 
En el menú de Azure Portal o la página **Inicio**, seleccione **Grupos de recursos**. A continuación, en la página **Grupos de recursos**, seleccione el grupo que ha creado.

En la página **myResourceGroup**, asegúrese de que los recursos enumerados sean los que desea eliminar.

Seleccione **Eliminar grupo de recursos**, escriba el nombre de su grupo en el cuadro de texto para confirmar la acción y, a continuación, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Ha usado Visual Studio 2019 para crear una función que se autodocumenta debido a la [extensión OpenAPI](https://github.com/Azure/azure-functions-openapi-extension) y está integrada con API Management. Ya puede refinar la definición de API Management en el portal. También puede obtener [más información acerca de API Management](../api-management/api-management-key-concepts.md).

> [!div class="nextstepaction"]
> [Edición de la definición de OpenAPI en API Management](../api-management/edit-api.md)
