---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: ac91ef5a56fe234cba47b430b78e74ce81c9d504
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96536935"
---
Azure Storage proporciona las siguientes métricas de transacciones en Azure Monitor.

| Métrica | Descripción |
| ------------------- | ----------------- |
| Transacciones | El número de solicitudes realizadas a un servicio de almacenamiento o la operación de API especificada. Este número incluye solicitudes correctas y con errores, así como las solicitudes que generaron errores. <br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: ResponseType, GeoType, ApiName y Authentication ([Definición](#metrics-dimensions))<br/> Ejemplo de valor: 1024 |
| Entrada | La cantidad de datos de entrada. Este número incluye la entradas desde un cliente externo en Azure Storage, así como la entrada dentro de Azure. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| Salida | La cantidad de datos de salida. Este número incluye la salida de un cliente externo en Azure Storage, así como la salida dentro de Azure. En consecuencia, este número no refleja las salidas facturables. <br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| SuccessServerLatency | El tiempo medio que se usa para que Azure Storage procese una solicitud correcta . Este valor no incluye la latencia de red especificada en SuccessE2ELatency. <br/><br/> Unidad: Milisegundos <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| SuccessE2ELatency | La latencia media de un extremo a otro de las solicitudes correctas realizadas a un servicio de almacenamiento o a la operación de API especificada. Este valor incluye el tiempo de procesamiento requerido dentro de Azure Storage para leer la solicitud, enviar la respuesta y recibir la confirmación de la respuesta. La diferencia entre los valores de SuccessE2ELatency y SuccessServerLatency es la latencia que probablemente causan la red y el cliente.<br/><br/> Unidad: Milisegundos <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 1024 |
| Disponibilidad | El porcentaje de disponibilidad para el servicio de almacenamiento o la operación de API especificada. Para calcular la disponibilidad hay que tomar el número total de solicitudes facturables y dividirlo por el número de solicitudes aplicables, incluidas las solicitudes que generaron errores inesperados. Todos los errores inesperados reducen la disponibilidad del servicio de almacenamiento o de la operación de API especificada. <br/><br/> Unidad: Percent <br/> Tipo de agregación: Average <br/> Dimensiones aplicables: GeoType, ApiName y Authentication ([definición](#metrics-dimensions)) <br/> Ejemplo de valor: 99,99 |
