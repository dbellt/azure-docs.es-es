---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 09/12/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: ec6cbcbc93fe87634c87caeb0041b75ec916a22f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186285"
---
En Azure, para abrir un puerto o crear un punto de conexión a una máquina virtual, debe crear un filtro de red en una subred o una interfaz de red de máquina virtual. Estos filtros, que controlan el tráfico entrante y saliente, se colocan en un grupo de seguridad de red asociado al recurso que va a recibir dicho tráfico.

El ejemplo de este artículo muestra cómo crear un filtro de red que utiliza el puerto TCP estándar 80 (se supone ya ha iniciado los servicios apropiados y ha abierto las reglas de firewall del sistema operativo en la máquina virtual).

Después de crear una máquina virtual configurada para atender las solicitudes web en el puerto TCP 80 estándar, puede:

1. Cree un grupo de seguridad de red.

2. Crear una regla de seguridad de entrada que permita el tráfico y asignar valores a las siguientes opciones:

   - **Intervalos de puertos de destino**: 80

   - **Intervalos de puertos de origen**: * (permite cualquier puerto de origen)

   - **Valor de prioridad**: escriba un valor inferior a 65.500 (para que tenga mayor prioridad que la regla de entrada predeterminada de denegación de comodín).

3. Asociar el grupo de seguridad de red con la subred o la interfaz de red de máquina virtual.

Aunque este ejemplo usa una regla sencilla para permitir el tráfico HTTP, también puede usar reglas y grupos de seguridad de red para crear configuraciones de red más complejas. 




