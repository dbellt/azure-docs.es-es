---
title: Configuración de DevOps para Azure Logic Apps de inquilino único
description: Configuración de la implementación de DevOps para flujos de trabajo en Azure Logic Apps de inquilino único.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: 41cc4c174028ff23cdcc248c6b10d746e5669349
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751242"
---
# <a name="set-up-devops-deployment-for-single-tenant-azure-logic-apps"></a>Configuración de la implementación de DevOps para Azure Logic Apps de inquilino único

En este artículo se muestra cómo implementar en su infraestructura un proyecto de aplicación lógica basado en un solo inquilino de Visual Studio Code mediante herramientas y procesos de DevOps. En función de si prefiere GitHub o Azure DevOps para la implementación, elija la ruta de acceso y las herramientas que mejor funcionen para su escenario. Puede usar los ejemplos incluidos que contienen proyectos de aplicación lógica de ejemplo más los ejemplos de implementación de Azure mediante GitHub o Azure DevOps. Para más información sobre DevOps en el caso de un solo inquilino, consulte [Introducción a la implementación de DevOps para Azure Logic Apps de inquilino único](devops-deployment-single-tenant-azure-logic-apps.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Un proyecto de aplicación lógica basado en un solo inquilino creado con [Visual Studio Code y la extensión Azure Logic Apps (estándar)](create-single-tenant-workflows-visual-studio-code.md#prerequisites).

  Si aún no ha configurado un proyecto de aplicación lógica o una infraestructura, puede usar los proyectos de ejemplo incluidos para implementar una aplicación y una infraestructura de ejemplo, en función de las opciones de origen e implementación que prefiera usar. Para más información sobre estos proyectos de ejemplo y los recursos incluidos para ejecutar la aplicación lógica de ejemplo, consulte [Implementación de la infraestructura](#deploy-infrastructure).

- Si la implementación prefiere realizarla en Azure, necesita un recurso de **aplicación lógica (estándar)** existente creado en Azure. Para crear rápidamente un recurso de aplicación lógica vacío, consulte [Creación de flujos de trabajo de aplicación lógica basados en un único inquilino: portal](create-single-tenant-workflows-azure-portal.md).

<a name="deploy-infrastructure"></a>

## <a name="deploy-infrastructure-resources"></a>Implementación de recursos de infraestructura

Si aún no tiene un proyecto de aplicación lógica o una infraestructura configurados, puede usar los proyectos de ejemplo incluidos para implementar una aplicación y una infraestructura de ejemplo, en función de las opciones de origen e implementación que prefiera usar:

- [Ejemplo de GitHub para Azure Logic Apps de inquilino único](https://github.com/Azure/logicapps/tree/master/github-sample)

  En este ejemplo se incluye un proyecto de aplicación lógica de ejemplo para Azure Logic Apps de inquilino único, además de ejemplos para la implementación de Azure y Acciones de GitHub.

- [Ejemplo de Azure DevOps para Azure Logic Apps de inquilino único](https://github.com/Azure/logicapps/tree/master/azure-devops-sample)

  En este ejemplo se incluye un proyecto de aplicación lógica de ejemplo para Azure Logic Apps de inquilino único, además de ejemplos para la implementación de Azure y Azure Pipelines.
  
Ambos ejemplos incluyen los siguientes recursos que emplea una aplicación lógica para ejecutarse.

| Nombre del recurso | Requerido | Descripción |
|---------------|----------|-------------|
| Aplicación lógica (estándar) | Yes | Este recurso de Azure contiene los flujos de trabajo que se ejecutan en Azure Logic Apps de inquilino único. |
| Plan de hospedaje de App Service o Prémium de Functions | Yes | Este recurso de Azure especifica los recursos de hospedaje que se usarán para ejecutar la aplicación lógica, por ejemplo, proceso, procesamiento, almacenamiento, redes, etc. <p><p>**Importante**: En la experiencia actual, el recurso **Aplicación lógica (estándar)** requiere el plan de hospedaje [**Flujo de trabajo estándar**](logic-apps-pricing.md#standard-pricing), que se basa en el plan de hospedaje Prémium de Functions. |
| Cuenta de almacenamiento de Azure | Sí, para flujos de trabajo sin estado | Este recurso de Azure almacena los metadatos, el estado, las entradas, las salidas, el historial de ejecución y otra información sobre los flujos de trabajo. |
| Application Insights | Opcionales | Este recurso de Azure proporciona funcionalidades de supervisión para los flujos de trabajo. |
| Conexiones de API | Opcional, si no existe ninguno | Estos recursos de Azure definen las conexiones de API administradas que los flujos de trabajo emplean para ejecutar operaciones de conector administrado, como Office 365, SharePoint, etc. <p><p>**Importante**: En el proyecto de aplicación lógica, el archivo **connections.json** contiene metadatos, puntos de conexión y claves para las conexiones de API administradas y las funciones de Azure que usan los flujos de trabajo. Para usar diferentes conexiones y funciones en cada entorno, asegúrese de parametrizar el archivo **connections.json** y actualizar los puntos de conexión. <p><p>Para más información, consulte [Recursos de conexión de API y directivas de acceso](#api-connection-resources). |
| Plantilla de Azure Resource Manager (ARM) | Opcionales | Este recurso de Azure define una implementación de infraestructura de línea de base que puede reutilizar o [exportar](../azure-resource-manager/templates/template-tutorial-export-template.md). La plantilla también incluye las directivas de acceso necesarias, por ejemplo, para usar las conexiones de API administradas. <p><p>**Importante**: La exportación de la plantilla de ARM no incluirá todos los parámetros relacionados para los recursos de conexión de API que usan los flujos de trabajo. Para más información, consulte [Búsqueda de los parámetros de conexión de API](#find-api-connection-parameters). |
||||

<a name="api-connection-resources"></a>

## <a name="api-connection-resources-and-access-policies"></a>Recursos de conexión de API y directivas de acceso

En Azure Logic Apps de inquilino único, todos los recursos de conexión de API o administrados de los flujos de trabajo requieren una directiva de acceso asociada. Esta directiva necesita la identidad de la aplicación lógica para proporcionar los permisos correctos para acceder a la infraestructura del conector administrado. Los proyectos de ejemplo incluidos incorporan una plantilla de ARM con todos los recursos de infraestructura necesarios, como estas directivas de acceso.

En el diagrama siguiente se muestran las dependencias entre el proyecto de aplicación lógica y los recursos de infraestructura:

![Diagrama conceptual que muestra las dependencias de infraestructura de un proyecto de aplicación lógica en el modelo de Azure Logic Apps de inquilino único.](./media/set-up-devops-deployment-single-tenant-azure-logic-apps/infrastructure-dependencies.png)

<a name="find-api-connection-parameters"></a>

### <a name="find-api-connection-parameters"></a>Búsqueda de los parámetros de conexión de API

Si los flujos de trabajo emplean conexiones de API administradas, el uso de la funcionalidad de la plantilla de exportación no incluirá todos los parámetros relacionados. En una plantilla de ARM, cada [definición de recurso de conexión de API](logic-apps-azure-resource-manager-templates-overview.md#connection-resource-definitions) tiene el formato general siguiente:

```json
{
   "type": "Microsoft.Web/connections",
   "apiVersion": "2016–06–01",
   "location": "[parameters('location')]",
   "name": "[parameters('connectionName')]",
   "properties": {}
}
```

Para encontrar los valores que debe usar en el objeto `properties` para completar la definición del recurso de conexión, puede usar la siguiente API en un conector concreto:

`PUT https://management.azure.com/subscriptions/{subscription-ID}/providers/Microsoft.Web/locations/{location}/managedApis/{connector-name}?api-version=2018–07–01-preview`

En la respuesta, busque el objeto `connectionParameters`, que contiene toda la información necesaria para completar la definición del recurso de ese conector en concreto. En el ejemplo siguiente se muestra una definición de recurso de ejemplo para una conexión administrada de SQL:

```json
{
   "type": "Microsoft.Web/connections",
   "apiVersion": "2016–06–01",
   "location": "[parameters('location')]",
   "name": "[parameters('connectionName')]",
   "properties": {
      "displayName": "sqltestconnector",
      "api": {
         "id": "/subscriptions/{subscription-ID}/providers/Microsoft.Web/locations/{location}/managedApis/sql"
      },
      "parameterValues": {
         "authType": "windows", 
         "database": "TestDB",
         "password": "TestPassword",
         "server": "TestServer",
         "username": "TestUserName"
      }
   }
}
```

También puede revisar el seguimiento de red para cuando cree una conexión en el Diseñador de aplicaciones lógicas. Busque la llamada `PUT` a la API administrada del conector como se ha descrito anteriormente, y revise el cuerpo de la solicitud para ver si tiene toda la información que necesita.

## <a name="deploy-logic-app-resources-zip-deploy"></a>Implementación de recursos de aplicación lógica (implementación de archivos ZIP)

Después de insertar el proyecto de aplicación lógica en el repositorio de origen, puede configurar canalizaciones de compilación y versión que implementen aplicaciones lógicas en la infraestructura dentro o fuera de Azure.

### <a name="build-your-project"></a>Compilación del proyecto

Para configurar una canalización de compilación basada en el tipo de proyecto de aplicación lógica, realice las acciones correspondientes de la siguiente tabla:

| Tipo de proyecto | Descripción y pasos |
|--------------|-----------------------|
| Basado en NuGet | La estructura del proyecto basado en NuGet se basa en .NET Framework. Para compilar estos proyectos, asegúrese de seguir los pasos de compilación de .NET Standard. Para más información, consulte [Creación de un paquete NuGet mediante MSBuild](/nuget/create-packages/creating-a-package-msbuild). |
| Basado en agrupación | El proyecto de extensiones basado en agrupación no es específico del lenguaje y, por tanto, no requiere ningún paso de compilación específico de este. Puede usar cualquier método para comprimir los archivos del proyecto. <p><p>**Importante:** Asegúrese de que el archivo .zip contenga los artefactos de compilación reales, incluidas todas las carpetas de flujo de trabajo, los archivos de configuración como host.json, connections.json y cualquier otro archivo relacionado. |
|||

### <a name="release-to-azure"></a>Publicación en Azure

Para configurar una canalización de versión que se implemente en Azure, elija la opción asociada de GitHub, Azure DevOps o CLI de Azure.

> [!NOTE]
> Azure Logic Apps actualmente no admite ranuras de implementación de Azure.

#### <a name="github"></a>[GitHub](#tab/github)

En el caso de las implementaciones de GitHub, puede implementar la aplicación lógica mediante [Acciones de GitHub](https://docs.github.com/actions), por ejemplo, la acción de GitHub en Azure Functions. Esta acción requiere que recorra la siguiente información:

- El nombre de la aplicación lógica que se usará para su implementación
- El archivo zip que contiene los artefactos de compilación reales, incluidas todas las carpetas de flujo de trabajo, archivos de configuración como host.js, connections.jsy cualquier otro archivo relacionado.
- El [perfil de publicación](../azure-functions/functions-how-to-github-actions.md#generate-deployment-credentials) que se usa para la autenticación.

```yaml
- name: 'Run Azure Functions Action'
  uses: Azure/functions-action@v1
  id: fa
  with:
   app-name: 'MyLogicAppName'
   package: 'MyBuildArtifact.zip'
   publish-profile: 'MyLogicAppPublishProfile'
```

Para más información, revise la documentación [Entrega continua mediante la acción de GitHub](../azure-functions/functions-how-to-github-actions.md).

#### <a name="azure-devops"></a>[Azure DevOps](#tab/azure-devops)

En las implementaciones de Azure DevOps, puede implementar la aplicación lógica mediante la [tarea de implementación de aplicaciones de funciones de Azure](/azure/devops/pipelines/tasks/deploy/azure-function-app?view=azure-devops&preserve-view=true) en Azure Pipelines. Esta acción requiere que recorra la siguiente información:

- El nombre de la aplicación lógica que se usará para su implementación
- El archivo zip que contiene los artefactos de compilación reales, incluidas todas las carpetas de flujo de trabajo, archivos de configuración como host.js, connections.jsy cualquier otro archivo relacionado.
- El [perfil de publicación](../azure-functions/functions-how-to-github-actions.md#generate-deployment-credentials) que se usa para la autenticación.

```yaml
- task: AzureFunctionApp@1
  displayName: 'Deploy logic app workflows'
  inputs:
     azureSubscription: 'MyServiceConnection'
     appType: 'workflowapp'
     appName: 'MyLogicAppName'
     package: 'MyBuildArtifact.zip'
     deploymentMethod: 'zipDeploy'
```

Para más información, consulte la documentación [Implementación de una función de Azure mediante Azure Pipelines](/azure/devops/pipelines/targets/azure-functions-windows).

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Si usa otras herramientas de implementación, puede implementar la aplicación lógica de inquilino único mediante la CLI de Azure. Antes de empezar, es preciso tener los siguientes elementos:

- La extensión más reciente de la CLI de Azure instalada en el equipo local.

  - Si no la tiene, consulte la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

  - Si no sabe seguro si tiene la versión más reciente, siga los [pasos para comprobar el entorno y la versión de la CLI](#check-environment-cli-version).

- La *versión preliminar* de la extensión de Azure Logic Apps (Estándar) de inquilino único para la CLI de Azure.

  Si no tiene esta extensión, siga los pasos [para instalarla](#install-logic-apps-cli-extension). Aunque Azure Logic Apps de inquilino único está disponible con carácter general, la extensión de Azure Logic Apps de inquilino único aún está en versión preliminar.

- Grupo de recursos de Azure que se va a usar para implementar la aplicación lógica.

  Si no tiene este grupo de recursos, siga los [pasos para crear el grupo de recursos](#create-resource-group).

- Una cuenta de Azure Storage que se usará con la aplicación lógica para la retención de datos y del historial de ejecución.

  Si no tiene esta cuenta de almacenamiento, siga los [pasos para crear una cuenta de almacenamiento](/cli/azure/storage/account#az_storage_account_create).

<a name="check-environment-cli-version"></a>

##### <a name="check-environment-and-cli-version"></a>Comprobación del entorno y la versión de la CLI

1. Inicie sesión en [Azure Portal](https://portal.azure.com). En una ventana de terminal o de comandos, ejecute el comando [`az login`](/cli/azure/authenticate-azure-cli) para confirmar que la suscripción está activa:

   ```azurecli-interactive
   az login
   ```

1. En una ventana de terminal o de comandos, ejecute el comando `az` para comprobar la versión del CLI de Azure, con el siguiente parámetro necesario:

   ```azurecli-interactive
   az --version
   ```

1. Si no tiene la versión más reciente de la CLI de Azure, actualice la instalación; para ello, siga las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

   Para obtener más información sobre la versión más reciente, consulte las [notas de la versión más recientes](/cli/azure/release-notes-azure-cli?tabs=azure-cli).

<a name="install-logic-apps-cli-extension"></a>

##### <a name="install-azure-logic-apps-standard-extension-for-azure-cli"></a>Instalación de la extensión de Azure Logic Apps (Estándar) para la CLI de Azure

Para instalar la *versión preliminar* de la extensión de Azure Logic Apps (Estándar) de inquilino único para la CLI de Azure, ejecute el comando `az extension add` con los siguientes parámetros requeridos:

```azurecli-interactive
az extension add --yes --source "https://aka.ms/logicapp-latest-py2.py3-none-any.whl"
```

<a name="create-resource-group"></a>

#### <a name="create-resource-group"></a>Creación de un grupo de recursos

Si aún no tiene un grupo de recursos para la aplicación lógica, ejecute el comando `az group create` para crearlo. A menos que ya haya establecido una suscripción predeterminada para su cuenta de Azure, asegúrese de usar el parámetro `--subscription` con el nombre o el identificador de la suscripción. De lo contrario, no tiene que usar el parámetro `--subscription`.

> [!TIP]
> Para establecer una suscripción predeterminada, ejecute el siguiente comando y reemplace `MySubscription` por el nombre o identificador de la suscripción.
>
> `az account set --subscription MySubscription`

Por ejemplo, el siguiente comando crea un grupo de recursos denominado `MyResourceGroupName` mediante la suscripción de Azure denominada `MySubscription` en la ubicación `eastus`:

```azurecli
az group create --name MyResourceGroupName 
   --subscription MySubscription 
   --location eastus
```

Si el grupo de recursos se crea correctamente, la salida muestra `provisioningState` como `Succeeded`:

```output
<...>
   "name": "testResourceGroup",
   "properties": {
      "provisioningState": "Succeeded"
    },
<...>
```

<a name="deploy-logic-app"></a>

##### <a name="deploy-logic-app"></a>Implementación de la aplicación lógica

Para implementar el artefacto comprimido en un grupo de recursos de Azure, ejecute el comando `az logicapp deployment` con los siguientes parámetros:

> [!IMPORTANT]
> Asegúrese de que el archivo zip contenga los artefactos del proyecto en el nivel raíz. Estos artefactos incluyen todas las carpetas del flujo de trabajo, los archivos de configuración, como host.json y connections.json, y cualquier otro archivo relacionado. No agregue carpetas adicionales ni coloque ningún artefacto en carpetas que no existan en la estructura del proyecto. Por ejemplo, en esta lista se muestra un estructura de archivos de ejemplo, en MyBuildArtifacts.zip:
>
> ```output
> MyStatefulWorkflow1-Folder
> MyStatefulWorkflow2-Folder
> connections.json
> host.json
> ```

```azurecli-interactive
az logicapp deployment source config-zip --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription 
   --src MyBuildArtifact.zip
```

---

### <a name="release-to-containers"></a>Publicación en contenedores

Si la aplicación lógica se incluye en contenedores, la implementación funciona principalmente igual que cualquier otro contenedor que implemente y administre.

Para ver ejemplos que muestran cómo implementar una canalización de implementación y compilación de contenedor de un extremo a otro, revise [CI/CD para Containers](https://azure.microsoft.com/solutions/architecture/cicd-for-containers/).

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de DevOps para Azure Logic Apps de inquilino único](devops-deployment-single-tenant-azure-logic-apps.md)

Además, nos gustaría conocer sus experiencias con Azure Logic Apps de inquilino único.

- En caso de errores o problemas, [dirija los problemas a GitHub](https://github.com/Azure/logicapps/issues).
- Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/logicappsdevops).
