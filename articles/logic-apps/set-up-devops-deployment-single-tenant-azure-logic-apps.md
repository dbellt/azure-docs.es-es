---
title: Configuración de DevOps para Azure Logic Apps de inquilino único
description: Configuración de la implementación de DevOps para flujos de trabajo en Azure Logic Apps de inquilino único.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: bd35af7ac6602bba7e23bd10eda2f2b0fdff71db
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110379766"
---
# <a name="set-up-devops-deployment-for-single-tenant-azure-logic-apps"></a>Configuración de la implementación de DevOps para Azure Logic Apps de inquilino único

En este artículo se muestra cómo implementar en su infraestructura un proyecto de aplicación lógica basado en un solo inquilino de Visual Studio Code mediante herramientas y procesos de DevOps. En función de si prefiere GitHub o Azure DevOps para la implementación, elija la ruta de acceso y las herramientas que mejor funcionen para su escenario. Puede usar los ejemplos incluidos que contienen proyectos de aplicación lógica de ejemplo más los ejemplos de implementación de Azure mediante GitHub o Azure DevOps. Para más información sobre DevOps en el caso de un solo inquilino, consulte [Introducción a la implementación de DevOps para Azure Logic Apps de inquilino único](devops-deployment-single-tenant-azure-logic-apps.md).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Un proyecto de aplicación lógica basado en un solo inquilino creado con [Visual Studio Code y la extensión Azure Logic Apps (estándar)](create-single-tenant-workflows-visual-studio-code.md#prerequisites).

  Si aún no tiene un proyecto de aplicación lógica o una infraestructura configurados, puede usar los proyectos de ejemplo incluidos para implementar una aplicación y una infraestructura de ejemplo, en función de las opciones de origen e implementación que prefiera usar. Para más información sobre estos proyectos de ejemplo y los recursos incluidos para ejecutar la aplicación lógica de ejemplo, consulte [Implementación de la infraestructura](#deploy-infrastructure).

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

Para configurar una canalización de compilación basada en el tipo de proyecto de aplicación lógica, siga las acciones correspondientes:

* Basado en NuGet: la estructura del proyecto basado en NuGet se basa en .NET Framework. Para compilar estos proyectos, asegúrese de seguir los pasos de compilación de .NET Standard. Para más información, consulte [Creación de un paquete NuGet mediante MSBuild](/nuget/create-packages/creating-a-package-msbuild).

* Basado en agrupación: el proyecto basado en agrupación de extensiones no es específico del lenguaje y, por tanto, no requiere ningún paso de compilación específico de este. Puede usar cualquier método para comprimir los archivos del proyecto.

  > [!IMPORTANT]
  > Asegúrese de que el archivo .zip incluya todas las carpetas del flujo de trabajo, los archivos de configuración, como host.json, y connections.json, y cualquier otro archivo relacionado.

### <a name="release-to-azure"></a>Publicación en Azure

Para configurar una canalización de versión que se implemente en Azure, elija la opción asociada de GitHub, Azure DevOps o CLI de Azure.

> [!NOTE]
> Azure Logic Apps actualmente no admite ranuras de implementación de Azure.

#### <a name="github"></a>[GitHub](#tab/github)

En el caso de las implementaciones de GitHub, puede implementar la aplicación lógica mediante [Acciones de GitHub](https://docs.github.com/actions), por ejemplo, la acción de GitHub en Azure Functions. Esta acción requiere que recorra la siguiente información:

* El artefacto de compilación
* El nombre de la aplicación lógica que se usará para su implementación
* El perfil de publicación

```yaml
- name: 'Run Azure Functions Action'
  uses: Azure/functions-action@v1
  id: fa
  with:
   app-name: {your-logic-app-name}
   package: '{your-build-artifact}.zip'
   publish-profile: {your-logic-app-publish-profile}
```

Para más información, revise la documentación [Entrega continua mediante la acción de GitHub](../azure-functions/functions-how-to-github-actions.md).

#### <a name="azure-devops"></a>[Azure DevOps](#tab/azure-devops)

En las implementaciones de Azure DevOps, puede implementar la aplicación lógica mediante la [tarea de implementación de aplicaciones de funciones de Azure](/devops/pipelines/tasks/deploy/azure-function-app) en Azure Pipelines. Esta acción requiere que recorra la siguiente información:

* El artefacto de compilación
* El nombre de la aplicación lógica que se usará para su implementación
* El perfil de publicación

```yaml
- task: AzureFunctionApp@1
  displayName: 'Deploy logic app workflows'
  inputs:
     azureSubscription: '{your-service-connection}'
     appType: 'workflowapp'
     appName: '{your-logic-app-name}'
     package: '{your-build-artifact}.zip'
     deploymentMethod: 'zipDeploy'
```

Para más información, consulte la documentación [Implementación de una función de Azure mediante Azure Pipelines](/devops/pipelines/targets/azure-functions-windows).

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Si usa otras herramientas de implementación, puede implementar la aplicación lógica mediante los comandos de la CLI de Azure para Azure Logic Apps de inquilino único. Por ejemplo, para implementar el artefacto comprimido en un grupo de recursos de Azure, ejecute el siguiente comando de la CLI:

`az logicapp deployment source config-zip -g {your-resource-group} --name {your-logic-app-name} --src {your-build-artifact}.zip`

---

### <a name="release-to-containers"></a>Publicación en contenedores

Si la aplicación lógica se incluye en contenedores, la implementación funciona principalmente igual que cualquier otro contenedor que implemente y administre.

Para ver ejemplos que muestran cómo implementar una canalización de implementación y compilación de contenedor de un extremo a otro, revise [CI/CD para Containers](https://azure.microsoft.com/solutions/architecture/cicd-for-containers/).

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de DevOps para Azure Logic Apps de inquilino único](devops-deployment-single-tenant-azure-logic-apps.md)

Además, nos gustaría conocer sus experiencias con Azure Logic Apps de inquilino único.

- En caso de errores o problemas, [dirija los problemas a GitHub](https://github.com/Azure/logicapps/issues).
- Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/logicappsdevops).