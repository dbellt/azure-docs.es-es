---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/26/2019
ms.author: glenga
ms.openlocfilehash: 642989df8dca9d4ae121d80e30a9fb6d8dd06286
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800065"
---
### <a name="query-the-storage-queue"></a>Consulta de la cola de almacenamiento

Puede usar el comando [`az storage queue list`](/cli/azure/storage/queue#az_storage_queue_list) para ver las colas de almacenamiento de la cuenta, como en el ejemplo siguiente:

```azurecli-interactive
az storage queue list --output tsv
```

La salida de este comando incluye una cola denominada `outqueue`, que es la cola que se creó cuando se ejecutó la función.

A continuación, use el comando [`az storage message peek`](/cli/azure/storage/message#az_storage_message_peek) para ver los mensajes de esta cola, como se muestra en este ejemplo:

```azurecli-interactive
echo `echo $(az storage message peek --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
```

La cadena devuelta debe ser la misma que el mensaje que envió para probar la función.

> [!NOTE]  
> En el ejemplo anterior se descodifica la cadena devuelta desde base64. Esto se debe a que los enlaces de Queue Storage escriben y leen en Azure Storage como [cadenas de base64](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#encoding).