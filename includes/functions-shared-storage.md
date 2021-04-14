---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/20/2018
ms.author: glenga
ms.openlocfilehash: 82d122ed236dc72ced7ebafe2301ef5f1143897f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "76963669"
---
Para maximizar el rendimiento, use una cuenta de almacenamiento independiente para cada aplicación de función. Esto es especialmente importante si tiene funciones desencadenadas por Durable Functions o Event Hubs, que generan un gran volumen de transacciones de almacenamiento. Cuando la lógica de la aplicación interactúa con Azure Storage, ya sea directamente (con el SDK de Storage) o a través de uno de los enlaces de almacenamiento, debe usar una cuenta de almacenamiento dedicada. Por ejemplo, si tiene una función desencadenada por Event Hubs que escribe datos en Blob Storage, use dos cuentas de almacenamiento: una para la aplicación de función y otra para los blobs que almacena la función.