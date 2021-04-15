---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: include
ms.date: 01/08/2021
ms.author: mimig
ms.custom: include file
ms.openlocfilehash: 698219a446240950b9a661f923109fbb2f863556
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106178457"
---
Si actualmente usa Azure Table Storage, obtendrá las siguientes ventajas al actualizar a Table API de Azure Cosmos DB:

|Característica | Almacenamiento de tablas de Azure | Table API de Azure Cosmos DB |
| --- | --- | --- |
| Latencia | Rápido, pero no hay límites máximos en la latencia. | Latencia en milisegundos de un solo dígito para lecturas y escrituras, respaldada con lecturas con una latencia inferior a 10 ms y escrituras con una latencia inferior a 15 ms en el percentil 99, a cualquier escala, en cualquier lugar del mundo. |
| Throughput | Modelo de rendimiento variable. Las tablas tienen un límite de escalabilidad de 20.000 operaciones por segundo. | Altamente escalable con [rendimiento reservado dedicado por tabla](../articles/cosmos-db/request-units.md) respaldado por los SLA. Las cuentas no tienen límite máximo en el rendimiento y admiten más de 10 millones de operaciones por segundo por tabla (en el modo de rendimiento aprovisionado). |
| Distribución global | Una sola región con una región de lectura secundaria legible opcional para alta disponibilidad que admite la conmutación por error de cuentas automática y manual. | [Distribución global inmediata](../articles/cosmos-db/distribute-data-globally.md) desde 1 a más de 30 regiones. Admite [conmutaciones por error automáticas y manuales](../articles/cosmos-db/high-availability.md) en cualquier momento y en cualquier lugar del mundo. |
| Indización | Índice principal solo en PartitionKey y RowKey. No hay índices secundarios. | Indexación automática y completa en todas las propiedades, sin administración de índices. |
| Consultar | La ejecución de consultas usa el índice de la clave principal y, en caso contrario, examina. | Las consultas pueden aprovechar la indexación automática en las propiedades para reducir el tiempo de consulta. |
| Coherencia | Seguro dentro de la región principal. Ocasional en la región secundaria. | [Cinco niveles de coherencia bien definidos](../articles/cosmos-db/consistency-levels.md) para compensar la disponibilidad, la latencia, el rendimiento y la coherencia en función de las necesidades de la aplicación. |
| Precios | Basado en el consumo. | Disponible en los modos [basado en el consumo](../articles/cosmos-db/serverless.md) y [capacidad aprovisionada](../articles/cosmos-db/set-throughput.md). |
| SLA | disponibilidad del 99,99%. | Acuerdo de Nivel de Servicio con disponibilidad del 99,99 % para todas las cuentas de región individual y todas las cuentas de varias regiones con coherencia menos estricta y disponibilidad del 99,999 % para todos los [Acuerdos de Nivel de Servicio completos líderes en el sector](https://azure.microsoft.com/support/legal/sla/cosmos-db/) de todas las cuentas de base de datos de varias regiones de lectura sobre la disponibilidad general. |
