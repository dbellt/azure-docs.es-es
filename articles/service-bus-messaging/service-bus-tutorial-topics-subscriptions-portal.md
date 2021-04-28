---
title: Actualización del inventario mediante Azure Portal y temas y suscripciones
description: En este tutorial, aprenderá a enviar y recibir mensajes desde un tema y una suscripción, y a agregar y usar las reglas de filtro mediante .NET
author: spelluru
ms.author: spelluru
ms.date: 10/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: a7b03727c574dcee1cd56144a521f36de6dc48c6
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107861380"
---
# <a name="tutorial-update-inventory-using-azure-portal-and-topicssubscriptions"></a>Tutorial: Actualización del inventario mediante Azure Portal y temas y suscripciones

Microsoft Azure Service Bus es un servicio de mensajería multiinquilino en la nube que envía información entre aplicaciones y servicios. Las operaciones asincrónicas le ofrecen una mensajería flexible y asincrónica, además de funcionalidades de mensajería estructurada de tipo FIFO (primero en entrar, primero en salir) y de publicación y suscripción. Este tutorial muestra cómo utilizar los temas y suscripciones de Service Bus en un escenario de inventario comercial con canales de publicación y suscripción mediante Azure Portal y .NET.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear un tema de Service Bus y una o varias suscripciones a ese tema mediante Azure Portal
> * Agregar filtros de tema mediante código .NET
> * Crear dos mensajes con contenido diferente
> * Enviar los mensajes y comprobar que llegaron a las suscripciones esperadas
> * Recibir mensajes de las suscripciones

Un ejemplo de este escenario es una actualización del surtido de inventario de varias tiendas. En este escenario, cada tienda o conjunto de tiendas, recibe mensajes dirigidos a que actualicen sus inventarios. En este tutorial se muestra cómo implementar este escenario mediante suscripciones y filtros. En primer lugar puede crear un tema con 3 suscripciones, agregar algunas reglas y filtros y, a continuación, enviar y recibir mensajes de los temas y las suscripciones.

![topic](./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png)

Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][] antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial, asegúrese de que tiene instalados los siguientes elementos:

- [Visual Studio 2017 Update 3 (versión 15.3, 26730.01)](https://www.visualstudio.com/vs) o posterior.
- [NET Core SDK](https://dotnet.microsoft.com/download), versión 2.0 o posterior.

## <a name="service-bus-topics-and-subscriptions"></a>Temas y suscripciones de Service Bus

Cada [suscripción a un tema](service-bus-messaging-overview.md#topics) puede recibir una copia de cada mensaje. Los temas son completamente compatibles desde el punto de vista semántico y del protocolo con las colas de Service Bus. Los temas de Service Bus admiten una amplia variedad de reglas de selección con condiciones de filtro, con acciones opcionales que establecerán o modificarán las propiedades del mensaje. Cada vez que coincide una regla, esta genera un mensaje. Para más información acerca de reglas, filtros y acciones, siga este [vínculo](topic-filters.md).

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

[!INCLUDE [service-bus-create-topics-three-subscriptions-portal](../../includes/service-bus-create-topics-three-subscriptions-portal.md)]



## <a name="create-filter-rules-on-subscriptions"></a>Creación de reglas de filtro en suscripciones

Después de aprovisionar el espacio de nombres y los temas y suscripciones, y de tener las credenciales necesarias, está listo para crear reglas de filtro en las suscripciones y, posteriormente, enviar y recibir mensajes. Puede examinar el código en [esta carpeta de ejemplos de GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/TopicFilters).

## <a name="send-and-receive-messages"></a>Envío y recepción de mensajes

Para ejecutar el código, haga lo siguiente:

1. En un símbolo del sistema o el símbolo del sistema de PowerShell, clone el [repositorio de GitHub de Service Bus](https://github.com/Azure/azure-service-bus/) emitiendo el comando siguiente:

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. Vaya a la carpeta de ejemplos `azure-service-bus\samples\DotNet\Azure.Messaging.ServiceBus\BasicSendReceiveTutorialwithFilters`.

3. Obtenga la cadena de conexión que ha copiado en el Bloc de notas en la sección "Obtención de las credenciales de administración" de este tutorial. También necesita el nombre del tema que creó en la sección anterior.

4. En el símbolo del sistema, escriba el siguiente comando:

   ```shell
   dotnet build
   ```

5. Vaya a la carpeta `BasicSendReceiveTutorialwithFilters\bin\Debug\netcoreapp3.1`.

6. Escriba el siguiente comando para ejecutar el programa. No olvide reemplazar `myConnectionString` por el valor que obtuvo previamente, y `myTopicName` por el nombre del tema que creó:

   ```shell
   dotnet BasicSendReceiveTutorialwithFilters.dll -ConnectionString "myConnectionString" -TopicName "myTopicName"
   ``` 
7. Siga las instrucciones de la consola para seleccionar la creación de filtros en primer lugar. Es parte de la creación de filtros quitar los filtros predeterminados. Cuando se usa PowerShell o la CLI no es necesario quitar los filtros predeterminados, pero si lo hace en el código, deberá quitarlos. Los comandos de consola 1 y 3 le ayudarán a administrar los filtros en las suscripciones que creó anteriormente:

   - Ejecute 1: para quitar los filtros predeterminados.
   - Ejecute 2: para agregar sus propios filtros.
   - Ejecute 3: para quitar opcionalmente sus propios filtros. Tenga en cuenta que esto no volverá a crear los filtros predeterminados.

     ![Mostrar la salida de 2](./media/service-bus-tutorial-topics-subscriptions-portal/create-rules.png)

8. Después de la creación de los filtros, puede enviar mensajes. Pulse 4 y observe los 10 mensajes que se envían al tema:

    ![Enviar los resultados](./media/service-bus-tutorial-topics-subscriptions-portal/send-output.png)

9. Pulse 5 y observe los mensajes que se reciben. Si no obtuvo 10 mensajes de vuelta, pulse "m" para mostrar el menú y, a continuación, pulse 5 de nuevo.

    ![Recibir salida](./media/service-bus-tutorial-topics-subscriptions-portal/receive-output.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el espacio de nombres y el tema, elimínelos. Para ello, seleccione estos recursos en el portal y haga clic en **Eliminar**.

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

Esta sección contiene información más detallada acerca de lo que hace el código de ejemplo.

### <a name="get-connection-string-and-topic"></a>Obtención de la cadena de conexión y el tema

En primer lugar, el código declara un conjunto de variables que dirige la ejecución del resto del programa.

```csharp
string ServiceBusConnectionString;
string TopicName;

static string[] Subscriptions = { "S1", "S2", "S3" };
static IDictionary<string, string[]> SubscriptionFilters = new Dictionary<string, string[]> {
    { "S1", new[] { "StoreId IN('Store1', 'Store2', 'Store3')", "StoreId = 'Store4'"} },
    { "S2", new[] { "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'" } },
    { "S3", new[] { "sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')" } }
};
// You can have only have one action per rule and this sample code supports only one action for the first filter, which is used to create the first rule. 
static IDictionary<string, string> SubscriptionAction = new Dictionary<string, string> {
    { "S1", "" },
    { "S2", "" },
    { "S3", "SET sys.Label = 'SalesEvent'"  }
};
static string[] Store = { "Store1", "Store2", "Store3", "Store4", "Store5", "Store6", "Store7", "Store8", "Store9", "Store10" };
static string SysField = "sys.To";
static string CustomField = "StoreId";
static int NrOfMessagesPerStore = 1; // Send at least 1.
```

Los nombres de la cadena de conexión y del tema se pasan mediante parámetros de la línea de comandos como se indica a continuación y, después, se leen en el método `Main()`:

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string TopicName = "";

    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i + 1]}");
            ServiceBusConnectionString = args[i + 1]; // Alternatively enter your connection string here.
        }
        else if (args[i] == "-TopicName")
        {
            Console.WriteLine($"TopicName: {args[i + 1]}");
            TopicName = args[i + 1]; // Alternatively enter your queue name here.
        }
    }

    if (ServiceBusConnectionString != "" && TopicName != "")
    {
        Program P = StartProgram(ServiceBusConnectionString, TopicName);
        P.PresentMenu().GetAwaiter().GetResult();
    }
    else
    {
        Console.WriteLine("Specify -Connectionstring and -TopicName to execute the example.");
        Console.ReadKey();
    }
}
```

### <a name="remove-default-filters"></a>Eliminación de los filtros predeterminados

Cuando se crea una suscripción, Service Bus crea un filtro predeterminado por suscripción. Este filtro permite recibir todos los mensajes enviados al tema. Si desea usar filtros personalizados, puede quitar el filtro predeterminado tal como se muestra en el código siguiente:

```csharp
private async Task RemoveDefaultFilters()
{
    Console.WriteLine($"Starting to remove default filters.");

    try
    {
        var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
        foreach (var subscription in Subscriptions)
        {
            await client.DeleteRuleAsync(TopicName, subscription, CreateRuleOptions.DefaultRuleName);
            Console.WriteLine($"Default filter for {subscription} has been removed.");
        }

        Console.WriteLine("All default Rules have been removed.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="create-filters"></a>Creación de filtros

El código siguiente agrega los filtros personalizados definidos en este tutorial:

```csharp
private async Task CreateCustomFilters()
{
    try
    {
        for (int i = 0; i < Subscriptions.Length; i++)
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            string[] filters = SubscriptionFilters[Subscriptions[i]];
            if (filters[0] != "")
            {
                int count = 0;
                foreach (var myFilter in filters)
                {
                    count++;

                    string action = SubscriptionAction[Subscriptions[i]];
                    if (action != "")
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Action = new SqlRuleAction(action),
                            Name = $"MyRule{count}"
                        });
                    }
                    else
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Name = $"MyRule{count}"
                        });
                    }
                }
            }

            Console.WriteLine($"Filters and actions for {Subscriptions[i]} have been created.");
        }

        Console.WriteLine("All filters and actions have been created.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="remove-your-custom-created-filters"></a>Eliminación de los filtros personalizados creados

Si desea quitar todos los filtros de la suscripción, en el código siguiente se muestra cómo hacerlo:

```csharp
private async Task CleanUpCustomFilters()
{
    foreach (var subscription in Subscriptions)
    {
        try
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            IAsyncEnumerator<RuleProperties> rules = client.GetRulesAsync(TopicName, subscription).GetAsyncEnumerator();
            while (await rules.MoveNextAsync())
            {
                await client.DeleteRuleAsync(TopicName, subscription, rules.Current.Name);
                Console.WriteLine($"Rule {rules.Current.Name} has been removed.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    Console.WriteLine("All default filters have been removed.\n");

    await PresentMenu();
}
```

### <a name="send-messages"></a>Envío de mensajes

El envío de mensajes a un tema es parecido al envío de mensajes a una cola. En este ejemplo se muestra cómo enviar mensajes mediante una lista de tareas y el procesamiento asincrónico:

```csharp
public async Task SendMessages()
{
    try
    {
        await using var client = new ServiceBusClient(ServiceBusConnectionString);
        var taskList = new List<Task>();
        for (int i = 0; i < Store.Length; i++)
        {
            taskList.Add(SendItems(client, Store[i]));
        }

        await Task.WhenAll(taskList);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    Console.WriteLine("\nAll messages sent.\n");
}

private async Task SendItems(ServiceBusClient client, string store)
{
    // create the sender
    ServiceBusSender tc = client.CreateSender(TopicName);

    for (int i = 0; i < NrOfMessagesPerStore; i++)
    {
        Random r = new Random();
        Item item = new Item(r.Next(5), r.Next(5), r.Next(5));

        // Note the extension class which is serializing an deserializing messages
        ServiceBusMessage message = item.AsMessage();
        message.To = store;
        message.ApplicationProperties.Add("StoreId", store);
        message.ApplicationProperties.Add("Price", item.GetPrice().ToString());
        message.ApplicationProperties.Add("Color", item.GetColor());
        message.ApplicationProperties.Add("Category", item.GetItemCategory());

        await tc.SendMessageAsync(message);
        Console.WriteLine($"Sent item to Store {store}. Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}"); ;
    }
}
```

### <a name="receive-messages"></a>Recepción de mensajes

Los mensajes se reciben de nuevo a través de una lista de tareas y el código usa el procesamiento por lotes. Puede enviar y recibir mediante el procesamiento por lotes, pero en este ejemplo solo se muestra cómo procesar por lotes la recepción. En realidad, no tendría que salir del bucle sino seguir en él y establecer un intervalo de tiempo mayor, de un minuto por ejemplo. La llamada de recepción para el agente se mantiene abierta durante este período de tiempo y si los mensajes llegan, se devuelven inmediatamente y se emite una nueva llamada de recepción. Este concepto se denomina *sondeo largo*. El uso del suministro de recepción que se puede ver en la [guía de inicio rápido](service-bus-quickstart-portal.md), y en otros varios ejemplos del repositorio, es una opción más habitual.

```csharp
public async Task Receive()
{
    var taskList = new List<Task>();
    for (var i = 0; i < Subscriptions.Length; i++)
    {
        taskList.Add(this.ReceiveMessages(Subscriptions[i]));
    }

    await Task.WhenAll(taskList);
}

private async Task ReceiveMessages(string subscription)
{
    await using var client = new ServiceBusClient(ServiceBusConnectionString);
    ServiceBusReceiver receiver = client.CreateReceiver(TopicName, subscription);

    // In reality you would not break out of the loop like in this example but would keep looping. The receiver keeps the connection open
    // to the broker for the specified amount of seconds and the broker returns messages as soon as they arrive. The client then initiates
    // a new connection. So in reality you would not want to break out of the loop. 
    // Also note that the code shows how to batch receive, which you would do for performance reasons. For convenience you can also always
    // use the regular receive pump which we show in our Quick Start and in other github samples.
    while (true)
    {
        try
        {
            //IList<Message> messages = await receiver.ReceiveAsync(10, TimeSpan.FromSeconds(2));
            // Note the extension class which is serializing an deserializing messages and testing messages is null or 0.
            // If you think you did not receive all messages, just press M and receive again via the menu.
            IReadOnlyList<ServiceBusReceivedMessage> messages = await receiver.ReceiveMessagesAsync(maxMessages: 100);

            if (messages.Any())
            {
                foreach (ServiceBusReceivedMessage message in messages)
                {
                    lock (Console.Out)
                    {
                        Item item = message.As<Item>();
                        IReadOnlyDictionary<string, object> myApplicationProperties = message.ApplicationProperties;
                        Console.WriteLine($"StoreId={myApplicationProperties["StoreId"]}");
                        if (message.Subject != null)
                        {
                            Console.WriteLine($"Subject={message.Subject}");
                        }
                        Console.WriteLine(
                            $"Item data: Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}");
                    }

                    await receiver.CompleteMessageAsync(message);
                }
            }
            else
            {
                break;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
}
```

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprovisionó recursos mediante Azure Portal, después, envió y recibió mensajes desde un tema de Service Bus y sus suscripciones. Ha aprendido a:

> [!div class="checklist"]
> * Crear un tema de Service Bus y una o varias suscripciones a ese tema mediante Azure Portal
> * Agregar filtros de tema mediante código .NET
> * Crear dos mensajes con contenido diferente
> * Enviar los mensajes y comprobar que llegaron a las suscripciones esperadas
> * Recibir mensajes de las suscripciones

Para más ejemplos de envío y recepción de mensajes, empiece a trabajar con los [ejemplos de Service Bus en GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted).

Avance hasta el siguiente tutorial para aprender más acerca del uso de las funcionalidades de publicación y suscripción de Service Bus.

> [!div class="nextstepaction"]
> [Respuesta a eventos mediante Event Grid](service-bus-to-event-grid-integration-example.md)

[cuenta gratuita]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Azure portal]: https://portal.azure.com/

[connection-string]: ./media/service-bus-tutorial-topics-subscriptions-portal/connection-string.png
[service-bus-flow]: ./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png
