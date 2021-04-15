---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/18/2019
ms.author: alkohli
ms.openlocfilehash: 05673885336dbfb9a70843bd0ccc4e700091ad7e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "67186984"
---
|     Código de error     |      Descripción del error     |
|--------------------|--------------------------|
|    100             | El nombre del contenedor o recurso compartido debe tener entre 3 y 63 caracteres.|
|    101             | El nombre del contenedor o recurso compartido debe constar solo de letras, números o guiones.|
|    102             | El nombre del contenedor o recurso compartido debe constar solo de letras, números o guiones.|
|    103             | El nombre del blob o archivo contiene caracteres de control no admitidos.|
|    104             | El nombre del blob o archivo contiene caracteres no válidos.|
|    105             | El nombre de archivo o blob contiene demasiados segmentos (cada segmento está separado por una barra diagonal: /).|
|    106             | El nombre de blob o archivo es demasiado largo.|
|    107             | Uno de los segmentos del nombre de blob o archivo es demasiado largo. |
|    108             | El tamaño del archivo supera el tamaño máximo para la carga.    |
|    109             | El blob o archivo no está alineado correctamente.  |
|    110             | El nombre de archivo con código Unicode o el blob no son válidos.|
|    111             | El nombre o el prefijo del archivo o blob es un nombre reservado que no se admite (por ejemplo, COM1).|
|    2000            | Un error de coincidencia de ETag indica que hay un conflicto entre un blob en bloques que se encuentra en la nube y otro que se encuentra en el dispositivo. Para resolver este conflicto, elimine uno de los archivos, ya sea la versión en la nube o la versión en el dispositivo.    |
|    2001            | Se ha producido un problema inesperado al procesar un archivo después de que se cargara el archivo.    Si ve este error y no desaparece antes de 24 horas, póngase en contacto con el equipo de soporte técnico. |
|    2002            | El archivo está abierto en otro proceso y no se puede cargar hasta que se cierre el manipulador.|
|    2003            | No se pudo abrir el archivo para cargarlo. Si ve este error, póngase en contacto con el servicio de soporte técnico de Microsoft.|
|    2004            | No se pudo conectar al contenedor para cargar datos en él.|
|    2005            | No se pudo conectar al contenedor porque los permisos de la cuenta están obsoletos o no son correctos. Compruebe los derechos de acceso.|
|    2006            | No se pudieron cargar datos en la cuenta porque la cuenta o el recurso compartido están deshabilitados.|
|    2007            | No se pudo conectar al contenedor porque los permisos de la cuenta están obsoletos o no son correctos. Compruebe los derechos de acceso.|
|    2008            | No se pudieron agregar datos nuevos porque el contenedor está lleno. Consulte en las especificaciones de Azure los tamaños de contenedor admitidos en función del tipo. Por ejemplo, Azure Files solo admite un tamaño máximo de 5 TB.|
|    2009            | No se pudieron cargar los datos porque no existe el contenedor asociado con el recurso compartido.|    
|    2997            | Se ha producido un error inesperado. Este es un error transitorio que se resolverá automáticamente.|
|    2998            | Se ha producido un error inesperado. El error se puede solucionar por sí solo, pero si persiste durante más de 24 horas, póngase en contacto con el Soporte técnico de Microsoft.|
|    16000           | No se pudo eliminar este archivo.|
|    16001           | No se pudo eliminar este archivo porque ya existe en el sistema local.|
|    16002           |No se pudo actualizar este archivo, ya que no está totalmente cargado.|

