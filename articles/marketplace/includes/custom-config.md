---
title: archivo de inclusión
description: archivo
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: emuench
ms.author: mingshen
ms.date: 10/09/2020
ms.openlocfilehash: a1bc7cf1fd339ca3660c7b39326f37d2763c74b2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92283390"
---
Si necesita alguna configuración adicional, use una tarea programada que se ejecute al inicio para realizar cambios finales en la máquina virtual una vez implementada. Tenga en cuenta lo siguiente:

- Si es una tarea de una ejecución, la tarea debe eliminarse a sí misma una vez que se realice correctamente.
- Las configuraciones no deben basarse en unidades distintas de C o D, ya que solo se garantiza la existencia continua de estas dos unidades (la unidad C es el disco del sistema operativo y la unidad D es el disco local temporal).

Para obtener más información sobre las personalizaciones de Linux, vea [Características y extensiones de las máquinas virtuales para Linux](../../virtual-machines/extensions/features-linux.md).