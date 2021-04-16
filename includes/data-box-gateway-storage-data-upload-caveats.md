---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 385ebffb4780543b532c81b89f1847f5c84cd0ac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96581202"
---
Las advertencias siguientes se aplican a los datos cuando se trasladan en Azure.

- Se recomienda que no haya más de un dispositivo que escriba en el mismo contenedor.
- Si tiene un objeto de Azure (por ejemplo, un blob o un archivo) en la nube con el mismo nombre que el objeto que se copia, el dispositivo sobrescribirá el archivo en la nube.
- Una jerarquía de directorios vacía (sin archivos) que se creó en las carpetas de recurso compartido no se carga en los contenedores de blobs.
- Para copiar los datos se puede usar el método de arrastrar y colocar con el Explorador de archivos, o bien la línea de comandos. Si el tamaño total de los archivos que se copian supera los 10 GB, se recomienda usar un programa de copia masiva, como `Robocopy` o `rsync`. Si se producen errores intermitentes, las herramientas de copia masiva reintentan la operación de copia y proporcionan mayor resistencia.
- Si el recurso compartido asociado al contenedor de almacenamiento de Azure carga los blobs que no coinciden con el tipo de blobs definido para el recurso compartido en el momento de creación, estos blobs no se actualizan. Por ejemplo, si crea un recurso compartido de blob en bloques en el dispositivo, asócielo con un contenedor en la nube existente que tenga blobs en páginas, actualícelo para descargar los archivos y modifique algunos de los archivos actualizados que ya están almacenados como blobs en páginas en la nube, y verá errores de carga.
- Una vez que se crea un archivo en los recursos compartidos, no se puede cambiar su nombre.
- La eliminación de un archivo de un recurso compartido no elimina la entrada en la cuenta de almacenamiento.
- Si usa `rsync` para copiar los datos, no se admite la opción `rsync -a`.