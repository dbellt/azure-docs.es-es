---
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: include
ms.date: 05/21/2019
ms.author: alkohli
ms.openlocfilehash: 9805f53d5901226fc9e32b24a323256cd1da6844
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "88655162"
---
Estos son los tamaños de los objetos de Azure que se pueden escribir. Asegúrese de que todos los archivos que se cargan se ajustan a estos límites.

| Tipo de objeto de Azure | Límite predeterminado                                             |
|-------------------|-----------------------------------------------------------|
| Blob en bloques        | ~4,75 TiB                                                 |
| Blob en páginas         | 8 TiB <br> Todos los archivos cargados en formato de blob en páginas deben tener 512 bytes alineados (un múltiplo entero), de lo contrario, se produce un error en la carga. <br> VHD y VHDX tienen 512 bytes alineados. |
| Azure Files        | 1 TiB                                                      |
| Discos administrados     | 4 TiB <br> Para más información sobre el tamaño y los límites, consulte: <li>[Objetivos de escalabilidad de SSD estándar](../articles/virtual-machines/disks-types.md#standard-ssd)</li><li>[Objetivos de escalabilidad de SSD Premium](../articles/virtual-machines/disks-types.md#standard-hdd)</li><li>[Objetivos de escalabilidad de unidades de disco duro estándar](../articles/virtual-machines/disks-types.md#premium-ssd)</li><li>[Descripción de precios y facturación de discos administrados](../articles/virtual-machines/disks-types.md#billing)</li>  
