---
title: Guía para desarrollar Azure Functions
description: Obtenga información sobre los conceptos y las técnicas de Azure Functions que necesita para desarrollar funciones en Azure, en todos los lenguajes de programación y enlaces.
ms.assetid: d8efe41a-bef8-4167-ba97-f3e016fcd39e
ms.topic: conceptual
ms.date: 10/12/2017
ms.openlocfilehash: a526edfccda1e4e0e60646989a59d23ad19501ab
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108227117"
---
# <a name="azure-functions-developer-guide"></a>Guía para desarrolladores de Azure Functions
En Azure Functions, determinadas funciones comparten algunos componentes y conceptos técnicos básicos, independientemente del idioma o el enlace que use. Antes de ir a detalles de aprendizaje específicos de un idioma o un enlace determinados, asegúrese de leer al completo esta información general que se aplica a todos ellos.

En este artículo se supone que ya ha leído la [Información general sobre Azure Functions](functions-overview.md).

## <a name="function-code"></a>Código de función
Una *función* es el concepto principal en las funciones de Azure. Una función contiene dos elementos importantes: el código, que se puede escribir en diversos lenguajes, y la configuración, el archivo function.json. Con los lenguajes compilados, este archivo de configuración se genera automáticamente a partir de las anotaciones del código. Para los lenguajes de scripting, debe proporcionar el archivo de configuración.

El archivo function.json define el desencadenador de la función, los enlaces y otras opciones de configuración. Cada función tiene un solo desencadenador. Este archivo de configuración se usa en tiempo de ejecución para determinar los eventos que se supervisarán y cómo pasar datos y devolverlos al ejecutarse una función. El siguiente es un ejemplo de archivo function.json.

```json
{
    "disabled":false,
    "bindings":[
        // ... bindings here
        {
            "type": "bindingType",
            "direction": "in",
            "name": "myParamName",
            // ... more depending on binding
        }
    ]
}
```

Para más información, consulte [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md).

La propiedad `bindings` es donde configura los enlaces y los desencadenadores. Cada enlace comparte unos ajustes de configuración comunes y algunos parámetros que son específicos de un determinado tipo de enlace. Cada enlace requiere la siguiente configuración:

| Propiedad    | Valores | Tipo | Comentarios|
|---|---|---|---|
| type  | Nombre del enlace.<br><br>Por ejemplo, `queueTrigger`. | string | |
| direction | `in`, `out`  | string | Indica si el enlace está disponible para recibir datos en la función o enviar datos de la función. |
| name | Identificador de función.<br><br>Por ejemplo, `myQueue`. | string | El nombre que se usa para los datos enlazados en la función. En C# es un nombre de argumento; en JavaScript es la clave en una lista de clave-valor. |

## <a name="function-app"></a>Aplicación de función
La aplicación de función proporciona un contexto de ejecución en Azure donde ejecutar las funciones. Como tal, es la unidad de implementación y administración de las funciones. Una aplicación de función se compone de una o varias funciones individuales que se administran, implementan y escalan conjuntamente. Todas las funciones de una aplicación de función comparten el mismo plan de precios, el mismo método de implementación y la misma versión en tiempo de ejecución. Una aplicación de función es como una forma de organizar y administrar las funciones de manera colectiva. Para obtener más información, vea [Administración de una Function App en Azure Portal](functions-how-to-use-azure-function-app-settings.md). 

> [!NOTE]
> Todas las funciones de una aplicación de función deben crearse en el mismo lenguaje. En [versiones anteriores](functions-versions.md) del tiempo de ejecución de Azure Functions, esto no era necesario.

## <a name="folder-structure"></a>Estructura de carpetas
[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

La estructura de carpetas anterior es la predeterminada (y recomendada) para una aplicación de función. Si desea cambiar la ubicación del archivo de código de una función, modifique la sección `scriptFile` del archivo _function.json_. Asimismo, para implementar el proyecto en la aplicación de función en Azure, se recomienda usar la [implementación de paquetes](deployment-zip-push.md). También puede usar herramientas existentes como la [integración e implementación continuas](functions-continuous-deployment.md) y Azure DevOps.

> [!NOTE]
> Si implementa un paquete manualmente, asegúrese de implementar el archivo _host.json_ y las carpetas de la función directamente en la carpeta `wwwroot`. No incluya la carpeta `wwwroot` en sus implementaciones. De lo contrario, acabará con carpetas `wwwroot\wwwroot`.

#### <a name="use-local-tools-and-publishing"></a>Uso de herramientas locales y publicación
Las aplicaciones de función pueden crearse y publicarse con diversas herramientas, incluidas [Visual Studio](./functions-develop-vs.md), [Visual Studio Code](./create-first-function-vs-code-csharp.md), [IntelliJ](./functions-create-maven-intellij.md), [Eclipse](./functions-create-maven-eclipse.md) y [Azure Functions Core Tools](./functions-develop-local.md). Para más información, consulte [Codificación y comprobación de las funciones de Azure Functions en un entorno local](./functions-develop-local.md).

<!--NOTE: I've removed documentation on FTP, because it does not sync triggers on the consumption plan --glenga -->

## <a name="how-to-edit-functions-in-the-azure-portal"></a><a id="fileupdate"></a> Edición de funciones en Azure Portal
El editor de funciones integrado en Azure Portal le permite actualizar el código y el archivo *function.json* directamente. Esto solo se recomienda para pequeños cambios o pruebas de concepto: el procedimiento recomendado es usar una herramienta de desarrollo local como VS Code.

## <a name="parallel-execution"></a>Ejecución en paralelo
Cuando se producen varios eventos de desencadenado más rápido de lo que un tiempo de ejecución de función de un solo subproceso pueda procesarlos, el tiempo de ejecución puede invocar la función varias veces en paralelo.  Si una aplicación de función usa el [plan de hospedaje de consumo](event-driven-scaling.md), esta aplicación podría escalarse horizontalmente de forma automática.  Cada instancia de la aplicación de función, tanto si la aplicación se ejecuta en el plan de hospedaje de consumo como en el [plan de hospedaje de App Service](../app-service/overview-hosting-plans.md) normal, puede procesar invocaciones de función simultáneas en paralelo mediante varios subprocesos.  El número máximo de invocaciones de función simultáneas en cada instancia de aplicación de función varía según el tipo de desencadenador usado y lo recursos empleados por otras funciones dentro de la aplicación de función.

## <a name="functions-runtime-versioning"></a>Versiones del entorno en tiempo de ejecución de Functions

Puede configurar la versión del entorno en tiempo de ejecución de Functions mediante la configuración de la aplicación `FUNCTIONS_EXTENSION_VERSION`. Por ejemplo, el valor "~3" indica que la aplicación de funciones utilizará 3.x como versión principal. Las aplicaciones de funciones se actualizan a las nuevas versiones secundarias a medida que se lanzan. Para más información y saber cómo ver la versión exacta de la aplicación de función, consulte [Cómo seleccionar un destino para versiones en tiempo de ejecución de Azure Functions](set-runtime-version.md).

## <a name="repositories"></a>Repositorios
El código de Azure Functions es código abierto y está almacenado en repositorios de GitHub:

* [Funciones de Azure](https://github.com/Azure/Azure-Functions)
* [Host de Azure Functions](https://github.com/Azure/azure-functions-host/)
* [Portal de Azure Functions](https://github.com/azure/azure-functions-ux)
* [Plantillas de Azure Functions](https://github.com/azure/azure-functions-templates)
* [SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/)
* [Extensiones del SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk-extensions/)

## <a name="bindings"></a>Enlaces
Esta es una tabla de todos los enlaces admitidos.

[!INCLUDE [dynamic compute](../../includes/functions-bindings.md)]

¿Tiene algún problema con errores procedentes de los enlaces? Revise la documentación de los [códigos de error de los enlaces de Azure Functions](functions-bindings-error-pages.md).


## <a name="connections"></a>Conexiones

El proyecto de función hace referencia a la información de conexión por nombre de su proveedor de configuración. No acepta directamente los detalles de conexión, lo que permite que se modifiquen en todos los entornos. Por ejemplo, una definición de desencadenador podría incluir una propiedad `connection`. Esta podría hacer referencia a una cadena de conexión, pero no puede establecer la cadena de conexión directamente en un objeto `function.json`. En su lugar, debe establecer `connection` en el nombre de una variable de entorno que contenga la cadena de conexión.

El proveedor de configuración predeterminado utiliza variables de entorno. Estas podrían establecerse mediante la [Configuración de la aplicación](./functions-how-to-use-azure-function-app-settings.md?tabs=portal#settings) cuando se ejecutan en el servicio Azure Functions, o desde el [archivo de configuración local](functions-run-local.md#local-settings-file) cuando el desarrollo se realiza localmente.

### <a name="connection-values"></a>Valores de conexión

Cuando el nombre de la conexión se resuelve en un solo valor exacto, el tiempo de ejecución identifica el valor como una _cadena de conexión_, que normalmente incluye un secreto. Los detalles de una cadena de conexión se definen mediante el servicio al que quiere conectarse.

Sin embargo, un nombre de conexión también puede hacer referencia a una colección de varios elementos de configuración. Las variables de entorno se pueden tratar como una colección mediante un prefijo compartido que termina en doble carácter de subrayado `__`. A continuación, se puede hacer referencia al grupo. Para ello, establezca el nombre de la conexión en este prefijo.

Por ejemplo, la propiedad `connection` de una definición de desencadenador de blob de Azure podría ser `Storage1`. Siempre que no haya ningún valor de cadena único configurado con `Storage1` como su nombre, se usará `Storage1__serviceUri` para la propiedad `serviceUri` de la conexión. Las propiedades de conexión son diferentes para cada servicio. Consulte la documentación de la extensión que utiliza la conexión.

### <a name="configure-an-identity-based-connection"></a>Configuración de una conexión basada en identidades

Algunas conexiones de Azure Functions están configuradas para usar una identidad en lugar de un secreto. La compatibilidad depende de la extensión que utiliza la conexión. En algunos casos, es posible que aún se necesite una cadena de conexión en Functions aunque el servicio al que se está conectando admita conexiones basadas en identidades.

> [!IMPORTANT]
> Aunque una extensión de enlace admita conexiones basadas en identidades, es posible que esa configuración no se admita todavía en el plan de consumo. Consulta la tabla de compatibilidad a continuación.

Las conexiones basadas en identidades son compatibles con las siguientes extensiones de desencadenador y enlace:

| Nombre de la extensión | Versión de la extensión                                                                                     | Compatible con el plan de consumo |
|----------------|-------------------------------------------------------------------------------------------------------|---------------------------------------|
| Blob de Azure     | [Versión 5.0.0-beta1 o posterior](./functions-bindings-storage-blob.md#storage-extension-5x-and-higher)  | No                                    |
| Azure Queue    | [Versión 5.0.0-beta1 o posterior](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher) | No                                    |
| Azure Event Hubs    | [Versión 5.0.0-beta1 o posterior](./functions-bindings-event-hubs.md#event-hubs-extension-5x-and-higher) | No                                    |
| Azure Service Bus    | [Versión 5.0.0-beta2 o posterior](./functions-bindings-service-bus.md#service-bus-extension-5x-and-higher) | No                                    |

> [!NOTE]
> La compatibilidad con conexiones basadas en identidades todavía no está disponible para las conexiones de almacenamiento que usa el tiempo de ejecución de Functions para los comportamientos básicos. Esto significa que la opción de configuración `AzureWebJobsStorage` debe ser una cadena de conexión.

#### <a name="connection-properties"></a>Propiedades de la conexión

Una conexión basada en identidades para un servicio de Azure acepta las siguientes propiedades:

| Propiedad    | Necesario para las extensiones | Variable de entorno | Descripción |
|---|---|---|---|
| URI de servicio | Blob de Azure, Cola de Azure | `<CONNECTION_NAME_PREFIX>__serviceUri` |  URI del plano de datos del servicio al que se está conectando. |
| Espacio de nombres completo | Event Hubs, Service Bus | `<CONNECTION_NAME_PREFIX>__fullyQualifiedNamespace` | Espacio de nombres completo de Event Hubs y Service Bus. |

Es posible que se admitan opciones adicionales para un tipo de conexión determinado. Consulte la documentación del componente que realiza la conexión.

Cuando se hospeda en el servicio de Azure Functions, las conexiones basadas en identidades usan una [identidad administrada](../app-service/overview-managed-identity.md?toc=%2fazure%2fazure-functions%2ftoc.json). Se usa la identidad asignada por el sistema de forma predeterminada. Cuando se ejecuta en otros contextos, como el desarrollo local, se usa la identidad del desarrollador en su lugar, aunque se puede personalizar mediante parámetros de conexión alternativos.

##### <a name="local-development"></a>Desarrollo local

Cuando se ejecuta localmente, la configuración anterior indica al tiempo de ejecución que use la identidad del desarrollador local. La conexión intentará obtener un token de las siguientes ubicaciones, en orden:

- Una caché local compartida entre las aplicaciones de Microsoft
- El contexto del usuario actual en Visual Studio
- El contexto del usuario actual en Visual Studio Code
- El contexto del usuario actual en la CLI de Azure

Si ninguna de estas opciones produce un resultado satisfactorio, se producirá un error.

En algunos casos, puede especificar el uso de una identidad diferente. Puede agregar propiedades de configuración para la conexión que apunten a la identidad alternativa.

> [!NOTE]
> No se admiten las siguientes opciones de configuración cuando se hospedan en el servicio Azure Functions.

Para conectarse mediante una entidad de servicio de Azure Active Directory con un identificador de cliente y un secreto, defina la conexión con las siguientes propiedades requeridas, aparte de las [propiedades de conexión](#connection-properties) anteriores:

| Propiedad    | Variable de entorno | Descripción |
|---|---|---|
| Id. de inquilino | `<CONNECTION_NAME_PREFIX>__tenantId` | Id. de inquilino (directorio) de Azure Active Directory. |
| Id. de cliente | `<CONNECTION_NAME_PREFIX>__clientId` |  Id. de cliente (aplicación) de un registro de aplicación en el inquilino. |
| Secreto del cliente | `<CONNECTION_NAME_PREFIX>__clientSecret` | Secreto de cliente generado para el registro de la aplicación. |

Ejemplo de las propiedades `local.settings.json` requeridas para la conexión basada en identidades con blobs de Azure: 
```json
{
  "IsEncrypted": false,
  "Values": {
    "<CONNECTION_NAME_PREFIX>__serviceUri": "<serviceUri>",
    "<CONNECTION_NAME_PREFIX>__tenantId": "<tenantId>",
    "<CONNECTION_NAME_PREFIX>__clientId": "<clientId>",
    "<CONNECTION_NAME_PREFIX>__clientSecret": "<clientSecret>"
  }
}
```

#### <a name="grant-permission-to-the-identity"></a>Concesión de permiso a la identidad

Cualquier identidad que se utilice debe tener permisos para realizar las acciones previstas. Normalmente, esto se realiza mediante la asignación de un rol en RBAC de Azure o la especificación de la identidad en una directiva de acceso, en función del servicio al que se esté conectando. Consulte la documentación de cada servicio para conocer los permisos necesarios y el procedimiento para establecerlos.

Los siguientes roles abarcan los permisos principales necesarios para cada extensión en condiciones de funcionamiento normal:

| Servicio     | Roles integrados de ejemplo |
|-------------|------------------------|
| Azure Blobs  | [Lector de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader), [Propietario de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)                 |
| Colas de Azure | [Lector de datos de la cola de Storage Blob](../role-based-access-control/built-in-roles.md#storage-queue-data-reader), [Procesador de mensajes de datos de Queue Storage](../role-based-access-control/built-in-roles.md#storage-queue-data-message-processor), [Remitente de mensajes de datos de Queue Storage](../role-based-access-control/built-in-roles.md#storage-queue-data-message-sender), [Colaborador de datos de la cola de Storage Blob](../role-based-access-control/built-in-roles.md#storage-queue-data-contributor)             |
| Event Hubs   |    [Receptor de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver), [Remitente de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender), [Propietario de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner)              |
| Azure Service Bus | [Receptor de los datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver), [Remitente de los datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender), [Propietario de los datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) |

> [!IMPORTANT]
> Es posible que el servicio muestre algunos permisos que no son necesarios para todos los contextos. Siempre que sea posible, respete el **principio de privilegios mínimos** y conceda solo los privilegios necesarios a la identidad. Por ejemplo, si la aplicación solo necesita leer un blob, use el rol [Lector de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader), ya que el rol [Propietario de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-owner) incluye permisos excesivos para una operación de lectura.


## <a name="reporting-issues"></a>Problemas de informes
[!INCLUDE [Reporting Issues](../../includes/functions-reporting-issues.md)]

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información, consulte los siguientes recursos:

* [Enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md)
* [Codificación y comprobación de Azure Functions en un entorno local](./functions-develop-local.md)
* [Procedimientos recomendados de Azure Functions](functions-best-practices.md)
* [Referencia para desarrolladores de C# de Azure Functions](functions-dotnet-class-library.md)
* [Referencia para desarrolladores de Node.js de Azure Functions](functions-reference-node.md)
