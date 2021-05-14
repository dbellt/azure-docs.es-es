---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 07/08/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: e9d7b39ce267202503b90e84e934d31501d45478
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107732485"
---
Toda la última generación de tamaños de máquina virtual admite el cifrado en el host:

|Tipo  |No compatible  |Compatible  |
|---------|---------|---------|
|Uso general     | Dv3, Dv2, Av2        | B, DSv2, Dsv3, DC, DCv2, Dav4, Dasv4, Ddv4, Ddsv4       |
|Proceso optimizado     |         | Fsv2        |
|Memoria optimizada     | Ev3        | Esv3, M, Mv2, Eav4, Easv4, Edv4, Edsv4        |
|Almacenamiento optimizado     |         | Ls, Lsv2 (discos NVMe no cifrados)        |
|GPU     | NC, NV        | NCv2, NCv3, ND, NVv3, NVv4, NDv2 (versión preliminar)        |
|Informática de alto rendimiento     | H        | HB, HC, HBv2        |
|Generaciones anteriores     | F, A, D, L, G        | DS, GS, Fs, NVv2        |
