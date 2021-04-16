---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 4/05/2021
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: 01512f0e37761915fcb6eeba8c162e1a3db62c48
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491696"
---
```console
Robocopy /MT:16 /R:5 /W:5 /B /MIR /IT /COPY:DATSO /DCOPY:DAT /NP /NFL /NDL /UNILOG:<FilePathAndName> <SourcePath> <Dest.Path> 
```

| Conmutador                | Significado |
|-----------------------|---------|
| `/MT:n`               | Permite que RoboCopy se ejecute en modo multiproceso. El valor predeterminado es n = 8 y el máximo son 128 subprocesos. Haga coincidir este valor con el número de núcleos del procesador y el número de subprocesos por núcleo. Considere si es necesario reservar los núcleos para otras tareas que quizá tenga un servidor de producción. |
| `/R:n`                | La velocidad de una ejecución de RoboCopy puede mejorarse si se especifica un número máximo de reintentos para un archivo que no se puede copiar en el primer intento. n = número máximo de reintentos antes de que el archivo no se pueda copiar de forma permanente en esta ejecución de RoboCopy. Este modificador funciona mejor en escenarios en los que ya está claro que habrá más ejecuciones de RoboCopy. Si el archivo no se puede copiar en esta ejecución, se volverá a intentar en el siguiente trabajo de RoboCopy. A menudo, los archivos que generan errores porque estaban en uso o debido a problemas de tiempo de espera, pueden copiarse correctamente con esta estrategia. |
| `/W:n`                | Este modificador especifica el tiempo que espera RoboCopy antes de intentar copiar un archivo que no se ha copiado correctamente en el último intento. n = número de segundos de espera entre reintentos. `/W:n` a menudo se usa junto con `/R:n`. |
| `/B`                  | Ejecuta RoboCopy en el mismo modo que usaría una aplicación de copia de seguridad. Permite que Robocopy mueva los archivos para los que el usuario actual no tiene permisos. |
| `/MIR`                | *Reflejar origen en destino* permite que RoboCopy solo tenga que copiar las diferencias entre el origen y el destino. Se copiarán los subdirectorios vacíos. Se copiarán los elementos (archivos o carpetas) que hayan cambiado o no existan en el destino. Los elementos que existan en el destino, pero no en el origen, se purgarán (se eliminarán) del destino. Al usar este modificador, es imperativo que haga coincidir de forma exacta la estructura de carpetas de origen y destino. "Coincidencia" significa que se copia desde el nivel de carpeta y origen correctos en el nivel de carpeta del destino coincidente. Solo entonces se puede realizar correctamente una copia de "puesta al día". Cuando el origen y el destino no coinciden, el uso de `/MIR` dará lugar a eliminaciones y nuevas copias a gran escala. |
| `/IT`                 | Garantiza que se conserve la fidelidad en ciertos escenarios de reflejo. </br>*Ejemplo*: Si entre dos ejecuciones de RoboCopy, un archivo experimenta un cambio de ACL y una actualización de atributo: por ejemplo, está marcado como *oculto*. Sin /IT, RoboCopy podría omitir el cambio de ACL y no se transferiría a la ubicación de destino. |
|`/COPY:[copyflags]`    | Fidelidad de la copia de archivos (el valor predeterminado, si no se especifica, es `/COPY:DAT`), marcas de copia: `D` = datos, `A` = atributos, `T` = marcas de tiempo, `S` = seguridad = ACL de NTFS, `O` = información del propietario, `U` = i<u>n</u>formación de auditoría. No se puede almacenar la información de auditoría en un recurso compartido de archivos de Azure. |
| `/DCOPY:[copyflags]`  | Fidelidad de la copia de directorios (el valor predeterminado, si no se especifica, es `/DCOPY:DA`), marcas de copia: `D` = datos, `A` = atributos, `T` = marcas de tiempo. |
| `/NP`                 | No se mostrará el progreso de la copia de cada archivo y carpeta. Mostrar el progreso reduce significativamente el rendimiento de la copia. |
| `/NFL`                | Especifica que los nombres de archivo no se han registrado. Mejora el rendimiento de la copia. |
| `/NDL`                | Especifica que los nombres de directorio no se han registrado. Mejora el rendimiento de la copia. |
| `/UNILOG:<file name>` | Envía el estado al archivo de registro como UNICODE (sobrescribe el registro existente). |
| `/LFSM`               | **solo para destinos con almacenamiento en capas** </br>El uso de /LFSM solicita RoboCopy para funcionar en "modo de poco espacio disponible". Este modificador solo es útil para destinos con almacenamiento en capas, que pueden quedarse sin capacidad local antes de que RoboCopy pueda finalizar. Este modificador se agregó específicamente para su uso con un destino habilitado de nube por niveles de Azure File Sync. Se puede usar con independencia de Azure File Sync. En este modo, RoboCopy se pondrá en pausa siempre que una copia de archivo haga que el espacio disponible del volumen de destino se sitúe por debajo de un valor 'floor'. El formato `/LFSM:n` de la marca puede especificar este valor. El parámetro `n` se especifica en la base 2: `nKB`, `nMB` o `nGB`. Si `/LFSM` se especifica sin ningún valor floor explícito, floor se establece en el 10 por ciento del tamaño del volumen de destino. El modo de poco espacio disponible es incompatible con /MT, /EFSRAW, /B, y /ZB. |
| `/Z`                  | **uso cuidadoso** </br>Copia los archivos en modo de reinicio; el uso solo se recomienda en un entorno de red inestable. Esta opción reduce significativamente el rendimiento de la copia debido al registro adicional. |
| `/ZB`                 | **uso cuidadoso** </br>Usa el modo de reinicio. Si se deniega el acceso, esta opción utiliza el modo de copia de seguridad. Esta opción reduce significativamente el rendimiento de la copia debido a los puntos de control. |
   