---
title: Envío o recepción de eventos desde Azure Event Hubs mediante JavaScript (recientes)
description: Este artículo es un tutorial para crear una aplicación de JavaScript que envía eventos a Azure Event Hubs, o los recibe de él, mediante el reciente paquete azure/event-hubs.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 8bf0b284a4434b15ae818693276cd04712496116
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113003201"
---
# <a name="send-events-to-or-receive-events-from-event-hubs-by-using-javascript--azureevent-hubs"></a>Envío o recepción de eventos en Event Hubs mediante JavaScript (azure/event-hubs)
En este inicio rápido se muestra cómo enviar y recibir eventos desde un centro de eventos mediante el paquete de JavaScript **azure/event-hubs**. 


## <a name="prerequisites"></a>Requisitos previos
Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- Node.js, versión 8.x o posteriores. Descargue la versión más reciente del [soporte técnico a largo plazo (LTS)](https://nodejs.org).  
- Visual Studio Code (se recomienda) o cualquier otro entorno de desarrollo integrado (IDE).  
- Un espacio de nombres de Event Hubs y un centro de eventos activos. Para crearlos, siga estos pasos: 

   1. En [Azure Portal](https://portal.azure.com), cree un espacio de nombres del tipo *Event Hubs* y obtenga las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. 
   1. Para crear tanto el espacio de nombres como el centro de eventos, siga las instrucciones que encontrará en [Inicio rápido: Creación de un centro de eventos desde Azure Portal](event-hubs-create.md).
   1. Para continuar, siga las instrucciones de este inicio rápido. 
   1. Para obtener la cadena de conexión del espacio de nombres del centro de eventos, siga las instrucciones de [Obtención de una cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Registre la cadena de conexión para usarla más adelante en este inicio rápido.
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). Después, obtenga la **cadena de conexión para el espacio de nombres de Event Hubs**. Para ello, siga las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). La cadena de conexión la utilizará más adelante en este inicio rápido.

### <a name="install-the-npm-package"></a>Instalación del paquete npm
Para instalar el [paquete del administrador de paquetes de nodos (npm) para Event Hubs](https://www.npmjs.com/package/@azure/event-hubs), abra un símbolo del sistema que tenga *npm* en su ruta de acceso, cambie el directorio a la carpeta en la que quiere tener los ejemplos y después ejecute este comando:

```shell
npm install @azure/event-hubs
```

En el lado del receptor, tiene que instalar dos paquetes más. En este inicio rápido se usar Azure Blob Storage para conservar los puntos de control, con el fin de que el programa no lea los eventos que ya ha leído. Ejecuta los puntos de comprobación de metadatos en los mensajes recibidos a intervalos regulares en un blob. Este enfoque facilita la tarea de seguir recibiendo posteriormente mensajes desde donde lo dejó.

Ejecute los comandos siguientes:

```shell
npm install @azure/storage-blob
```

```shell
npm install @azure/eventhubs-checkpointstore-blob
```

## <a name="send-events"></a>Envío de eventos

En esta sección se crea una aplicación de JavaScript que envía eventos a un centro de eventos.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com).
1. Cree un archivo denominado *send.js* y pegue en él el código siguiente:

    ```javascript
    const { EventHubProducerClient } = require("@azure/event-hubs");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";
    const eventHubName = "EVENT HUB NAME";

    async function main() {

      // Create a producer client to send messages to the event hub.
      const producer = new EventHubProducerClient(connectionString, eventHubName);

      // Prepare a batch of three events.
      const batch = await producer.createBatch();
      batch.tryAdd({ body: "First event" });
      batch.tryAdd({ body: "Second event" });
      batch.tryAdd({ body: "Third event" });    

      // Send the batch to the event hub.
      await producer.sendBatch(batch);

      // Close the producer client.
      await producer.close();

      console.log("A batch of three events have been sent to the event hub");
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
1. En el código, use valores reales para reemplazar los siguientes:
    * `EVENT HUBS NAMESPACE CONNECTION STRING` 
    * `EVENT HUB NAME`
1. Ejecute `node send.js` para ejecutar este archivo. Este comando envía un lote de tres eventos al centro de eventos.
1. En Azure Portal, compruebe que el centro de eventos ha recibido los mensajes. En la sección **Métricas**, cambie a la vista **Mensajes**. Actualice la página para actualizar el gráfico. Pueden pasar unos segundos hasta que aparezca que los mensajes se han recibido.

    [![Comprobación de que el centro de eventos ha recibido los mensajes](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)

    > [!NOTE]
    > Para ver todo el código fuente, incluidos los comentarios informativos adicionales, vaya a la [página sendEvents.js de GitHub](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/samples/javascript/sendEvents.js).

Felicidades. Ha enviado eventos a un centro de eventos.


## <a name="receive-events"></a>Recepción de eventos
En esta sección recibe eventos de un centro de eventos mediante un almacén de puntos de comprobación de Azure Blob Storage en una aplicación de JavaScript. Ejecuta los puntos de comprobación de metadatos en los mensajes recibidos a intervalos regulares en Azure Storage Blob. Este enfoque facilita la tarea de seguir recibiendo posteriormente mensajes desde donde lo dejó.

> [!WARNING]
> Si ejecuta este código en Azure Stack Hub, experimentará errores en tiempo de ejecución a menos que tenga como destino una versión específica de la API de Storage. Esto se debe a que el SDK de Event Hubs usa la API de Azure Storage más reciente disponible en Azure, que puede que no esté disponible en la plataforma de Azure Stack Hub. Azure Stack Hub puede admitir una versión diferente del SDK de Blob Storage que las que suelen estar disponibles en Azure. Si usa Azure Blob Storage como almacén de puntos de control, compruebe la [versión de la API de Azure Storage admitida para la compilación de Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) y establezca esa versión como destino en el código. 
>
> Por ejemplo, si trabaja en la versión 2005 de Azure Stack Hub, la versión más reciente disponible para el servicio Storage es la 2019-02-02. De forma predeterminada, la biblioteca de cliente del SDK de Event Hubs usa la versión más reciente disponible en Azure (2019-07-07 en el momento de la versión del SDK). En este caso, además de seguir los pasos de esta sección, también tendrá que agregar código para usar como destino la versión 2019-02-02 de la API del servicio de almacenamiento. Para ver un ejemplo de cómo establecer como destino una versión específica de la API de Storage, consulte los ejemplos de [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/eventhub/eventhubs-checkpointstore-blob/samples/v1/javascript/receiveEventsWithApiSpecificStorage.js) y [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/eventhub/eventhubs-checkpointstore-blob/samples/v1/typescript/src/receiveEventsWithApiSpecificStorage.ts) en GitHub. 


### <a name="create-an-azure-storage-account-and-a-blob-container"></a>Creación de una cuenta de Azure Storage y un contenedor de blobs
Para crear una cuenta de Azure Storage y un contenedor de blobs en ella, realice estas acciones:

1. [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal)  
2. [Creación de un contenedor de blobs en la cuenta de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)  
3. [Obtención de la cadena de conexión para una cuenta de almacenamiento](../storage/common/storage-configure-connection-string.md)

Asegúrese de registrar la cadena de conexión y el nombre del contenedor para usarlo posteriormente en el código de recepción.

### <a name="write-code-to-receive-events"></a>Escritura de código para recibir eventos

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com).
1. Cree un archivo denominado *receive.js* y pegue en él el código siguiente:

    ```javascript
    const { EventHubConsumerClient } = require("@azure/event-hubs");
    const { ContainerClient } = require("@azure/storage-blob");    
    const { BlobCheckpointStore } = require("@azure/eventhubs-checkpointstore-blob");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";    
    const eventHubName = "EVENT HUB NAME";
    const consumerGroup = "$Default"; // name of the default consumer group
    const storageConnectionString = "AZURE STORAGE CONNECTION STRING";
    const containerName = "BLOB CONTAINER NAME";

    async function main() {
      // Create a blob container client and a blob checkpoint store using the client.
      const containerClient = new ContainerClient(storageConnectionString, containerName);
      const checkpointStore = new BlobCheckpointStore(containerClient);

      // Create a consumer client for the event hub by specifying the checkpoint store.
      const consumerClient = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName, checkpointStore);

      // Subscribe to the events, and specify handlers for processing the events and errors.
      const subscription = consumerClient.subscribe({
          processEvents: async (events, context) => {
            if (events.length === 0) {
              console.log(`No events received within wait time. Waiting for next interval`);
              return;
            }
          
            for (const event of events) {
              console.log(`Received event: '${event.body}' from partition: '${context.partitionId}' and consumer group: '${context.consumerGroup}'`);
            }
            // Update the checkpoint.
            await context.updateCheckpoint(events[events.length - 1]);
          },

          processError: async (err, context) => {
            console.log(`Error : ${err}`);
          }
        }
      );

      // After 30 seconds, stop processing.
      await new Promise((resolve) => {
        setTimeout(async () => {
          await subscription.close();
          await consumerClient.close();
          resolve();
        }, 30000);
      });
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });    
    ```
1. En el código, use valores reales para reemplazar los siguientes:
    - `EVENT HUBS NAMESPACE CONNECTION STRING`
    - `EVENT HUB NAME`
    - `AZURE STORAGE CONNECTION STRING`
    - `BLOB CONTAINER NAME`
1. Ejecute `node receive.js` en un símbolo del sistema para ejecutar este archivo. La ventana debe mostrar mensajes sobre los eventos recibidos.

    > [!NOTE]
    > Para ver todo el código fuente, incluidos los comentarios informativos adicionales, vaya a la [página receiveEventsUsingCheckpointStore.js de GitHub](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/eventhub/eventhubs-checkpointstore-blob/samples/v1/javascript/receiveEventsUsingCheckpointStore.js).

Felicidades. Ya ha recibido eventos procedentes del centro de eventos. El programa del destinatario recibirá los eventos de todas las particiones del grupo de consumidores predeterminado en el centro de eventos.

## <a name="next-steps"></a>Pasos siguientes
Consulte estos ejemplos de código en GitHub:

- [Ejemplos de JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/javascript)
- [Ejemplos de TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/typescript)
