---
title: Enlaces de Azure Service Bus en Azure Functions
description: Obtenga información sobre cómo enviar desencadenadores y enlaces de Azure Service Bus en Azure Functions.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: fasttrack-edit
ms.openlocfilehash: 4b0daecadd3af5f1322afc97f91706098aade768
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110080495"
---
# <a name="azure-service-bus-bindings-for-azure-functions"></a>Enlaces de Azure Service Bus en Azure Functions

Azure Functions se integra con [Azure Service Bus](https://azure.microsoft.com/services/service-bus) mediante [desencadenadores y enlaces](./functions-triggers-bindings.md). La integración con Service Bus permite crear funciones que reaccionan ante y envían mensajes de cola o tema.

| Acción | Tipo |
|---------|---------|
| Ejecutar una función cuando se crea un mensaje de cola o tema de Service Bus | [Desencadenador](./functions-bindings-service-bus-trigger.md) |
| Enviar mensajes de Azure Service Bus |[Enlace de salida](./functions-bindings-service-bus-output.md) |

## <a name="add-to-your-functions-app"></a>Adición a la aplicación de Functions

> [!NOTE]
> El enlace de Service Bus no admite actualmente la autenticación mediante una identidad administrada. En su lugar, use una [firma de acceso compartido de Service Bus](../service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature).

### <a name="functions-2x-and-higher"></a>Functions 2.x y superiores

Para trabajar con el desencadenador y los enlaces, es necesario hacer referencia al paquete adecuado. En las bibliotecas de clases de .NET se usa el paquete NuGet, mientras que en los demás tipos de aplicaciones se emplea el conjunto de extensiones.

| Idioma                                        | Agregar mediante...                                   | Observaciones 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Instalación del [paquete NuGet], versión 4.x | |
| Script de C#, Java, JavaScript, Python, PowerShell | Registro de [conjunto de extensiones]          | Se recomienda usar la [extensión Azure Tools] con Visual Studio Code. |
| Script de C# (solo en línea en Azure Portal)         | Adición de un enlace                            | Para actualizar extensiones de enlace existentes sin tener que volver a publicar la aplicación de funciones, consulte [Actualización de las extensiones]. |

[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.ServiceBus/
[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Actualización de las extensiones]: ./functions-bindings-register.md
[Extensión Azure Tools]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

#### <a name="service-bus-extension-5x-and-higher"></a>Extensión 5.x y superior de Service Bus

Hay disponible una nueva versión de la extensión de enlaces de Service Bus en un [paquete NuGet en versión preliminar](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.ServiceBus/5.0.0-beta.2). Esta versión preliminar presenta la posibilidad de [conectarse con una identidad en lugar de un secreto](./functions-reference.md#configure-an-identity-based-connection). En el caso de las aplicaciones .NET, también cambian los tipos con los que se puede enlazar; así, los tipos `Microsoft.ServiceBus.Messaging` y `Microsoft.Azure.ServiceBus` se reemplazan por otros tipos más recientes de [Azure.Messaging.ServiceBus](/dotnet/api/azure.messaging.servicebus).

> [!NOTE]
> El paquete en versión preliminar no se incluye en un conjunto de extensiones y debe instalarse manualmente. En el caso de las aplicaciones .NET, agregue una referencia al paquete. Con el resto de tipos de aplicaciones, consulte [Actualización de las extensiones].

[core tools]: ./functions-run-local.md
[conjunto de extensiones]: ./functions-bindings-register.md#extension-bundles
[Paquete NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Actualización de las extensiones]: ./functions-bindings-register.md
[Extensión Azure Tools]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Las aplicaciones de Functions 1.x tienen automáticamente una referencia al paquete NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), versión 2.x.


<a name="host-json"></a>  

## <a name="hostjson-settings"></a>configuración de host.json

En esta sección se describen las opciones de configuración globales disponibles para este enlace en las versiones 2.x y posteriores. El siguiente archivo host.json de ejemplo contiene solo la configuración para este enlace. Para más información acerca de las opciones de configuración globales, consulte [Referencia de host.json para Azure Functions](functions-host-json.md).

> [!NOTE]
> Para obtener una referencia de host.json en Functions 1.x, consulte la [referencia de host.json para Azure Functions, versión 1.x](functions-host-json-v1.md).

```json
{
    "version": "2.0",
    "extensions": {
        "serviceBus": {
            "prefetchCount": 100,
            "messageHandlerOptions": {
                "autoComplete": true,
                "maxConcurrentCalls": 32,
                "maxAutoRenewDuration": "00:05:00"
            },
            "sessionHandlerOptions": {
                "autoComplete": false,
                "messageWaitTimeout": "00:00:30",
                "maxAutoRenewDuration": "00:55:00",
                "maxConcurrentSessions": 16
            }
        }
    }
}
```

Si `isSessionsEnabled` se ha establecido en `true`, se respetará `sessionHandlerOptions`.  Si `isSessionsEnabled` se ha establecido en `false`, se respetará `messageHandlerOptions`.

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|prefetchCount|0|Obtiene o establece el número de mensajes que el destinatario del mensaje puede solicitar simultáneamente.|
|maxAutoRenewDuration|00:05:00|Duración máxima dentro de la cual el bloqueo de mensajes se renovará automáticamente.|
|autoComplete|true|Si el desencadenador debe llamar automáticamente a Complete después del procesamiento o si el código de la función llamará manualmente a Complete.<br><br>La configuración en `false` solo se admite en C#.<br><br>Si se establece en `true`, el desencadenador completa automáticamente el mensaje si la ejecución de la función se completa correctamente y abandona el mensaje en caso contrario.<br><br>Cuando se establece en `false`, usted es responsable de llamar a los métodos [MessageReceiver](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver) para completar, abandonar o cerrar el mensaje. Si se produce una excepción (y no se llama a ninguno de los métodos `MessageReceiver`), se mantiene el bloqueo. Una vez que el bloqueo expira, el mensaje se vuelve a poner en cola con la `DeliveryCount` incrementada y el bloqueo se renueva automáticamente.<br><br>En las funciones que no son C#, las excepciones en la función dan como resultado las llamadas en tiempo de ejecución `abandonAsync` en segundo plano. Si no se produce ninguna excepción, se llama a `completeAsync` en segundo plano. |
|maxConcurrentCalls|16|Número máximo de llamadas simultáneas a la devolución de llamada que el bombeo de mensajes debe iniciar por instancia con escala. De forma predeterminada, el entorno de ejecución de Functions procesa simultáneamente varios mensajes.|
|maxConcurrentSessions|2000|Número máximo de sesiones que se puede administrar simultáneamente por instancia con escala.|

### <a name="additional-settings-for-version-5x"></a>Configuración adicional para la versión 5.x+

El archivo host.json de ejemplo a continuación contiene solo la configuración para la versión 5.0.0 y posterior de la extensión de Service Bus.

```json
{
    "version": "2.0",
    "extensions": {
        "serviceBus": {
            "serviceBusOptions": {
                "retryOptions":{
                    "mode": "exponential",
                    "tryTimeout": "00:00:10",
                    "delay": "00:00:00.80",
                    "maxDelay": "00:01:00",
                    "maxRetries": 4
                },
                "prefetchCount": 100,
                "autoCompleteMessages": true,
                "maxAutoLockRenewalDuration": "00:05:00",
                "maxConcurrentCalls": 32,
                "maxConcurrentSessions": 10,
                "maxMessages": 2000,
                "sessionIdleTimeout": "00:01:00"
            }
        }
    }
}
```

Al usar la versión 5.x y posteriores de la extensión de Service Bus, se admiten las siguientes opciones de configuración global, además de la configuración 2.x en `ServiceBusOptions`.

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|prefetchCount|0|Obtiene o establece el número de mensajes que el destinatario del mensaje puede solicitar simultáneamente.|
|autoCompleteMessages|true|Determina si los mensajes se completan automáticamente o no después de la ejecución correcta de la función, y se debe usar en lugar de la opción de configuración `autoComplete`.|
|maxAutoLockRenewalDuration|00:05:00|Se tiene que usar en lugar de `maxAutoRenewDuration`.|
|maxConcurrentCalls|16|Número máximo de llamadas simultáneas a la devolución de llamada que el bombeo de mensajes debe iniciar por instancia con escala. De forma predeterminada, el entorno de ejecución de Functions procesa simultáneamente varios mensajes.|
|maxConcurrentSessions|8|Número máximo de sesiones que se puede administrar simultáneamente por instancia con escala.|
|maxMessages|1000|Número máximo de mensajes que se pasarán a cada llamada de función. Esto solo se aplica a las funciones que reciben un lote de mensajes.|
|sessionIdleTimeout|N/D|Cantidad máxima de tiempo que se esperará a que se reciba un mensaje para la sesión activa actualmente. Una vez transcurrido este tiempo, el procesador cerrará la sesión e intentará procesar otra sesión.|

### <a name="retry-settings"></a>Configuración de los reintentos

Además de las propiedades de configuración anteriores al usar la versión 5.x y posteriores de la extensión de Service Bus, también puede configurar `RetryOptions` desde dentro de `ServiceBusOptions`. Esta configuración determina si se debe reintentar una operación con errores y, si es así, la cantidad de tiempo que se debe esperar entre reintentos. Las opciones también controlan la cantidad de tiempo permitido para recibir mensajes y otras interacciones con el servicio Service Bus.

|Propiedad  |Valor predeterminado | Descripción |
|---------|---------|---------|
|mode|Exponencial|Enfoque que se debe usar para calcular los retrasos de reintento. El modo exponencial predeterminado aplicará un retraso a los reintentos en función de una estrategia de interrupción, en la que cada intento aumentará la duración que espera antes de volver a intentarlo. El modo `Fixed` reintentará a intervalos fijos, donde cada retraso tendrá una duración uniforme.|
|tryTimeout|00:00:10|Duración máxima que se debe esperar para una operación por intento.|
|delay|00:00:00.80|Factor de retraso o de interrupción que se aplicará entre reintentos.|
|maxDelay|00:01:00|Retraso máximo para permitir entre los reintentos.|
|maxRetries|3|Número máximo de reintentos antes de considerar que hay un error en la operación asociada.|

---

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de una función cuando se crea un mensaje de cola o tema de Service Bus (Desencadenador)](./functions-bindings-service-bus-trigger.md)
- [Envío de mensajes de Azure Service Bus desde Azure Functions (Enlace de salida)](./functions-bindings-service-bus-output.md)
