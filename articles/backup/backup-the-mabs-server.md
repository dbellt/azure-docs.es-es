---
title: Copia de seguridad del servidor de MABS
description: Aprenda a realizar copias de seguridad con Microsoft Azure Backup Server (MABS).
ms.topic: conceptual
ms.date: 09/24/2020
ms.openlocfilehash: fbd3d1f2d7cb24c21962dbe5c88acebf73652a04
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107519126"
---
# <a name="back-up-the-mabs-server"></a>Copia de seguridad del servidor de MABS

Para asegurarse de que los datos se pueden recuperar si se produce un error en Microsoft Azure Backup Server (MABS), necesitará una estrategia para hacer una copia de seguridad del servidor de MABS. Si no se realiza una copia de seguridad, tendrá que recompilarla manualmente después de un error, y los puntos de recuperación basados en disco no se podrán recuperar. Puede realizar copias de seguridad de los servidores de MABS mediante una copia de seguridad de la base de datos MABS.

## <a name="back-up-the-mabs-database"></a>Copia de seguridad de la base de datos de MABS

Como parte de su estrategia de copia de seguridad de MABS, debe realizar una copia de la base de datos de MABS. La base de datos MABS se denomina DPMDB. Esta base de datos contiene la configuración MABS junto con datos sobre las copias de seguridad de MABS. Si se produce un desastre, puede volver a generar la mayor parte de la funcionalidad de un servidor MABS mediante una copia de seguridad reciente de la base de datos. Suponiendo que puede restaurar la base de datos, las copias de seguridad basadas en cinta estarían accesibles y mantendrían todas las configuraciones de grupo de protección y las programaciones de copia de seguridad. Si la interrupción no ha afectado a los discos del bloque de almacenamiento de MABS, las copias de seguridad basadas en disco también podrían utilizarse después de una recopilación. Dispone de varios métodos para hacer una copia de seguridad de la base de datos.

|Método de copia de seguridad de base de datos|Ventajas|Desventajas|
|--------------------------|--------------|-----------------|
|[Copia de seguridad en Azure](#back-up-to-azure)|<li>Realice la configuración y la supervisión fácilmente en MABS.<li>Varias ubicaciones de los archivos de la base de datos de copia de seguridad.<li>El almacenamiento en nube proporciona una solución sólida para la recuperación ante desastres.<li>Almacenamiento muy seguro para la base de datos.<li>Admite 120 puntos de recuperación en línea.|<li>Requiere una cuenta de Azure y una configuración adicional de MABS. Incurre en algún coste por el almacenamiento de Azure.<li> Requiere una versión compatible del sistema basado en Windows Server con el agente de Azure para obtener acceso a las copias de seguridad de MABS almacenadas en el almacén de Azure Backup. No puede ser otro servidor de MABS.<li>No es una opción si la base de datos se hospeda localmente y desea habilitar la protección secundaria. <li>Se incurre en algún tiempo de recuperación y preparación adicional.|
|[Copia de seguridad de la base de datos mediante la copia de seguridad del bloque de almacenamiento de MABS](#back-up-the-database-by-backing-up-the-mabs-storage-pool)|<li>Fácil de configurar y supervisar.<li>La copia de seguridad se mantiene en los discos del bloque de almacenamiento de MABS y es fácil obtener acceso a ella localmente.<li>Las copias de seguridad programadas de MABS admiten copias de seguridad completas rápidas 512. Si realiza una copia de seguridad cada hora, tendrá 21 días de protección completa.|<li>No es una opción conveniente para recuperación ante desastres. Es en línea y la recuperación podría no funcionar según lo esperado si se produce un error en el disco del bloque de almacenamiento o en el servidor de MABS.<li>No es una opción si la base de datos se hospeda localmente y desea habilitar la protección secundaria. <li>Se necesita una fase de preparación y de pasos especiales para obtener acceso a los puntos de recuperación si la consola o el servicio de MABS no se ejecuta o no funciona.|
|[Realización de copias de seguridad con la copia de seguridad nativa de SQL Server en un disco local](#back-up-with-native-sql-server-backup-to-a-local-disk)|<li>Está integrada en SQL Server.<li>La copia de seguridad se mantiene en un disco local que es fácilmente accesible.<li>Se puede programar para que se ejecute con tanta frecuencia como desee.<li>Totalmente independiente de MABS.<li>Puede programar una limpieza del archivo de copia de seguridad.|<li>No es una opción conveniente para la recuperación ante desastres, a menos que las copias de seguridad se copien en una ubicación remota.<li>Se requiere almacenamiento local para las copias de seguridad, que pueden limitar la retención y la frecuencia.|
|[Copia de seguridad con la copia de seguridad de SQL nativa y la protección de MABS en un recurso compartido protegido por MABS](#back-up-to-a-share-protected-by-mabs)|<li>Se supervisa fácilmente en MABS.<li>Varias ubicaciones de los archivos de la base de datos de copia de seguridad.<li>Fácil acceso desde cualquier equipo de Windows en la red.<li>Posiblemente el método de recuperación más rápido.|<li>Solo admite 64 puntos de recuperación.<li>No es una opción conveniente para recuperación del sitio ante desastres. El error del servidor o el disco del bloque de almacenamiento de MABS puede dificultar los esfuerzos de recuperación.<li>No es una opción si la base de datos de MABS se hospeda localmente y quiere habilitar la protección secundaria. <li>Es necesaria una preparación adicional para realizar la configuración y las pruebas.<li>Se necesita algún tiempo de recuperación y preparación adicional si el propio servidor de MABS está inactivo, pero los discos del bloque de almacenamiento de MABS son correctos.|

- Si hace una copia de seguridad con un grupo de protección de MABS, se recomienda usar un grupo de protección único para la base de datos.

    > [!NOTE]
    > Para fines de restauración, la instalación de MABS que quiere restaurar con la base de datos MABS debe coincidir con la versión de la base de datos MABS.  Por ejemplo, si la base de datos que quiere recuperar es de una instalación MABS V3 con el paquete acumulativo de actualizaciones 1, el servidor de MABS debe ejecutar la misma versión con el paquete acumulativo de actualizaciones 1. Esto puede implicar tener que desinstalar y reinstalar MABS con una versión compatible antes de restaurar la base de datos.  Para consultar la versión de la base de datos que puede necesitar para el montaje manual en un nombre de base de datos temporal, ejecute una consulta SQL en la base de datos para comprobar el último paquete acumulativo instalado basado en las versiones principal y secundaria.

- Para ver la versión de la base de datos de MABS, siga estos pasos:

    1. Para ejecutar la consulta, abra SQL Management Studio y conéctese a la instancia de SQL que ejecuta la base de datos de MABS.

    2. Seleccione la base de datos de MABS y, a continuación, inicie una nueva consulta.

    3. Pegue la siguiente consulta SQL en el panel de consulta y ejecútela:

        `Select distinct MajorVersionNumber,MinorVersionNumber ,BuildNumber, FileName FROM dbo.tbl\_AM\_AgentPatch order byMajorVersionNumber,MinorVersionNumber,BuildNumber`

    Si no se devuelve nada en los resultados de la consulta o si el servidor de MABS se actualizó a partir de versiones anteriores, pero no se instaló un nuevo paquete acumulativo de actualizaciones desde entonces, no habrá ninguna entrada para la versión principal o secundaria de una instalación base de MABS. Para comprobar las versiones de MABS asociadas a paquetes acumulativos de actualizaciones, vea [Lista de números de compilación de MABS](https://social.technet.microsoft.com/wiki/contents/articles/36381.microsoft-azure-backup-server-list-of-build-numbers.aspx).

### <a name="back-up-to-azure"></a>Copia de seguridad en Azure

1. Antes de comenzar, deberá ejecutar un script para recuperar la ruta de acceso del punto de montaje del volumen de réplica de MABS para saber qué punto de recuperación contiene la copia de seguridad de MABS. Puede hacerlo después de la replicación inicial con Azure Backup. En el script, reemplace `dplsqlservername%` por el nombre de la instancia de SQL Server que hospeda la base de datos de MABS.

    ```SQL
    Select ag.NetbiosName as ServerName,ds.DataSourceName,vol.MountPointPath
    from tbl_IM_DataSource as ds
    join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
    join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
    join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
    and vol.Usage =1
    and lr.Validity in (1,2)
    where ds.datasourcename like '%dpmdb%'
    and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB
    ```

    Asegúrese de que tiene el código de acceso que se especificó al instalar el agente de Azure Recovery Services y que el servidor de MABS se registró en el almacén de Azure Backup. Necesitará este código de acceso para restaurar la copia de seguridad.

2. Cree un almacén de copia de seguridad de Azure y descargue las credenciales del almacén y el archivo de instalación del Agente de copia de seguridad de Azure. Ejecute el archivo de instalación para instalar el agente en el servidor de MABS y use las credenciales del almacén para registrar el servidor de MABS en el almacén. [Más información](backup-azure-microsoft-azure-backup.md).

3. Una vez configurado el almacén, configure un grupo de protección de MABS que contenga la base de datos de MABS. Seleccione hacer una copia de seguridad en disco y en Azure.

#### <a name="recover-the-mabs-database-from-azure"></a>Recuperación de la base de datos de MABS desde Azure

Puede recuperar la base de datos de Azure con cualquier servidor de MABS que esté registrado en el almacén de Azure Backup, como se indica a continuación:

1. En la consola de MABS, seleccione **Recuperar** > **Agregar MABS externo**.

2. Proporcione las credenciales del almacén (descárguelas desde el almacén de Azure Backup). Tenga en cuenta que las credenciales solo son válidas durante dos días.

3. En **Seleccionar MABS externo para la recuperación**, seleccione el servidor de MABS para el que quiere recuperar la base de datos, escriba la frase de contraseña de cifrado y seleccione **Aceptar.**

4. Seleccione el punto de recuperación que quiere usar en la lista de puntos disponibles. Seleccione **Borrar MABS externo** para volver a la vista de MABS local.

## <a name="back-up-the-mabs-database-to-mabs-storage-pool"></a>Copia de seguridad de la base de datos de MABS en el bloque de almacenamiento de MABS

> [!NOTE]  
> Esta opción es aplicable a MABS con Modern Backup Storage.

1. En la consola de MABS, seleccione **Protección** > **Crear grupo de protección**.
2. En la página **Seleccionar tipo de grupo de protección** , seleccione **Servidores**.
3. En la página **Seleccionar miembros del grupo**, seleccione **Base de datos DPM**. Expanda el servidor de MABS y seleccione DPMDB.
4. En la página **Seleccionar método de protección de datos**, seleccione **Deseo protección a corto plazo mediante disco**. Especifique las opciones de directiva de protección a corto plazo.
5. Después de la replicación inicial de la base de datos de MABS, ejecute el siguiente script de SQL:

```SQL
select AG.NetbiosName, DS.DatasourceName, V.AccessPath, LR.PhysicalReplicaId from tbl_IM_DataSource DS
join tbl_PRM_LogicalReplica as LR
on DS.DataSourceId = LR.DataSourceId
join tbl_AM_Server as AG
on DS.ServerId=AG.ServerId
join tbl_PRM_ReplicaVolume RV
on RV.ReplicaId = LR.PhysicalReplicaId
join tbl_STM_Volume V
on RV.StorageId = V.StorageId
where datasourcename like N'%dpmdb%' and ds.ProtectedGroupId is not null
and LR.Validity in (1,2)
and AG.ServerName like N'%<dpmsqlservername>%' -- <dpmsqlservername> is a placeholder, put netbios name of server hosting DPMDB
```

### <a name="recover-mabs-database"></a>Recuperación de la base de datos de MABS

Para reconstruir MABS con la misma base de datos, primero debe recuperar la base de datos de MABS y sincronizarla con la instancia de MABS recién instalada.

#### <a name="use-the-following-steps"></a>Siga estos pasos

1. Abra un símbolo del sistema administrativo y ejecute `psexec.exe -s powershell.exe` para iniciar una ventana de PowerShell en el contexto del sistema.
2. Decida la ubicación desde la que desea recuperar la base de datos:

#### <a name="to-copy-the-database-from-the-last-backup"></a>Para copiar la base de datos de la última copia de seguridad

1. Vaya a la ruta de acceso del disco duro virtual de la réplica `\<MABSServer FQDN\>\<PhysicalReplicaId\>\<PhysicalReplicaId\>`
2. Monte el archivo **disk0.vhdx** presente en ella con el comando `mount-vhd disk0.vhdx`.
3. Una vez montado el disco duro virtual de la réplica, use `mountvol.exe` para asignar una letra de unidad al volumen de réplica con el id. de réplica física de la salida del script de SQL. Por ejemplo: `mountvol X: \?\Volume{}\`

#### <a name="to-copy-the-database-from-a-previous-recovery-point"></a>Para copiar la base de datos desde un punto de recuperación anterior

1. Navegue hasta el directorio del contenedor DPMDB `\<MABSServer FQDN\>\<PhysicalReplicaId\>`. Verá varios directorios con algunos identificadores GUID únicos en sus correspondiente puntos de recuperación para la base de datos de MABS. Otros directorios representan un punto de recuperación o PIT.
2. Navegue a cualquier ruta de acceso de VHD de PIT, por ejemplo, `\<MABSServer FQDN\>\<PhysicalReplicaId\>\<PITId\>`, y monte el archivo **disk0.vhdx** presente en ella con el comando `mount-vhd disk0.vhdx`.
3. Una vez montado el disco duro virtual de la réplica, use `mountvol.exe` para asignar una letra de unidad al volumen de réplica con el id. de réplica física de la salida del script de SQL. Por ejemplo: `mountvol X: \?\Volume{}\`

   Todos los términos que aparecen con llaves angulares en los pasos anteriores son marcadores de posición. Reemplácelos por los valores adecuados de la manera siguiente:
   - **ReFSVolume**: ruta de acceso de la salida del script de SQL.
   - **FQDN de MABSServer**: nombre completo del servidor de MABS.
   - **PhysicalReplicaId**: id. de réplica física de la salida del script de SQL.
   - **PITId**: identificador GUID distinto del id. de réplica física del directorio del contenedor.
4. Abra otro símbolo del sistema administrativo y ejecute `psexec.exe -s cmd.exe` para iniciar un símbolo del sistema en el contexto del sistema.
5. Cambie el directorio a la unidad X: y vaya hasta la ubicación de los archivos de base de datos de MABS.
6. Cópielos en una ubicación desde la que sea fácil restaurar. Cierre la ventana de psexec cmd después de copiar.
7. Vaya a la ventana psexec de PowerShell abierta en el paso 1, navegue a la ruta de acceso VHDX y desmonte el archivo VHDX mediante el comando `dismount-vhd disk0.vhdx`.
8. Después de volver a instalar el servidor de MABS, puede usar la base de datos DPMDB restaurada para conectarse al servidor de MABS mediante la ejecución del comando `DPMSYNC-RESTOREDB`.
9. Ejecute `DPMSYNC-SYNC` una vez que haya terminado `DPMSYNC-RESTOREDB`.

### <a name="back-up-the-database-by-backing-up-the-mabs-storage-pool"></a>Copia de seguridad de la base de datos mediante la copia de seguridad del bloque de almacenamiento de MABS

> [!NOTE]
> Esta opción es aplicable a MABS con almacenamiento heredado.

Antes de comenzar, deberá ejecutar un script para recuperar la ruta de acceso del punto de montaje del volumen de réplica de MABS para saber qué punto de recuperación contiene la copia de seguridad de MABS. Puede hacerlo después de la replicación inicial con Azure Backup. En el script, reemplace `dplsqlservername%` por el nombre de la instancia de SQL Server que hospeda la base de datos de MABS.

```SQL
Select ag.NetbiosName as ServerName,ds.DataSourceName,vol.MountPointPath
from tbl_IM_DataSource as ds
join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
and vol.Usage =1
and lr.Validity in (1,2)
where ds.datasourcename like '%dpmdb%'
and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB
```

1. En la consola de MABS, seleccione **Protección** > **Crear grupo de protección**.

2. En la página **Seleccionar tipo de grupo de protección**, seleccione **Servidores**.

3. En la página **Seleccionar miembros del grupo**, seleccione la base de datos de MABS. Expanda el elemento del servidor de MABS y seleccione **DPMDB**.

4. En la página **Seleccionar método de protección de datos**, seleccione **Deseo protección a corto plazo mediante disco**. Especifique las opciones de directiva de protección a corto plazo. Se recomienda una duración de retención de dos semanas para las bases de datos de MABS.

#### <a name="recover-the-database"></a>Recuperación de la base de datos

Si el servidor de MABS sigue estando operativo y el bloque de almacenamiento está intacto (como en el caso de los problemas con la consola o el servicio de MABS), copie la base de datos del volumen de réplica o una instantánea como se indica a continuación:

1. Decida desde dónde quiere recuperar la base de datos.

    - Si desea copiar la base de datos desde la última copia de seguridad realizada directamente desde el volumen de réplica de MABS, utilice **mountvol.exe** para asignar una letra de unidad al volumen de réplica con el GUID de la salida del script de SQL. Por ejemplo: `C:\Mountvol X: \\?\Volume{d7a4fd76\-a0a8\-11e2\-8fd3\-001c23cb7375}\`

    - Si desea copiar la base de datos desde un punto de recuperación (instantánea) anterior, debe enumerar todas las instantáneas de la réplica con el GUID de volumen de la salida del script de SQL. Este comando enumera las instantáneas de dicho volumen: `C:\>Vssadmin list shadows /for\=\\?\Volume{d7a4fd76-a0a8-11e2-8fd3-001c23cb7375}\`. Anote la hora de creación y el id. de instantánea a partir de los cuales desea realizar la recuperación.

2. A continuación, use **diskshadow.exe** para montar la instantánea en una letra de unidad X: que no se use mediante el identificador de instantánea para poder copiar los archivos de base de datos.

3. Abra un símbolo del sistema administrativo y ejecute `psexec.exe -s cmd.exe` para iniciar un símbolo del sistema en el contexto de sistema para tener permiso para explorar el volumen de réplica (X:) y copiar los archivos.

4. Realice la implementación continua en la unidad X: y vaya hasta la ubicación de los archivos de base de datos de MABS. Cópielos en una ubicación desde la que sea fácil restaurar. Una vez finalizada la copia, salga de la ventana del símbolo del sistema psexec, ejecute **diskshadow.exe** y anule la exposición del volumen de X:.

5. Ahora puede restaurar los archivos de base de datos mediante SQL Management Studio o ejecutando **DPMSYNC\-RESTOREDB**.

## <a name="back-up-with-native-sql-server-backup-to-a-local-disk"></a>Hacer copias de seguridad con la copia de seguridad nativa de SQL Server en un disco local

Puede realizar una copia de seguridad de la base de datos de MABS en un disco local con copia de seguridad de SQL Server nativa, independiente de MABS.

- Vea una [introducción](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) a las copias de seguridad de SQL Server.

- [Obtenga más información](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service) sobre las copias de seguridad de SQL Server en la nube.

## <a name="back-up-to-a-share-protected-by-mabs"></a>Copia de seguridad en un recurso compartido protegido por MABS

Esta opción de copia de seguridad usa SQL nativo para realizar una copia de seguridad de la base de datos de MABS en un recurso compartido, protege el recurso compartido con MABS y usa las versiones anteriores de Windows VSS para facilitar la restauración.

### <a name="before-you-start"></a>Antes de comenzar

1. En SQL Server, cree una carpeta en una unidad de disco con suficiente espacio disponible para contener una sola copia de una copia de seguridad. Por ejemplo: `C:\MABSBACKUP`.

1. Comparta la carpeta. Por ejemplo, comparta la carpeta `C:\MABSBACKUP` como *DPMBACKUP*.

1. Copie y pegue el comando OSQL siguiente en el Bloc de notas y guárdelo en un archivo denominado `C:\MABSACKUP\bkupdb.cmd`. Asegúrese de que no hay ninguna extensión .txt. Modifique los valores de SQL_Instance_name y DPMDB_NAME para que coincidan con el nombre de instancia y de DPMDB que utiliza el servidor de MABS.

    ```SQL
    OSQL -E -S localhost\SQL_INSTANCE_NAME -Q "BACKUP DATABASE DPMDB_NAME TO DISK='C:\DPMBACKUP\dpmdb.bak' WITH FORMAT"
    ```

1. Con el Bloc de notas, abra el archivo **ScriptingConfig.xml** ubicado en la carpeta `C:\Program Files\Microsoft System Center\DPM\DPM\Scripting` en el servidor de MABS.

1. Modifique **ScriptingConfig.xml** y cambie **DataSourceName=** para que sea la letra de la unidad que contiene el recurso compartido o la carpeta DPMDBBACKUP. Cambie la entrada PreBackupScript a la ruta de acceso completa y el nombre de **bkupdb.cmd** guardado en el paso 3.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ScriptConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns="https://schemas.microsoft.com/2003/dls/ScriptingConfig.xsd">
    <DatasourceScriptConfig DataSourceName="C:">
    <PreBackupScript>C:\MABSDBBACKUP\bkupdb.cmd</PreBackupScript>
    <TimeOut>120</TimeOut>
    </DatasourceScriptConfig>
    </ScriptConfiguration>
    ```

1. Guarde los cambios en **ScriptingConfig.xml**.

1. Proteja la carpeta C:\MABSBACKUP o el recurso compartido `\sqlservername\MABSBACKUP` con MABS y espere a que se cree la réplica inicial. Debe haber un archivo **dpmdb.bak** en la carpeta C:\MABSBACKUP como resultado de la ejecución del script anterior a la copia de seguridad que, a su vez, se copió en la réplica de MABS.

1. Si no habilita la recuperación de autoservicio, necesitará ejecutar algunos pasos adicionales para compartir la carpeta MABSBACKUP en la réplica:

    1. En la consola de MABS > **Protección**, busque el origen de datos MABSBACKUP y selecciónelo. En la sección de detalles, seleccione **Haga clic para ver los detalles** en el vínculo de la ruta de acceso a la réplica y copie la ruta de acceso en el Bloc de notas. Quite la ruta de acceso de origen y conserve la ruta de acceso de destino. La ruta de acceso debe ser similar a la siguiente: `C:\Program Files\Microsoft System Center\DPM\DPM\Volumes\Replica\File System\vol_c9aea05f-31e6-45e5-880c-92ce5fba0a58\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP`.

    2. Coloque un recurso compartido en dicha ruta de acceso con el nombre **MABSSERVERNAME-DPMDB**. Puede usar el comando Net Share siguiente desde un símbolo del sistema administrativo.

        ```cmd
        Net Share MABSSERVERNAME-dpmdb="C:\Program Files\Microsoft System Center\DPM\DPM\Volumes\Replica\File System\vol_c9aea05f-31e6-45e5-880c-92ce5fba0a58\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP"
        ```

### <a name="configure-the-backup"></a>Configuración de la copia de seguridad

Puede realizar una copia de seguridad de la base de datos de MABS como lo haría con cualquier otra base de datos de SQL Server con la copia de seguridad nativa de SQL Server.

- Vea una [introducción](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) a las copias de seguridad de SQL Server.

- [Obtenga más información](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service) sobre las copias de seguridad de SQL Server en la nube.

### <a name="recover-the-mabs-database"></a>Recuperación de la base de datos de MABS

1. Conéctese al recurso compartido `\\MABSServer\MABSSERVERNAME-dpmdb` mediante el Explorador desde cualquier equipo Windows.

2. Haga clic con el botón derecho en el archivo **dpmdb.bak** para ver las propiedades. En la pestaña **Versiones anteriores** aparecen todas las copias de seguridad que puede seleccionar y copiar. También está la última copia de seguridad que aún se encuentra en la carpeta C:\MABSBACKUP, a la que también se puede tener acceso con facilidad.

3. Si necesita mover el disco del bloque de almacenamiento de MABS conectado a una SAN a otro servidor para poder leer el volumen de réplica, o volver a instalar Windows para leer localmente los discos conectados, necesitará saber la ruta de acceso del punto de montaje del volumen de réplica de MABS o el GUID de volumen con antelación para saber qué volumen contiene la copia de seguridad de la base de datos. Puede usar el script de SQL siguiente para extraer esa información en cualquier momento tras la protección inicial, pero antes de que sea necesario realizar la restauración. Reemplace `%dpmsqlservername%` con el nombre de la instancia de SQL Server que hospeda la base de datos.

    ```sql
    Select ag.NetbiosName as
    ServerName,ds.DataSourceName,vol.MountPointPath,vol.GuidName
    from tbl_IM_DataSource as ds
    join tbl_PRM_LogicalReplica as lr on ds.DataSourceId=lr.DataSourceId
    join tbl_AM_Server as ag on ds.ServerId=ag.ServerId
    join tbl_SPM_Volume as vol on lr.PhysicalReplicaId=vol.VolumeSetID
    and vol.Usage =1
    and lr.Validity in (1,2)
    where ds.datasourcename like '%C:\%' -- volume drive letter for DPMBACKUP
    and servername like '%dpmsqlservername%' --netbios name of server hosting DPMDB

    ```

4. Si necesita realizar una recuperación después de mover los discos del bloque de almacenamiento de MABS o una recopilación del servidor de MABS:

    1. Tiene el GUID del volumen, por lo que, si dicho volumen necesita montarse en otra instancia de Windows Server o después de la recopilación de un servidor de MABS, use **mountvol.exe** para asignarle una letra de unidad mediante el GUID del volumen de la salida del script de SQL: `C:\Mountvol X: \\?\Volume{d7a4fd76-a0a8-11e2-8fd3-001c23cb7375}\`.

    2. Vuelva a compartir la carpeta MABSBACKUP del volumen de réplica mediante la letra de unidad y la parte de la ruta de acceso de la réplica que representa la estructura de carpetas.

        ```cmd
        net share SERVERNAME-DPMDB="X:\454d81a0-0d9d-4e07-9617-d49e3f2aa5de\Full\DPMBACKUP"
        ```

    3. Conéctese al recurso compartido `\\SERVERNAME\MABSSERVERNAME-dpmdb` mediante el Explorador desde cualquier equipo Windows.

    4. Haga clic con el botón derecho en el archivo **dpmdb.bak** para ver las propiedades. En la pestaña **Versiones anteriores** aparecen todas las copias de seguridad que puede seleccionar y copiar.

## <a name="using-dpmsync"></a>Uso de DPMSync

**DpmSync** es una herramienta de línea de comandos que permite sincronizar la base de datos de MABS con el estado de los discos del bloque de almacenamiento y con los agentes de protección instalados. DpmSync restaura la base de datos de MABS, la sincroniza con las réplicas en el bloque de almacenamiento, restaura la base de datos de informes y reasigna las réplicas que faltan.

### <a name="parameters"></a>Parámetros

| Parámetro      | Descripción    |
|----------------|-----------------------------|
| **-RestoreDb**                       | Restaura una base de datos de MABS desde una ubicación especificada.|
| **-Sync**                            | Sincroniza las bases de datos restauradas. Debe ejecutar **DpmSync –Sync** después de restaurar las bases de datos. Después de ejecutar **DpmSync –Sync**, es posible que algunas réplicas sigan marcadas como Missing. |
| *Ubicación* de **-DbLoc**                | Identifica la ubicación de la copia de seguridad de la base de datos de MABS.|
| **-InstanceName** <br/>*server\instance*     | Instancia en la que se debe restaurar DPMDB.|
| **-ReallocateReplica**         | Reasigna todos los volúmenes de réplica que faltan sin sincronización. |
| **-DataCopied**                      | Indica que ha completado la carga de datos en los volúmenes de réplica recién asignados. Esto solo se aplica a los equipos cliente. |

**Ejemplo 1:** Para restaurar la base de datos de MABS desde un medio de copia de seguridad local en el servidor de MABS, ejecute el comando siguiente:

```cmd
DpmSync –RestoreDb -DbLoc G:\DPM\Backups\2005\November\DPMDB.bak
```

Después de restaurar la base de datos de MABS, para sincronizar las bases de datos, ejecute el comando siguiente:

```cmd
DpmSync -Sync
```

Después de restaurar y sincronizar la base de datos de MABS y antes de restaurar la réplica, ejecute el siguiente comando para reasignar el espacio en disco para la réplica:

```cmd
DpmSync -ReallocateReplica
```

**Ejemplo 2:** Para restaurar la base de datos de MABS desde una base de datos remota, ejecute el siguiente comando en el equipo remoto:

```cmd
DpmSync –RestoreDb -DbLoc G:\DPM\Backups\2005\November\DPMDB.bak –InstanceName contoso\ms$dpm
```

Después de restaurar la base de datos de MABS, para sincronizar las bases de datos, ejecute el comando siguiente en el servidor de MABS:

```cmd
DpmSync -Sync
```

Después de restaurar y sincronizar la base de datos de MABS y antes de restaurar la réplica, ejecute el siguiente comando en el servidor de MABS para reasignar el espacio en disco para la réplica:

```cmd
DpmSync -ReallocateReplica
```

## <a name="next-steps"></a>Pasos siguientes

- [Matriz de compatibilidad de MABS](backup-support-matrix-mabs-dpm.md)
- [PREGUNTAS MÁS FRECUENTES SOBRE MABS](backup-azure-dpm-azure-server-faq.yml)