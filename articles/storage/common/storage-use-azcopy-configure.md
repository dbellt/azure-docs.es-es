---
title: Búsqueda de errores y reanudación de trabajos con registros en AzCopy (Azure Storage) | Microsoft Docs
description: Aprenda a usar registros para diagnosticar errores y reanudar trabajos en pausa mediante archivos de plan.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: d3b956803e9a796c49288f90873e88c3b69f1c7b
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502902"
---
# <a name="find-errors-and-resume-jobs-by-using-log-and-plan-files-in-azcopy"></a>Búsqueda de errores y reanudación de trabajos mediante archivos de registro y de plan en AzCopy

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. En este artículo se describe cómo usar registros para diagnosticar errores y, a continuación, usar archivos de plan para reanudar trabajos. En este artículo también se muestra cómo configurar archivos de registro y de plan cambiando su nivel de detalle y la ubicación predeterminada donde se almacenan.

> [!NOTE]
> Si busca contenido que le ayude a empezar a trabajar con AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

## <a name="log-and-plan-files"></a>Archivos de registro y de plan

AzCopy crea archivos de *registro* y de *plan* para cada trabajo. Puede usar los registros para investigar y solucionar los posibles problemas. 

Los registros contendrán el estado de error (`UPLOADFAILED`, `COPYFAILED` y `DOWNLOADFAILED`), la ruta de acceso completa y el motivo del error.

De forma predeterminada, los archivos de registro y de plan se encuentran en el directorio `%USERPROFILE%\.azcopy` de Windows o en el directorio `$HOME$\.azcopy` en Mac y Linux, pero puede cambiar la ubicación. 

El error pertinente no es necesariamente el primer error que aparece en el archivo. En el caso de errores como errores de red, tiempos de espera y errores de servidor ocupado, AzCopy volverá a intentarlo hasta 20 veces y, normalmente, el proceso de reintento se realizará correctamente.  El primer error que se ve podría ser algo inofensivo que se reintentara correctamente.  Por lo tanto, en lugar de examinar el primer error del archivo, busque los errores cercanos `UPLOADFAILED`, `COPYFAILED` o `DOWNLOADFAILED`. 

> [!IMPORTANT]
> Al enviar una solicitud a Soporte técnico de Microsoft (o al solucionar el problema con la participación de terceros), comparta la versión censurada del comando que quiere ejecutar. Esto garantiza que la SAS no se comparta de forma accidental con nadie. Puede encontrar la versión censurada al principio del archivo de registro.

## <a name="review-the-logs-for-errors"></a>Revisión de los registros en busca errores

El comando siguiente obtiene todos los errores con el estado `UPLOADFAILED` del registro `04dc9ca9-158f-7945-5933-564021086c79`:

**Windows (PowerShell)**

```
Select-String UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

**Linux**

```
grep UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

## <a name="view-and-resume-jobs"></a>Visualización y reanudación de trabajos

Cada operación de transferencia creará un trabajo de AzCopy. Use el comando siguiente para ver el historial de trabajos:

```
azcopy jobs list
```

Para ver las estadísticas del trabajo, use el siguiente comando:

```
azcopy jobs show <job-id>
```

Para filtrar a las transferencias por estado, use el siguiente comando:

```
azcopy jobs show <job-id> --with-status=Failed
```

Use el comando siguiente para reanudar un trabajo con error o cancelado. Este comando usa su identificador, junto con el token de SAS, ya que no es persistente por motivos de seguridad:

```
azcopy jobs resume <job-id> --source-sas="<sas-token>"
azcopy jobs resume <job-id> --destination-sas="<sas-token>"
```

> [!TIP]
> Encierre los argumentos de ruta de acceso, como el token de SAS, entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

Al reanudar un trabajo, AzCopy examina el archivo de plan de trabajo. En el archivo de plan se enumeran todos los archivos que se han identificado para el procesamiento al crear el trabajo por primera vez. Al reanudar un trabajo, AzCopy intentará transferir todos los archivos que aparecen en el archivo de plan que no se han transferido todavía.

## <a name="change-the-location-of-plan-files"></a>Cambio de la ubicación de los archivos de plan

Use cualquiera de estos comandos.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | PowerShell:`$env:AZCOPY_JOB_PLAN_LOCATION="<value>"` <br> En un símbolo del sistema, use: `set AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **Linux** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **macOS** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, los registros se escriben en la ubicación predeterminada.

## <a name="change-the-location-of-log-files"></a>Cambio de la ubicación de los archivos de registro

Use cualquiera de estos comandos.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | PowerShell:`$env:AZCOPY_LOG_LOCATION="<value>"` <br> En un símbolo del sistema, use: `set AZCOPY_LOG_LOCATION=<value>`|
| **Linux** | `export AZCOPY_LOG_LOCATION=<value>` |
| **macOS** | `export AZCOPY_LOG_LOCATION=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, los registros se escriben en la ubicación predeterminada.

## <a name="change-the-default-log-level"></a>Cambiar el nivel de registro predeterminado

De forma predeterminada, el nivel de registro de AzCopy se establece en `INFO`. Si quiere reducir el nivel de detalle del registro para ahorrar espacio en disco, sobrescriba este valor mediante la opción ``--log-level``. 

Los niveles de registro disponibles son: `NONE`, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `PANIC` y `FATAL`.

## <a name="remove-plan-and-log-files"></a>Eliminación de archivos de registro y de plan

Si quiere quitar todos los archivos de registro y de plan de la máquina local para ahorrar espacio en disco, use el comando `azcopy jobs clean`.

Para quitar los archivos de registro y de plan asociados a un solo trabajo, use `azcopy jobs rm <job-id>`. Reemplace el marcador de posición `<job-id>` en este ejemplo por el identificador del trabajo.

## <a name="see-also"></a>Consulte también

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
