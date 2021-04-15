---
title: archivo de inclusión
description: archivo
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: emuench
ms.author: mingshen
ms.date: 10/09/2020
ms.openlocfilehash: f4e34e850391696506beed9f6f386f85528dff24
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92283385"
---
Si seleccionó una de las máquinas virtuales preconfiguradas con un sistema operativo (y, opcionalmente, otros servicios) ya ha seleccionado un tamaño de máquina virtual de Azure estándar. Iniciar la solución con un sistema operativo configurado previamente es el enfoque recomendado. Pero si va a instalar un sistema operativo manualmente, debe ajustar el tamaño del disco duro virtual principal en la imagen de la máquina virtual. Asegúrese de que el tamaño del disco del sistema operativo esté dentro de los límites de Linux o Windows.

| SO | Tamaño de VHD |
| --- | --- |
| Linux | 30 a 1023 GB |
| Windows | 30 a 250 GB |
|

Las imágenes base de Windows o SQL Server proporcionadas como base aprobada ya cumplen estos requisitos, por lo que no es necesario que cambie el formato ni el tamaño del disco duro virtual.

Los discos de datos pueden tener un tamaño de hasta 1 TB. A la hora de decidir el tamaño, tenga en cuenta que los clientes no pueden cambiar el tamaño de los VHD de una imagen en el momento de la implementación. Creación de discos duros virtuales de discos de datos como discos duros virtuales con un formato fijo. También deberían poderse expandir (dispersos o dinámicos). Los discos de datos inicialmente pueden estar vacíos o contener datos.