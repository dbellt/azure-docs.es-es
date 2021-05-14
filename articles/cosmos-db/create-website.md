---
title: 'Implementación de una aplicación web con una plantilla: Azure Cosmos DB'
description: Aprenda a implementar una cuenta de Azure Cosmos, Azure App Service Web Apps y una aplicación web de muestra con una plantilla de Azure Resource Manager.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/19/2020
ms.author: mjbrown
ms.openlocfilehash: 1ff0543ebc341788ee9492f3e2a117c8e64c44f5
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737391"
---
# <a name="deploy-azure-cosmos-db-and-azure-app-service-with-a-web-app-from-github-using-an-azure-resource-manager-template"></a>Implementación de Azure Cosmos DB y Azure App Service con una aplicación web desde GitHub con una plantilla de Azure Resource Manager
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En este tutorial se muestra cómo realizar una implementación "sin tocar" de una aplicación web que se conecta a Azure Cosmos DB en la primera ejecución sin tener que cortar y pegar la información de conexión de Azure Cosmos DB a `appsettings.json` o a la configuración de Azure App Service en el Azure Portal. Todas estas acciones se realizan con una plantilla de Azure Resource Manager en una sola operación. En el siguiente ejemplo, se implementará la [muestra de Azure Cosmos DB ToDo](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) desde un tutorial de [aplicación Web](sql-api-dotnet-application.md).

Las plantillas de Resource Manager son bastante flexibles y permiten crear implementaciones complejas en cualquier servicio de Azure. Esto incluye tareas avanzadas, como la implementación de aplicaciones desde GitHub y la inserción de información de conexión en la configuración de la aplicación de Azure App Service en el Azure Portal. En este tutorial se muestra cómo realizar las siguientes acciones con una sola plantilla de Resource Manager.

* Implementar una cuenta de Azure Cosmos.
* Implementar un plan de hospedaje de Azure App Service.
* Implementar un Azure App Service.
* Insertar el punto de conexión y las claves de la cuenta de Azure Cosmos en la configuración de la aplicación App Service en el Azure Portal.
* Implementar una aplicación web desde un repositorio de GitHub en el App Service.

La implementación resultante tiene una aplicación web totalmente funcional que puede conectarse a Azure Cosmos DB sin tener que cortar y pegar la dirección URL del punto de conexión del Azure Cosmos DB o las claves de autenticación de la Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

> [!TIP]
> Aunque en este tutorial no se exige experiencia con plantillas de Azure Resource Manager o JSON, si quiere modificar las plantillas o las opciones de implementación a las que se hace referencia, entonces sí es necesario tener conocimientos sobre cada una de estas áreas.

## <a name="step-1-deploy-the-template"></a>Paso 1: Implementación de la plantilla

En primer lugar, seleccione el botón **Implementar en Azure** para abrir el Azure Portal para crear una implementación personalizada. También puede ver la plantilla de Resource Management en la [Galería de plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.documentdb/cosmosdb-webapp)

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación en Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.documentdb%2Fcosmosdb-webapp%2Fazuredeploy.json)

Una vez en el Azure Portal, seleccione la suscripción en la que desea realizar la implementación y seleccione o cree un nuevo grupo de recursos. Después, rellene los siguientes valores.

:::image type="content" source="./media/create-website/template-deployment.png" alt-text="Captura de pantalla de la interfaz de usuario de implementación de plantillas":::

* **Región**: es necesario para el Resource Manager. Escriba la misma región utilizada por el parámetro de ubicación donde se encuentran los recursos.
* **Nombre de la aplicación**: todos los recursos de esta implementación usan este nombre. Asegúrese de elegir un nombre único para evitar conflictos con las cuentas de App Service y Azure Cosmos DB existentes.
* **Ubicación**: la región donde se implementan los recursos.
* **El plan de tarifa de App Service**: plan de tarifa de plan de App Service.
* **Instancias de plan de App Service**: el número de trabajadores para el plan de App Service.
* **URL del repositorio**: el repositorio a la aplicación web en GitHub.
* **Rama**: la rama para el repositorio de GitHub.
* **Nombre de la base de datos**: nombre de la base de datos de Azure Cosmos.
* **Nombre del contenedor**: el nombre del contenedor de Cosmos de Azure.

Después de rellenar los valores, seleccione el botón **Crear** para iniciar la implementación. Este paso debe completarse entre 5 y 10 minutos.

> [!TIP]
> La plantilla no valida que el nombre del Azure App Service y el nombre de la cuenta de Azure Cosmos especificado en la plantilla sean válidos y estén disponibles. Es muy recomendable comprobar la disponibilidad de los nombres que planea suministrar antes de enviar la implementación.


## <a name="step-2-explore-the-resources"></a>Paso 2: Explorar los recursos

### <a name="view-the-deployed-resources"></a>Ver los recursos implementados

Después de que la plantilla haya implementado los recursos, ahora puede ver cada uno de ellos en el grupo de recursos.

:::image type="content" source="./media/create-website/resource-group.png" alt-text="Grupo de recursos":::

### <a name="view-cosmos-db-endpoint-and-keys"></a>Ver el punto de conexión y las claves de Cosmos DB

Después, abra la cuenta de Azure Cosmos en el portal. En la siguiente captura de pantalla se muestra el punto de conexión y las claves de una cuenta de Azure Cosmos.

:::image type="content" source="./media/create-website/cosmos-keys.png" alt-text="Claves de Cosmos":::

### <a name="view-the-azure-cosmos-db-keys-in-application-settings"></a>Vea las claves de Azure Cosmos DB en la configuración de la aplicación

Después, vaya al Azure App Service del grupo de recursos. Haga clic en la pestaña Configuración para ver la configuración de la aplicación para el App Service. La Configuración de la aplicación contiene la cuenta Cosmos DB y los valores de clave principal necesarios para conectarse a Cosmos DB así como los nombres de base de datos y contenedor que se pasaron desde la implementación de plantilla.

:::image type="content" source="./media/create-website/application-settings.png" alt-text="Configuración de la aplicación":::

### <a name="view-web-app-in-deployment-center"></a>Vea la aplicación web en el centro de implementación

Después, vaya al centro de implementación para el App Service. Aquí verá los puntos del repositorio en el repositorio de GitHub que se pasan a la plantilla. Además, el siguiente estado indica Éxito (Activo), lo que significa que la aplicación se implementó e inició correctamente.

:::image type="content" source="./media/create-website/deployment-center.png" alt-text="Centro de implementación":::

### <a name="run-the-web-application"></a>Ejecución de la aplicación web

Haga clic en **Examinar** en la parte superior del Centro de implementación para abrir la aplicación web. La aplicación web se abrirá en la pantalla principal. Haga clic en **Crear nuevo** y escriba algunos datos en los campos y haga clic en Guardar. La pantalla resultante muestra los datos guardados en Cosmos DB.

:::image type="content" source="./media/create-website/app-home-screen.png" alt-text="Pantalla principal":::

## <a name="step-3-how-does-it-work"></a>Paso 3: ¿Cómo funciona?

Hay tres elementos necesarios para que funcione.

### <a name="reading-app-settings-at-runtime"></a>Lectura de la configuración de la aplicación en tiempo de ejecución

En primer lugar, la aplicación necesita solicitar el Cosmos DB un punto de conexión y la clave en la clase `Startup` de la aplicación web MVC de ASP.NET. La [Muestra de Cosmos To Do](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) se puede ejecutar localmente, donde puede especificar la información de conexión en appsettings.json. Pero cuando se implementa, este archivo se implementa con la aplicación. Si estas líneas de color rojo no pueden tener acceso a la configuración de appsettings.json, se intentará desde la configuración de la aplicación en Azure App Service.

:::image type="content" source="./media/create-website/startup.png" alt-text="Captura de pantalla que muestra un método con varias variables de tipo string marcadas en rojo, como databaseName, containerName, account y key.":::

### <a name="using-special-azure-resource-management-functions"></a>Uso de funciones especiales de administración de recursos de Azure

Para que estos valores estén disponibles para la aplicación cuando se implementa, la plantilla de Azure Resource Manager puede solicitar esos valores de la cuenta de Cosmos DB al usar funciones especiales de Azure Resource Management que incluyen [referencia](../azure-resource-manager/templates/template-functions-resource.md#reference) y [listKeys](../azure-resource-manager/templates/template-functions-resource.md#listkeys) que tome los valores de la cuenta de Cosmos DB e insértelos en los valores de configuración de la aplicación con nombres de clave que coincidan con lo que se usa en la aplicación anterior en un formato "{section:key}". Por ejemplo, `CosmosDb:Account`.

:::image type="content" source="./media/create-website/template-keys.png" alt-text="Claves de plantilla":::

### <a name="deploying-web-apps-from-github"></a>Implementación de aplicaciones web desde GitHub

Por último, es necesario implementar la aplicación web desde GitHub en el App Service. Esto se hace con el siguiente código JSON. Dos aspectos que se deben tener en consideración son el tipo y el nombre de este recurso. Los valores de propiedad `"type": "sourcecontrols"` y `"name": "web"` están codificados y no deben cambiarse.

:::image type="content" source="./media/create-website/deploy-from-github.png" alt-text="Implementación desde GitHub":::

## <a name="next-steps"></a>Pasos siguientes

Felicidades. Ha implementado Azure Cosmos DB, Azure App Service y una aplicación web de muestra que tiene automáticamente la información de conexión necesaria para conectarse a Cosmos DB, todo en una sola operación y sin tener que cortar y pegar información confidencial. Con esta plantilla como punto de partida, puede modificarla para implementar sus propias aplicaciones web de la misma manera.

* Para ver la plantilla de Azure Resource Manager de esta muestra, vaya a [Galería de plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.documentdb/cosmosdb-webapp)
* Para el código fuente de la aplicación de muestra, vaya a [Cosmos DB para realizar la aplicación en GitHub](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app).
