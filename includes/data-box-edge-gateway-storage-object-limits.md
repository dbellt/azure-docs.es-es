---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 02/04/2019
ms.author: alkohli
ms.openlocfilehash: 563849d3875ed0156d81770f58340633d90d515b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186768"
---
Estos son los tamaños de los objetos de Azure que se pueden escribir. Asegúrese de que todos los archivos que se cargan se ajustan a estos límites.

| Tipo de objeto de Azure | Límite de carga                                             |
|-------------------|-----------------------------------------------------------|
| Blob en bloques        | ~ 4,75 TB                                                 |
| Blob en páginas         | 1 TB <br> Todos los archivos cargados en formato de blob en páginas deben tener 512 bytes alineados (un múltiplo entero); de lo contrario, se produce un error en la carga. <br> VHD y VHDX tienen 512 bytes alineados. |
| Azure Files         | 1 TB <br> Todos los archivos cargados en formato de blob en páginas deben tener 512 bytes alineados (un múltiplo entero); de lo contrario, se produce un error en la carga. <br> VHD y VHDX tienen 512 bytes alineados. |

> [!IMPORTANT]
> Se permite la creación de archivos (independientemente del tipo de almacenamiento) de hasta 5 TB. Sin embargo, si crea un archivo cuyo tamaño sea mayor que el límite de carga definido en la tabla anterior, no se cargará el archivo. Tendrá que eliminar manualmente el archivo para recuperar el espacio.