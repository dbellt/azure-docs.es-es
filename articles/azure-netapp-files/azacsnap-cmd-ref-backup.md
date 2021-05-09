---
title: Copias de seguridad con la herramienta Azure Application Consistent Snapshot para Azure NetApp Files | Microsoft Docs
description: Ofrece una guía para ejecutar el comando de copia de seguridad de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.
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
ms.topic: reference
ms.date: 04/21/2021
ms.author: phjensen
ms.openlocfilehash: 4e0091d1d94a173df07f956959580f7f862ec08f
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930036"
---
# <a name="back-up-using-azure-application-consistent-snapshot-tool"></a>Copias de seguridad con la herramienta Azure Application Consistent Snapshot

En este artículo se ofrece una guía para ejecutar el comando de copia de seguridad de la herramienta Azure Application Consistent Snapshot que puede usar con Azure NetApp Files.

## <a name="introduction"></a>Introducción

Una copia de seguridad basada en una instantánea de almacenamiento se ejecuta con el comando `azacsnap -c backup`.  Este comando realiza la orquestación de una instantánea de almacenamiento coherente con la base de datos en los volúmenes de DATOS y una instantánea de almacenamiento (sin configuración de coherencia con la base de datos) en los OTROS volúmenes.  

En el caso de los volúmenes de DATOS, `azacsnap` preparará la base de datos para una instantánea de almacenamiento, tomará la instantánea de almacenamiento de todos los volúmenes configurados y, por último, se aconsejará a la base de datos que se finalice la instantánea.  También administrará las entradas de base de datos que registran la actividad de copia de seguridad de instantáneas (por ejemplo, el catálogo de copia de seguridad de SAP HANA).

## <a name="command-options"></a>Opciones de comando

El comando `-c backup` toma los argumentos siguientes:

- `--volume=`: tipo de volumen para las instantáneas; este parámetro puede contener `data` o `other`.
  - Instantáneas `data`: volúmenes dentro de la estrofa dataVolume del archivo de configuración.
  - Instantáneas `other`: volúmenes dentro de la estrofa otherVolume del archivo de configuración.
  
  > [!NOTE]
  > Al crear un archivo de configuración independiente con el volumen de arranque como otherVolume, es posible que se tomen instantáneas de `boot` en una programación totalmente diferente (por ejemplo, diariamente).

- `--prefix=`: prefijo de la instantánea del cliente para el nombre de la instantánea. Este parámetro tiene dos propósitos. En primer lugar, debe proporcionar un nombre único para la agrupación de instantáneas. En segundo lugar, para determinar el número `--retention` de instantáneas de almacenamiento que se conservan para el `--prefix`especificado.

    > [!IMPORTANT]
    > Solo se admiten caracteres alfanuméricos ("A-Z, a-z, 0-9"), guión bajo ("_") y guiones ("-").

- `--retention`: número de instantáneas del `--prefix` definido que se va a conservar. Las instantáneas adicionales se quitan después de tomar una nueva instantánea para este `--prefix`.

- `--trim`: disponible para SAP HANA V2 y versiones posteriores, esta opción mantiene el catálogo de copias de seguridad y el catálogo de discos y las copias de seguridad de registros. El número de entradas que se conservan en el catálogo de copias de seguridad viene determinado por la opción `--retention` anterior y elimina las entradas más antiguas para el prefijo definido (`--prefix`) del catálogo de copias de seguridad y la copia de seguridad de registros físicos relacionada. También elimina las entradas de copia de seguridad de registros anteriores a la entrada de copia de seguridad que no es de registro más antigua. Esta operación ayuda a evitar que las copias de seguridad de registros usen todo el espacio en disco disponible.

  > [!NOTE]
  > El siguiente comando de ejemplo mantendrá nueve instantáneas de almacenamiento y garantizará que el catálogo de copias de seguridad se recorte continuamente para que coincida con las nueve instantáneas de almacenamiento que se conservan.

    ```bash
    azacsnap -c backup --volume data --prefix hana_TEST --retention 9 --trim
    ```

- `[--ssl=]`: parámetro opcional que define el método de cifrado que se usa para comunicarse con SAP HANA, ya sea `openssl` o `commoncrypto`. Si se define, el comando `azacsnap -c backup` espera encontrar dos archivos en el mismo directorio. Estos archivos se deben denominar después del SID correspondiente. Consulte [Uso de SSL para la comunicación con SAP HANA](azacsnap-installation.md#using-ssl-for-communication-with-sap-hana). En el ejemplo siguiente, se toma una instantánea de tipo `hana` con un prefijo de `hana_TEST` y se mantendrán `9` de ellas comunicándose con SAP HANA mediante SSL (`openssl`).

    ```bash
    azacsnap -c backup --volume data --prefix hana_TEST --retention 9 --trim --ssl=openssl
    ```

- `[--configfile <config filename>]`: es un parámetro opcional que permite nombres de archivo de configuración personalizados.

## <a name="snapshot-backups-are-fast"></a>Las copias de seguridad de instantáneas son rápidas

La duración de una copia de seguridad de instantáneas es independiente del tamaño del volumen: un volumen de 10 TB se ajusta en el mismo tiempo aproximado que un volumen de 10 GB.  

Los principales factores que afectan al tiempo de ejecución global son el número de volúmenes de instantáneas y los cambios en el parámetro `--retention` (donde una reducción puede aumentar el tiempo de ejecución a medida que se quitan las instantáneas sobrantes).

En la configuración de ejemplo anterior, para **Azure (instancias grandes)** , las instantáneas de los dos volúmenes tardaron menos de 5 segundos en completarse. Para **Azure NetApp Files**, las instantáneas de los dos volúmenes tardarán aproximadamente 60 segundos.

> [!NOTE]
> Si `--retention` se reduce significativamente de la hora anterior en que se ejecuta `azacsnap` (por ejemplo, de `--retention 50` a `--retention 5`), el tiempo invertido aumentará a medida que `azacsnap` necesite quitar las instantáneas adicionales.

## <a name="example-with-data-parameter"></a>Ejemplo con el parámetro `data`

```bash
azacsnap -c backup --volume data --prefix hana_TEST --retention 9 --trim
```

El comando no genera resultados en la consola, sino que escribe en un archivo de registro, un archivo de resultados y `/var/log/messages`.

El *archivo de registro* se compone del nombre del comando + la opción -c + el nombre de archivo de configuración. De manera predeterminada, un nombre de archivo de registro para `-c backup` se ejecuta con un nombre de archivo de configuración `azacsnap-backup-azacsnap.log` predeterminado.

El archivo de *resultados* tiene el mismo nombre base que el archivo de registro, con `.result` como sufijo; por ejemplo, `azacsnap-backup-azacsnap.result`, que contiene el siguiente resultado:

```bash
cat logs/azacsnap-backup-azacsnap.result
```

```output
Database # 1 (H80) : completed ok
```

El archivo `/var/log/messages` contiene el mismo resultado que el archivo `.result`. Vea el siguiente ejemplo (ejecución como raíz):

```bash
grep "azacsnap.*Database" /var/log/messages | tail -n10
```

```output
Jul  2 05:22:07 server01 azacsnap[183868]: Database # 1 (H80) : completed ok
Jul  2 05:27:06 server01 azacsnap[4069]: Database # 1 (H80) : completed ok
Jul  2 05:32:07 server01 azacsnap[19769]: Database # 1 (H80) : completed ok
Jul  2 05:37:06 server01 azacsnap[35312]: Database # 1 (H80) : completed ok
Jul  2 05:42:06 server01 azacsnap[50877]: Database # 1 (H80) : completed ok
Jul  2 05:47:06 server01 azacsnap[66429]: Database # 1 (H80) : completed ok
Jul  2 05:52:06 server01 azacsnap[82964]: Database # 1 (H80) : completed ok
Jul  2 05:57:06 server01 azacsnap[98522]: Database # 1 (H80) : completed ok
Jul  2 05:59:13 server01 azacsnap[105519]: Database # 1 (H80) : completed ok
Jul  2 06:02:06 server01 azacsnap[114280]: Database # 1 (H80) : completed ok
```

## <a name="example-with-other-parameter"></a>Ejemplo con el parámetro `other`

```bash
azacsnap -c backup --volume other --prefix logs_TEST --retention 9
```

El comando no genera resultados en la consola, sino que escribe en un archivo de registro únicamente.  _No_ escribe en un archivo de resultados ni en `/var/log/messages`.

El *archivo de registro* se compone del nombre del comando + la opción -c + el nombre de archivo de configuración. De manera predeterminada, un nombre de archivo de registro para `-c backup` se ejecuta con un nombre de archivo de configuración `azacsnap-backup-azacsnap.log` predeterminado.

## <a name="example-with-other-parameter-to-backup-host-os"></a>Ejemplo con el parámetro `other` (para el sistema operativo del host de copia de seguridad)

> [!NOTE]
> Uso de otro archivo de configuración (`--configfile bootVol.json`) que solo contiene los volúmenes de arranque.

```bash
azacsnap -c backup --volume other --prefix boot_TEST --retention 9 --configfile bootVol.json
```

El comando no genera resultados en la consola, sino que escribe en un archivo de registro únicamente.  _No_ escribe en un archivo de resultados ni en `/var/log/messages`.

El nombre de *archivo de registro* en este ejemplo es `azacsnap-backup-bootVol.log`.

> [!NOTE]
> El nombre de archivo de registro se compone del "(nombre del comando - (la opción `-c`) - (el nombre de archivo de configuración)".  Por ejemplo, si usa la opción `-c backup` con un nombre de archivo de registro de `h80.json`, el archivo de registro se denominará `azacsnap-backup-h80.log`.  O bien, si usa la opción `-c test` con el mismo archivo de configuración, el archivo de registro se denominará `azacsnap-test-h80.log`.

- Tipo de instancia grande de HANA: Hay dos valores válidos con `TYPEI` o `TYPEII`, en función de la unidad de instancia grande de HANA.
- Consulte [SKU disponibles para instancias grandes de HANA](../virtual-machines/workloads/sap/hana-available-skus.md) para confirmar las SKU disponibles.

## <a name="next-steps"></a>Pasos siguientes

- [Obtención de los detalles de la instantánea](azacsnap-cmd-ref-details.md)
- [Eliminar instantáneas](azacsnap-cmd-ref-delete.md)
