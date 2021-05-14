---
title: Sugerencias y trucos de uso de la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece sugerencias y trucos de uso de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: 6b777c834917d2a5b1fba72d7c18ef536109675a
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107929658"
---
# <a name="tips-and-tricks-for-using-azure-application-consistent-snapshot-tool"></a>Sugerencias y trucos de uso de la herramienta Azure Application Consistent Snapshot

En este artículo se ofrecen sugerencias y trucos que pueden resultar útiles cuando se usa AzAcSnap.

## <a name="limit-service-principal-permissions"></a>Limitación de permisos de entidad de servicio

Puede que sea necesario limitar el ámbito de la entidad de servicio de AzAcSnap.  Revise la [documentación de Azure RBAC](../role-based-access-control/index.yml) para obtener más información sobre la administración de acceso específico de recursos de Azure.  

A continuación se proporciona una definición de roles de ejemplo con las acciones mínimas necesarias para que AzAcSnap funcione.

```bash
az role definition create --role-definition '{ \
  "Name": "Azure Application Consistent Snapshot tool", \
  "IsCustom": "true", \
  "Description": "Perform snapshots on ANF volumes.", \
  "Actions": [ \
    "Microsoft.NetApp/*/read", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/write", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/delete" \
  ], \
  "NotActions": [], \
  "DataActions": [], \
  "NotDataActions": [], \
  "AssignableScopes": ["/subscriptions/<insert your subscription id>"] \
}'
```

Para que las opciones de restauración funcionen correctamente, la entidad de servicio AzAcSnap también debe ser capaz de crear volúmenes.  En este caso, la definición de rol necesita una acción adicional, por lo que la entidad de servicio completa debe ser similar a la del ejemplo siguiente.

```bash
az role definition create --role-definition '{ \
  "Name": "Azure Application Consistent Snapshot tool", \
  "IsCustom": "true", \
  "Description": "Perform snapshots and restores on ANF volumes.", \
  "Actions": [ \
    "Microsoft.NetApp/*/read", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/write", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/snapshots/delete", \
    "Microsoft.NetApp/netAppAccounts/capacityPools/volumes/write" \
  ], \
  "NotActions": [], \
  "DataActions": [], \
  "NotDataActions": [], \
  "AssignableScopes": ["/subscriptions/<insert your subscription id>"] \
}'
```


## <a name="take-snapshots-manually"></a>Realización manual de instantáneas

Antes de ejecutar los comandos de copia de seguridad (`azacsnap -c backup`), compruebe la configuración ejecutando los comandos de prueba y compruebe que se ejecutan correctamente.  La ejecución correcta de estas pruebas ha demostrado que `azacsnap` puede comunicarse con la base de datos de SAP HANA y el sistema de almacenamiento subyacente del sistema SAP HANA en **Azure (instancias grandes)** o **Azure NetApp Files**.

- `azacsnap -c test --test hana`
- `azacsnap -c test --test storage`

A continuación, para realizar una copia de seguridad manual de la instantánea de base de datos, ejecute el siguiente comando:

```bash
azacsnap -c backup --volume data --prefix hana_TEST --retention=1
```

## <a name="setup-automatic-snapshot-backup"></a>Configuración de la copia de seguridad automática de instantáneas

Es una práctica común en los sistemas UNIX/Linux usar `cron` para automatizar comandos de ejecución en un sistema. La práctica estándar de las herramientas de instantáneas consiste en configurar el `crontab`del usuario.

A continuación se muestra un ejemplo de un `crontab` para que el `azacsnap` del usuario automatice las instantáneas.

```output
MAILTO=""
# =============== TEST snapshot schedule ===============
# Data Volume Snapshots - taken every hour.
@hourly (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume data --prefix hana_TEST --retention=9)
# Other Volume Snapshots - taken every 5 minutes, excluding the top of the hour when hana snapshots taken
5,10,15,20,25,30,35,40,45,50,55 * * * * (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume other --prefix logs_TEST --retention=9)
# Other Volume Snapshots - using an alternate config file to snapshot the boot volume daily.
@daily (. /home/azacsnap/.profile ; cd /home/azacsnap/bin ; azacsnap -c backup --volume other --prefix DailyBootVol --retention=7 --configfile boot-vol.json)
```

Explicación del crontab anterior.

- `MAILTO=""`: si se tiene un valor vacío, se evita que cron intente enviar automáticamente por correo electrónico al usuario cuando se ejecute la entrada de crontab, ya que podría acabar en el archivo de correo del usuario local.
- Las versiones abreviadas de intervalos para las entradas de crontab se explican por sí mismas:
  - `@monthly` = se ejecuta una vez al mes, es decir, "0 0 1 * *".
  - `@weekly` = se ejecuta una vez a la semana, es decir, "0 0 * * 0".
  - `@daily` = se ejecuta una vez al día, es decir, "0 0 * * *".
  - `@hourly` = se ejecuta una vez a la hora, es decir, "0 * * * *".
- Las cinco primeras columnas se usan para designar horas, consulte los ejemplos de columna siguientes:
  - `0,15,30,45`: Cada 15 minutos
  - `0-23`: Cada hora
  - `*`: Todos los días
  - `*`: Cada mes
  - `*`: Todos los días de la semana
- Línea de comandos que se va a ejecutar entre paréntesis "()"
  - `. /home/azacsnap/.profile` = extraer el perfil del usuario para configurar su entorno, incluidos $PATH, etc.
  - `cd /home/azacsnap/bin` = cambiar el directorio de ejecución a la ubicación "/home/azacsnap/bin" donde se encuentran los archivos de configuración.
  - `azacsnap -c .....` = comando azacsnap completo que se va a ejecutar, incluidas todas las opciones.

Explicación más detallada de cron y el formato del archivo crontab aquí: <https://en.wikipedia.org/wiki/Cron>

> [!NOTE]
> Los usuarios son responsables de supervisar los trabajos cron para asegurarse de que las instantáneas se generan correctamente.

## <a name="monitor-the-snapshots"></a>Supervisión de las instantáneas

Se deben supervisar las siguientes condiciones para garantizar un sistema correcto:

1. Espacio disponible en disco. Las instantáneas consumirán lentamente espacio en disco, ya que los bloques de disco más antiguos se conservan en la instantánea.
    1. Para ayudar a automatizar la administración del espacio en disco, use las opciones `--retention` y `--trim` para limpiar automáticamente las instantáneas y los archivos de registro de base de datos antiguos.
1. Ejecución correcta de las herramientas de instantáneas
    1. Compruebe el archivo `*.result` para ver si se ha realizado la última ejecución de `azacsnap`.
    1. Compruebe `/var/log/messages` para ver la salida del comando `azacsnap`.
1. Coherencia de las instantáneas mediante su restauración en otro sistema periódicamente.

> [!NOTE]
> Para mostrar los detalles de la instantánea, ejecute el comando `azacsnap -c details`.

## <a name="delete-a-snapshot"></a>Eliminación de una instantánea

Para eliminar una instantánea, ejecute el comando `azacsnap -c delete`. No es posible eliminar instantáneas del nivel de sistema operativo. Debe usar el comando correcto (`azacsnap -c delete`) para eliminar las instantáneas de almacenamiento.

> [!IMPORTANT]
> Tenga cuidado al eliminar una instantánea. Una vez eliminadas, es **IMPOSIBLE** recuperar las instantáneas eliminadas.

## <a name="restore-a-snapshot"></a>Restauración de una instantánea

Una instantánea de volumen de almacenamiento se puede restaurar en un otro volumen (`-c restore --restore snaptovol`).  En el caso de Azure (instancias grandes), el volumen se puede revertir a una instantánea (`-c restore --restore revertvolume`).

> [!NOTE]
> **NO** se ha proporcionado ningún comando de recuperación de base de datos.

Una instantánea puede volver a copiarse en el área de datos de SAP HANA, pero SAP HANA no debe estar en ejecución cuando se realiza una copia (`cp /hana/data/H80/mnt00001/.snapshot/hana_hourly.2020-06-17T113043.1586971Z/*`).

En el caso de Azure (instancias grandes), puede ponerse en contacto con el equipo de operaciones de Microsoft; para ello, abra una solicitud de servicio para restaurar la instantánea que quiera a partir de las instantáneas existentes disponibles. Puede abrir una solicitud de servicio desde Azure Portal: <https://portal.azure.com>

Si decide realizar la conmutación por error de recuperación ante desastres, el comando `azacsnap -c restore --restore revertvolume` en el sitio de recuperación ante desastres pondrá automáticamente a disposición las instantáneas de volumen más recientes (`/hana/data` y `/hana/logbackups`) para permitir una recuperación de SAP HANA. Use este comando con precaución, ya que interrumpe la replicación entre los sitios de producción y de recuperación ante desastres.

## <a name="set-up-snapshots-for-boot-volumes-only"></a>Configuración de instantáneas solo para volúmenes de "arranque"

> [!IMPORTANT]
> Esta operación solo se aplica a Azure (instancias grandes).

En algunos casos, los clientes ya tienen herramientas para proteger SAP HANA y solo quieren configurar instantáneas de volumen de "arranque".  En este caso, se simplifica la tarea y se deben realizar los siguientes pasos.

1. Complete los pasos del 1 al 4 de los requisitos previos para la instalación.
1. Habilite la comunicación con el almacenamiento.
1. Descargue la ejecución del instalador para instalar las herramientas de instantáneas.
1. Complete la configuración de las herramientas de instantáneas.
1. Cree otro archivo de configuración tal y como se indica a continuación. Los detalles del volumen de arranque deben estar en la estrofa OtherVolume (entradas de usuario en <span style="color:red">rojo</span>):
    ```output
    > <span style="color:red">azacsnap -c configure --configuration new --configfile BootVolume.json</span>
    Building new config file
    Add comment to config file (blank entry to exit adding comments):<span style="color:red">Boot only config file.</span>
    Add comment to config file (blank entry to exit adding comments):
    Add database to config? (y/n) [n]: <span style="color:red">y</span>
    HANA SID (for example, H80): <span style="color:red">X</span>
    HANA Instance Number (for example, 00): <span style="color:red">X</span>
    HANA HDB User Store Key (for example, `hdbuserstore List`): <span style="color:red">X</span>
    HANA Server's Address (hostname or IP address): <span style="color:red">X</span>
    Add ANF Storage to database section? (y/n) [n]:
    Add HLI Storage to database section? (y/n) [n]: <span style="color:red">y</span>
    Add DATA Volume to HLI Storage section of Database section? (y/n) [n]:
    Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]: <span style="color:red">y</span>
    Storage User Name (for example, clbackup25): <span style="color:red">shoasnap</span>
    Storage IP Address (for example, 192.168.1.30): <span style="color:red">10.1.1.10</span>
    Storage Volume Name (for example, hana_data_soldub41_t250_vol): <span style="color:red">t210_sles_boot_azsollabbl20a31_vol</span>
    Add OTHER Volume to HLI Storage section of Database section? (y/n) [n]:
    Add HLI Storage to database section? (y/n) [n]:
    Add database to config? (y/n) [n]:

    Editing configuration complete, writing output to 'BootVolume.json'.
    ```
1. Compruebe el archivo de configuración, consulte el ejemplo siguiente:

    Use el comando `cat` para ver el contenido del archivo de configuración:

    ```bash
    cat BootVolume.json
    ```

    ```output
    {
      "version": "5.0",
      "logPath": "./logs",
      "securityPath": "./security",
      "comments": [
        "Boot only config file."
      ],
      "database": [
        {
          "hana": {
            "serverAddress": "X",
            "sid": "X",
            "instanceNumber": "X",
            "hdbUserStoreName": "X",
            "savePointAbortWaitSeconds": 600,
            "hliStorage": [
              {
                "dataVolume": [],
                "otherVolume": [
                  {
                    "backupName": "shoasnap",
                    "ipAddress": "10.1.1.10",
                    "volume&quot;: &quot;t210_sles_boot_azsollabbl20a31_vol"
                  }
                ]
              }
            ],
            "anfStorage": []
          }
        }
      ]
    }
    ```

1. Prueba de una copia de seguridad de volumen de arranque

    ```bash
    azacsnap -c backup --volume other --prefix TestBootVolume --retention 1 --configfile BootVolume.json
    ```

1. Compruebe que aparece, tenga en cuenta la adición de la opción `--snapshotfilter` para limitar la lista de instantáneas devuelta.

    ```bash
    azacsnap -c details --snapshotfilter TestBootVolume --configfile BootVolume.json
    ```

    Salida del comando:
    ```output
    List snapshot details called with snapshotFilter 'TestBootVolume'
    #, Volume, Snapshot, Create Time, HANA Backup ID, Snapshot Size
    #1, t210_sles_boot_azsollabbl20a31_vol, TestBootVolume.2020-07-03T034651.7059085Z, "Fri Jul 03 03:48:24 2020", "otherVolume Backup|azacsnap version: 5.0 (Build: 20210421.6349)", 200KB
    , t210_sles_boot_azsollabbl20a31_vol, , , Size used by Snapshots, 1.31GB
    ```

1. Configure la copia de seguridad automática de instantáneas.

> [!NOTE]
> No es necesario configurar la comunicación con SAP HANA.

## <a name="restore-a-boot-snapshot"></a>Restauración de una instantánea de "arranque"

> [!IMPORTANT]
> Esta operación solo se destina a Azure (instancias grandes).
> El servidor se restaurará hasta el momento en que se tomó la instantánea.

Se puede recuperar una instantánea de "arranque" tal y como se indica a continuación:

1. El cliente tendrá que apagar el servidor.
1. Después de apagar el servidor, el cliente tendrá que abrir una solicitud de servicio que contenga el identificador de la máquina y la instantánea que se va a restaurar.
    > Los clientes pueden abrir una solicitud de servicio en Azure Portal: <https://portal.azure.com>
1. Microsoft restaurará el LUN del sistema operativo con el identificador de la máquina y la instantánea especificados y luego arrancará el servidor.
1. El cliente tendrá que confirmar que el servidor se ha arrancado y su estado es correcto.

No se realizarán pasos adicionales después de la restauración.

## <a name="key-facts-to-know-about-snapshots"></a>Hechos clave que se deben conocer sobre las instantáneas

Atributos clave de las instantáneas de volumen de almacenamiento:

- **Location of snapshots** (Ubicación de las instantáneas) Las instantáneas se pueden encontrar en un directorio virtual (`.snapshot`) dentro del volumen.  Consulte los siguientes ejemplos de **Azure (instancias grandes)** .
  - Base de datos: `/hana/data/<SID>/mnt00001/.snapshot`
  - Elementos compartidos: `/hana/shared/<SID>/.snapshot`
  - Registros: `/hana/logbackups/<SID>/.snapshot`
  - Arranque: las instantáneas de arranque para HLI son **no visibles** desde el nivel de sistema operativo, pero se pueden mostrar con `azacsnap -c details`.

  > [!NOTE]
  >  `.snapshot` es una carpeta *virtual* oculta de solo lectura que proporciona acceso de solo lectura a las instantáneas.

- **Max snapshot:** (Número máximo de instantáneas) El hardware puede admitir hasta 250 instantáneas por volumen. El comando de instantánea mantendrá un número máximo de instantáneas para el prefijo en función de la retención establecida en la línea de comandos y eliminará la instantánea más antigua si se supera el número máximo que se va a conservar.
- **Nombre de la instantánea:** El nombre de la instantánea incluye la etiqueta de prefijo proporcionada por el cliente.
- **Tamaño de la instantánea:** Depende del tamaño o de los cambios en el nivel de base de datos.
- **Ubicación del archivo de registro:** Los archivos de registro generados por los comandos se muestran en carpetas, tal y como se define en el archivo de configuración JSON, que, de forma predeterminada, es una subcarpeta en la que se ejecuta el comando (por ejemplo, `./logs`).

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas](azacsnap-troubleshoot.md)
