---
title: archivo de inclusión
description: archivo de inclusión
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/21/2021
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: f2dcb92d0a600c57d96348413704863285cea3f0
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987205"
---
## <a name="allow-azure-to-call-your-local-function"></a>Permitir que Azure llame a su función local

Para interrumpir la depuración de una función en su máquina, debe habilitar una manera de que Azure se comunique con la función local desde la nube.

La utilidad [ngrok](https://ngrok.com/) proporciona una manera de que Azure llame a la función que se ejecuta en la máquina. Inicie *ngrok* con el comando siguiente:

```bash
ngrok http -host-header=localhost 7071
```
Al configurar la utilidad, la ventana de comandos debe ser similar a la que se muestra en la siguiente captura de pantalla:

![Captura de pantalla que muestra el símbolo del sistema después de iniciar la utilidad "ngrok".](./media/functions-event-grid-local-dev/functions-event-grid-local-dev-ngrok.png)

Copie la dirección URL **HTTPS** generada al ejecutar *ngrok*. Este valor se utiliza al configurar el punto de conexión del desencadenador de Event Grid.

## <a name="add-a-storage-event"></a>Agregar un evento de almacenamiento

1. Abra Azure Portal, vaya a la cuenta de almacenamiento y seleccione la opción **Eventos** en el menú izquierdo.

    ![Agregar un evento de cuenta de almacenamiento](./media/functions-event-grid-local-dev/functions-event-grid-local-dev-add-event.png)

1. En la ventana *Eventos*, seleccione el botón **Suscripción de eventos**. 

1. En la ventana *Suscripción de eventos*, seleccione el menú desplegable *Tipo de punto de conexión* y seleccione **Webhook**.

    ![Seleccionar el tipo de suscripción](./media/functions-event-grid-local-dev/functions-event-grid-local-dev-event-subscription-type.png)

1. Una vez configurado el tipo de punto de conexión, haga clic en **Select an endpoint** (Seleccionar un punto de conexión) para configurar el valor del punto de conexión.

    ![Seleccionar el tipo de punto de conexión](./media/functions-event-grid-local-dev/functions-event-grid-local-dev-event-subscription-endpoint.png)

    El valor *Punto de conexión de suscriptor* se compone de tres valores diferentes. El prefijo es la dirección URL HTTPS generada por *ngrok*. El resto de la dirección URL procede de la dirección URL de localhost que copió anteriormente en la guía de procedimiento, con el nombre de la función agregado al final. A partir de la dirección URL del localhost, la dirección URL de *ngrok* reemplaza a `http://localhost:7071` y el nombre de la función reemplaza a `{functionname}`.

1. En la captura de pantalla siguiente se muestra un ejemplo de cómo debe ser la dirección URL final cuando se usa un tipo de desencadenador `Event Grid`.

    ![Selección de punto de conexión](./media/functions-event-grid-local-dev/functions-event-grid-local-dev-event-subscription-endpoint-selection.png)

1. Una vez que haya escrito el valor adecuado, haga clic en **Confirmar selección**.

> [!IMPORTANT]
> Cada vez que inicie *ngrok*, se vuelve a generar la dirección URL HTTPS y cambia el valor. Por lo tanto, debe crear una nueva suscripción a eventos cada vez que exponga la función en Azure mediante *ngrok*.

## <a name="upload-a-file"></a>Cargar un archivo

Ahora puede cargar un archivo a su cuenta de almacenamiento para desencadenar un evento de Event Grid para que lo controle su función local. 

Abra [Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) y conéctelo a la cuenta de almacenamiento. 

- Expanda **Contenedores de blobs**. 
- Haga clic con el botón derecho en **Crear contenedor de blobs**.
- Asigne al contenedor el nombre **samples-workitems**.
- Seleccione el contenedor *samples-workitems*.
- Haga clic en el botón **Cargar**.
- Haga clic en **Cargar archivos**.
- Seleccione un archivo y cárguelo en el contenedor de blobs.

