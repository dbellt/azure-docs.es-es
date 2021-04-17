---
ms.service: hpc-cache
ms.topic: include
ms.date: 03/15/2021
author: ekpgh
ms.author: v-erkel
ms.openlocfilehash: 2804b3bb36a31c9cfbf4cce8db94b7469c8975c7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "103563250"
---
| Modelo de uso | Modo de almacenamiento en caché | Comprobación de back-end | Retraso máximo de reescritura |
|--|--|--|--|
| Lectura de textos densos y poco frecuentes | Lectura | Nunca | None |
| Mayor que el 15 % de textos | Lectura/escritura | 8 horas | 20 minutos |
| Los clientes omiten la memoria caché | Lectura | 30 segundos | None |
| Mayor que el 15 % de escrituras, comprobación de back-end frecuente (30 segundos) | Lectura/escritura | 30 segundos | 20 minutos |
| Mayor que el 15 % de escrituras, comprobación de back-end frecuente (60 segundos) | Lectura/escritura | 60 segundos | 20 minutos |
| Mayor que el 15 % de escrituras, reescritura frecuente | Lectura/escritura | 30 segundos | 30 segundos |
| Lectura intensiva, comprobación del servidor de copia de seguridad cada 3 horas | Lectura | 3 horas | None |
