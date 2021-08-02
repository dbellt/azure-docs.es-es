---
title: 'Copia de seguridad y restauración de los grupos de servidores de Azure Database for PostgreSQL: hiperescala'
description: 'Copia de seguridad y restauración de los grupos de servidores de Azure Database for PostgreSQL: hiperescala'
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 06/02/2021
ms.topic: how-to
ms.openlocfilehash: 4ad0fbacdfe8b6205dd32ecd75e5291b504adcac
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111407784"
---
# <a name="back-up-and-restore-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Copia de seguridad y restauración de grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

Al realizar una copia de seguridad del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc, o restaurarlo, la copia de seguridad o la restauración abarcan el conjunto completo de bases de datos en todos los nodos de PostgreSQL del grupo de servidores.

## <a name="take-a-manual-full-backup"></a>Realización de una copia de seguridad completa manual

Para hacer una copia de seguridad completa de todas las carpetas de datos y de registro del grupo de servidores, ejecute el siguiente comando:
```console
azdata arc postgres backup create [--name <backup name>] --server-name <server group name> [--no-wait] 
```
Donde:
- __name__ indica el nombre de una copia de seguridad
- __server-name__ indica un grupo de servidores
- __no-wait__ indica que la línea de comandos no esperará a que se complete la copia de seguridad para que pueda seguir usando esta ventana de línea de comandos.

Este comando coordinará una copia de seguridad completa distribuida en todos los nodos que constituyen el grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc. En otras palabras, hará una copia de seguridad de todos los datos de los nodos Coordinador y Trabajador.

Por ejemplo:

```console
azdata arc postgres backup create --name backup12082020-0250pm --server-name postgres01
```

Cuando finalice la copia de seguridad, se devolverá el identificador, el nombre, el tamaño, el estado y la marca de tiempo de la copia de seguridad. Por ejemplo:
```console
{
  "ID": "8085723fcbae4aafb24798c1458f4bb7",
  "name": "backup12082020-0250pm",
  "size": "9.04 MiB",
  "state": "Done",
  "timestamp": "2020-12-08 22:50:22+00:00"
}
```
`+xx:yy` indica la zona horaria de la hora a la que se realizó la copia de seguridad. En este ejemplo, "+ 00:00" significa la hora UTC (UTC + 00 horas 00 minutos).

> [!NOTE]
> Todavía no es posible:
> - Programación de copias de seguridad automáticas
> - Mostrar el progreso de una copia de seguridad mientras se está llevando a cabo

## <a name="list-backups"></a>Enumeración de copias de seguridad

Para enumerar las copias de seguridad que están disponibles para restaurar, ejecute el siguiente comando:

```console
azdata arc postgres backup list --server-name <servergroup name>
```

Por ejemplo:

```console
azdata arc postgres backup list --server-name postgres01
```

Devuelve una salida como la siguiente:

```output
ID                                Name                   Size       State    Timestamp
--------------------------------  ---------------------  ---------  -------  -------------------------
d744303b1b224ef48be9cba4f58c7cb9  backup12072020-0731pm  13.83 MiB  Done     2020-12-08 03:32:09+00:00
c4f964d28da34318a420e6d14374bd36  backup12072020-0819pm  9.04 MiB   Done     2020-12-08 04:19:49+00:00
a304c6ef99694645a2a90ce339e94714  backup12072020-0822pm  9.1 MiB    Done     2020-12-08 04:22:26+00:00
47d1f57ec9014328abb0d8fe56020760  backup12072020-0827pm  9.06 MiB   Done     2020-12-08 04:27:22+00:00
8085723fcbae4aafb24798c1458f4bb7  backup12082020-0250pm  9.04 MiB   Done     2020-12-08 22:50:22+00:00
```

La columna Timestamp indica la hora UTC a la que se realizó la copia de seguridad.

## <a name="restore-a-backup"></a>Restaurar una copia de seguridad
En esta sección se muestra cómo realizar una restauración completa o una restauración a un momento dado. Al restaurar una copia de seguridad completa, se restaura todo su contenido. Cuando la restauración es a un momento dado, se restaura hasta el momento dado que se indique. Las transacciones que se realizaron a partir de ahí no se restauran.

> [!CAUTION]
> Solo puede restaurar en un grupo de servidores que tenga el mismo número de nodos de trabajo que tenía cuando se tomó la copia de seguridad. Si ha aumentado o reducido el número de nodos de trabajo desde que se ha realizado la copia de seguridad, antes de realizar la restauración, debe aumentar o reducir dicho número (o crear un nuevo grupo de servidores) para que coincida con el contenido de la copia de seguridad. Se producirá un error en la restauración cuando el número de nodos de trabajo no coincida.

### <a name="restore-a-full-backup"></a>Restauración de una copia de seguridad completa
Para restaurar todo el contenido de una copia de seguridad, ejecute el comando:
```console
azdata arc postgres backup restore --server-name <target server group name> [--source-server-name <source server group name> --backup-id <backup id>]
or
azdata arc postgres backup restore -sn <target server group name> [-ssn <source server group name> --backup-id <backup id>]
```
<!--To read the general format of restore command, run: azdata arc postgres backup restore --help -->

Donde:
- __backup-id__ es el identificador de la copia de seguridad que se muestra anteriormente en el comando list backup.
De este modo se coordinará una restauración completa distribuida en todos los nodos que constituyen el grupo de servidores de hiperescala de PostgreSQL habilitado para Azure Arc. En otras palabras, hará una restauración de todos los datos de los nodos Coordinador y Trabajador.

#### <a name="examples"></a>Ejemplos:

__Restauración del grupo de servidores postgres01 en sí mismo:__

```console
azdata arc postgres backup restore -sn postgres01 --backup-id d134f51aa87f4044b5fb07cf95cf797f
```

Esta operación solo se admite para PostgreSQL versión 12 y versiones posteriores.

__Restauración del grupo de servidores postgres01 en un grupo de servidores diferente postgres02:__

```console
azdata arc postgres backup restore -sn postgres02 -ssn postgres01 --backup-id d134f51aa87f4044b5fb07cf95cf797f
```
Esta operación es compatible con cualquier versión de PostgreSQL a partir de la versión 11. El grupo de servidores de destino debe crearse antes de la operación de restauración, debe tener la misma configuración y debe usar el mismo PVC de copia de seguridad que el grupo de servidores de origen.

Una vez completada la operación de restauración, devolverá una salida similar a la siguiente en la línea de comandos:

```json
{
  "ID": "d134f51aa87f4044b5fb07cf95cf797f",
  "state": "Done"
}
```

> [!NOTE]
> Todavía no es posible:
> - Restaurar una copia de seguridad indicando su nombre
> - Mostrar el progreso de una operación de restauración


### <a name="do-a-point-in-time-restore"></a>Restauración a un momento dado

Para restaurar un grupo de servidores a un momento dado específico, ejecute el comando:
```console
azdata arc postgres backup restore --server-name <target server group name> --source-server-name <source server group name> --time <point in time to restore to>
or
azdata arc postgres backup restore -sn <target server group name> -ssn <source server group name> -t <point in time to restore to>
```

Para leer el formato general del comando de restauración, ejecute: `azdata arc postgres backup restore --help`.

Donde `time` es el momento al que se va a restaurar. Proporcione una marca de tiempo o un número y un sufijo ( `m` para minutos, `h` para horas, `d` para días o `w` para semanas). Por ejemplo, `1.5h` retrocede 90 minutos.

#### <a name="examples"></a>Ejemplos:
__Restauración a un momento dado del grupo de servidores postgres01 en sí mismo:__

Todavía no es posible restaurar a un momento dado un grupo de servidores en sí mismo.

__Restauración a un momento dado del grupo de servidores postgres01 en un grupo de servidores diferente postgres02 a una marca de tiempo específica:__
```console
azdata arc postgres backup restore -sn postgres02 -ssn postgres01 -t "2020-12-08 04:23:48.751326+00"
``` 

En este ejemplo se restaura el grupo de servidores postgres01 en el grupo de servidores postgres02 al estado en el que estaba el 8 de diciembre de 2020 a las 04:23:48.75 UTC. Tenga en cuenta que "+ 00" indica la zona horaria del momento en el tiempo que se indica. Si no se indica una zona horaria, se usará la zona horaria del cliente desde donde se ejecuta la operación de restauración.

Por ejemplo:
- `2020-12-08 04:23:48.751326+00` se interpreta como `2020-12-08 04:23:48.751326` UTC
- si se encuentra en la zona horaria estándar del Pacífico (PST = UTC + 08), `2020-12-08 04:23:48.751326` se interpreta como `2020-12-08 12:23:48.751326` UTC. Esta operación es compatible con cualquier versión de PostgreSQL a partir de la versión 11. El grupo de servidores de destino debe crearse antes de la operación de restauración y debe usar el mismo PVC de copia de seguridad que el grupo de servidores de origen.


__Restauración a un momento dado del grupo de servidores postgres01 en un grupo de servidores diferente postgres02 a una hora determinada del pasado:__
```console
azdata arc postgres backup restore -sn postgres02 -ssn postgres01 -t "22m"
```

En este ejemplo se restaura el grupo de servidores postgres01 en el grupo de servidores postgres02 al estado en el que estaba hace 22 minutos.
Esta operación es compatible con cualquier versión de PostgreSQL a partir de la versión 11. El grupo de servidores de destino debe crearse antes de la operación de restauración y debe usar el mismo PVC de copia de seguridad que el grupo de servidores de origen.

> [!NOTE]
> Todavía no es posible:
> - Mostrar el progreso de una operación de restauración

## <a name="delete-backups"></a>Eliminar copias de seguridad

No se puede establecer la retención de copias de seguridad en la versión preliminar. Sin embargo, puede eliminar manualmente las copias de seguridad que no necesite.
El comando general para eliminar copias de seguridad es:

```console
azdata arc postgres backup delete  [--server-name, -sn] {[--name, -n], -id}
```

donde:
- `--server-name` es el nombre del grupo de servidores del que el usuario desea eliminar una copia de seguridad.
- `--name` es el nombre de la copia de seguridad que se va a eliminar.
- `-id` es el identificador de la copia de seguridad que se va a eliminar.

> [!NOTE]
> `--name` y `-id` se excluyen mutuamente.

Por ejemplo:

```console
azdata arc postgres backup delete -sn postgres01 -n MyBackup091720200110am
{
  "ID": "5b0481dfc1c94b4cac79dd56a1bb21f4",
  "name": "MyBackup091720200110am",
  "state": "Done"
}
```

Puede recuperar el nombre y el identificador de las copias de seguridad ejecutando el comando de enumeración de copias de seguridad como se explicó en el párrafo anterior.

Para más detalles sobre el comando de eliminación, ejecute:

```console
azdata arc postgres backup delete --help
```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre el [escalado horizontal (adición de nodos de trabajo)](scale-out-in-postgresql-hyperscale-server-group.md) del grupo de servidores.
- Obtenga información sobre [cómo escalar o reducir verticalmente (aumentar o reducir la memoria o los núcleos virtuales)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) el grupo de servidores.
