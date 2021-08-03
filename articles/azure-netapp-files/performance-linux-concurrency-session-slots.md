---
title: 'Procedimientos recomendados de simultaneidad de Linux para Azure NetApp Files: ranuras de sesión y entradas de tabla de ranuras | Microsoft Docs'
description: Describe los procedimientos recomendados sobre las ranuras de sesión y las entradas de tabla de ranuras para el protocolo NFS de Azure NetApp Files.
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
ms.date: 06/03/2021
ms.author: b-juche
ms.openlocfilehash: 3158d4fae313afcb1fef69ba7a2728df4d235175
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111525343"
---
# <a name="linux-concurrency-best-practices-for-azure-netapp-files---session-slots-and-slot-table-entries"></a>Procedimientos recomendados de simultaneidad de Linux para Azure NetApp Files: ranuras de sesión y entradas de tabla de ranuras

Este artículo le ayudará a comprender los procedimientos recomendados de simultaneidad sobre las ranuras de sesión y las entradas de tabla de ranuras para el protocolo NFS de Azure NetApp Files. 

## <a name="nfsv3"></a>NFSv3

El protocolo NFS v3 no tiene ningún mecanismo para negociar la simultaneidad entre el cliente y el servidor. El cliente y el servidor definen su límite sin consultar al otro.  Para obtener el mejor rendimiento, debe alinear el máximo de entradas de tabla de ranuras `sunrpc` del lado cliente con las que se admiten, sin retroceso en el servidor.  Cuando un cliente sobrecarga la capacidad de la pila de red del servidor que procesa una carga de trabajo, el servidor responde reduciendo el tamaño de la ventana para la conexión, que no es el escenario de rendimiento ideal.

De manera predeterminada, los kernels modernos de Linux definen el tamaño de entrada de tabla de ranuras `sunrpc` por conexión `sunrpc.max_tcp_slot_table_entries` como compatible con 65 536 operaciones pendientes, como se muestra en la tabla siguiente. 

| Servidor de NFS v3 de Azure NetApp Files <br> Contextos de ejecución máximos por conexión | Cliente Linux <br> Máximo predeterminado de entradas de tabla de ranuras `sunrpc` por conexión |
|-|-|
| 128 | 65 536 |

Estas entradas de tabla de ranuras definen los límites de simultaneidad. Estos valores tan altos son innecesarios.  Por ejemplo, si usa la teoría de colas conocida como *Littles Law*, verá que la tasa de E/S viene determinada por la simultaneidad (es decir, la E/S pendiente) y la latencia. Por lo tanto, el algoritmo demuestra que 65 536 ranuras son órdenes de magnitud superior a lo necesario para impulsar incluso cargas de trabajo extremadamente exigentes.

`Littles Law: (concurrency = operation rate × latency in seconds)`

Un nivel de simultaneidad de 155 es suficiente para lograr 155 000 operaciones NFS de Oracle DB por segundo mediante Oracle Direct NFS, que es una tecnología similar a nivel de concepto a la opción de montaje `nconnect`:

* Teniendo en cuenta una latencia de 0,5 ms, se necesita una simultaneidad de 55 para lograr 110 000 IOPS.
* Teniendo en cuenta una latencia de 1 ms, se necesita una simultaneidad de 155 para lograr 155 000 IOPS.

![Curva de latencia de Oracle DNFS](../media/azure-netapp-files/performance-oracle-dnfs-latency-curve.png)  

Si quiere obtener más información, consulte el documento [Rendimiento de la base de datos de Oracle en volúmenes individuales de Azure NetApp Files](performance-oracle-single-volumes.md).

El valor ajustable `sunrpc.max_tcp_slot_table_entries` es un parámetro de ajuste del nivel de conexión.  *Como procedimiento recomendado, establezca este valor en 128 o menos por conexión, sin superar las 3000 ranuras en todo el entorno.*

### <a name="examples-of-slot-count-based-on-concurrency-recommendation"></a>Ejemplos de recuento de ranuras en función de la recomendación de simultaneidad 

En los ejemplos de esta sección se muestra el recuento de ranuras en función de la recomendación de simultaneidad.  

#### <a name="example-1--one-nfs-client-65536-sunrpcmax_tcp_slot_table_entries-and-no-nconnect-for-a-maximum-concurrency-of-128-based-on-the-server-side-limit-of-128"></a>Ejemplo 1: un cliente NFS, 65 536 `sunrpc.max_tcp_slot_table_entries` y ningún `nconnect` para una simultaneidad máxima de 128 según el límite del lado del servidor de 128

El ejemplo 1 se basa en la carga de trabajo de un único cliente con el valor predeterminado `sunrpc.max_tcp_slot_table_entry` de 65 536 y una única conexión de red, es decir, sin `nconnect`.  En este caso, se puede lograr una simultaneidad de 128.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection (10.10.10.10:2049, 10.10.10.11:6543,TCP`) 
        * En teoría, el cliente no puede emitir más de 65 536 solicitudes activas al servidor por conexión.
        * El servidor no aceptará más de 128 solicitudes activas de esta conexión.

#### <a name="example-2--one-nfs-client-128-sunrpcmax_tcp_slot_table_entries-and-no-nconnect-for-a-maximum-concurrency-of-128"></a>Ejemplo 2: un cliente NFS, 128 `sunrpc.max_tcp_slot_table_entries` y ningún `nconnect` para una simultaneidad máxima de 128

El ejemplo 2 se basa en la carga de trabajo de un único cliente con un valor `sunrpc.max_tcp_slot_table_entry` de 128, pero sin la opción de montaje `nconnect`.  Con esta configuración, se puede lograr una simultaneidad de 128 de una única conexión de red.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection (10.10.10.10:2049, 10.10.10.11:6543,TCP) `
        * El cliente no podrá emitir más de 128 solicitudes activas al servidor por conexión.
        * El servidor no aceptará más de 128 solicitudes activas de esta conexión.

#### <a name="example-3--one-nfs-client-100-sunrpcmax_tcp_slot_table_entries-and-nconnect8-for-a-maximum-concurrency-of-800"></a>Ejemplo 3: un cliente NFS, 100 `sunrpc.max_tcp_slot_table_entries` y `nconnect=8` para una simultaneidad máxima de 800

El ejemplo 3 se basa en la carga de trabajo de un único cliente, pero con un valor `sunrpc.max_tcp_slot_table_entry` más bajo, de 100.  Esta vez, la opción de montaje `nconnect=8` usó la propagación de la carga de trabajo en 8 conexiones.  Con esta configuración, se puede lograr una simultaneidad de 800 entre las 8 conexiones.  Esta cantidad es la simultaneidad necesaria para lograr 400 000 IOPS.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection 1 (10.10.10.10:2049, 10.10.10.11:6543,TCP), Connection 2 (10.10.10.10:2049, 10.10.10.11:6454,TCP)… Connection 8 (10.10.10.10:2049, 10.10.10.11:7321,TCP)`
    * Conexión 1
        * El cliente no podrá emitir más de 100 solicitudes activas al servidor de esta conexión.
        * Se espera que el servidor no acepte más de 128 solicitudes activas del cliente para esta conexión.
    * Conexión 2
        * El cliente no podrá emitir más de 100 solicitudes activas al servidor de esta conexión.
        * Se espera que el servidor no acepte más de 128 solicitudes activas del cliente para esta conexión.
    * `…`
    * `…`
    * Conexión 8
        * El cliente no podrá emitir más de 100 solicitudes activas al servidor de esta conexión.
        * Se espera que el servidor no acepte más de 128 solicitudes activas del cliente para esta conexión.

#### <a name="example-4--250-nfs-clients-8-sunrpcmax_tcp_slot_table_entries-and-no-nconnect-for-a-maximum-concurrency-of-2000"></a>Ejemplo 4: 250 clientes NFS, 8 `sunrpc.max_tcp_slot_table_entries` y ningún `nconnect` para una simultaneidad máxima de 2000

En el ejemplo 4 se usa el valor reducido `sunrpc.max_tcp_slot_table_entry` por cliente de 8 para un entorno EDA de 250 equipos. En este escenario, se alcanza una simultaneidad de 2000 en todo el entorno, un valor más que suficiente para impulsar 4000 MiB/s de una carga de trabajo EDA de back-end.

* `NFS_Server=10.10.10.10, NFS_Client1=10.10.10.11`
    * `Connection (10.10.10.10:2049, 10.10.10.11:6543,TCP)` 
        * El cliente no podrá emitir más de 8 solicitudes activas al servidor por conexión.
        * El servidor no aceptará más de 128 solicitudes activas de esta conexión.
* `NFS_Server=10.10.10.10, NFS_Client2=10.10.10.12`
    * `Connection (10.10.10.10:2049, 10.10.10.12:7820,TCP) `
        * El cliente no podrá emitir más de 8 solicitudes activas al servidor por conexión.
        * El servidor no aceptará más de 128 solicitudes activas de esta conexión.
* `…`
* `…`
* `NFS_Server=10.10.10.10, NFS_Client250=10.10.11.13`
    * `Connection (10.10.10.10:2049, 10.10.11.13:4320,TCP) `
        * El cliente no podrá emitir más de 8 solicitudes activas al servidor por conexión.
        * El servidor no aceptará más de 128 solicitudes activas de esta conexión.

Al usar NFS v3, *debe mantener colectivamente el número de ranuras del punto de conexión de almacenamiento en 2000 como máximo*. Es mejor establecer el valor por conexión de `sunrpc.max_tcp_slot_table_entries` en menos de 128 cuando una aplicación escala horizontalmente en muchas conexiones de red (`nconnect` y HPC en general y EDA en particular).  

### <a name="how-to-calculate-the-best-sunrpcmax_tcp_slot_table_entries"></a>Cómo calcular el mejor `sunrpc.max_tcp_slot_table_entries` 

Mediante *Littles Law*, puede calcular el recuento total de entradas de tabla de ranuras necesarias. En general, tenga en cuenta los siguientes factores:  

* Las cargas de trabajo de escalabilidad horizontal suelen ser mayoritariamente secuenciales de gran tamaño por naturaleza.
* Las cargas de trabajo de base de datos, especialmente OLTP, suelen ser aleatorias por naturaleza. 

En la tabla siguiente aparece un estudio de muestra de simultaneidad con latencias arbitrarias:

|     Tamaño de E/S    |     Latencia    |     E/S o rendimiento    |     Simultaneidad    |
|-|-|-|-|
|     8 KiB    |     0,5 ms    |     110 000 IOPS \| 859 MiB/s    |     55    |
|     8 KiB    |     2 ms    |     400 000 IOPS \| 3125 MiB/s    |     800    |
|     256 KiB    |     2 ms    |     16 000 IOPS \| 4000 MiB/s    |     32    |
|     256 KiB    |     4 ms    |     32 000 IOPS \| 8000 MiB/s    |     128    |

### <a name="how-to-calculate-concurrency-settings-by-connection-count"></a>Cómo calcular la configuración de simultaneidad mediante el recuento de conexiones

Por ejemplo, la carga de trabajo es una granja de EDA y 200 clientes dirigen carga de trabajo al mismo punto de conexión de almacenamiento (un punto de conexión de almacenamiento es una dirección IP de almacenamiento). A continuación, se calcula la tasa de E/S necesaria y se divide la simultaneidad entre la granja.

Suponga que la carga de trabajo es de 4000 MiB/s con un tamaño medio de operación de 256 KiB y una latencia media de 10 ms. Para calcular la simultaneidad, use la siguiente fórmula:

`(concurrency = operation rate × latency in seconds)`

El cálculo se traduce en una simultaneidad de 160: 
 
`(160 = 16,000 × 0.010)`

Dada la necesidad de 200 clientes, podría establecer de forma segura `sunrpc.max_tcp_slot_table_entries` en 2 por cliente para alcanzar los 4000 MiB/s.  Sin embargo, puede decidir compilar capacidad de aumento adicional si establece el número por cliente en 4 o incluso 8 y mantenerlo por debajo del límite máximo de 2000 ranuras recomendado. 

### <a name="how-to-set-sunrpcmax_tcp_slot_table_entries-on-the-client"></a>Cómo configurar `sunrpc.max_tcp_slot_table_entries` en el cliente

1.  Agregue `sunrpc.max_tcp_slot_table_entries=<n>` al archivo de configuración `/etc/sysctl.conf`.   
    Durante el ajuste, si se encuentra un valor óptimo inferior a 128, reemplace 128 por el número adecuado.
2.  Ejecute el siguiente comando:   
    `$ sysctl -p`
3.  Monte (o vuelva a montar) todos los sistemas de archivos de NFS, ya que el valor ajustable solo se aplica a los montajes realizados después de establecer el valor ajustable.

## <a name="nfsv41"></a>NFSv4.1 

En NFS v4.1, las sesiones definen la relación entre el cliente y el servidor. Tanto si los sistemas de archivos de NFS montados se encuentran en una o en varias conexiones (como es el caso de `nconnect`), se aplican las reglas para la sesión. En la configuración de la sesión, el cliente y el servidor negocian el máximo de solicitudes para la sesión, poniéndose de acuerdo en el más bajo de los dos valores admitidos. Azure NetApp Files admite 180 solicitudes pendientes y los clientes de Linux tienen como valor predeterminado 64. En la tabla siguiente se muestran los límites de la sesión: 

| Servidor de NFS v4.1 de Azure NetApp Files <br>  Máximo de comandos por sesión | Cliente de Linux <br>  Máximo de comandos por defecto por sesión    | Número máximo de comandos negociados para la sesión |
|-|-|-|
|     180    |     64    |     64    |

Aunque los clientes de Linux tienen como valor predeterminado un máximo de 64 solicitudes por sesión, el valor de `max_session_slots` es ajustable.  Debe reiniciarse el equipo para que los cambios tengan efecto.  Use el comando `systool -v -m nfs` para ver el máximo actual que usa el cliente.  Para que el comando funcione, debe haber al menos un montaje de NFS v4.1:

```
$ systool -v -m nfs
{
Module = "nfs"
…
  Parameters:
…
    max_session_slots   = "64"   
…
}
```

Para ajustar `max_session_slots`, cree un archivo de configuración en `/etc/modprobe.d` como tal.  Asegúrese de que no haya "comillas" en la línea del archivo. De lo contrario, la opción no tendrá efecto.

`$ echo “options nfs max_session_slots=180” > /etc/modprobe.d/nfsclient.conf`
`$ reboot`

Azure NetApp Files limita cada sesión a 180 comandos como máximo. Por lo tanto, considere 180 como el valor máximo que se puede configurar actualmente.  El cliente no podrá lograr una simultaneidad mayor que 128 a menos que la sesión se divida en más de una conexión, ya que Azure NetApp Files restringe cada conexión a 128 comandos NFS como máximo.  Para obtener más de una conexión, se recomienda la opción de montaje `nconnect` y se requiere al menos un valor de dos.

### <a name="examples-of-expected-concurrency-maximums"></a>Ejemplos de máximos de simultaneidad esperados

En los ejemplos de esta sección se muestran los máximos de simultaneidad esperados.  

#### <a name="example-1--64-max_session_slots-and-no-nconnect"></a>Ejemplo 1: 64 `max_session_slots` y ningún `nconnect`

El ejemplo 1 se basa en la configuración predeterminada de 64 `max_session_slots` y ningún `nconnect`. Con esta configuración, se puede lograr una simultaneidad de 64 de una única conexión de red.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection (10.10.10.10:2049, 10.10.10.11:6543,TCP)` 
        * El cliente no podrá emitir más de 64 solicitudes activas al servidor para la sesión.
        * El servidor no podrá aceptar más de 64 solicitudes activas del cliente para la sesión. (64 es el valor negociado).

#### <a name="example-2--64-max_session_slots-and-nconnect2"></a>Ejemplo 2: 64 `max_session_slots` y `nconnect=2` 

El ejemplo 2 se basa en 64 `session_slots` como máximo, pero con la opción de montaje agregada de `nconnect=2`.  Una simultaneidad de 64 es factible si se divide en dos conexiones.  Aunque varias conexiones no aportan mayor simultaneidad en este escenario, la disminución de la profundidad de la cola por conexión tiene un impacto positivo en la latencia.

Con el `max_session_slots` todavía en 64 pero con `nconnect=2`, observe que el número máximo de solicitudes se divide entre las conexiones.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection 1 (10.10.10.10:2049, 10.10.10.11:6543,TCP) &&  Connection 2 (10.10.10.10:2049, 10.10.10.11:6454,TCP)`
    * Conexión 1
        * El cliente no podrá emitir más de 32 solicitudes activas al servidor de esta conexión.
        * Se espera que el servidor no acepte más de 32 solicitudes activas del cliente para esta conexión.
    * Conexión 2
        * El cliente no podrá emitir más de 32 solicitudes activas al servidor de esta conexión.
        * Se espera que el servidor no acepte más de 32 solicitudes activas del cliente para esta conexión.

#### <a name="example-3--180-max_session_slots-and-no-nconnect"></a>Ejemplo 3: 180 `max_session_slots` y ningún `nconnect`

En el ejemplo 3 se anula la opción de montaje `nconnect` y se establece el valor `max_session_slots` en 180, que coincide con la simultaneidad máxima de sesión de NFS v4.1 del servidor.  En este escenario, con una sola conexión y dado el máximo de 128 operaciones pendientes de Azure NetApp Files por conexión NFS, la sesión se limita a 128 operaciones activas.

Aunque `max_session_slots` se ha establecido en 180, la conexión de red única está limitada a 128 solicitudes como máximo: 

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection (10.10.10.10:2049, 10.10.10.11:6543,TCP) `
        * El cliente no podrá emitir más de 180 solicitudes activas al servidor para la sesión.
        * El servidor no podrá aceptar más de 180 solicitudes activas del cliente para la sesión.
        * *El servidor no aceptará más de 128 solicitudes activas para la conexión única.*

#### <a name="example-4--180-max_session_slots-and-nconnect2-"></a>Ejemplo 4: 180 `max_session_slots` y `nconnect=2 ` 

En el ejemplo 4 se agrega la opción de montaje `nconnect=2` y se reutiliza el valor `max_session_slots` a 180. Dado que la carga de trabajo general se divide en dos conexiones, se pueden realizar 180 operaciones pendientes.

Con dos conexiones en juego, la sesión admite la asignación completa de 180 solicitudes pendientes.

* `NFS_Server=10.10.10.10, NFS_Client=10.10.10.11`
    * `Connection 1 (10.10.10.10:2049, 10.10.10.11:6543,TCP) &&  Connection 2 (10.10.10.10:2049, 10.10.10.11:6454,TCP)`
    * Conexión 1
        * Se espera que el cliente no mantenga más de 90 solicitudes activas al servidor desde la conexión uno.
        * *Se espera que el servidor no mantenga más de 90 solicitudes activas del cliente para esta conexión durante la sesión.*
    * Conexión 2
        * Se espera que el cliente no mantenga más de 90 solicitudes activas al servidor desde la conexión uno.
        * *Se espera que el servidor no mantenga más de 90 solicitudes activas del cliente para esta conexión durante la sesión.*

> [!NOTE]
> Para una simultaneidad máxima, establezca `max_session_slots` en 180, que es la simultaneidad de nivel de sesión máxima admitida por Azure NetApp Files actualmente.

### <a name="how-to-check-for-the-maximum-requests-outstanding-for-the-session"></a>Cómo comprobar el número máximo de solicitudes pendientes para la sesión 

Para ver los tamaños de `session_slot` admitidos por el cliente y el servidor, capture el comando de montaje en un seguimiento de paquetes.  Busque la llamada `CREATE_SESSION` y la respuesta `CREATE_SESSION` como se muestra en el ejemplo siguiente.  La llamada se originó en el cliente y la respuesta se originó en el servidor.

Use el comando `tcpdump` siguiente para capturar el comando de montaje:

`$ tcpdump -i eth0 -s 900 -w /tmp/write.trc port 2049`

Con Wireshark, los paquetes de interés son los siguientes:

![Instantánea que muestra paquetes de interés.](../media/azure-netapp-files/performance-packets-interest.png)  

Dentro de estos dos paquetes, observe el campo `max_reqs` en la sección central del archivo de seguimiento.

* Network File System
    * Operations
        * `Opcode`
            * `csa_fore_channel_attrs`
            * `max reqs`

El paquete 12 (máximo de solicitudes de cliente) muestra que el cliente tenía un valor `max_session_slots` de 64.  En la sección siguiente, observe que el servidor admite una simultaneidad de 180 para la sesión.  La sesión acaba negociando el menor de los dos valores proporcionados.

![Instantánea que muestra el número máximo de ranuras de sesión para el paquete 12.](../media/azure-netapp-files/performance-max-session-packet-12.png)  

En el ejemplo siguiente se muestra el paquete 14 (máximo de solicitudes del servidor):

![Instantánea que muestra el número máximo de ranuras de sesión para el paquete 14.](../media/azure-netapp-files/performance-max-session-packet-14.png) 

## <a name="next-steps"></a>Pasos siguientes  

* [Procedimientos recomendados de las opciones de montaje de NFS de Linux para Azure NetApp Files](performance-linux-mount-options.md)
* [Bancos de pruebas de rendimiento para Linux](performance-benchmarks-linux.md) 
