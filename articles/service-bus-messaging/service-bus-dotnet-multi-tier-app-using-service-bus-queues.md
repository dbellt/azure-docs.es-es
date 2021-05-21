---
title: Aplicación de niveles múltiples .NET mediante Azure Service Bus | Microsoft Docs
description: Un tutorial .NET que le ayuda a desarrollar una aplicación de varios niveles en Azure que usa colas de Service Bus para comunicarse entre los niveles.
ms.devlang: dotnet
ms.topic: article
ms.date: 04/30/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 38a1c975df578b32ec2d6cac9ff5c6ad4acb3687
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759198"
---
# <a name="net-multi-tier-application-using-azure-service-bus-queues"></a>Aplicación de niveles múltiples .NET con colas de Azure Service Bus

Desarrollar para Microsoft Azure es muy sencillo con Visual Studio y el SDK de Azure para .NET gratis. Este tutorial lo guía por los pasos para crear una aplicación que usa varios recursos de Azure que se ejecutan en su entorno local.

Aprenderá lo siguiente:

* Cómo habilitar el equipo para el desarrollo de Azure con una única descarga e instalación.
* Cómo utilizar Visual Studio para desarrollar para Azure.
* Cómo crear una aplicación de niveles múltiples en Azure mediante el uso de roles web y de trabajo.
* Cómo comunicarse entre niveles mediante las colas de Service Bus.

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

En este tutorial compilará y ejecutará la aplicación de niveles múltiples en un servicio en la nube de Azure. El front-end es un rol web de ASP.NET MVC y el back-end es un rol de trabajo que usa una cola de Service Bus. Puede crear la misma aplicación de niveles múltiples con el front-end como un proyecto web, que se implementa en un sitio web de Azure, en lugar de en un servicio en la nube. También puede probar el tutorial de [aplicación híbrida en la nube/local .NET](../azure-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay.md).

La siguiente captura de pantalla muestra la aplicación completada:

:::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png" alt-text="Página de envío de la aplicación":::

## <a name="scenario-overview-inter-role-communication"></a>Información general del escenario: comunicación entre roles
Para enviar una orden para su procesamiento, el componente de la interfaz de usuario de front-end, que se ejecuta en el rol web, debe interactuar con la lógica del nivel intermedio que se ejecuta en el rol de trabajo. En ese ejemplo se utiliza la mensajería de Service Bus para la comunicación entre los niveles.

El uso de la mensajería de Service Bus entre los niveles de web e intermedio desacopla los dos componentes. Al contrario que en la mensajería directa (es decir, TCP o HTTP), el nivel web no se conecta directamente al nivel intermedio; por el contrario, inserta unidades de trabajo, como mensajes, en Service Bus, que los conserva de manera confiable hasta que el nivel intermedio esté preparado para consumirlas y procesarlas.

Service Bus ofrece dos entidades para admitir la mensajería asincrónica: colas y temas. Con las colas, cada mensaje enviado a la cola lo consume un único receptor. Los temas admiten el patrón de publicación/suscripción, en el cual cada mensaje publicado está disponible para una suscripción registrada con el tema. Cada suscripción mantiene lógicamente su propia cola de mensajes. Las suscripciones también se pueden configurar con reglas de filtro que restringen el conjunto de mensajes pasados a la cola de suscripción a aquellos que coinciden con el filtro. En este ejemplo se usan las colas de Service Bus.

:::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png" alt-text="Diagrama que muestra la comunicación entre el rol web, Service Bus y el rol de trabajo.":::

Este mecanismo de comunicación tiene varias ventajas sobre la mensajería directa:

* **Desacoplamiento temporal.** Con el patrón de mensajería asincrónica, los productores y los consumidores no tienen que estar en línea al mismo tiempo. Service Bus almacena los mensajes de manera confiable hasta que la parte consumidora esté lista para recibirlos. De esta forma, los componentes de la aplicación distribuida se pueden desconectar, ya sea voluntariamente, por ejemplo, para mantenimiento, o debido a un bloqueo del componente, sin que afecte al sistema en su conjunto. Es más, puede que la aplicación consumidora solo necesite estar en línea durante determinados períodos del día.
* **Redistribución de la carga.** En muchas aplicaciones, la carga del sistema varía con el tiempo, mientras que el tiempo de procesamiento requerido para cada unidad de trabajo suele ser constante. La intermediación de productores y consumidores de mensajes con una cola implica que la aplicación consumidora (el trabajador) solo necesita ser aprovisionada para acomodar una carga promedio en lugar de una carga pico. La profundidad de la cola aumenta y se contrae a medida que varíe la carga entrante, lo que permite ahorrar dinero directamente en función de la cantidad de infraestructura requerida para dar servicio a la carga de la aplicación.
* **Equilibrio de carga.** A medida que aumenta la carga, se pueden agregar más procesos de trabajo para que puedan leerse desde la cola. Cada mensaje se procesa únicamente por uno de los procesos de trabajo. Además, este equilibrio de carga permite el uso óptimo de las máquinas de trabajo aunque estas máquinas difieran en términos de capacidad de procesamiento, ya que extraerán mensajes a su frecuencia máxima propia. Este patrón con frecuencia se denomina patrón de *consumo de competidor*.
  
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png" alt-text="Diagrama que muestra la comunicación entre el rol web, Service Bus y dos roles de trabajo.":::
  
En las secciones siguientes se explica el código que implementa esta arquitectura.

## <a name="prerequisites"></a>Requisitos previos
En este tutorial, usará la autenticación de Azure Active Directory (Azure AD) para crear objetos `ServiceBusClient` y `ServiceBusAdministrationClient`. También usará `DefaultAzureCredential`; para usarlo, debe realizar los pasos siguientes que le permiten probar la aplicación localmente en un entorno de desarrollo.

1. [Registrar una aplicación en Azure AD](../active-directory/develop/quickstart-register-app.md).
1. [Agregar la aplicación al rol `Service Bus Data Owner`](service-bus-managed-service-identity.md#to-assign-azure-roles-using-the-azure-portal).
1. Establecer las variables de entorno `AZURE-CLIENT-ID`, `AZURE-TENANT-ID` y `AZURE-CLIENT-SECRET`. Para obtener instrucciones, consulte [este artículo](/dotnet/api/overview/azure/identity-readme#environment-variables).


## <a name="create-a-namespace"></a>Creación de un espacio de nombres

El primer paso consiste en crear un *espacio de nombres* y obtener una [clave de Firma de acceso compartido (SAS)](service-bus-sas.md) para dicho espacio de nombres. Un espacio de nombres proporciona un límite de aplicación para cada aplicación que se expone a través de Service Bus. El sistema genera una clave SAS cuando se crea un espacio de nombres. La combinación del espacio de nombres y la clave SAS proporciona las credenciales de Service Bus para autenticar el acceso a una aplicación.

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="create-a-web-role"></a>Creación de un rol web

En esta sección, va a crear el front-end de la aplicación. En primer lugar, va a crear las páginas que mostrará la aplicación.
Después, va a agregar el código que envía elementos a una cola de Service Bus y muestra información de estado sobre la cola.

### <a name="create-the-project"></a>Creación del proyecto

1. Inicie Visual Studio con privilegios de administrador: haga clic con el botón derecho en el icono del programa **Visual Studio** y después haga clic en **Ejecutar como administrador**. El Emulador de Azure Compute, descrito posteriormente en este artículo, requiere que se inicie Visual Studio con privilegios de administrador.
   
   En Visual Studio, en el menú **Archivo**, haga clic en **Nuevo** y, a continuación, en **Proyecto**.
2. En la página **Plantillas**, siga estos pasos:
    1. Seleccione **C#** como lenguaje de programación.
    1. Seleccione **Nube** como tipo de proyecto.
    1. Seleccione **Servicio en la nube de Azure**.
    1. Seleccione **Next** (Siguiente). 
   
        :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png" alt-text="Captura de pantalla del cuadro de diálogo Nuevo proyecto con Nube seleccionada y Azure Cloud Service Visual C# resaltado y con contorno en rojo.":::
3.  Asigne al proyecto el nombre **MultiTierApp**, seleccione la ubicación del proyecto y, a continuación, seleccione **Crear**.

    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/project-name.png" alt-text="Especifique el nombre del proyecto.":::    
1. En la página **Roles**, haga doble clic en **Rol web de ASP.NET** y seleccione **Aceptar**. 
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png" alt-text="Seleccionar el rol web":::
4. Mantenga el cursor sobre **WebRole1** en **Solución de servicio de nube de Azure**, haga clic en el icono de lápiz y cambie el nombre del rol web a **FrontendWebRole**. A continuación, haga clic en **Aceptar**. (Asegúrese de que escribe "Frontend" con "e" minúscula, no "FrontEnd").
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png" alt-text="Captura de pantalla del cuadro de diálogo Nuevo servicio de nube de Microsoft Azure con la solución con nombre cambiado a FrontendWebRole.":::
5. En el cuadro de diálogo **Crear una aplicación web ASP.NET**, seleccione **MVC** y, a continuación, **Crear**.
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png" alt-text="Captura de pantalla del cuadro de diálogo Nuevo proyecto de ASP.NET con MVC resaltado y con contorno en rojo, y la opción Cambiar autenticación con contorno en rojo.":::
8. En el **Explorador de soluciones**, en el proyecto **FrontendWebRole**, haga clic con el botón derecho en **Referencias** y haga clic en **Administrar paquetes NuGet**.
9. Haga clic en la pestaña **Examinar** y, después, busque **Azure.Messaging.ServiceBus**. Seleccione el paquete **Azure.Messaging.ServiceBus**, seleccione **Instalar** y acepte las condiciones de uso.
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png" alt-text="Captura de pantalla del cuadro de diálogo Administrar paquetes NuGet con Azure.Messaging.ServiceBus resaltado y la opción Instalar con contorno en rojo.":::

   Tenga en cuenta que ahora se hace referencia a los ensamblados del cliente requeridos y que se han agregado algunos archivos de código nuevos.
10. Siga los mismos pasos para agregar el paquete NuGet `Azure.Identity` al proyecto.  
10. En el **Explorador de soluciones**, expanda **FronendWebRole**, haga clic con el botón derecho en **Modelos**, luego en **Agregar** y, por último, en **Clase**. En el cuadro **Nombre**, escriba el nombre **OnlineOrder.cs**. A continuación, haga clic en **Agregar**.

### <a name="write-the-code-for-your-web-role"></a>Especificación del código del rol web
En esta sección, creará las distintas páginas que va a mostrar la aplicación.

1. En el archivo OnlineOrder.cs en Visual Studio, sustituya la definición del espacio de nombres existentes por el código siguiente:
   
   ```csharp
   namespace FrontendWebRole.Models
   {
       public class OnlineOrder
       {
           public string Customer { get; set; }
           public string Product { get; set; }
       }
   }
   ```
2. En el **Explorador de soluciones**, haga doble clic en **Controllers\HomeController.cs**. Agregue las siguientes instrucciones **using** en la parte superior del archivo para incluir los espacios de nombres en el modelo que acaba de crear, así como Service Bus.
   
   ```csharp
    using FrontendWebRole.Models;
    using Azure.Messaging.ServiceBus;    
   ```
3. En el archivo HomeController.cs en Visual Studio, sustituya la definición del espacio de nombres existentes por el código siguiente. Este código contiene métodos para controlar el envío de elementos a la cola.
   
   ```csharp
   namespace FrontendWebRole.Controllers
   {
       public class HomeController : Controller
       {
           public ActionResult Index()
           {
               // Simply redirect to Submit, since Submit will serve as the
               // front page of this application.
               return RedirectToAction("Submit");
           }
   
           public ActionResult About()
           {
               return View();
           }
   
           // GET: /Home/Submit.
           // Controller method for a view you will create for the submission
           // form.
           public ActionResult Submit()
           {
               // Will put code for displaying queue message count here.
   
               return View();
           }
   
           // POST: /Home/Submit.
           // Controller method for handling submissions from the submission
           // form.
           [HttpPost]
           // Attribute to help prevent cross-site scripting attacks and
           // cross-site request forgery.  
           [ValidateAntiForgeryToken]
           public ActionResult Submit(OnlineOrder order)
           {
               if (ModelState.IsValid)
               {
                   // Will put code for submitting to queue here.
   
                   return RedirectToAction("Submit");
               }
               else
               {
                   return View(order);
               }
           }
       }
   }
   ```
4. En el menú **Compilar**, haga clic en **Compilar solución** para probar la precisión del trabajo hasta ahora.
5. Ahora, va a crear la vista para el método `Submit()` que creó antes. Haga clic con el botón derecho en el método `Submit()` (la sobrecarga de `Submit()` que no acepta parámetros) del archivo **HomeController.cs** y elija **Agregar vista**.
6. En el cuadro de diálogo **Agregar nuevo elemento con scaffolding**, seleccione **Agregar**. 
1. En el cuadro de diálogo **Agregar vista**, siga estos pasos:
    1. En la lista **Plantilla**, elija **Crear**. 
    1. En la lista **Clase de modelo**, seleccione la clase **OnlineOrder**.
    1. Seleccione **Agregar**. 
   
        :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar vista con las listas desplegables Plantilla y Clase de modelo con contorno en rojo.":::
8. Ahora, cambie el nombre mostrado de la aplicación. En el **Explorador de soluciones**, haga doble clic en el archivo **Views\Shared\\_Layout.cshtml** para abrirlo en el editor de Visual Studio.
9. Reemplace todas las repeticiones de **Mi aplicación ASP.NET** por **Productos de Northwind Traders**.
10. Quite los vínculos **Página principal**, **Acerca de** y **Contacto**. Elimine el código resaltado:
    
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png" alt-text="Captura de pantalla del código con tres líneas del código de Vínculo de acción HTML resaltado.":::
11. Finalmente, modifique la página de envío para incluir información sobre la cola. En el **Explorador de soluciones**, haga doble clic en el archivo **Views\Home\Submit.cshtml** para abrirlo en el editor de Visual Studio. Agregue la línea siguiente después de `<h2>Submit</h2>`. Por ahora, `ViewBag.MessageCount` está vacío. Lo rellenará más adelante.
    
    ```html
    <p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>
    ```
12. Acaba de implementar su interfaz de usuario. Puede presionar **F5** para ejecutar la aplicación y confirmar que tiene el aspecto previsto.
    
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png" alt-text="Captura de pantalla de la página Enviar de la aplicación":::

### <a name="write-the-code-for-submitting-items-to-a-service-bus-queue"></a>Especificación del código para enviar elementos a una cola de Service Bus
Ahora, agregue el código para enviar elementos a una cola. En primer lugar, va a crear una clase que contiene la información de conexión a la cola de Service Bus. A continuación, inicialice la conexión en Global.aspx.cs. Finalmente, actualice el código de envío que creó antes en HomeController.cs para enviar realmente los elementos a una cola de Service Bus.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en **FrontendWebRole** (haga clic con el botón derecho en el proyecto, no en el rol). Haga clic en **Agregar** y, a continuación, en **Clase**.
2. Asigne a la clase el nombre **QueueConnector.cs**. Haga clic en **Agregar** para crear la clase.
3. Ahora va a agregar código que encapsula la información de conexión e inicializará la conexión a una cola de Service Bus. Reemplace todo el contenido de QueueConnector.cs por el código siguiente y escriba valores para `your Service Bus namespace` (el nombre del espacio de nombres) y `yourKey`, la **clave principal** que obtuvo antes en Azure Portal.
   
   ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using System.Threading.Tasks;
    using Azure.Messaging.ServiceBus;
    using Azure.Messaging.ServiceBus.Administration;
       
   namespace FrontendWebRole
   {
        public static class QueueConnector
        {
            // object to send messages to a Service Bus queue
            internal static ServiceBusSender SBSender;
    
            // object to create a queue and get runtime properties (like message count) of queue
            internal static ServiceBusAdministrationClient SBAdminClient;
        
            // Fully qualified Service Bus namespace
            private const string FullyQualifiedNamespace = "<SERVICE BUS NAMESPACE NAME>.servicebus.windows.net";
            
            // The name of your queue.
            internal const string QueueName = "OrdersQueue";
        
            public static async Task Initialize()
            {
                // Create a Service Bus client that you can use to send or receive messages
                ServiceBusClient SBClient = new ServiceBusClient(FullyQualifiedNamespace, new DefaultAzureCredential());
        
                // Create a Service Bus admin client to create queue if it doesn't exist or to get message count
                SBAdminClient = new ServiceBusAdministrationClient(FullyQualifiedNamespace, new DefaultAzureCredential());
        
                // create the OrdersQueue if it doesn't exist already
                if (!(await SBAdminClient.QueueExistsAsync(QueueName)))
                {
                    await SBAdminClient.CreateQueueAsync(QueueName);
                }
        
                // create a sender for the queue 
                SBSender = SBClient.CreateSender(QueueName);    
            }
        }    
   }
   ```
4. Ahora, asegúrese de que se llama al método **Initialize**. En el **Explorador de soluciones**, haga doble clic en **Global.asax\Global.asax.cs**.
5. Agregue la siguiente línea de código al final del método **Application_Start**.
   
   ```csharp
    FrontendWebRole.QueueConnector.Initialize().Wait();
   ```
6. Por último, actualice el código web que creó anteriormente para enviar elementos a la cola. En el **Explorador de soluciones**, haga doble clic en **Controllers\HomeController.cs**.
7. Actualice el método `Submit()` (la sobrecarga que no acepta parámetros) como sigue para obtener el número de mensajes de la cola.
   
   ```csharp
        public ActionResult Submit()
        {
            QueueRuntimeProperties properties = QueueConnector.adminClient.GetQueueRuntimePropertiesAsync(QueueConnector.queueName).Result;
            ViewBag.MessageCount = properties.ActiveMessageCount;

            return View();
        }
   ```
8. Actualice el método `Submit(OnlineOrder order)` (la sobrecarga que no acepta parámetros) como sigue para enviar información de pedidos a la cola.
   
   ```csharp
        public ActionResult Submit(OnlineOrder order)
        {
            if (ModelState.IsValid)
            {
                // create a message 
                var message = new ServiceBusMessage(new BinaryData(order));

                // send the message to the queue
                QueueConnector.sbSender.SendMessageAsync(message);

                return RedirectToAction("Submit");
            }
            else
            {
                return View(order);
            }
        }
   ```
9. Ahora puede volver a ejecutar la aplicación. Cada vez que envía un pedido, el contador de mensajes aumenta.
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app2.png" alt-text="Captura de pantalla de la página de envío de la aplicación con el recuento de mensajes incrementado en 1.":::

## <a name="create-the-worker-role"></a>Creación del rol de trabajo
Ahora creará el rol de trabajo que procesa los envíos del pedido. Este ejemplo utiliza la plantilla de proyecto de Visual Studio de **rol de trabajo con cola de Service Bus**. Ya ha obtenido las credenciales necesarias del portal.

1. Asegúrese de que se ha conectado Visual Studio a su cuenta de Azure.
2. En Visual Studio, en el **Explorador de soluciones**, haga clic con el botón derecho en la carpeta **Roles** en el proyecto **MultiTierApp**.
3. Haga clic en **Agregar** y, a continuación, en **Nuevo proyecto de rol de trabajo**. Aparecerá el cuadro de diálogo **Agregar nuevo proyecto de rol**.

   :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png" alt-text="Captura de pantalla del panel Explorador de soluciones con las opciones Nuevo proyecto de rol de trabajo y Agregar resaltadas.":::
1. En el cuadro de diálogo **Agregar nuevo proyecto de rol**, seleccione **Rol de trabajo**. No seleccione **Rol de trabajo con cola de Service Bus**, ya que genera código que usa el SDK de Service Bus heredado. 
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar nuevo proyecto de rol con el rol de trabajo con la opción Cola de Service Bus resaltada y con contorno en rojo.":::
5. En el cuadro **Nombre**, asigne al proyecto el nombre **OrderProcessingRole**. A continuación, haga clic en **Agregar**.
1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **OrderProcessingRole** y seleccione **Administrar paquetes NuGet**.
9. Seleccione la pestaña **Examinar** y, después, busque **Azure.Messaging.ServiceBus**. Seleccione el paquete **Azure.Messaging.ServiceBus**, seleccione **Instalar** y acepte las condiciones de uso.
   
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png" alt-text="Captura de pantalla del cuadro de diálogo Administrar paquetes NuGet con Azure.Messaging.ServiceBus resaltado y la opción Instalar con contorno en rojo.":::
1. Siga los mismos pasos para agregar el paquete NuGet `Azure.Identity` al proyecto.  
1. Cree una clase **OnlineOrder** para representar los pedidos cuando los procese desde la cola. Puede reutilizar una clase que ya ha creado. En el **Explorador de soluciones**, haga clic con el botón derecho en la clase **OrderProcessingRole** (haga clic con el botón derecho en el icono de clase, no en el rol). Haga clic en **Agregar** y, a continuación, en **Elemento existente**.
1. Busque **FrontendWebRole\Models** en la subcarpeta y haga doble clic en **OnlineOrder.cs** para agregarlo a este proyecto.
1. Agregue la siguiente instrucción `using` al archivo **WorkerRole.cs** del proyecto **OrderProcessingRole**. 

    ```csharp
    using FrontendWebRole.Models;
    using Azure.Messaging.ServiceBus;
    using Azure.Messaging.ServiceBus.Administration; 
    ```    
1. En **WorkerRole.cs**, agregue las siguientes propiedades. 

    > [!IMPORTANT]
    > Use la cadena de conexión para el espacio de nombres que anotó como parte de los requisitos previos. 

    ```csharp
        // Fully qualified Service Bus namespace
        private const string FullyQualifiedNamespace = "<SERVICE BUS NAMESPACE NAME>.servicebus.windows.net";

        // The name of your queue.
        private const string QueueName = "OrdersQueue";

        // Service Bus Receiver object to receive messages message the specific queue
        private ServiceBusReceiver SBReceiver;

    ```
1. Actualice el método `OnStart` para crear un objeto `ServiceBusClient` y, a continuación, un objeto `ServiceBusReceiver` para recibir mensajes de `OrdersQueue`. 
    
    ```csharp
        public override bool OnStart()
        {
            // Create a Service Bus client that you can use to send or receive messages
            ServiceBusClient SBClient = new ServiceBusClient(FullyQualifiedNamespace, new DefaultAzureCredential());

            CreateQueue(QueueName).Wait();

            // create a receiver that we can use to receive the message
            SBReceiver = SBClient.CreateReceiver(QueueName);

            return base.OnStart();
        }
        private async Task CreateQueue(string queueName)
        {
            // Create a Service Bus admin client to create queue if it doesn't exist or to get message count
            ServiceBusAdministrationClient SBAdminClient = new ServiceBusAdministrationClient(FullyQualifiedNamespace, new DefaultAzureCredential());

            // create the OrdersQueue if it doesn't exist already
            if (!(await SBAdminClient.QueueExistsAsync(queueName)))
            {
                await SBAdminClient.CreateQueueAsync(queueName);
            }
        }
    ```
12. Actualice el método `RunAsync` para incluir el código para recibir mensajes. 

    ```csharp
        private async Task RunAsync(CancellationToken cancellationToken)
        {
            // TODO: Replace the following with your own logic.
            while (!cancellationToken.IsCancellationRequested)
            {
                // receive message from the queue
                ServiceBusReceivedMessage receivedMessage = await SBReceiver.ReceiveMessageAsync();

                if (receivedMessage != null)
                {
                    Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());

                    // view the message as an OnlineOrder
                    OnlineOrder order = receivedMessage.Body.ToObjectFromJson<OnlineOrder>();
                    Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");

                    // complete message so that it's removed from the queue
                    await SBReceiver.CompleteMessageAsync(receivedMessage);
                }
            }
        }
    ```
14. Ha completado la aplicación. Puede probar la aplicación completa haciendo clic con el botón derecho en el proyecto MultiTierApp en el Explorador de soluciones, seleccionando **Establecer como proyecto de inicio** y, a continuación, presionando F5. Tenga en cuenta que el contador de mensajes no se aumenta, ya que el rol de trabajo procesa los elementos de la cola y los marca como finalizados. Puede ver el resultado de seguimiento de su rol de trabajo viendo la interfaz de usuario del emulador de Azure Compute. Para ello, haga clic con el botón derecho en el icono del emulador del área de notificación de la barra de tareas y seleccione **Mostrar interfaz de usuario del emulador de proceso**.
    
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png" alt-text="Captura de pantalla de lo que aparece al hacer clic en el icono del emulador. En la lista de opciones aparece Mostrar interfaz de usuario del emulador de proceso.":::
    
    :::image type="content" source="./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png" alt-text="Captura de pantalla del cuadro de diálogo Emulador de proceso de Microsoft Azure (Rápido).":::

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre Service Bus, consulte los siguientes recursos:  

* [Introducción al uso de las colas de Service Bus][sbacomqhowto]
* [Página de servicio de Service Bus][sbacom]  

Para más información sobre los escenarios de niveles múltiples, consulte:  

* [Aplicación de niveles múltiples .NET mediante tablas, colas y blobs de Storage][mutitierstorage]  



[sbacom]: https://azure.microsoft.com/services/service-bus/  
[sbacomqhowto]: service-bus-dotnet-get-started-with-queues.md  
[mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
