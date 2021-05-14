---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 04/15/2021
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 4435784592954e4df44d5ea8dc05b33a0023fd43
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108313415"
---
- No es compatible con discos Ultra.
- No se puede habilitar si Azure Disk Encryption (cifrado de máquina virtual de invitado con BitLocker/VM-Decrypt) está habilitado en las máquinas virtuales o en los conjuntos de escalado de máquinas virtuales.
- No se puede habilitar Azure Disk Encryption en discos que tienen habilitado el cifrado en el host.
- El cifrado se puede habilitar en el conjunto de escalado de máquinas virtuales existente. Sin embargo, solo se cifrarán automáticamente las nuevas máquinas virtuales creadas después de habilitar el cifrado.
- Las máquinas virtuales existentes se deben desasignar y reasignar para su cifrado.
- Admite discos del sistema operativo efímeros, pero solo con claves administradas por la plataforma.