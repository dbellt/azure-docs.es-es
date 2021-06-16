---
title: 'Procedimiento para usar Queue Storage desde Ruby: Azure Storage'
description: Aprenda a utilizar Azure Queue Storage para crear y eliminar colas, e insertar, obtener y eliminar mensajes. Los ejemplos están escritos en Ruby.
author: twooley
ms.author: twooley
ms.reviewer: dineshm
ms.date: 06/08/2021
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 7850e7fef034fd618c86400e17ddaa319c45005a
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081000"
---
# <a name="how-to-use-queue-storage-from-ruby"></a>Procedimiento para usar Queue Storage desde Ruby

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Información general

Esta guía muestra cómo realizar algunas tareas comunes a través del servicio Microsoft Azure Queue Storage. Los ejemplos están escritos usando la API Ruby de Azure. Entre los escenarios descritos se incluyen **insertar**, **ojear**, **obtener** y **eliminar** mensajes de la cola, así como **crear y eliminar colas**.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-ruby-application"></a>Creación de una aplicación de Ruby

Cree una aplicación de Ruby. Para obtener instrucciones, vea [Creación de una aplicación de Ruby en App Service en Linux](../../app-service/quickstart-ruby.md).

## <a name="configure-your-application-to-access-storage"></a>Configuración de la aplicación para acceder al almacenamiento

Para usar Azure Storage, tendrá que descargar y usar el paquete Ruby Azure, que incluye un conjunto de bibliotecas útiles que se comunican con los servicios REST de almacenamiento.

<!-- docutune:ignore Terminal -->

### <a name="use-rubygems-to-obtain-the-package"></a>Uso de RubyGems para obtener el paquete

1. Use una interfaz de línea de comandos como PowerShell (Windows), Terminal (Mac) o Bash (Unix).
2. Escriba `gem install azure` en la ventana de comandos para instalar la gema y las dependencias.

### <a name="import-the-package"></a>Importación del paquete

Con el editor de texto que prefiera, agregue lo siguiente al principio del archivo de Ruby en el que pretenda utilizar el almacenamiento:

```ruby
require "azure"
```

## <a name="setup-an-azure-storage-connection"></a>Configuración de una conexión de Azure Storage

El módulo de Azure leerá las variables de entorno `AZURE_STORAGE_ACCOUNT` y `AZURE_STORAGE_ACCESS_KEY` para obtener la información necesaria para conectarse a la cuenta de Azure Storage. Si no se establecen estas variables de entorno, tendrá que especificar la información de la cuenta antes de usar `Azure::QueueService` con el código siguiente:

```ruby
Azure.config.storage_account_name = "<your azure storage account>"
Azure.config.storage_access_key = "<your Azure storage access key>"
```

Para obtener estos valores desde una cuenta de almacenamiento de Azure Resource Manager o clásica en el Portal de Azure:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a la cuenta de almacenamiento que desea utilizar.
3. En la hoja **Configuración** de la derecha, haga clic en **Claves de acceso**.
4. En la hoja **Claves de acceso** que aparece, verá las claves de acceso 1 y 2. Puede usar cualquiera de estas.
5. Haga clic en el icono de copia para copiar la clave en el Portapapeles.

## <a name="how-to-create-a-queue"></a>Procedimientos: Creación de una cola

El siguiente código crea un objeto `Azure::QueueService` , que le permite trabajar con colas.

```ruby
azure_queue_service = Azure::QueueService.new
```

Use el método `create_queue()` para crear una cola con el nombre especificado.

```ruby
begin
  azure_queue_service.create_queue("test-queue")
rescue
  puts $!
end
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Procedimientos: un mensaje en una cola

Para insertar un mensaje en una cola, use el método `create_message()` para crear un mensaje y agregarlo a la cola.

```ruby
azure_queue_service.create_message("test-queue", "test message")
```

## <a name="how-to-peek-at-the-next-message"></a>Procedimientos: siguiente mensaje

Puede ver el código sin salir del mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, mediante una llamada al método `peek_messages()`. De forma predeterminada, `peek_messages()` inspecciona un único mensaje. También puede indicar cuántos mensajes desea inspeccionar.

```ruby
result = azure_queue_service.peek_messages("test-queue",
  {:number_of_messages => 10})
```

## <a name="how-to-dequeue-the-next-message"></a>Procedimientos: Extracción del siguiente mensaje de la cola

Puede borrar un mensaje de una cola en dos pasos.

1. Si llama a `list_messages()`, obtiene, de forma predeterminada, el siguiente mensaje de una cola. También puede indicar cuántos mensajes desea obtener. Los mensajes devueltos por `list_messages()` se hacen invisibles para cualquier otro código de lectura de mensajes de esta cola. Usted proporciona el tiempo de espera de la visibilidad en segundos a modo de parámetro.
2. Para terminar quitando el mensaje de la cola, también debe llamar a `delete_message()`.

Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código siguiente llama a `delete_message()` justo después de haberse procesado el mensaje.

```ruby
messages = azure_queue_service.list_messages("test-queue", 30)
azure_queue_service.delete_message("test-queue",
  messages[0].id, messages[0].pop_receipt)
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Procedimientos: contenido de un mensaje en cola

Puede cambiar el contenido de un mensaje local en la cola. En el código siguiente se usa el método `update_message()` para actualizar un mensaje. El método devolverá una tupla que contiene la recepción de confirmación del mensaje en cola y un valor `DateTime` UTC que representa el momento en que el mensaje será visible en la cola.

```ruby
message = azure_queue_service.list_messages("test-queue", 30)
pop_receipt, time_next_visible = azure_queue_service.update_message(
  "test-queue", message.id, message.pop_receipt, "updated test message",
  30)
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>Procedimiento: Opciones adicionales para quitar mensajes de la cola

Hay dos formas de personalizar la recuperación de mensajes de una cola.

1. En primer lugar, puede obtener un lote de mensajes.
2. En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje.

En el ejemplo de código siguiente se usa el método `list_messages()` para obtener 15 mensajes en una llamada. A continuación, imprime y elimina cada mensaje. También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje.

```ruby
azure_queue_service.list_messages("test-queue", 300
  {:number_of_messages => 15}).each do |m|
  puts m.message_text
  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
end
```

## <a name="how-to-get-the-queue-length"></a>Procedimientos: la longitud de la cola

Puede obtener una estimación del número de mensajes existentes en la cola. El método `get_queue_metadata()` devuelve el recuento aproximado de mensajes y otros metadatos de la cola.

```ruby
message_count, metadata = azure_queue_service.get_queue_metadata(
  "test-queue")
```

## <a name="how-to-delete-a-queue"></a>Procedimientos: Eliminación de una cola

Para eliminar una cola y todos los mensajes que contiene, llame al método `delete_queue()` en el objeto de cola.

```ruby
azure_queue_service.delete_queue("test-queue")
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido los aspectos básicos de Queue Storage, siga estos vínculos para obtener más información sobre tareas de almacenamiento más complejas.

- Visite el [Blog del equipo de Azure Storage](/archive/blogs/windowsazurestorage/)
- Visite el repositorio de [SDK de Azure para Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) en GitHub

Para obtener una comparación entre Azure Queue Storage, que se explica en este artículo, y las colas de Azure Service Bus, que se explican en el artículo [Uso de colas de Service Bus con Ruby](https://azure.microsoft.com/develop/ruby/how-to-guides/service-bus-queues/), vea [Colas de Storage y de Service Bus: comparación y diferencias](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md).
