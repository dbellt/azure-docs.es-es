---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/27/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: e96d205ef1a8f94baa3a0cfe6c5127b6cf570e5a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "80420948"
---
En este artículo se explica cómo cargar un disco duro virtual de la máquina local a un disco administrado de Azure o copiar un disco administrado a otra región mediante AzCopy. Este proceso, la carga directa, también permite cargar un disco duro virtual de hasta 32 TiB directamente en un disco administrado. Actualmente, la carga directa es compatible con los HDD estándar, la unidad de estado sólido estándar y los discos administrados SSD prémium. Aún no es compatible con Ultra Disks.

Si va a proporcionar una solución de copia de seguridad para las máquinas virtuales de IaaS en Azure, se recomienda usar la carga directa para restaurar las copias de seguridad de clientes en discos administrados. Al cargar un disco duro virtual desde una fuente externa en Azure, las velocidades dependerán del ancho de banda local. Al cargar o copiar desde una máquina virtual de Azure, el ancho de banda sería el mismo que el de los discos duros estándar.