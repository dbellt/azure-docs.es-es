---
title: Exclusión de discos de una replicación con Azure Site Recovery
description: Cómo excluir discos de una replicación en Azure con Azure Site Recovery.
ms.topic: conceptual
ms.date: 12/17/2019
ms.openlocfilehash: 45ee7b23d01ff80bcb2e090688626f3f7fccb146
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131125"
---
# <a name="exclude-disks-from-disaster-recovery"></a>Exclusión de discos de una recuperación ante desastres

En este artículo se describe cómo excluir discos de una replicación durante una recuperación ante desastres desde una ubicación local a Azure con [Azure Site Recovery](site-recovery-overview.md). Es posible que se quiera excluir discos de una replicación por diversos motivos:

- Para que los datos sin importancia que hay en el disco excluido no se repliquen.
- Para optimizar el ancho de banda que la replicación consume, o los recursos del destino.
- Para ahorrar recursos de almacenamiento y red, al no replicar los datos que no se necesitan.
- Las máquinas virtuales de Azure han alcanzado los límites de replicación de Site Recovery.


## <a name="supported-scenarios"></a>Escenarios admitidos

Se pueden excluir discos de una replicación según se indica en la tabla.

**De Azure a Azure** | **VMware a Azure** | **Hyper-V en Azure** | **Servidor físico en Azure**
--- | --- | --- | ---
Sí | Sí | Sí | Sí

## <a name="exclude-limitations"></a>Limitaciones de exclusión

**Limitación** | **Máquinas virtuales de Azure** | **Máquinas virtuales de VMware** | **Máquinas virtuales de Hyper-V**
--- | --- | ---
**Tipos de disco** | Se pueden excluir los discos básicos de la replicación.<br/><br/> No se pueden excluir los discos dinámicos ni de sistema operativo. Los discos temporales se excluyen de forma predeterminada. | Se pueden excluir los discos básicos de la replicación.<br/><br/> No se pueden excluir los discos dinámicos ni de sistema operativo. | Se pueden excluir los discos básicos de la replicación.<br/><br/> No se pueden excluir los discos del sistema operativo. Se recomienda no excluir discos dinámicos. Site Recovery no puede identificar qué disco VHS es básico o dinámico en la máquina virtual invitada. Si no se excluyen todos los discos del volumen dinámico dependientes, el disco dinámico protegido aparecerá como erróneo en la máquina virtual de conmutación por error y no se podrá acceder a los datos de ese disco.
**Disco en fase de replicación** | No se puede excluir un disco que se está replicando.<br/><br/> Deshabilite y vuelva a habilitar la replicación de la máquina virtual. |  No se puede excluir un disco que se está replicando. |  No se puede excluir un disco que se está replicando.
**Servicio Mobility (VMware)** | No es relevante. | Solo se pueden excluir discos en máquinas virtuales que ya tengan instalado el servicio Mobility.<br/><br/> Esto significa que hay que instalar el servicio Mobility manualmente en las máquinas virtuales de las que se quiera excluir discos. No se puede usar el mecanismo de instalación de inserción, porque instala el servicio Mobility solo después de habilitar la replicación. | No es relevante.
**Agregar o quitar** | Puede agregar discos administrados en máquinas virtuales de Azure con replicación habilitada con discos administrados. No se pueden quitar discos en máquinas virtuales de Azure con replicación habilitada. | Una vez habilitada la replicación, no se pueden agregar ni quitar discos. Deshabilite y vuelva a habilitar la replicación para agregar un disco. | Una vez habilitada la replicación, no se pueden agregar ni quitar discos. Deshabilite y vuelva a habilitar la replicación.
**Conmutación por error** | Si una aplicación necesita un disco que se ha excluido, después de la conmutación por error debe crear el disco manualmente para que la aplicación replicada se pueda ejecutar.<br/><br/> También puede crear el disco durante la conmutación por error de la máquina virtual, integrando Azure Automation en un plan de recuperación. | Si excluye un disco que una aplicación necesita, créelo manualmente en Azure después de la conmutación por error. | Si excluye un disco que una aplicación necesita, créelo manualmente en Azure después de la conmutación por error.
**Conmutación por recuperación local: discos creados manualmente** | No es relevante. | **Máquinas virtuales Windows**: Los discos creados manualmente en Azure no conmutan por recuperación. Por ejemplo, si se conmutan por error tres discos, y se crean dos discos directamente en una máquina virtual de Azure, solo conmutarán por recuperación los tres discos que conmutaron por error.<br/><br/> **Máquinas virtuales Linux**: Los discos creados manualmente en Azure conmutan por recuperación. Por ejemplo, si se conmutan por error tres discos, y se crean dos discos directamente en una máquina virtual de Azure, los cinco discos conmutarán por recuperación. No se pueden excluir discos creados manualmente de la conmutación por recuperación. | Los discos creados manualmente en Azure no conmutan por recuperación. Por ejemplo, si se conmutan por error tres discos, y se crean dos discos directamente en una máquina virtual de Azure, solo conmutarán por recuperación los tres discos que conmutaron por error.
**Conmutación por recuperación local: discos excluidos** | No es relevante. | Si se conmuta por recuperación a la máquina original, la configuración de disco de máquina virtual de conmutación por recuperación no incluye los discos excluidos. Los discos que se excluyeron de la replicación de VMware a Azure no estarán disponibles en la máquina virtual de conmutación por recuperación. | Cuando la conmutación por recuperación se realiza en la ubicación de Hyper-V original, la configuración de disco de máquina virtual de conmutación por recuperación es la misma que la del disco de máquina virtual de origen inicial. Los discos que se excluyeron de la replicación del sitio de Hyper-V a Azure estarán disponibles en la máquina virtual de conmutación por recuperación.



## <a name="typical-scenarios"></a>Escenarios típicos

Algunos ejemplos de actividad de datos que son perfectos candidatos para excluirse son las operaciones de escritura en un archivo de paginación (pagefile.sys) y en el archivo tempdb de Microsoft SQL Server. Según la carga de trabajo y el subsistema de almacenamiento, los archivos de paginación y tempdb pueden registrar una actividad considerable. La replicación de este tipo de datos en Azure requiere un consumo muy elevado de recursos.

- Para optimizar la replicación de una máquina virtual con un único disco virtual que incluye tanto el sistema operativo como el archivo de paginación, se puede hacer lo siguiente:
    1. Divida el disco virtual único en dos. Uno para el sistema operativo y el otro para el archivo de paginación.
    2. Excluya el disco del archivo de paginación de la replicación.

- Para optimizar la replicación de un disco que incluye tanto el archivo de la base de datos tempbd de Microsoft SQL Server como el de la base de datos del sistema, se podría hacer lo siguiente:
    1. Guarde la base de datos del sistema y tempdb en dos discos diferentes.
    2. Excluya el disco de tempdb de la replicación.

## <a name="example-1-exclude-the-sql-server-tempdb-disk"></a>Ejemplo 1: Exclusión del disco de la base de datos tempdb de SQL Server

Veamos cómo controlar la exclusión de discos, la conmutación por error y la conmutación por recuperación de una máquina virtual SQL Server Windows de origen (**SalesDB** *) donde queremos excluir tempdb. 

### <a name="exclude-disks-from-replication"></a>Excluir discos de la replicación

Tenemos los siguientes discos en la máquina virtual Windows de origen SalesDB.

**Nombre del disco** | **Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disco del sistema operativo.
DB-Disk1| Disk1 | D:\ | Base de datos del sistema SQL y Database1 del usuario.
DB-Disk2 (disco excluido de la protección) | Disk2 | E:\ | Archivos temporales.
DB-Disk3 (disco excluido de la protección) | Disk3 | F:\ | Base de datos tempdb de SQL.<br/><br/> Ruta de acceso de la carpeta: F:\MSSQL\Data\. Anote la ruta de acceso de la carpeta antes de que se produzca la conmutación por error.
DB-Disk4 | Disk4 |G:\ | Database2 del usuario

1. Habilitamos la replicación de la máquina virtual SalesDB.
2. Excluimos Disk2 y Disk3 de la replicación porque la actividad de los datos en esos discos es temporal. 


### <a name="handle-disks-during-failover"></a>Administración de discos durante la conmutación por error

Como los discos no se replican, cuando se conmuta por error a Azure, estos discos no están presentes en la máquina virtual de Azure creada después de la conmutación por error. La máquina virtual de Azure tiene los discos que se muestran en esta tabla.

**Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | ---
Disk0 | C:\ | Disco del sistema operativo.
Disk1 | E:\ | Almacenamiento temporal<br/><br/>Azure agrega este disco. Como Disk2 y Disk3 se excluyeron de la replicación, E: es la primera unidad disponible de la lista. Azure la asigna al volumen de almacenamiento temporal. Las demás letras de los discos replicados permanecen igual.
Disk2 | D:\ | Base de datos del sistema SQL y Database1 del usuario
Disk3 | G:\ | Database2 del usuario

En nuestro ejemplo, dado que Disk3 (el disco de tempdb de SQL) se excluyó de la replicación y no está disponible en la máquina virtual de Azure, el servicio SQL está en estado detenido y necesita la ruta de acceso F:\MSSQL\Data. Esta ruta de acceso se puede crear de varias maneras: 

- Agregando un disco nuevo después de la conmutación por error y asignando la ruta de acceso de carpeta de tempdb.
- Usar un disco de almacenamiento temporal existente para la ruta de acceso de carpeta de tempdb.

#### <a name="add-a-new-disk-after-failover"></a>Adición de un nuevo disco después de la conmutación por error

1. Anote las rutas de acceso de tempdb.mdf y tempdb.ldf de SQL antes de la conmutación por error.
2. En Azure Portal, agregue un disco nuevo a la máquina virtual de Azure de conmutación por error. El disco debe tener el mismo tamaño (o mayor) que el disco de tempdb de SQL de origen (Disk3).
3. Inicie sesión en la máquina virtual de Azure.
4. Desde la consola de administración de disco (diskmgmt.msc), inicialice y formatee el disco recién agregado.
5. Asigne la misma letra de unidad de la base de datos tempdb de SQL (F:).
6. Cree la carpeta para tempdb en el volumen F: (F:\MSSQL\Data).
7. Inicie el servicio SQL desde la consola de servicio.

#### <a name="use-an-existing-temporary-storage-disk"></a>Uso de un disco de almacenamiento temporal existente 

1. Abra un símbolo del sistema.
2. Ejecute SQL Server en modo de recuperación desde el símbolo del sistema.

    ```console
    Net start MSSQLSERVER /f / T3608
    ```

3. Ejecute el siguiente sqlcmd para cambiar a la nueva ruta de acceso de tempdb.

    ```sql
    sqlcmd -A -S SalesDB        **Use your SQL DBname**
    USE master;     
    GO      
    ALTER DATABASE tempdb       
    MODIFY FILE (NAME = tempdev, FILENAME = 'E:\MSSQL\tempdata\tempdb.mdf');
    GO      
    ALTER DATABASE tempdb       
    MODIFY FILE (NAME = templog, FILENAME = 'E:\MSSQL\tempdata\templog.ldf');       
    GO
    ```

4. Detenga el servicio Microsoft SQL Server.

    ```console
    Net stop MSSQLSERVER
    ```

5. Inicie el servicio Microsoft SQL Server.

    ```console
    Net start MSSQLSERVER
    ```

### <a name="vmware-vms-disks-during-failback-to-original-location"></a>Máquinas virtuales de VMware: discos durante la conmutación por recuperación en la ubicación original

Veamos ahora cómo administrar discos en máquinas virtuales de VMware cuando se conmuta por recuperación en la ubicación local original.

- **Discos creados en Azure**: dado que en nuestro ejemplo se usa una máquina virtual de Windows, los discos que creamos manualmente en Azure no se replican de vuelta en el sitio al conmutar por recuperación o al reproteger una máquina virtual.
- **Disco de almacenamiento temporal en Azure**: el disco de almacenamiento temporal tampoco se replica en el host local.
- **Discos excluidos**: los discos que se excluyeron de la replicación de VMware a Azure no están disponibles en la máquina virtual local tras la conmutación por recuperación.

Antes de conmutar por recuperación las máquinas virtuales de VMware en la ubicación original, la configuración de disco de la máquina virtual de Azure es la siguiente.

**Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | ---
Disk0 | C:\ | Disco del sistema operativo.
Disk1 | E:\ | Almacenamiento temporal.
Disk2 | D:\ | Base de datos del sistema SQL y Database1 del usuario.
Disk3 | G:\ | Database2 del usuario.

Después de la conmutación por recuperación, la máquina virtual de VMware en la ubicación original tiene los discos que se muestran en esta tabla.

**Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | ---
Disk0 | C:\ | Disco del sistema operativo.
Disk1 | D:\ | Base de datos del sistema SQL y Database1 del usuario.
Disk2 | G:\ | Database2 del usuario.


### <a name="hyper-v-vms-disks-during-failback-to-original-location"></a>Máquinas virtuales de Hyper-V: discos durante la conmutación por recuperación en la ubicación original

Veamos ahora cómo administrar discos en máquinas virtuales de Hyper-V cuando se conmuta por recuperación en la ubicación local original.

- **Discos creados en Azure**: Los discos que creamos manualmente en Azure no se replican de vuelta en el sitio al conmutar por recuperación o al reproteger una máquina virtual.
- **Disco de almacenamiento temporal en Azure**: el disco de almacenamiento temporal tampoco se replica en el host local.
- **Discos excluidos**: Después de la conmutación por recuperación, la configuración de disco de la máquina virtual es la misma que la configuración de disco de la máquina virtual original. Los discos que se excluyeron de la replicación de Hyper-V a Azure estarán disponibles en la máquina virtual de conmutación por recuperación.

Antes de conmutar por recuperación las máquinas virtuales de Hyper-V en la ubicación original, la configuración de disco de la máquina virtual de Azure es la siguiente.

**Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | ---
Disk0 | C:\ | Disco del sistema operativo.
Disk1 | E:\ | Almacenamiento temporal.
Disk2 | D:\ | Base de datos del sistema SQL y Database1 del usuario.
Disk3 | G:\ | Database2 del usuario.

Después de una conmutación por error planeada (conmutación por recuperación) de Azure a una instancia local de Hyper-V, la máquina virtual de Hyper-V de la ubicación original tiene los discos que se muestran en esta tabla.

**Nombre del disco** | **N.º del disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
 --- | --- | --- | ---
DB-Disk0-OS | Disk0 |   C:\ | Disco del sistema operativo.
DB-Disk1 | Disk1 | D:\ | Base de datos del sistema SQL y Database1 del usuario.
DB-Disk2 (disco excluido) | Disk2 | E:\ | Archivos temporales.
DB-Disk3 (disco excluido) | Disk3 | F:\ | Base de datos tempdb de SQL.<br/><br/> Ruta de acceso de la carpeta (F:\MSSQL\Data\).
DB-Disk4 | Disk4 | G:\ | Database2 del usuario


## <a name="example-2-exclude-the-paging-file-disk"></a>Ejemplo 2: Exclusión del disco del archivo de paginación

Veamos cómo controlar la exclusión de discos, la conmutación por error y la conmutación por recuperación de una máquina virtual Windows de origen, donde queremos excluir el disco del archivo pagefile.sys tanto en la unidad D como en una unidad alternativa.


### <a name="paging-file-on-the-d-drive"></a>Archivo de paginación en la unidad D

Tenemos los siguientes discos en la máquina virtual de origen.

**Nombre del disco** | **Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disco del sistema operativo
DB-Disk1 (excluir de la replicación) | Disk1 | D:\ | pagefile.sys
DB-Disk2 | Disk2 | E:\ | Datos del usuario 1
DB-Disk3 | Disk3 | F:\ | Datos del usuario 2

La configuración del archivo de paginación en la máquina virtual de origen es la siguiente:

![Captura de pantalla del cuadro de diálogo de memoria virtual, con la unidad D: resaltada. Línea de unidad resaltada que muestra [Pagefile volume] y un tamaño de archivo de paginación (MB) de 3000-7000.](./media/exclude-disks-replication/pagefile-d-drive-source-vm.png)

1. Habilitamos la replicación de la máquina virtual.
2. Excluimos DB-Disk1 de la replicación.

#### <a name="disks-after-failover"></a>Discos después de la conmutación por error

Tras la conmutación por error, la máquina virtual de Azure tiene los discos que se muestran en esta tabla.

**Nombre del disco** | **Sistema operativo invitado** | **Unidad** | **Tipo de datos en el disco**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disco del sistema operativo
DB-Disk1 | Disk1 | D:\ | Almacenamiento temporal/pagefile.sys <br/><br/> Como DB-Disk1 (D:) se ha excluido, D: es la primera letra de unidad disponible de la lista.<br/><br/> Azure la asigna al volumen de almacenamiento temporal.<br/><br/> Dado que D: está disponible, la configuración del archivo de paginación de la máquina virtual sigue siendo la misma.
DB-Disk2 | Disk2 | E:\ | Datos del usuario 1
DB-Disk3 | Disk3 | F:\ | Datos del usuario 2

La configuración de nuestro archivo de paginación en la máquina virtual de Azure es la siguiente:

![Configuración del archivo de paginación en la máquina virtual de Azure](./media/exclude-disks-replication/pagefile-azure-vm-after-failover.png)

### <a name="paging-file-on-another-drive-not-d"></a>Archivo de paginación en otra unidad (que no es D:)

Veamos un ejemplo en el que el archivo de paginación no está en la unidad D.  

Tenemos los siguientes discos en la máquina virtual de origen.

**Nombre del disco** | **Disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | --- | ---
DB-Disk0-OS | Disk0 | C:\ | Disco del sistema operativo
DB-Disk1 (excluir de la replicación) | Disk1 | G:\ | pagefile.sys
DB-Disk2 | Disk2 | E:\ | Datos del usuario 1
DB-Disk3 | Disk3 | F:\ | Datos del usuario 2

La configuración de nuestro archivo de paginación en la máquina virtual local es la siguiente:

![Configuración del archivo de paginación en la máquina virtual local](./media/exclude-disks-replication/pagefile-g-drive-source-vm.png)

1. Habilitamos la replicación de la máquina virtual.
2. Excluimos DB-Disk1 de la replicación.

#### <a name="disks-after-failover"></a>Discos después de la conmutación por error

Tras la conmutación por error, la máquina virtual de Azure tiene los discos que se muestran en esta tabla.

**Nombre del disco** | **N.º del disco de SO invitado** | **Unidad** | **Tipo de datos del disco**
--- | --- | --- | ---
DB-Disk0-OS | Disk0  |C:\ | Disco del sistema operativo
DB-Disk1 | Disk1 | D:\ | Almacenamiento temporal<br/><br/> Como D: es la primera unidad disponible en la lista, Azure la asigna al volumen de almacenamiento temporal.<br/><br/> Para todos los discos replicados, la unidad permanece igual.<br/><br/> Dado que el disco G: no está disponible, el sistema usará la unidad C: para el archivo de paginación.
DB-Disk2 | Disk2 | E:\ | Datos del usuario 1
DB-Disk3 | Disk3 | F:\ | Datos del usuario 2

La configuración de nuestro archivo de paginación en la máquina virtual de Azure es la siguiente:

![Captura de pantalla del cuadro de diálogo de memoria virtual, con la unidad C: resaltada. Línea de la unidad que aparece resaltada y muestra la opción de tamaño de archivo de paginación como "Administrado por el sistema".](./media/exclude-disks-replication/pagefile-azure-vm-after-failover-2.png)


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las directrices relativas al disco de almacenamiento temporal:
    - [Obtenga información sobre cómo](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/) usar SSD en máquinas virtuales de Azure para almacenar la base de datos TempDB de SQL Server y las extensiones del grupo de búferes.
    - [Repase ](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices-checklist.md) los procedimientos recomendados de rendimiento de SQL Server en máquinas virtuales de Azure.
- Después de que la implementación esté configurada y en ejecución, [obtenga más información](failover-failback-overview.md) sobre los diferentes tipos de conmutación por error.