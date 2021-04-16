---
title: archivo de inclusión
description: archivo de inclusión
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: include
ms.date: 12/05/2019
ms.author: duau
ms.custom: include file
ms.openlocfilehash: b1465b9bf855d592d195277b616c860a23c179f6
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106504718"
---
| SKU de puerta de enlace | Conexiones por segundo | Número de flujos | Megabits por segundo | Paquetes por segundo | Ancho de banda del circuito | Número de rutas anunciadas por la puerta de enlace (a MSEE) | Número de rutas que ha aprendido la puerta de enlace (de MSEE) | Número de máquinas virtuales en la red virtual |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **SKU básica (obsoleto)** | N/D | N/D | 500 | N/D | N/D | N/D | N/D | N/D |
| **SKU estándar/ErGw1AZ** | 7000 | 400.000 | >1000 | >100 000 | 1 Gbps |  500 | 4\.000 | 2000 (reducir a 1000 durante el mantenimiento, se restaura después). | 
| **SKU de alto rendimiento/ErGw2AZ** | 14 000 | 840 000 | >2000 | 250 000 | 1 Gbps | 500 | ~9500 (reducir a 4000 si hay más de 6500 máquinas virtuales en la red virtual). | 4500 |
| **SKU de ultrarrendimiento/ErGw3AZ** | 16 000 | 950 000 | ~10 000 | 1 000 000 | 1 Gbps | 500 | ~9500 | 11 000 |
