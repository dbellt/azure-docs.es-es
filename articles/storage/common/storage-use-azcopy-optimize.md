---
title: Optimización del rendimiento de AzCopy v10 con Azure Storage | Microsoft Docs
description: Este artículo le ayuda a optimizar el rendimiento de AzCopy v10 con Azure Storage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: b60fc4b1fc20c455c2c409f544a8af16f1dbf8d1
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107509009"
---
# <a name="optimize-the-performance-of-azcopy-with-azure-storage"></a>Optimización del rendimiento de AzCopy con Azure Storage

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. Este artículo le ayuda a optimizar el rendimiento.

> [!NOTE]
> Si busca contenido que le ayude a empezar a trabajar con AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

Puede realizar un banco de pruebas de rendimiento y, después, usar comandos y variables de entorno para encontrar un equilibrio óptimo entre el rendimiento y el consumo de recursos.

## <a name="run-benchmark-tests"></a>Ejecución de pruebas del banco de pruebas

Puede ejecutar una prueba del banco de pruebas de rendimiento en contenedores de blobs o recursos compartidos de archivos específicos para ver las estadísticas generales e identificar los cuellos de botella de rendimiento. Puede ejecutar la prueba al cargar o descargar los datos de prueba generados. 

Utilice el siguiente comando para ejecutar un banco de pruebas de rendimiento.

**Sintaxis**

`azcopy benchmark 'https://<storage-account-name>.blob.core.windows.net/<container-name>'`

**Ejemplo**

```azcopy
azcopy benchmark 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D'
```

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

Este comando ejecuta un banco de pruebas de rendimiento mediante la carga los datos de prueba en un destino especificado. Los datos de prueba se generan en la memoria, se cargan en el destino y, a continuación, se eliminan del destino una vez completada la prueba. Puede especificar el número de archivos que se van a generar y el tamaño que desea que se utilicen mediante parámetros de comando opcionales.

Si prefiere descargar datos para ejecutar esta prueba, establezca el parámetro `mode` en `download`. Para ver documentos de referencia detallados, consulte [azcopy benchmark](storage-ref-azcopy-bench.md). 

## <a name="optimize-for-large-numbers-of-small-files"></a>Optimización para grandes cantidades de archivos pequeños

El rendimiento puede disminuir al transferir archivos pequeños, especialmente al transferir grandes cantidades de ellos. Para maximizar el rendimiento, reduzca el tamaño de cada trabajo. Para las operaciones de carga y descarga, aumente la simultaneidad, reduzca la actividad de registro y desactive las características que incurran en costos de alto rendimiento.

#### <a name="reduce-the-size-of-each-job"></a>Reducción del tamaño de cada trabajo

Para lograr un rendimiento óptimo, asegúrese de que cada trabajo transfiere menos de 10 millones de archivos. Los trabajos que transfieren más de 50 millones de archivos tienen un rendimiento menor, ya que el mecanismo de seguimiento de trabajos de AzCopy incurre en una gran sobrecarga. Para reducir la sobrecarga, considere la posibilidad de dividir los trabajos grandes en otros más pequeños. 

Una manera de hacerlo es limitar el número de archivos por trabajo. Para ello, puede usar parámetros de comando. Por ejemplo, un trabajo solo puede copiar un subconjunto de directorios mediante el parámetro `include path` como parte del comando [azcopy copy](storage-ref-azcopy-copy.md). 

Use el parámetro `include-pattern` para copiar archivos con una extensión específica (por ejemplo, `*.pdf`). En un trabajo independiente, use el parámetro `exclude-pattern` para copiar todos los archivos que no tengan la extensión `*.pdf`. Consulte [Carga de archivos específicos](storage-use-azcopy-blobs-upload.md#upload-specific-files) y [Descarga de blobs específicos](storage-use-azcopy-blobs-download.md#download-specific-blobs) para ejemplos.

Cuando haya decidido cómo dividir los trabajos grandes en otros más pequeños, considere la posibilidad de ejecutarlos en más de una máquina virtual.

#### <a name="increase-concurrency"></a>Aumento de la simultaneidad

Si va a cargar o descargar archivos, use la variable de entorno `AZCOPY_CONCURRENCY_VALUE` para aumentar el número de solicitudes simultáneas que pueden producirse en la máquina. Establezca esta variable lo más alta posible sin poner en peligro el rendimiento de la máquina. Para más información sobre esta variable, consulte la sección [Aumento del número de solicitudes simultáneas](#increase-the-number-of-concurrent-requests) de este artículo.

Si va a copiar blobs entre cuentas de almacenamiento, considere la posibilidad de establecer el valor de la variable de entorno `AZCOPY_CONCURRENCY_VALUE` en un valor mayor que `1000`. Puede establecer esta variable alta porque AzCopy usa las API de servidor a servidor, de manera que los datos se copian directamente entre servidores de almacenamiento y no usan la potencia de procesamiento de la máquina.  

#### <a name="decrease-the-number-of-logs-generated"></a>Reducción del número de registros generados

Puede mejorar el rendimiento reduciendo el número de entradas de registro que AzCopy crea a medida que completa operaciones. De forma predeterminada, AzCopy registra toda la actividad relacionada con una operación. Para lograr un rendimiento óptimo, considere la posibilidad de establecer el parámetro `log-level` del comando de copia, sincronización o eliminación en `ERROR`. De este modo, AzCopy solo registrará los errores. De forma predeterminada, el nivel de registro de valores se establece en `INFO`. 

#### <a name="turn-off-length-checking"></a>Desactivación de la comprobación de longitud 

Si va a cargar o descargar archivos, considere la posibilidad de establecer `--check-length` de los comandos de copia y sincronización en `false`. Esto impide que AzCopy compruebe la longitud de un archivo tras la transferencia. De forma predeterminada, AzCopy comprueba la longitud para asegurarse de que los archivos de origen y destino coinciden una vez completada la transferencia. AzCopy realiza esta comprobación después de cada transferencia de archivos. No obstante, esto puede degradar el rendimiento si los trabajos transfieren grandes cantidades de archivos pequeños. 

#### <a name="turn-on-concurrent-local-scanning-linux"></a>Activación del examen local simultáneo (Linux)

Los exámenes de archivos en algunos sistemas Linux no se ejecutan lo suficientemente rápido como para saturar todas las conexiones de red paralelas. En estos casos, puede establecer `AZCOPY_CONCURRENT_SCAN` en `true`. 

## <a name="increase-the-number-of-concurrent-requests"></a>Aumento del número de solicitudes simultáneas

Puede aumentar el rendimiento si establece la variable de entorno `AZCOPY_CONCURRENCY_VALUE`. Esta variable especifica el número de solicitudes simultáneas que pueden producirse.  

Si el equipo tiene menos de cinco CPU, el valor de esta variable se establece en `32`. En caso contrario, el valor predeterminado es igual a 16 multiplicado por el número de CPU. El valor máximo predeterminado de esta variable es `3000`, pero puede establecerlo manualmente en un valor superior o inferior. 

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set AZCOPY_CONCURRENCY_VALUE=<value>` |
| **Linux** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |
| **macOS** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, puede leer el valor que se está usando al examinar el principio de cualquier archivo de registro de AzCopy. El valor seleccionado, y el motivo por el que se seleccionó, se indican allí.

Antes de establecer esta variable, se recomienda ejecutar una prueba del banco de pruebas. El proceso de prueba del banco de pruebas informará del valor de simultaneidad recomendado. Como alternativa, si las condiciones de la red y las cargas varían, establezca esta variable en la palabra `AUTO` en lugar de en un número determinado. Esto hará que AzCopy ejecute siempre el mismo proceso de ajuste automático que utiliza en las pruebas del banco de pruebas.

## <a name="limit-the-throughput-data-rate"></a>Limitación de la velocidad de los datos de rendimiento

Puede usar la marca `cap-mbps` en los comandos para colocar un límite superior en la velocidad de datos de rendimiento. Por ejemplo, el siguiente comando reanuda un trabajo y limita el rendimiento a `10` megabits (MB) por segundo. 

```azcopy
azcopy jobs resume <job-id> --cap-mbps 10
```

## <a name="optimize-memory-use"></a>Optimización del uso de memoria

Establezca la variable de entorno `AZCOPY_BUFFER_GB` para especificar la cantidad máxima de memoria del sistema que quiere que use AzCopy para el almacenamiento en búfer al descargar y cargar archivos. Exprese este valor en gigabytes (GB).

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set AZCOPY_BUFFER_GB=<value>` |
| **Linux** | `export AZCOPY_BUFFER_GB=<value>` |
| **macOS** | `export AZCOPY_BUFFER_GB=<value>` |

> [!NOTE]
> El seguimiento de trabajos siempre conlleva una sobrecarga adicional en el uso de memoria. La cantidad varía en función del número de transferencias de un trabajo. Los búferes son el componente más grande del uso de memoria. Puede ayudar a controlar la sobrecarga usando `AZCOPY_BUFFER_GB` para cumplir los requisitos en general, pero no hay ninguna marca disponible que limite estrictamente el uso de memoria total.

## <a name="optimize-file-synchronization"></a>Optimización de la sincronización de archivos

El comando [sync](storage-ref-azcopy-sync.md) identifica todos los archivos en el destino y, luego, compara los nombres de archivo y las marcas de tiempo de última modificación antes de iniciar la operación de sincronización. Si tiene un gran número de archivos, puede eliminar este procesamiento por adelantado para mejorar el rendimiento. 

Para ello, use en su lugar el comando [azcopy copy](storage-ref-azcopy-copy.md) y establezca la marca `--overwrite` en `ifSourceNewer`. AzCopy comparará los archivos a medida que se copien sin necesidad de realizar exámenes y comparaciones por adelantado. Esto proporciona una ventaja para el rendimiento en los casos en los que hay un gran número de archivos que se van a comparar.

El comando [azcopy copy](storage-ref-azcopy-copy.md) no elimina archivos del destino, por lo que si desea eliminar archivos en el destino cuando ya no existan en el origen, use el comando [azcopy sync](storage-ref-azcopy-sync.md) con la marca `--delete-destination` establecida en el valor `true` o `prompt`.

## <a name="see-also"></a>Consulte también

- [Introducción a AzCopy](storage-use-azcopy-v10.md)