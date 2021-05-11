---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 03/15/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: b566fc5c8d490d9fbad8ba64217d9ab3110cd709
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108313418"
---
> [!NOTE]
> Se pueden escribir datos en Blob Storage con formato [Apache Avro](https://avro.apache.org/), que es el predeterminado, o JSON. 
>    
> El formato de codificación solo se puede establecer en el momento en que se configura el punto de conexión de Blob Storage. No se puede cambiar el formato de un punto de conexión que ya se ha configurado. Cuando se usa la codificación JSON, debe establecer contentType en JSON y contentEncoding en UTF-8 en las propiedades del sistema de mensajes. 
>
> Para obtener más información sobre el uso de un punto de conexión de Blob Storage, consulte la [guía sobre el enrutamiento Azure Storage](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint).
>