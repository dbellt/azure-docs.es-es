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
ms.openlocfilehash: 57d14ae6e6da7cfa883f1aed74cce390c0185d98
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491698"
---
La velocidad y la tasa de éxito de una ejecución determinada de RoboCopy dependerán de varios factores:

* El número de IOPS en el almacenamiento de origen y de destino.
* El ancho de banda de red disponible entre el origen y el destino.
* La capacidad de procesar rápidamente archivos y carpetas en un espacio de nombres.
* El número de cambios entre ejecuciones de RoboCopy.


### <a name="iops-and-bandwidth-considerations"></a>Consideraciones sobre el ancho de banda y el número de IOPS.

En esta categoría, debe tener en cuenta la capacidad del **almacenamiento de origen**, el **almacenamiento de destino** y la **red** que los conecta. El mayor rendimiento posible viene determinado por el más lento de estos tres componentes. Asegúrese de que la infraestructura de red se haya configurado para admitir las mejores velocidades de transferencia.

> [!CAUTION]
> Aunque es deseable copiar lo más rápido posible, tenga en cuenta el uso de la red local y el dispositivo NAS en otras tareas que suelen ser esenciales para la empresa.

Copiar lo más rápido posible podría no ser deseable si existe el riesgo de que la migración acapare los recursos disponibles.

* Tenga en cuenta cuándo es mejor para su entorno hacer migraciones: durante el día, fuera del horario laboral o en los fines de semana.
* Considere también la calidad de servicio de redes en Windows Server para limitar la velocidad de RoboCopy.
* Evite trabajo innecesario a las herramientas de migración.

RobCopy puede introducir retrasos entre paquetes al especificar el modificador `/IPG:n`, en donde el valor `n` se calcula en milisegundos entre los paquetes de RoboCopy. El uso de este modificador puede ayudar a evitar el acaparamiento de los recursos en dispositivos de E/S restringidos y vínculos de red saturados.

`/IPG:n` no se puede usar para limitar la red a un número exacto de megabits por segundo. En su lugar, use la calidad de servicio de red de Windows Server. RoboCopy se basa íntegramente en el protocolo SMB para todas las necesidades de red. El uso de SMB es el motivo por el que RoboCopy no puede influir en el propio rendimiento de la red, pero puede ralentizar su uso. 

Un enfoque similar se aplica a las operaciones de IOPS observadas en el dispositivo NAS. El tamaño del clúster en el volumen de NAS y los tamaños de paquete, entre otros factores, influyen en las IOPS observadas. La introducción de retraso entre paquetes suele ser la manera más fácil de controlar la carga en el dispositivo NAS. Pruebe varios valores, por ejemplo, de 20 milisegundos (n = 20) a múltiplos de ese número. Una vez que introduzca un retraso, podrá valorar si las otras aplicaciones funcionan según lo esperado. Esta estrategia de optimización le permitirá encontrar la velocidad de RoboCopy óptima para su entorno.

### <a name="processing-speed"></a>Velocidad de procesamiento

RoboCopy recorrerá el espacio de nombres al que se apunta y evaluará cada archivo y carpeta con fines de copia. Los archivos se evaluarán durante una copia inicial y durante las copias de puesta al día. Por ejemplo, ejecuciones repetidas de RoboCopy/MIR en las mismas ubicaciones de almacenamiento de origen y de destino. Estas ejecuciones repetidas son útiles para minimizar el tiempo de inactividad de los usuarios y las aplicaciones, así como para mejorar la tasa de éxito general de los archivos migrados.

A menudo, el ancho de banda suele considerarse como el factor más restrictivo en una migración, algo que puede ser cierto. Pero la posibilidad de enumerar un espacio de nombres puede influir en el tiempo total de copia para espacios de nombres más largos con archivos más pequeños. Tenga en cuenta que copiar 1 TiB de archivos pequeños tardará mucho más que copiar 1 TiB de un número inferior de archivos de mayor tamaño. Esto es así si damos por hecho que las demás variables no cambian.

La razón de esta diferencia es la potencia de procesamiento necesaria para recorrer un espacio de nombres. RoboCopy admite copias multiproceso a través del parámetro `/MT:n`, en donde "n" indica el número de subprocesos del procesador. Por lo tanto, al aprovisionar una máquina específicamente para RoboCopy, tenga en cuenta el número de núcleos de procesador y su relación con el número de subprocesos que proporcionan. Lo más habitual son dos subprocesos por núcleo. El número de núcleos y subprocesos de una máquina es un punto de datos importante para determinar qué valores multiproceso `/MT:n` se deberían especificar. Tenga en cuenta también cuántos trabajos de RoboCopy tiene previsto ejecutar al mismo tiempo en una máquina determinada.

Un número mayor de subprocesos copiarán nuestro ejemplo de 1 TiB de archivos pequeños considerablemente más rápido que un número menor de subprocesos. Al mismo tiempo, la inversión adicional en recursos en 1 TiB de archivos de más grandes podría no aportar ventajas proporcionales. Un número mayor de subprocesos intentará copiar simultáneamente más archivos grandes a través de la red. Esta actividad de red adicional aumentará la probabilidad de sufrir restricciones asociadas al rendimiento o a las operaciones de IOPS de almacenamiento.

### <a name="avoid-unnecessary-work"></a>Evitar el trabajo innecesario

Evite cambios a gran escala en el espacio de nombres. Por ejemplo, mover archivos entre directorios, cambiar propiedades a gran escala o cambiar permisos (ACL de NTFS). Los cambios en la ACL en especial pueden tener una importante repercusión, ya que con frecuencia tienen un efecto de cambio en cascada en los archivos de nivel inferior en la jerarquía de carpetas. Entre las consecuencias, cabe destacar las siguientes:

* La ampliación del tiempo de ejecución del trabajo de RoboCopy, ya que será necesario actualizar cada archivo y carpeta a los que afecte un cambio de ACL.
* Es posible que haya que volver a copiar los datos que se migraron anteriormente. Por ejemplo, habrá que copiar más datos si cambian las estructuras de carpetas, aunque los archivos se hubieran copiado anteriormente. Un trabajo de RoboCopy no puede "reproducir" un cambio del espacio de nombres. El siguiente trabajo debe purgar los archivos transportados previamente en la estructura de carpetas antigua y volver a cargar los archivos en la nueva estructura de carpetas.

Otro aspecto importante es usar la herramienta RoboCopy de forma eficaz. Con el script de RoboCopy recomendado, se creará y guardará un archivo de registro de los errores. Es normal que puedan producirse errores de copia. Estos errores suelen hacer que sea necesario ejecutar varias rondas de una herramienta de copia como RoboCopy. Una ejecución inicial, por ejemplo, de un dispositivo NAS a DataBox o de un servidor a un recurso compartido de archivos de Azure. Y una o varias ejecuciones adicionales con el modificador /MIR para identificar los archivos que no se han copiado y volver a intentarlo.

Debe estar preparado para ejecutar varias rondas de RoboCopy en el ámbito de un espacio de nombres determinado. Las sucesivas ejecuciones finalizarán más rápido, ya que tienen menos que copiar, pero cada vez tendrán más restricciones debido a la velocidad de procesamiento del espacio de nombres. Cuando ejecute varias rondas, puede acelerar cada una de ellas si evita que RoboCopy intente copiarlo todo en una misma ejecución. Los siguientes modificadores RoboCopy pueden marcar una diferencia importante:

* `/R:n` n = la frecuencia con la que se vuelve a intentar copiar un archivo con errores 
* `/W:n` n = el número de segundos que hay que esperar entre reintentos

`/R:5 /W:5` es un valor razonable que puede ajustar a su gusto. En este ejemplo, un archivo con errores se intentará volver a copiar cinco veces, con un tiempo de espera de cinco segundos entre un reintento y otro. Si el archivo sigue sin copiarse, el siguiente trabajo de RoboCopy lo volverá a intentar. A menudo, los archivos que generan errores por estar en uso o debido a problemas de tiempo de espera se pueden copiar correctamente de esta manera.
   
