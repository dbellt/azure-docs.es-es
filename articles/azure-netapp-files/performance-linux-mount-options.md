---
title: Procedimientos recomendados en las opciones de montaje de NFS de Linux para Azure NetApp Files | Microsoft Docs
description: Describe las opciones de montaje y los procedimientos recomendados para usarlas con Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/01/2021
ms.author: b-juche
ms.openlocfilehash: f62c22c5615b1494ad8c1ebb966db9f1c25a8df7
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111442125"
---
# <a name="linux-nfs-mount-options-best-practices-for-azure-netapp-files"></a>Procedimientos recomendados en las opciones de montaje de NFS de Linux para Azure NetApp Files

Este artículo le ayudará a conocer mejor las opciones de montaje y los procedimientos recomendados para usarlas con Azure NetApp Files.

## `Nconnect` 

El uso de la opción de montaje `nconnect` permite especificar el número de conexiones (flujos de red) que se deben establecer entre el cliente NFS y el punto de conexión NFS, hasta un límite de 16. Tradicionalmente, los clientes NFS usan una única conexión entre ellos y el punto de conexión.  Si aumenta el número de flujos de red, los límites superiores de E/S y el rendimiento aumentan significativamente. Las pruebas han indicado que `nconnect=8` es el que tiene un mayor rendimiento.  

Al preparar un entorno de SAS GRID de varios nodos para producción, es posible que observe una reducción repetible del 30 % en el tiempo de ejecución que va de 8 a 5,5 horas: 

| Opción de montaje | Tiempos de ejecución del trabajo |
|-|-|
| No `nconnect` | 8 horas |
| `nconnect=8`  | 5,5 horas | 

Ambos conjuntos de pruebas usaban la misma máquina virtual E32-8_v4 y RHEL8.3, con la lectura anticipada establecida en 15 MiB.

Cuando use `nconnect`, tenga en cuenta las siguientes reglas:

* `nconnect` es compatible con Azure NetApp Files en todas las distribuciones principales de Linux, pero solo en las versiones más recientes:

    | Versión de Linux | NFSv3 (versión mínima) | NFSv4.1 (versión mínima) |
    |-|-|-|
    | Red Hat Enterprise Linux | RHEL8.3 | RHEL8.3 |
    | SUSE | SLES12SP4 o SLES15SP1 | SLES15SP2 |
    | Ubuntu | Ubuntu18.04 |          |

    > [!NOTE]
    > SLES15SP2 es la versión mínima de SUSE en la que Azure NetApp Files para NFSv4.1 admite `nconnect`.  Las restantes versiones especificadas son las primeras que introdujeron la característica `nconnect`.

* Todos los montajes de un único punto de conexión heredarán el valor `nconnect` de la primera exportación montada, como se muestra en los siguientes escenarios: 

    Escenario 1: el primer montaje usa `nconnect`. Por consiguiente, todos los montajes del mismo punto de conexión usan `nconnect=8`.

    * `mount 10.10.10.10:/volume1 /mnt/volume1 -o nconnect=8`
    * `mount 10.10.10.10:/volume2 /mnt/volume2`
    * `mount 10.10.10.10:/volume3 /mnt/volume3`

    Escenario 2: el primer montaje no usa `nconnect`. Por tanto, ninguno de los montajes del mismo punto de conexión usa `nconnect`, aunque se pueda especificar `nconnect` en él.

    * `mount 10.10.10.10:/volume1 /mnt/volume1`
    * `mount 10.10.10.10:/volume2 /mnt/volume2 -o nconnect=8`
    * `mount 10.10.10.10:/volume3 /mnt/volume3 -o nconnect=8`

    Escenario 3: la configuración de `nconnect` no se propaga entre puntos de conexión de almacenamiento independientes.  `nconnect` lo usa el montaje procedente de `10.10.10.10`, pero no el montaje procedente de `10.12.12.12`.

    * `mount 10.10.10.10:/volume1 /mnt/volume1 -o nconnect=8`
    * `mount 10.12.12.12:/volume2 /mnt/volume2`

* `nconnect` se puede usar para aumentar la simultaneidad de almacenamiento desde cualquier cliente dado. 

Para más información, consulte el artículo sobre los [procedimientos recomendados de simultaneidad de Linux para Azure NetApp Files](performance-linux-concurrency-session-slots.md).

## <a name="rsize-and-wsize"></a>`Rsize` y `Wsize`
 
Las marcas `rsize` y `wsize` establecen el tamaño máximo de transferencia de las operaciones de NFS.  Si `rsize` o `wsize` no se especifican en el montaje, el cliente y el servidor negocian el mayor tamaño que admiten ambos.   Actualmente, tanto Azure NetApp Files como las distribuciones modernas de Linux admiten tamaños de lectura y escritura de hasta 1 048 576 bytes (1 MiB).   Sin embargo, para obtener los mejores rendimiento general y latencia, Azure NetApp Files recomienda establecer tanto `rsize` como `wsize` con un valor inferior a 262 144 bytes (256 K). Si usa unos valores mayores de 256 KiB en `rsize` y `wsize`, es posible que observe una mayor latencia y un menor rendimiento. 

Por ejemplo, en la [implementación de un sistema de escalabilidad horizontal de SAP HANA con nodos en espera en máquinas virtuales de Azure mediante Azure NetApp Files en SUSE Linux Enterprise Server](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse.md#mount-the-azure-netapp-files-volumes), se muestra el máximo de 256 KiB de `rsize` y `wsize` como se indica a continuación:

```
sudo vi /etc/fstab
# Add the following entries
10.23.1.5:/HN1-data-mnt00001 /hana/data/HN1/mnt00001  nfs rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
10.23.1.6:/HN1-data-mnt00002 /hana/data/HN1/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
10.23.1.4:/HN1-log-mnt00001 /hana/log/HN1/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
10.23.1.6:/HN1-log-mnt00002 /hana/log/HN1/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
10.23.1.4:/HN1-shared/shared /hana/shared  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
```
 
En otro ejemplo, SAS Viya recomienda un tamaño de lectura y escritura de 256 KiB, y [SAS GRID](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/606973/highlight/true#M17740) limita el valor de `r/wsize` a 64 KiB, a la vez que aumenta el rendimiento de lectura con mayor lectura por adelantado para los montajes de NFS.  <!-- For more information on readahead, see the article “NFS Readahead”. --> 

Las siguientes consideraciones son válidas para el uso de `rsize` y `wsize`:

* Los tamaños de las operaciones de E/S aleatorias suelen ser menores que las opciones de montaje de `rsize` y `wsize`. Por lo tanto, en efecto, no se restringirán.
* Al usar la caché del sistema de archivos, se producirá una E/S secuencial en el tamaño predicado por las opciones de montaje `rsize` y `wsize`, salvo que el tamaño del archivo sea menor que `rsize` y `wsize`.
* Las operaciones que omiten la caché del sistema de archivos, aunque estén restringidas por las opciones de montaje `rsize` y `wsize`, no emitirán necesariamente un tamaño tan grande como el máximo especificado por `rsize`o `wsize`.  Esta consideración es importante cuando se usan generadores de cargas de trabajo que tienen la opción `directio`.

*Como procedimiento recomendado con Azure NetApp Files, para obtener los mejores rendimiento general y latencia es preciso que el valor de `rsize` y `wsize` sea inferior a de 262 144 bytes.*

## <a name="close-to-open-consistency-and-cache-attribute-timers"></a>Coherencia casi abierta y temporizadores de atributos de caché   

NFS usa un modelo de coherencia flexible. La coherencia es flexible porque la aplicación no tiene que ir al almacenamiento compartido y capturar datos cada vez para usarlos, lo que afectaría considerablemente al rendimiento de la aplicación.  Hay dos mecanismos que administran este proceso: los temporizadores de atributos de la caché y la coherencia casi abierta.

*Si el cliente tiene la propiedad completa de los datos, es decir, no se comparte entre varios nodos o sistemas, la coherencia está garantizada.* En ese caso, puede reducir las operaciones de acceso de `getattr` para almacenar y agilizar la aplicación mediante la desactivación de la coherencia casi abierta (`cto`) (`nocto` como opción de montaje) y la activación de los tiempos de expiración para la administración de la caché de atributos (`actimeo=600` como opción de montaje cambia el temporizador a 10 m, frente a los valores predeterminados `acregmin=3,acregmax=30,acdirmin=30,acdirmax=60`). En algunas pruebas, `nocto` reduce aproximadamente entre el 65 y el 70 % de las llamadas de acceso de `getattr`, mientras que el ajuste de `actimeo` reduce estas llamadas otro 20 a 25 %.

### <a name="how-attribute-cache-timers-work"></a>Funcionamiento de los temporizadores de la caché de atributos  

Los atributos `acregmin`, `acregmax`, `acdirmin` y `acdirmax` controlan la coherencia de la caché. Los dos primeros atributos controlan durante cuánto tiempo son de confianza los atributos de los archivos. Los dos últimos atributos controlan durante cuánto tiempo son de confianza los atributos del propio archivo de directorio (tamaño del directorio, propiedad del directorio y permisos de directorio).  Los atributos `min` y `max` definen la duración mínima y máxima en la que los atributos de un directorio, los atributos de un archivo y el contenido de caché de un archivo se consideran de confianza, respectivamente. Entre `min` y `max`, se usa un algoritmo para definir la cantidad de tiempo durante el que se confía en una entrada almacenada en la caché.

Por ejemplo, considere los valores `acregmin` y `acregmax`predeterminados, 3 y 30 segundos, respectivamente.  Por ejemplo, los atributos se evalúan repetidamente para los archivos de un directorio.  3 segundos después, se realiza una consulta del servicio NFS para realizar la actualización.  Si los atributos se consideran válidos, el cliente duplica el tiempo de confianza a 6 segundos, 12 segundos, 24 segundos y el máximo se establece en 30, 30 segundos.  A partir de ese momento, hasta que los atributos almacenados en caché se consideren obsoletos (momento en el que se inicia de nuevo el ciclo), la confiabilidad se define como 30 segundos, que es el valor especificado por `acregmax`.

Hay otros casos que pueden beneficiarse de un conjunto similar de opciones de montaje, incluso cuando los clientes no tienen una propiedad completa, por ejemplo, si los clientes usan los datos como de solo lectura y la actualización de datos se administra a través de otra ruta de acceso.  En el caso de las aplicaciones que usan cuadrículas de clientes como EDA, el hospedaje web y la representación de películas, y tienen conjuntos de datos relativamente estáticos (herramientas o bibliotecas EDA, contenido web, datos de texturas), el comportamiento típico es que el conjunto de datos se almacene principalmente en la caché de los clientes. Hay muy pocas lecturas y ninguna escritura. Habrá muchas `getattr` o llamadas de acceso que volverán al almacenamiento.  Estos conjuntos de datos normalmente se actualizan a través de otro cliente que monta los sistemas de archivos y envía periódicamente actualizaciones de contenido.

En estos casos, se produce un retraso conocido al seleccionar contenido nuevo y la aplicación sigue funcionando con datos potencialmente obsoletos.  En estos casos, se pueden usar `nocto` y `actimeo` para controlar el período en el que se puede administrar la fecha en que empiezan a estar obsoletos.  Por ejemplo, en las bibliotecas y herramientas de EDA, `actimeo=600` funciona bien porque estos datos normalmente se actualizan con poca frecuencia.  En el caso de un hospedaje web pequeño en el que los clientes necesitan ver las actualizaciones de manera puntual mientras editan sus sitios, `actimeo=10` puede ser aceptable. En el caso de los sitios web a gran escala, en los que el contenido se inserta en varios sistemas de archivos, `actimeo=60` puede ser aceptable.

El uso de estas opciones de montaje reduce significativamente la carga de trabajo del almacenamiento en estos casos (por ejemplo, una experiencia reciente de EDA redujo las E/S por segundo del volumen de la herramientas de 150 000 a aproximadamente 6000). Las aplicaciones se pueden ejecutar considerablemente más rápido porque pueden confiar en los datos que hay en la memoria (el tiempo de acceso a la memoria se mide en nanosegundos, frente a cientos de microsegundos para `getattr`/acceso en una red rápida).

### <a name="close-to-open-consistency"></a>Coherencia casi abierta 

La coherencia casi abierta (la opción de montaje `cto`) garantiza que, independientemente del estado de la caché, al abrir los datos más recientes de un archivo siempre se presentan a la aplicación.  

* Cuando se rastrea un directorio (por ejemplo `ls` o `ls -l`), se emite un determinado conjunto de llamadas PRC.  
    El servidor NFS comparte su vista del sistema de archivos. Mientras todos los clientes NFS que accedan a una exportación de NFS determinada usen `cto`, todos los clientes verán la misma lista de archivos y directorios.  La actualización de los atributos de los archivos del directorio se controla mediante los [temporizadores de la caché de atributos](#how-attribute-cache-timers-work).  En otras palabras, mientras se utilice `cto`, los archivos aparecen en los clientes remotos en cuanto se crea el archivo y el archivo llega al almacenamiento.
* Cuando se abre un archivo, se garantiza la actualización de su contenido desde la perspectiva del servidor NFS.  
    Si hay una condición de carrera en la que el contenido no ha terminado de vaciarse de Máquina 1 cuando se abre un archivo en Máquina 2, Máquina 2 solo recibirá los datos presentes en el servidor en el momento de la apertura. En este caso, la máquina 2 no recuperará más datos del archivo hasta que se alcance el temporizador `acreg` y la máquina 2 vuelve a comprobar la coherencia de su caché desde el servidor.  Para observar este escenario se puede usar una cola `-f` de la máquina 2 cuando el archivo todavía se escribe desde la máquina 1.

### <a name="no-close-to-open-consistency"></a>No hay coherencia casi abierta  

Si no se usa la coherencia casi abierta (`nocto`), el cliente confiará en la actualización de su vista actual del archivo y del directorio hasta que se hayan infringido los temporizadores de atributos de la caché.  

* Cuando se rastrea un directorio (por ejemplo `ls` o `ls -l`), se emite un determinado conjunto de llamadas PRC.  
    El cliente solo emitirá una llamada al servidor para obtener una lista de archivos actual cuando se haya infringido el valor del temporizador de la caché `acdir`.  En este caso, los archivos y directorios creados recientemente no aparecerán, seguirán apareciendo los archivos y directorios eliminados recientemente.  

* Cuando se abre un archivo, si todavía esta en la caché, se devuelve su contenido almacenado en la caché (si existe) sin validar la coherencia con el servidor NFS.

## <a name="next-steps"></a>Pasos siguientes  

* [Procedimientos recomendados de simultaneidad de Linux para Azure NetApp Files](performance-linux-concurrency-session-slots.md)
* [Bancos de pruebas de rendimiento para Linux](performance-benchmarks-linux.md) 
