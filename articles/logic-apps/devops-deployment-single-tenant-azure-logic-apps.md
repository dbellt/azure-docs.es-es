---
title: Implementación de DevOps para Azure Logic Apps de inquilino único
description: Obtenga información sobre la implementación de DevOps para Azure Logic Apps de inquilino único.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: conceptual
ms.date: 05/25/2021
ms.openlocfilehash: bae25b29fb0244641f3f1db80d8f2679d2070777
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369118"
---
# <a name="devops-deployment-for-single-tenant-azure-logic-apps"></a>Implementación de DevOps para Azure Logic Apps de inquilino único

Con la tendencia hacia las aplicaciones en la nube distribuidas y nativas, las organizaciones están tratando con componentes más distribuidos en más entornos. Para mantener el control y la coherencia, puede automatizar los entornos e implementar más componentes de forma más rápida y segura mediante herramientas y procesos de DevOps.

En este artículo se proporciona una introducción e información general sobre la experiencia actual de integración continua e implementación continua (CI/CD) para aplicaciones de Azure Logic Apps de inquilino único.

<a name="single-tenant-versus-multi-tenant"></a>

## <a name="single-tenant-versus-multi-tenant"></a>Inquilino único frente a multiinquilino

En Azure Logic Apps *multiinquilino*, la implementación de recursos se basa en las plantillas de Azure Resource Manager (plantillas de ARM), que combinan y controlan el aprovisionamiento de recursos para las aplicaciones lógicas y la infraestructura. En Azure Logic Apps de *inquilino único*, la implementación es más fácil porque puede separar el aprovisionamiento de recursos entre las aplicaciones y la infraestructura.

Cuando se crean aplicaciones lógicas con el tipo de recurso **Logic App (estándar)** , los flujos de trabajo tienen la tecnología del entorno de ejecución de Azure Logic Apps de inquilino único rediseñado. El entorno de ejecución usa el [modelo de extensibilidad de Azure Functions](../azure-functions/functions-bindings-register.md) y se [hospeda como una extensión en el entorno de ejecución de Azure Functions](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564). Este diseño proporciona portabilidad, flexibilidad y más rendimiento para las aplicaciones lógicas, además de otras funcionalidades y ventajas heredadas de la plataforma Azure Functions y el ecosistema Azure App Service.

Por ejemplo, puede empaquetar el entorno de ejecución y los flujos de trabajo en contenedores rediseñados juntos como parte de la aplicación lógica. Puede usar pasos genéricos o tareas que compilan, ensamblan y comprimen los recursos de la aplicación lógica en artefactos listos para implementarse. Para implementar las aplicaciones, copie los artefactos en el entorno host y, a continuación, inicie las aplicaciones para ejecutar los flujos de trabajo. O bien, integre los artefactos en canalizaciones de implementación mediante las herramientas y los procesos que ya conoce y usa. Por ejemplo, si el escenario requiere contenedores, puede crear incluir en contenedores las aplicaciones lógicas e integrarlas en las canalizaciones existentes.

Para configurar e implementar los recursos de infraestructura, como redes virtuales y conectividad, puede seguir usando plantillas de ARM y aprovisionar por separado esos recursos junto con otros procesos y canalizaciones que use para esos fines. 

Mediante el uso de las opciones de compilación e implementación estándar, puede centrarse en el desarrollo de aplicaciones por separado de la implementación de infraestructura. Como resultado, obtiene un modelo de proyecto más genérico en el que se pueden aplicar muchas opciones de implementación similares o las mismas que se usan para una aplicación genérica. También se beneficia de una experiencia más coherente para crear canalizaciones de implementación en torno a los proyectos de aplicación y para ejecutar las pruebas y validaciones necesarias antes de publicar en producción. Independientemente de la pila de tecnología que use, puede implementar aplicaciones lógicas con sus propias herramientas elegidas.

<a name="devops-deployment-features"></a>

## <a name="devops-deployment-capabilities"></a>Funcionalidades de implementación de DevOps

La plataforma Azure Logic Apps de inquilino único hereda muchas funcionalidades y ventajas de la plataforma Azure Functions y el ecosistema Azure App Service. Estas actualizaciones incluyen todo un nuevo modelo de implementación y más formas de usar DevOps para los flujos de trabajo de la aplicación lógica.

<a name="local-development-testing"></a>

### <a name="local-development-and-testing"></a>Desarrollo y pruebas locales

Al usar Visual Studio Code con la extensión Azure Logic Apps (Estándar), puede desarrollar, compilar y ejecutar localmente flujos de trabajo de aplicación lógica basados en un solo inquilino en el entorno de desarrollo sin tener que implementar en Azure. También puede ejecutar los flujos de trabajo en cualquier lugar que pueda ejecutar Azure Functions. Por ejemplo, si el escenario requiere contenedores, puede incluir en contenedores las aplicaciones lógicas e implementarlas como contenedores.

Esta funcionalidad es una mejora importante y proporciona una ventaja considerable en comparación con el modelo multiinquilino, que requiere que se desarrolle con un recurso existente y en ejecución en Azure.

<a name="separate-concerns"></a>

### <a name="separate-concerns"></a>Separación de las preocupaciones

El modelo de inquilino único ofrece la capacidad de separar las preocupaciones entre la aplicación y la infraestructura subyacente. Por ejemplo, puede desarrollar, compilar, comprimir e implementar la aplicación por separado como un artefacto inmutable en diferentes entornos. Normalmente, los flujos de trabajo de aplicación lógica tienen "código de aplicación" que se actualiza con más frecuencia que la infraestructura subyacente. Al separar estas capas, puede centrarse más en la creación del flujo de trabajo de la aplicación lógica y dedicar menos esfuerzo a implementar los recursos necesarios en varios entornos.

![Diagrama conceptual que muestra canalizaciones de implementación independientes para aplicaciones e infraestructura.](./media/devops-deployment-single-tenant/deployment-pipelines-logic-apps.png)

<a name="resource-structure"></a>

### <a name="logic-app-resource-structure"></a>Estructura de recursos de la aplicación lógica

[!INCLUDE [Visual Studio Code - logic app resource structure](../../includes/logic-apps-single-tenant-resource-structure.md)]

### <a name="logic-app-project-structure"></a>Estructura de proyecto de la aplicación lógica

[!INCLUDE [Visual Studio Code - logic app project structure](../../includes/logic-apps-single-tenant-project-structure-visual-studio-code.md)]

<a name="deployment-containers"></a>

### <a name="container-deployment"></a>Implementación de contenedor

Las aplicaciones de Azure Logic Apps de inquilino único admiten la implementación en contenedores, lo que significa que puede incluir en contenedores los flujos de trabajo de la aplicación lógica y ejecutarlos en cualquier lugar en el que puedan ejecutarse los contenedores. Después de incluir la aplicación lógica en un contenedor, la implementación funciona principalmente igual que cualquier otro contenedor que implemente y administre.

Para obtener ejemplos que incluyen Azure DevOps, revise [CI/CD para contenedores](https://azure.microsoft.com/solutions/architecture/cicd-for-containers/).

<a name="app-settings-parameters"></a>

### <a name="app-settings-and-parameters"></a>Configuración y parámetros de la aplicación

En Azure Logic Apps multiinquilino, la plantilla de ARM supone un desafío cuando tiene que mantener variables de entorno para aplicaciones lógicas en varios entornos de desarrollo, pruebas y producción. Todo lo que hay en una plantilla de ARM se define en la implementación. Si necesita cambiar solo una variable, tendrá que volver a implementarlo todo.

En el entorno de Azure Logic Apps de inquilino único, puede llamar a las variables de entorno en el entorno de ejecución y hacer referencia a estas mediante la configuración y los parámetros de la aplicación, por lo que no tiene que volver a implementar con tanta frecuencia.

<a name="managed-connectors-built-in-operations"></a>

## <a name="managed-connectors-and-built-in-operations"></a>Conectores administrados y operaciones integradas

El ecosistema Azure Logic Apps proporciona [cientos de conectores administrados por Microsoft](/connectors/connector-reference/connector-reference-logicapps-connectors) y operaciones integradas como parte de una colección en constante crecimiento que se puede usar en aplicaciones Azure Logic Apps de inquilino único. La forma en que Microsoft mantiene estos conectores y operaciones integradas permanece principalmente igual en el caso de las aplicaciones Azure Logic Apps de inquilino único.

La mejora más significativa es que el servicio de inquilino único hace que los conectores administrados más populares también estén disponibles como operaciones integradas. Por ejemplo, puede usar operaciones integradas para Azure Service Bus, Azure Event Hubs y otros. Mientras tanto, las versiones del conector administrado siguen estando disponibles y siguen funcionando.

Las conexiones que se crean mediante operaciones integradas se denominan conexiones integradas, o *conexiones de proveedor de servicios*. Las operaciones integradas y sus conexiones se ejecutan localmente en el mismo proceso que ejecuta los flujos de trabajo. Ambos se hospedan en el entorno de ejecución de Logic Apps rediseñado. Por el contrario, las conexiones administradas, o las conexiones de API, se crean y ejecutan por separado como recursos de Azure, que se implementan mediante plantillas de ARM. Como resultado, las operaciones integradas y sus conexiones proporcionan un mejor rendimiento debido a su proximidad a los flujos de trabajo. Este diseño también funciona bien con las canalizaciones de implementación porque las conexiones del proveedor de servicios se empaquetan en el mismo artefacto de compilación.

En Visual Studio Code, cuando se usa el diseñador para desarrollar o realizar cambios en los flujos de trabajo, el motor de Logic Apps genera automáticamente los metadatos de conexión necesarios en el archivo **connections.json** del proyecto. En las secciones siguientes se describen los tres tipos de conexiones que puede crear en los flujos de trabajo. Cada tipo de conexión tiene una estructura JSON diferente, que es importante comprender porque los puntos de conexión cambian al moverse entre entornos.

<a name="service-provider-connections"></a>

### <a name="service-provider-connections"></a>Conexiones del proveedor de servicios

Cuando se usa una operación integrada para un servicio como Azure Service Bus o Azure Event Hubs en aplicaciones Azure Logic Apps de inquilino único, se crea una conexión de proveedor de servicios que se ejecuta en el mismo proceso que el flujo de trabajo. Esta infraestructura de conexión se hospeda y administra como parte de la aplicación lógica, y la configuración de la aplicación almacena las cadenas de conexión para cualquier operación integrada basada en el proveedor de servicios que usan los flujos de trabajo.

En el proyecto de aplicación lógica, cada flujo de trabajo tiene un archivo workflow.json que contiene la definición JSON subyacente del flujo de trabajo. A continuación, esta definición de flujo de trabajo hace referencia a las cadenas de conexión necesarias en el archivo connections.json del proyecto.

En el ejemplo siguiente se muestra cómo aparece la conexión del proveedor de servicios para una operación de Service Bus integrada en el archivo connections.json del proyecto:

```json
"serviceProviderConnections": {
   "{service-bus-connection-name}": {
      "parameterValues": {
         "connectionString": "@appsetting('servicebus_connectionString')"
      },
      "serviceProvider": {
         "id": "/serviceProviders/serviceBus"
      },
      "displayName": "{service-bus-connection-name}"
   },
   ...
}
```

<a name="managed-connections"></a>

### <a name="managed-connections"></a>Conexiones administradas

Cuando use un conector administrado por primera vez en el flujo de trabajo, se le pedirá que cree una conexión de API administrada para el servicio o sistema de destino y autentique su identidad. Estos conectores se administran por el ecosistema de conectores compartidos en Azure. Las conexiones de API existen y se ejecutan como recursos independientes en Azure.

En Visual Studio Code, mientras continúa creando y desarrollando el flujo de trabajo mediante el diseñador, el motor de Logic Apps crea automáticamente los recursos necesarios en Azure para los conectores administrados del flujo de trabajo. El motor agrega automáticamente estos recursos de conexión al grupo de recursos de Azure que ha diseñado para contener la aplicación lógica.

En el ejemplo siguiente se muestra cómo aparece una conexión de API para el conector de Service Bus administrado en el archivo connections.json del proyecto:

```json
"managedApiConnections": {
   "{service-bus-connection-name}": { 
      "api": {
         "id": "/subscriptions/{subscription-ID}/providers/Microsoft.Web/locations/{region}/managedApis/servicebus"
      },
      "connection": { 
         "id": "/subscriptions/{subscription-ID}/resourcegroups/{resource-group-name}/providers/Microsoft.Web/connections/servicebus"
      }, 
      "connectionRuntimeUrl": "{connection-runtime-URL}",
      "authentication": { 
         "type": "Raw",
         "scheme": "Key",
         "parameter": "@appsetting('servicebus_1-connectionKey')"
      },
   },
   <...>
}
```

<a name="azure-functions-connections"></a>

### <a name="azure-functions-connections"></a>Conexiones de Azure Functions

Para llamar a funciones creadas y hospedadas en Azure Functions, use la operación de Azure Functions integrada. Los metadatos de conexión para las llamadas de Azure Functions son diferentes a los de otras conexiones integradas. Estos metadatos se almacenan en el archivo connections.json del proyecto de aplicación lógica, pero tienen un aspecto diferente:

```json
"functionConnections": {
   "{function-operation-name}": {
      "function": { 
         "id": "/subscriptions/{subscription-ID}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{function-app-name}/functions/{function-name}"
      },
      "triggerUrl": "{function-url}",
      "authentication": {
        "type": "QueryString",
         "name": "Code",
         "value": "@appsetting('azureFunctionOperation_functionAppKey')"
      }, 
      "displayName": "{functions-connection-display-name}"
   },
   <...>
}
```

## <a name="authentication"></a>Authentication

En Azure Logic Apps de inquilino único, el modelo de hospedaje para los flujos de trabajo de las aplicaciones lógicas es un único inquilino en el que sus cargas de trabajo se benefician de un mayor aislamiento que en el modelo multiinquilino. Además, el entorno de ejecución de Azure Logic Apps de inquilino único es portátil, lo que significa que puede ejecutar los flujos de trabajo en cualquier lugar que Azure Functions pueda ejecutar. Aun así, este diseño requiere una manera de que las aplicaciones lógicas autentiquen su identidad para que puedan acceder al ecosistema de conectores administrados en Azure. Las aplicaciones también necesitan los permisos correctos para ejecutar operaciones cuando se usan conexiones administradas.

De forma predeterminada, cada aplicación lógica basada en un inquilino único tiene una identidad administrada asignada por el sistema habilitada automáticamente. Esta identidad se diferencia de las credenciales de autenticación o de la cadena de conexión que se usan al crear una conexión. En el entorno de ejecución, la aplicación lógica usa esta identidad para autenticar sus conexiones a través de directivas de acceso de Azure. Si deshabilita esta identidad, las conexiones no funcionarán en runtime.

En las secciones siguientes se proporciona más información sobre los tipos de autenticación que puede usar para autenticar las conexiones administradas, en función de dónde se ejecuta la aplicación lógica. Para cada conexión administrada, el archivo connections.json del proyecto de aplicación lógica tiene un objeto `authentication` que especifica el tipo de autenticación que la aplicación lógica puede usar para autenticar esa conexión administrada.

### <a name="managed-identity"></a>Identidad administrada

En el caso de una aplicación lógica hospedada y ejecutada en Azure, una [identidad administrada](create-managed-service-identity.md) es el tipo de autenticación predeterminado y recomendado que se usará para autenticar las conexiones administradas hospedadas y ejecutadas en Azure. En el archivo connections.json del proyecto de aplicación lógica, la conexión administrada tiene un objeto `authentication` que especifica `ManagedServiceIdentity` como tipo de autenticación:

```json
"authentication": {
   "type": "ManagedServiceIdentity"
}
```

### <a name="raw"></a>Raw

En el caso de las aplicaciones lógicas que se ejecutan en el entorno de desarrollo local mediante Visual Studio Code, las claves de autenticación sin formato se usan para autenticar las conexiones administradas hospedadas y ejecutadas en Azure. Estas claves están diseñadas solo para el desarrollo, no para producción, y expiran en 7 días. En el archivo connections.json del proyecto de aplicación lógica, la conexión administrada tiene un objeto `authentication` que especifica la siguiente información de autenticación:

```json
"authentication": {
   "type": "Raw", 
   "scheme": "Key", 
   "parameter": "@appsetting('connectionKey')"
 }
```

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la implementación de DevOps para Azure Logic Apps de inquilino único](set-up-devops-deployment-single-tenant-azure-logic-apps.md)
