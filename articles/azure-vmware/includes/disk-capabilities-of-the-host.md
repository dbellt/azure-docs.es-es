---
title: Funcionalidad de disco de los hosts
description: Los hosts que se usan para crear o escalar clústeres provienen de un grupo de hosts aislado.
ms.topic: include
ms.date: 04/23/2021
ms.openlocfilehash: 7ee15fbfd668d5db24282a6e4de8e4dfc6639c3e
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945776"
---
<!-- Used in production-ready-deployment-steps.md and concepts-private-cloud-clusters.md -->


Los clústeres de Azure VMware Solution se basan en la infraestructura de reconstrucción completa e hiperconvergida. En la tabla siguiente se muestran las capacidades de RAM, CPU y disco del host.

| Tipo de host | CPU   | RAM (GB)  | Nivel de caché NVMe de vSAN (TB, sin procesar)  | Nivel de capacidad SSD de vSAN (TB, sin procesar)  |
| :---      | :---: | :---:     | :---:                           | :---:                             |
| AV36     |  Intel doble de 18 núcleos, 2,3 GHz  |     576      |                3.2               |                15,20               |

Los hosts que se usan para crear o escalar clústeres provienen de un grupo de hosts aislado. Esos hosts han superado las pruebas de hardware, y se han eliminado todos los datos de manera segura. 