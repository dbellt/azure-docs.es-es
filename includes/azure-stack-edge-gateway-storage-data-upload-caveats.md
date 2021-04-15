---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/13/2020
ms.author: alkohli
ms.openlocfilehash: dc18efe03cbc8a3f657ae4afc781941e8e76611c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96465860"
---
Las advertencias siguientes se aplican a los datos cuando se trasladan en Azure.

- Se recomienda que no haya más de un dispositivo que escriba en el mismo contenedor.
- Si tiene un objeto existente de Azure (por ejemplo, un blob o un archivo) en la nube con el mismo nombre que el objeto que se está copiando, el dispositivo sobrescribirá el archivo en la nube.
- Una jerarquía de directorios vacía (sin archivos) que se creó en las carpetas de recurso compartido no se carga en los contenedores de blobs.
- Puede copiar los datos mediante el método de arrastrar y colocar con el Explorador de archivos o mediante la línea de comandos. Si el tamaño total de los archivos que se copian es mayor que 10 GB, se recomienda usar un programa de copia masiva, como Robocopy o rsync. Las herramientas de copia masiva reintentan la operación de copia en caso de errores intermitentes y proporcionan resistencia adicional. Si se usa Blob Storage a través de REST, puede usar AzCopy o el Explorador de Azure Storage.
- Si el recurso compartido asociado al contenedor de almacenamiento de Azure carga los blobs que no coinciden con el tipo de blobs definido para el recurso compartido en el momento de creación, estos blobs no se actualizan. Por ejemplo, se crea un recurso compartido de blob en bloques en el dispositivo. Se asocia el recurso compartido a un contenedor en la nube existente que tiene blobs en páginas. Se actualiza ese recurso compartido para descargar los archivos. Se modifican algunos de los archivos actualizados que ya están almacenados como blobs en páginas en la nube. Aparecerán errores de carga.
- Una vez que se crea un archivo en los recursos compartidos, no se puede cambiar su nombre.
- La eliminación de un archivo de un recurso compartido no elimina la entrada en la cuenta de almacenamiento.
- Si usa rsync para copiar los datos, la opción `rsync -a` no se admitirá.
