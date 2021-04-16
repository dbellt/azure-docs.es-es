---
author: cynthn
ms.service: azure
ms.topic: include
ms.date: 11/09/2018
ms.author: cynthn
ms.openlocfilehash: c2ed33aea77b5478e8d17f6bd0213ef3e778b806
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186652"
---
## <a name="transfer-local-files-to-cloud-shell"></a>Transferir archivos locales a Cloud Shell
El directorio `clouddrive` se sincroniza con la hoja de almacenamiento de Azure Portal. Use esta hoja para transferir archivos locales al recurso compartido de archivos y viceversa. La actualización de archivos en Cloud Shell se refleja en la GUI de File Storage al actualizarse las hojas.

### <a name="download-files"></a>Descarga de archivos

![Lista de archivos locales](../articles/cloud-shell/media/persisting-shell-storage/download.png)
1. En Azure Portal, vaya al recurso compartido de archivos montados.
2. Seleccione el archivo de destino.
3. Seleccione el botón **Descargar**.

### <a name="upload-files"></a>Carga de archivos

![Archivos locales para cargarse](../articles/cloud-shell/media/persisting-shell-storage/upload.png)
1. Vaya al recurso compartido de archivos montado.
2. Seleccione el botón **Cargar**.
3. Seleccione el archivo o archivos que desea cargar.
4. Confirmación de la actualización.

Ahora debería ver los archivos accesibles en el directorio `clouddrive` en Cloud Shell.