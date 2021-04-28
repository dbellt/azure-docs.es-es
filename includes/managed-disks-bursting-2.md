---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 03/04/2021
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 4162fe12ff54f16cd5f982f6a576905227c9a107
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107821027"
---
## <a name="disk-level-bursting"></a>Expansión en el nivel de disco

### <a name="on-demand-bursting-preview"></a>Expansión a petición (versión preliminar)

Los discos que usan el modelo de expansión a petición pueden expandirse más allá de los objetivos aprovisionados originales, con tanta frecuencia como sea necesario para su carga de trabajo, hasta el máximo del objetivo de expansión. Por ejemplo, en un disco P30 de 1 TiB, la E/S por segundo aprovisionada es de 5000 IOPS. Cuando se habilita la expansión en este disco, las cargas de trabajo pueden emitir entradas y salidas en este disco hasta alcanzar el rendimiento máximo de expansión de 30 000 IOPS y 1000 MBps.

Si se cuenta con que las cargas de trabajo superarán con frecuencia el objetivo de rendimiento aprovisionado, la expansión de disco no resultará rentable. En este caso, se recomienda cambiar el rendimiento del disco a un [nivel superior](../articles/virtual-machines/disks-performance-tiers.md), para conseguir un mejor rendimiento de base. Revise los detalles de su facturación y evalúela teniendo en cuenta el patrón de tráfico de sus cargas de trabajo.

Antes de habilitar la expansión de disco a petición, debe comprender lo siguiente:

[!INCLUDE [managed-disk-bursting-regions-limitations](managed-disk-bursting-regions-limitations.md)]

#### <a name="regional-availability"></a>Disponibilidad regional

[!INCLUDE [managed-disk-bursting-availability](managed-disk-bursting-availability.md)]

#### <a name="billing"></a>Facturación

Cuando se usan discos con el modelo de expansión a petición, se cobra una tarifa fija por hora de habilitación de expansión y se aplican costos de transacción por las transacciones de expansión que superen el objetivo aprovisionado. Los costos por transacción se cobran mediante el modelo de pago por uso, en función del número de E/S en discos sin caché, incluyendo tanto las lecturas como las escrituras que superen los objetivos aprovisionados. A continuación se incluye un ejemplo de patrones de tráfico de disco en una hora de facturación:

Configuración de disco: SSD prémium de 1 TiB (P30) con la expansión de disco habilitada.

- De 00:00:00 a 00:10:00: la IOPS del disco está por debajo del objetivo aprovisionado de 5000 IOPS. 
- De 00:10:01 a 00:10:10: la aplicación envía un trabajo por lotes, lo que provoca que la IOPS de disco se expanda a 6000 IOPS durante 10 segundos. 
- De 00:10:11 a 00:59:00: la IOPS del disco está por debajo del objetivo aprovisionado de 5000 IOPS. 
- De 00:59:01 a 01:00:00: la aplicación envía otro trabajo por lotes, lo que provoca que la IOPS de disco se expanda a 7000 IOPS durante 60 segundos. 

En esta hora de facturación, el costo de expansión incluye dos cargos:

El primer cargo es la tarifa fija por habilitación de la expansión de X € (determinada según la región). Esta tarifa fija se cobra siempre por disco, sin tener en cuenta el estado de conexión, hasta que se deshabilita. 

El segundo es el costo por transacción de la expansión. La expansión del disco se produjo en dos intervalos de tiempo. De 00:10:01 a 00:10:10, la transacción acumulada de expansión es (6000 – 5000) x 10 = 10 000. De 00:59:01 a 01:00:00, la transacción acumulada de expansión es (7000 – 5000) x 60 = 120 000. El total de transacciones de expansión es de 10 000 + 120 000 = 130 000. El costo de las transacciones de expansión se cobrará a Y €, basándose en 13 unidades de 10 000 transacciones (en función de los precios regionales).

Así, el costo total de la expansión del disco en esta hora de facturación es de X + Y €. Se aplicaría el mismo cálculo en la expansión por encima del objetivo aprovisionado de MBps. Trasladamos uso de MB por encima del límite a las transacciones con un tamaño de E/S de 256 KB. Si el tráfico de disco supera los objetivos de IOPS y MBps aprovisionados, puede consultar el ejemplo siguiente para calcular las transacciones de expansión. 

Configuración de disco: SSD prémium de 1 TB (P30) con la expansión de disco habilitada.

- De 00:00:01 a 00:00:05: la aplicación envía un trabajo por lotes, lo que provoca que la IOPS de disco se expanda a 10000 IOPS y 300 MBps durante 5 segundos.
- De 00:00:06 a 00:00:10: la aplicación envía un trabajo de recuperación, lo que provoca que la IOPS de disco se expanda a 6000 IOPS y 600 MBps durante 5 segundos.

La transacción de expansión se cuenta como el número máximo de transacciones de expansión de IOPS o de MBps. De 00:00:01 a 00:00:05, la transacción de expansión acumulada máxima es ((10 000 – 5000), (300 - 200) * 1024 / 256)) * 5 = 25 000 transacciones. De 00:00:06 a 00:00:10, la transacción de expansión acumulada máxima es ((6000 – 5000), (600 - 200) * 1024 / 256)) * 5 = 8000 transacciones. Además, se incluye la tarifa fija de habilitación de expansión para obtener el costo total de habilitar la expansión de disco a petición. 

Puede consultar la [página de precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/) para obtener más información sobre los precios y usar la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=storage) para evaluar las cargas de trabajo. 


Para habilitar la expansión a petición, consulte [Habilitación de la expansión a petición](../articles/virtual-machines/disks-enable-bursting.md).

### <a name="credit-based-bursting"></a>Expansión basada en crédito

La expansión basada en crédito está disponible en los tamaños de disco P20 e inferiores en todas las regiones de la nube pública de Azure, la nube de Azure Government y la nube de China. La expansión del disco está habilitada de forma predeterminada en las implementaciones nuevas y existentes de los tamaños del disco que la admiten. La expansión en el nivel de máquina virtual solo usa la opción basada en crédito.

## <a name="virtual-machine-level-bursting"></a>Expansión en el nivel de máquina virtual

La expansión en el nivel de máquina virtual solo usa el modelo basado en crédito; está habilitada de forma predeterminada en todas las máquinas virtuales que la admiten.

Dicha característica está habilitada en todas las regiones de la nube pública de Azure en los tamaños admitidos siguientes: 
- [Serie Lsv2](../articles/virtual-machines/lsv2-series.md)
- [Series Dv3 y DSv3](../articles/virtual-machines/dv3-dsv3-series.md)
- [Series Ev3 y Esv3](../articles/virtual-machines/ev3-esv3-series.md)

## <a name="bursting-flow"></a>Flujo de expansión

El sistema de créditos de expansión se aplica de la misma manera tanto en el nivel de máquina virtual como en el de disco. El recurso, ya sea una VM o un disco, se iniciará con su reserva de créditos de expansión totalmente llena. Estos créditos le otorgarán 30 minutos de expansión a la velocidad máxima. Se acumulan créditos cada vez que se usan IOPS o MB/s del recurso por debajo de su objetivo de rendimiento. Si el recurso ha acumulado créditos de expansión y la carga de trabajo necesita un rendimiento adicional, el recurso puede usar esos créditos para superar los límites del rendimiento y aumentarlo hasta satisfacer la demanda de la carga de trabajo.

![Diagrama de la reserva de expansión](media/managed-disks-bursting/bucket-diagram.jpg)

Es usted quien decide el uso que se hace de los créditos disponibles. Puede usar los 30 minutos de créditos de expansión seguidos o de forma esporádica a lo largo del día. Cuando se implementan los recursos, incluyen una asignación completa de créditos. Cuando se agotan estos créditos, tardan menos de un día en reponerse. Puede gastar los créditos a su discreción; no es necesario que la reserva esté llena para expandir los recursos. La acumulación de la expansión varía en función de cada recurso, ya que se basa en IOPS y MB/s sin usar por debajo de sus objetivos de rendimiento. Los recursos con mayor rendimiento de base pueden acumular sus créditos de expansión más rápido que los recursos con un rendimiento de base más bajo. Por ejemplo, un disco P1 inactivo acumulará 120 IOPS por segundo, en tanto que un disco P20 inactivo acumulará 2300 IOPS por segundo.

## <a name="bursting-states"></a>Estados de expansión
Hay tres estados que puede tener el recurso con la expansión habilitada:
- **Acumulación** : el tráfico de E/S del recurso tiene un rendimiento inferior al de destino. La acumulación de créditos de expansión para IOPS y MB/s se realiza de forma independiente. El recurso puede acumular créditos de IOPS y gastar créditos de MB/s o viceversa.
- **Expansión**: el tráfico del recurso tiene un rendimiento superior al de destino. El tráfico de ráfaga consumirá créditos de IOPS o ancho de banda de forma independiente.
- **Constante**: el tráfico del recurso tiene un rendimiento igual al de destino.

## <a name="bursting-examples"></a>Ejemplos de expansión

En los siguientes ejemplos se muestra cómo funciona la expansión con varias combinaciones de máquinas virtuales y discos. Para facilitar el seguimiento de los ejemplos, nos centraremos en los MB/s, pero se aplica la misma lógica independientemente de las IOPS.

### <a name="non-burstable-virtual-machine-with-burstable-disks"></a>Máquina virtual no ampliable con discos ampliables
**Combinación de VM y discos:** 
- Standard_D8as_v4 
    - MB/s no almacenados en caché: 192
- Disco de SO P4
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 
- 2 discos de datos P10 
    - MB/s aprovisionados: 100
    - Máximo de MB/s de expansión: 170

 Cuando la VM se inicia, recupera los datos del disco del SO. Puesto que el disco de SO forma parte de una VM que se está iniciando, el disco del SO estará lleno de créditos de expansión. Estos créditos permitirán expandir el inicio del disco del SO a 170 MB/s segundos.

![La máquina virtual envía una solicitud de 192 MB/s de rendimiento al disco del sistema operativo, el disco del sistema operativo responde con datos de 170 MB/s.](media/managed-disks-bursting/nonbursting-vm-bursting-disk/nonbursting-vm-bursting-disk-startup.jpg)

Una vez completado el arranque, se ejecuta una aplicación en la VM y tiene una carga de trabajo no crítica. Esta carga de trabajo requiere 15 MB/s que se distribuyen uniformemente entre todos los discos.

![La aplicación envía una solicitud de 15 MB/s de rendimiento a la máquina virtual, la máquina virtual toma la solicitud y envía a cada uno de sus discos una solicitud de 5 MB/s, cada disco devuelve 5 MB/s, la máquina virtual devuelve 15 MB/s a la aplicación.](media/managed-disks-bursting/nonbursting-vm-bursting-disk/nonbursting-vm-bursting-disk-idling.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 192 MB/s. El disco del SO usa 2 MB/s y el resto se divide uniformemente entre los discos de datos.

![La aplicación envía una solicitud de 192 MB/s de rendimiento a la máquina virtual, la máquina virtual toma la solicitud y envía la mayor parte de su solicitud a los discos de datos (95 MB/s cada uno) y 2 MB/s al disco del sistema operativo, los discos de datos se expanden para satisfacer la demanda y todos los discos devuelven el rendimiento solicitado a la máquina virtual, que lo devuelve a la aplicación.](media/managed-disks-bursting/nonbursting-vm-bursting-disk/nonbursting-vm-bursting-disk-bursting.jpg)

### <a name="burstable-virtual-machine-with-non-burstable-disks"></a>Máquina virtual expansible con discos no expansibles
**Combinación de VM y discos:** 
- Standard_L8s_v2 
    - MB/s no almacenados en caché: 160
    - Máximo de MB/s de expansión: 1 280
- Disco de SO P50
    - MB/s aprovisionados: 250 
- 2 discos de datos P10 
    - MB/s aprovisionados: 250

 Una vez completado el arranque inicial, se ejecuta una aplicación en la VM y tiene una carga de trabajo no crítica. Esta carga de trabajo requiere 30 MB/s, que se distribuyen uniformemente entre todos los discos.
![La aplicación envía una solicitud de 30 MB/s de rendimiento a la máquina virtual, la máquina virtual toma la solicitud y envía a cada uno de sus discos una solicitud de 10 MB/s, cada disco devuelve 10 MB/s, la máquina virtual devuelve 30 MB/s a la aplicación.](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-normal.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 600 MB/s. Standard_L8s_v2 se expande para satisfacer esta demanda y, a continuación, las solicitudes a los discos se distribuyen uniformemente para los discos P50.

![La aplicación envía una solicitud de 600 MB/s de rendimiento a la máquina virtual, la máquina virtual se expande para tomar la solicitud y envía a cada uno de sus discos una solicitud de 200 MB/s, cada disco devuelve 200 MB/s, la máquina virtual se expande para devolver 600 MB/s a la aplicación.](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-bursting.jpg)
### <a name="burstable-virtual-machine-with-burstable-disks"></a>Máquina virtual ampliable con discos ampliables
**Combinación de VM y discos:** 
- Standard_L8s_v2 
    - MB/s no almacenados en caché: 160
    - Máximo de MB/s de expansión: 1 280
- Disco de SO P4
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 
- 2 discos de datos P4 
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 

Cuando se inicia la VM, se expande para solicitar su límite de expansión de 1280 MB/s del disco de SO y este disco responde con su rendimiento de expansión de 170 MB/s.

![En el inicio, la máquina virtual se expande para enviar una solicitud de 1280 MB/s al disco del sistema operativo, el disco del sistema operativo se expande para devolver los 1280 MB/s.](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-startup.jpg)

Después del inicio, inicie una aplicación que tenga una carga de trabajo no crítica. Esta aplicación requiere 15 MB/s que se distribuyen uniformemente entre todos los discos.

![La aplicación envía una solicitud de 15 MB/s de rendimiento a la máquina virtual, la máquina virtual toma la solicitud y envía a cada uno de sus discos una solicitud de 5 MB/s, cada disco devuelve respuestas de 5 MB/s, la máquina virtual devuelve 15 MB/s a la aplicación.](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-idling.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 360 MB/s. Standard_L8s_v2 se expande para satisfacer esta demanda y, a continuación, realiza una solicitud. El disco del SO solo necesita 20 MB/s. Los 340 MB/s restantes se administran mediante los discos de datos P4 de expansión.

![La aplicación envía una solicitud de 360 MB/s de rendimiento a la máquina virtual, la máquina virtual se expande para tomar la solicitud y envía a cada uno de sus discos de datos una solicitud de 170 MB/s y 20 MB/s desde el disco del sistema operativo, cada disco devuelve los MB/s solicitados, la máquina virtual se expande para devolver 360 MB/s a la aplicación.](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-bursting.jpg)
