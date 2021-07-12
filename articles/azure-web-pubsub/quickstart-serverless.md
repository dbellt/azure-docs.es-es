---
title: Inicio rápido del servicio Azure Web PubSub sin servidor
description: Un inicio rápido para usar el servicio Azure Web PubSub y Azure Functions en una aplicación sin servidor.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 03/11/2021
ms.openlocfilehash: 573e0dc028391c2eea9d412bfe68c07a2e95aec3
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111963150"
---
# <a name="quickstart-create-a-serverless-simple-chat-application-with-azure-functions-and-azure-web-pubsub-service"></a>Inicio rápido: Creación de una aplicación de chat simple sin servidor con Azure Functions y el servicio Azure Web PubSub 

El servicio Azure Web PubSub le ayuda a crear aplicaciones web de mensajería en tiempo real mediante WebSockets y el patrón de publicación y suscripción fácilmente. Azure Functions es una plataforma sin servidor que le permite ejecutar el código sin tener que administrar ninguna infraestructura. En este inicio rápido, aprenderá a usar el servicio Azure Web PubSub y Azure Functions para compilar una aplicación sin servidor con mensajería en tiempo real y el patrón de publicación y suscripción.

## <a name="prerequisites"></a>Requisitos previos

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Instale un editor de código, como [Visual Studio Code](https://code.visualstudio.com/), y la versión 10.x de la biblioteca [Node.js](https://nodejs.org/en/download/).

   > [!NOTE]
   > Para más información sobre las versiones de Node.js que se admiten, consulte la [documentación de las versiones del runtime de Azure Functions.](../azure-functions/functions-versions.md#languages)

Instale [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools#installing) (versión 2.7.1505 o posteriores) para ejecutar las aplicaciones de Azure Functions de forma local.

---

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [create-instance-portal](includes/create-instance-portal.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Mientras se implementa el servicio, pasaremos a trabajar con el código. Clone la [aplicación de ejemplo de GitHub](https://github.com/Azure/azure-webpubsub/tree/main/samples/functions/js/simplechat) como primer paso.

1. Abra una ventana de terminal de GIT. Cambie a la carpeta en la que desea clonar el proyecto de ejemplo.

1. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure/azure-webpubsub.git
    ```

## <a name="configure-and-run-the-azure-function-app"></a>Configuración y ejecución de la aplicación Azure Function

- En el explorador, abra **Azure Portal** y confirme que la instancia del servicio Azure Web PubSub que ha implementado anteriormente se creó correctamente. Vaya a la instancia.
- Seleccione **Claves** y copie la cadena de conexión.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

- Actualice la configuración de la función.

  Abra la carpeta */samples/functions/js/simplechat* en el repositorio clonado. Edite *local.settings.jspara* para agregar la cadena de conexión del servicio.
  En *local.settings.jsen*, debe realizar estos cambios y, después, guardar el archivo.
    - Reemplace el marcador de posición *<cadena de conexión>* por el real copiado de **Azure Portal** en el valor **`WebPubSubConnectionString`** . 
    - En el caso del valor **`AzureWebJobsStorage`** , esto es necesario porque [Azure Functions requiere una cuenta de Azure Storage](../azure-functions/storage-considerations.md).
        - Si el emulador de Azure Storage se ejecuta en local, mantenga la configuración original de "UseDevelopmentStorage=true".
        - Si tiene una cadena de conexión de Azure Storage, reemplace el valor por ella.
 
- Las funciones de JavaScript se organizan en carpetas. En cada carpeta hay dos archivos: `function.json`, que define los enlaces que se usan en la función, e `index.js`, que es el cuerpo de la función. Hay varias funciones desencadenadas en esta aplicación de función:

    - **login**: esta es la función desencadenada por HTTP. Usa el enlace de entrada *webPubSubConnection* para generar y devolver información de conexión del servicio válida.
    - **messages**: esta es la función desencadenada por `WebPubSubTrigger`. Recibe un mensaje de chat en el cuerpo de la solicitud y usa el enlace de salida `WebPubSub` para difundir el mensaje a todas las aplicaciones cliente conectadas.
    - **connect** y **connected**: son las funciones desencadenadas por `WebPubSubTrigger`. Controle los eventos conectados y de conexión.

- En el terminal, asegúrese de que está en la carpeta */samples/functions/js/simplechat*. Instale las extensiones y ejecute la aplicación de funciones.

    ```bash
    func extensions install

    func start
    ```

- La función local usará el puerto definido en el archivo `local.settings.json`. Para que esté disponible en la red pública, debe usar [ngrok](https://ngrok.com) para exponer este punto de conexión. Ejecute el comando siguiente y obtendrá un punto de conexión de reenvío, por ejemplo: http://{ngrok-id}.ngrok.io -> http://localhost:7071.

    ```bash
    ngrok http 7071
    ```    

- Establezca `Event Handler` en el servicio Azure Web PubSub. Vaya a **Azure Portal** -> Busque su recurso de Azure Web PubSub -> **Configuración**. Agregue una nueva asignación de configuración del centro a la única función en uso, como se indica a continuación. Reemplace {ngrok-id} por el suyo.

   - Nombre del centro: simplechat
   - Plantilla de dirección URL: **http://{ngrok-id}.ngrok.io/runtime/webhooks/webpubsub**
   - Patrón de evento de usuario: *
   - Eventos del sistema: conexión, conectado y desconectado.

---

## <a name="run-the-web-application"></a>Ejecución de la aplicación web

1. Para simplificar las pruebas del cliente, abra el explorador en nuestra [aplicación web de página única](http://jialinxin.github.io/webpubsub/) de ejemplo. 

1. Escriba la dirección URL base de la aplicación de funciones como local: `http://localhost:7071`.

1. Especifique un nombre de usuario.

1. La aplicación web llama a la función *login* de la aplicación de función para recuperar la información de conexión y conectarse al servicio Azure Web PubSub. Cuando vea `Client websocket opened.`, significará que se ha establecido la conexión. 

1. Escriba un mensaje y presione ENTRAR. La aplicación envía el mensaje a la función *messages* de la aplicación de Azure Functions que, a continuación, usa el enlace de salida de Azure Web PubSub para difundir el mensaje a todos los clientes conectados. Si todo funciona correctamente, el mensaje aparecerá en la aplicación.

1. Abra otra instancia de la aplicación web en otra ventana del explorador. Verá que los mensajes enviados aparecerán en todas las instancias de la aplicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, siga los pasos que se indican a continuación para eliminar todos los recursos creados por este documento, con el fin de que no se le apliquen cargos:

1. En Azure Portal, seleccione **Grupos de recursos** en el extremo izquierdo y luego seleccione el grupo de recursos que creó. Puede usar el cuadro de búsqueda para buscar el grupo de recursos por su nombre en su lugar.

1. En la ventana que se abrirá, seleccione el grupo de recursos y luego seleccione **Eliminar grupo de recursos**.

1. En la nueva ventana, escriba el nombre del grupo de recursos que desea eliminar y, después, seleccione **Eliminar**.