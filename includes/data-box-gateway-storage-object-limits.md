---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 20d30935fe03bc002ab63f2f8ca1ce890ef9e3b4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96581191"
---
Estos son los tamaños de los objetos de Azure que se pueden escribir. Asegúrese de que todos los archivos que se cargan se ajustan a estos límites.

| Tipo de objeto de Azure | Límite de carga                                             |
|-------------------|-----------------------------------------------------------|
| Blob en bloques        | ~ 4,75 TB                                                 |
| Blob en páginas         | 1 TB <br> Todos los archivos cargados en formato de blob en páginas deben tener 512 bytes alineados (un múltiplo entero); de lo contrario, se produce un error en la carga. <br> VHD y VHDX tienen 512 bytes alineados. |
| Azure Files         | 1 TB <br> Todos los archivos cargados en formato de blob en páginas deben tener 512 bytes alineados (un múltiplo entero); de lo contrario, se produce un error en la carga. <br> VHD y VHDX tienen 512 bytes alineados. |

> [!IMPORTANT]
> Se permite la creación de archivos (independientemente del tipo de almacenamiento) de hasta 5 TB. Sin embargo, si crea un archivo cuyo tamaño sea mayor que el límite de carga definido en la tabla anterior, no se cargará el archivo. Tendrá que eliminar manualmente el archivo para recuperar el espacio.