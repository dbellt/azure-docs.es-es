---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 93f4f74d435cc14130668da102d1246c5fad5872
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186713"
---
| Identificador de límites | Límite | Comentarios |
| --- | --- | --- |
| Número máximo de credenciales de la cuenta de almacenamiento |64 | |
| Número máximo de contenedores de volúmenes |64 | |
| Número máximo de volúmenes |255 | |
| Número máximo de programaciones por plantilla de ancho de banda |168 |Una programación para cada hora y cada día de la semana. |
| Tamaño máximo de un volumen en capas en dispositivos físicos |64 TB para StorSimple 8100 y StorSimple 8600 |StorSimple 8100 y StorSimple 8600 son dispositivos físicos. |
| Tamaño máximo de un volumen en capas en dispositivos virtuales de Azure |30 TB para StorSimple 8010 <br></br> 64 TB para StorSimple 8020 |StorSimple 8010 y StorSimple 8020 son dispositivos virtuales de Azure que utilizan almacenamiento Standard y almacenamiento Premium, respectivamente. |
| Tamaño máximo de un volumen anclado localmente en dispositivos físicos |9 TB para StorSimple 8100 <br></br> 24 TB para StorSimple 8600 |StorSimple 8100 y StorSimple 8600 son dispositivos físicos. |
| Número máximo de conexiones iSCSI |512 | |
| Número máximo de conexiones de iSCSI de iniciadores |512 | |
| Número máximo de registros de control de acceso por dispositivo |64 | |
| Número máximo de volúmenes por directiva de copia de seguridad |24 | |
| Número máximo de copias de seguridad por directiva de copia de seguridad |64 | |
| Número máximo de programaciones por directiva de copia de seguridad |10 | |
| Número máximo de instantáneas de cualquier tipo que se pueden retener por volumen |256 |Esta cantidad incluye las instantáneas locales y en la nube. |
| Número máximo de instantáneas que pueden estar presentes en cualquier dispositivo |10 000 | |
| Número máximo de volúmenes que se pueden procesar en paralelo para copia de seguridad, restauración o clonación |16 |<ul><li>Si hay más de 16 volúmenes, se procesarán secuencialmente a medida que las ranuras de procesamiento estén disponibles.</li><li>No se pueden realizar copias de seguridad nuevas de un volumen clonado o de un volumen en capas restaurado hasta que no haya finalizado la operación. Para un volumen local, se permiten las copias de seguridad después de que el volumen está en línea.</li></ul> |
| Tiempo de recuperación de la restauración y la clonación |<2 minutos |<ul><li>El volumen estará disponible en dos minutos de una operación de restauración o clonación, independientemente del tamaño del volumen.</li><li>Es posible que el rendimiento del volumen resulte más lento de lo normal inicialmente, ya que la mayoría de los datos y metadatos todavía residen en la nube. Es posible que el rendimiento aumente a medida que los datos fluyan de la nube al dispositivo StorSimple.</li><li>El tiempo total para descargar los metadatos depende del tamaño del volumen asignado. Los metadatos se traen automáticamente al dispositivo en segundo plano a una velocidad de 5 minutos por cada TB de datos de volumen asignado. Es posible que esta velocidad se vea afectada por el ancho de banda de Internet a la nube.</li><li>La operación de restauración o clonación se completará cuando todos los metadatos estén en el dispositivo.</li><li>No se pueden realizar operaciones de copia de seguridad hasta que se haya completado totalmente la operación de restauración o clonación. |
| Restauración del tiempo de recuperación de volúmenes anclados localmente |<2 minutos |<ul><li>El volumen estará disponible en dos minutos de operación de restauración, independientemente del tamaño del volumen.</li><li>Es posible que el rendimiento del volumen resulte más lento de lo normal inicialmente, ya que la mayoría de los datos y metadatos todavía residen en la nube. Es posible que el rendimiento aumente a medida que los datos fluyan de la nube al dispositivo StorSimple.</li><li>El tiempo total para descargar los metadatos depende del tamaño del volumen asignado. Los metadatos se traen automáticamente al dispositivo en segundo plano a una velocidad de 5 minutos por cada TB de datos de volumen asignado. Es posible que esta velocidad se vea afectada por el ancho de banda de Internet a la nube.</li><li>A diferencia de los volúmenes en capas, si hay volúmenes anclados localmente, también se descargan los datos del volumen localmente en el dispositivo. La operación de restauración finaliza cuando todos los datos de volúmenes se transfieren al dispositivo.</li><li>Las operaciones de restauración pueden ser largas y el tiempo total para completar la restauración dependerá del tamaño del volumen local aprovisionado, el ancho de banda de Internet y los datos existentes en el dispositivo. Se permiten las operaciones de copia de seguridad en el volumen anclado localmente mientras la operación de restauración está en curso. |
| Disponibilidad de restauración fina |Última conmutación por error | |
| Rendimiento máximo de lectura y escritura del cliente, cuando se obtiene desde la capa SSD* |920/720 MB/s con una única interfaz de red Ethernet de 10 gigabits |Hasta dos veces con MPIO y dos interfaces de red. |
| Rendimiento máximo de lectura y escritura del cliente, cuando se obtiene desde la capa HDD* |120/250 MB/s | |
| Rendimiento máximo de lectura y escritura de cliente, cuando se obtiene desde la capa de la nube* |11/41 MB/s |El rendimiento de lectura depende de que los clientes generen y mantengan una profundidad de cola de E/S suficiente. |

&#42;Se ha medido el rendimiento máximo por tipo de E/S con escenarios de escritura y de lectura del 100 %. Es posible que el rendimiento real sea inferior y dependa de las condiciones de la red y de la mezcla de E/S.

