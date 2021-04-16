---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 10/12/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 82b4c127f983f3133326bf7fb538e40713ef9655
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "100580377"
---
![Gráfico en el que se muestran las especificaciones de Dsv3.](media/vm-disk-performance/dsv3-documentation.jpg)

- El rendimiento de disco máximo *no almacenado en caché* es el límite máximo de almacenamiento predeterminado que la máquina virtual puede controlar.
- El límite del rendimiento de almacenamiento *almacenado en caché* máximo es un límite independiente cuando se habilita el almacenamiento en caché del host.

El almacenamiento en caché de host funciona con el almacenamiento más cercano a la máquina virtual que puede escribirse o leerse rápidamente. La cantidad de almacenamiento que está disponible para la máquina virtual para el almacenamiento en caché del host se encuentra en la documentación. Por ejemplo, puede ver que la Standard_D8s_v3 incluye 200 GiB de almacenamiento en caché.

Puede habilitar el almacenamiento en caché de host al crear la máquina virtual y conectar los discos. También puede activar y desactivar el almacenamiento en caché de host de los discos en una máquina virtual existente.

![Captura de pantalla en la que se muestra el almacenamiento en caché de host.](media/vm-disk-performance/host-caching.jpg)

Puede ajustar el almacenamiento en caché de host para que coincida con los requisitos de carga de trabajo de cada disco. Puede configurar el almacenamiento en caché de host para que sea:

- **Solo lectura**: para cargas de trabajo que solo realizan operaciones de lectura
- **Lectura/escritura**: para cargas de trabajo que realizan un equilibrio de operaciones de lectura y escritura

Si la carga de trabajo no sigue ninguno de estos patrones, no se recomienda usar el almacenamiento en caché de host.

Ahora se verán un par de ejemplos de diferentes opciones de configuración de caché de host y cómo afecta al flujo de datos y al rendimiento. En este primer ejemplo, se examina lo que sucede con las solicitudes de E/S cuando la opción de almacenamiento en caché de host está establecida en **Solo lectura**.

**Configuración:**

- Standard_D8s_v3
  - IOPS en caché: 16 000
  - IOPS no almacenadas en caché: 12.800
- Disco de datos P30
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Solo lectura**

Cuando se realiza una lectura y los datos deseados están disponibles en la caché, esta devuelve los datos solicitados. No es necesario leer desde el disco. Esta lectura se cuenta para los límites en caché de la máquina virtual.

![Diagrama en el que se muestra un acierto de lectura del almacenamiento en caché de host de lectura.](media/vm-disk-performance/host-caching-read-hit.jpg)

Cuando se realiza una lectura y los datos deseados *no* están disponibles en la caché, la solicitud de lectura se retransmite al disco. Después, el disco la muestra en la caché y en la máquina virtual. Esta lectura se cuenta para el límite sin almacenamiento en caché de la máquina virtual y el límite almacenado en caché de la misma.

![Diagrama en el que se muestra un fallo de lectura del almacenamiento en caché de host de lectura.](media/vm-disk-performance/host-caching-read-miss.jpg)

Cuando se realiza una escritura, esta tiene que hacerse en la memoria caché y en el disco antes de que se considere completa. Esta escritura se cuenta para el límite sin almacenamiento en caché de la máquina virtual y el límite almacenado en caché de la misma.

![Diagrama en el que se muestra una escritura de almacenamiento en caché de host de lectura.](media/vm-disk-performance/host-caching-write.jpg)

Ahora se verá lo que sucede con las solicitudes de E/S cuando la opción de almacenamiento en caché de host está establecida en **Lectura/escritura**.

**Configuración:**

- Standard_D8s_v3
  - IOPS en caché: 16 000
  - IOPS no almacenadas en caché: 12.800
- Disco de datos P30
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Lectura/escritura**

Una lectura se controla de la misma manera que una operación de solo lectura. Las escrituras son lo único que es diferente con el almacenamiento en caché de lectura y escritura. Al escribir con el almacenamiento en caché de host establecido en **Lectura/escritura**, la escritura solo se debe realizar en la caché de host para que se considere completada. Después, la escritura se escribe de forma diferida en el disco, como un proceso en segundo plano. Esto significa que una escritura se cuenta para la E/S almacenada en caché cuando se escribe en la caché. Cuando se escribe de forma diferida en el disco, se cuenta para la E/S sin almacenamiento en caché.

![Diagrama en el que se muestra una escritura de almacenamiento en caché de host de lectura/escritura.](media/vm-disk-performance/host-caching-read-write.jpg)

Ahora continuará con la máquina virtual Standard_D8s_v3. Menos en esta ocasión, se habilitará el almacenamiento en caché de host en los discos. Además, ahora el límite de IOPS de la máquina virtual es 16 000. Hay tres discos P30 subyacentes conectados a la máquina virtual y cada uno puede administrar 5000 IOPS.

**Configuración:**

- Standard_D8s_v3
  - IOPS en caché: 16 000
  - IOPS no almacenadas en caché: 12.800
- Disco de SO P30
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Lectura/escritura**
- Dos discos de datos P30 × 2
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Lectura/escritura**

![Diagrama en el que se muestra un ejemplo de almacenamiento en caché de host.](media/vm-disk-performance/host-caching-example-without-remote.jpg)

La aplicación usa una máquina virtual Standard_D8s_v3 con el almacenamiento en caché habilitado. Realiza una solicitud de 15 000 IOPS. Las solicitudes se dividen en 5000 IOPS en cada disco subyacente conectado. No se produce ningún límite de rendimiento.

## <a name="combined-uncached-and-cached-limits"></a>Límites no almacenados en caché y almacenados en caché combinados

Los límites en caché de una máquina virtual son independientes de los límites no almacenados en caché. Esto significa que puede habilitar el almacenamiento en caché de host en los discos conectados a una máquina virtual mientras no lo habilita en otros discos. Esta configuración permite que las máquinas virtuales obtengan una E/S de almacenamiento total del límite en caché más el límite no almacenado en caché.

Ahora se verá un ejemplo para ayudarle a comprender cómo funcionan estos límites de forma conjunta. Se continuará con la configuración de una máquina virtual Standard_D8s_v3 y discos Premium conectados.

**Configuración:**

- Standard_D8s_v3
  - IOPS en caché: 16 000
  - IOPS no almacenadas en caché: 12.800
- Disco de SO P30
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Lectura/escritura**
- Dos discos de datos P30 × 2
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Lectura/escritura**
- Dos discos de datos P30 × 2
  - IOPS: 5.000
  - Almacenamiento en caché de host: **Disabled** (Deshabilitado)

![Diagrama en el que se muestra un ejemplo de almacenamiento en caché de host con almacenamiento remoto.](media/vm-disk-performance/host-caching-example-with-remote.jpg)

En este caso, la aplicación que se ejecuta en una máquina virtual Standard_D8s_v3 realiza una solicitud de 25 000 IOPS. La solicitud se divide en 5000 IOPS en cada uno de los discos conectados. En tres discos se usa el almacenamiento en caché de host y en dos discos no.

- Como los tres discos que usan el almacenamiento en caché de host están dentro de los límites en caché de 16 000, esas solicitudes se completan correctamente. No se produce ningún límite de rendimiento de almacenamiento.
- Como los dos discos que no usan el almacenamiento en caché de host están dentro de los límites no almacenados en caché de 12 800, esas solicitudes también se completan correctamente. No se produce ningún límite.

