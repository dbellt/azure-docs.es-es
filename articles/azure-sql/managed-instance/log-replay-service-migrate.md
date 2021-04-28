---
title: Migración de bases de datos a SQL Managed Instance mediante el servicio de reproducción de registros
description: Aprenda a migrar bases de datos de SQL Server a SQL Managed Instance mediante el servicio de reproducción de registros.
services: sql-database
ms.service: sql-managed-instance
ms.custom: seo-lt-2019, sqldbrb=1, devx-track-azurecli
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: sstein
ms.date: 03/31/2021
ms.openlocfilehash: 522f4ec2f28f9d8975a8a7a7b10e435f602af855
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484037"
---
# <a name="migrate-databases-from-sql-server-to-sql-managed-instance-by-using-log-replay-service-preview"></a>Migración de bases de datos de SQL Server a SQL Managed Instance mediante el servicio de reproducción de registros (versión preliminar)
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica cómo configurar manualmente la migración de bases de datos de SQL Server 2008-2019 a Azure SQL Managed Instance mediante el servicio de reproducción de registros (LRS) actualmente en versión preliminar pública. Se trata de un servicio en la nube habilitado para SQL Managed Instance y basado en la tecnología de trasvase de registros de SQL Server. 

[Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md) y LRS usan la misma tecnología de migración subyacente y las mismas API. LRS permite realizar migraciones personalizadas complejas y utilizar arquitectura híbrida entre la instancia de SQL Server local y SQL Managed Instance.

## <a name="when-to-use-log-replay-service"></a>Cuándo usar el servicio de reproducción de registros

Si no puede usar Azure Database Migration Service para la migración, puede usar LRS directamente con PowerShell, cmdlets de la CLI de Azure o API para compilar y organizar manualmente las migraciones de base de datos a SQL Managed Instance. 

Puede considerar el uso de LRS en los siguientes casos:
- Necesita más control sobre el proyecto de migración de base de datos.
- Existe poca tolerancia al tiempo de inactividad en la migración total.
- El archivo ejecutable de Database Migration Service no se puede instalar en el entorno.
- El archivo ejecutable de Database Migration Service no tiene acceso a los archivos de copia de seguridad de la base de datos.
- No dispone de acceso al sistema operativo del host o no tiene privilegios de administrador.
- No puede abrir puertos de red entre su entorno y Azure.
- Limitación de la red o problemas de bloqueo del proxy en el entorno.
- Las copias de seguridad se almacenan directamente en Azure Blob Storage con la opción `TO URL`.
- Necesita usar copias de seguridad diferenciales.

> [!NOTE]
> Se recomienda automatizar la migración de bases de datos de SQL Server a SQL Managed Instance mediante Database Migration Service. Este servicio emplea el mismo servicio en la nube LRS en el back-end con el trasvase de registros en modo `NORECOVERY`. Considere la posibilidad de usar LRS manualmente para orquestar las migraciones cuando Database Migration Service no sea totalmente compatible con sus escenarios.

## <a name="how-it-works"></a>Funcionamiento

Para crear una solución personalizada que utilice LRS para migrar bases de datos a la nube, se necesitan varios pasos de orquestación, tal como se muestra en el diagrama y en la tabla incluidos más adelante en esta sección.

La migración consiste en realizar copias de seguridad completas de la base de datos en SQL Server con `CHECKSUM` habilitado y copiar después los archivos de copia de seguridad en Azure Blob Storage. LRS se usa para restaurar los archivos de copia de seguridad de Blob Storage a SQL Managed Instance. Blob Storage es el almacenamiento intermedio entre SQL Server y SQL Managed Instance.

LRS supervisa Blob Storage para comprobar si se han agregado nuevas copias de seguridad diferenciales o de registros después de haber restaurado la copia de seguridad completa. Si las hay, restaura automáticamente esos nuevos archivos. Puede usar el servicio para supervisar el progreso de los archivos de copia de seguridad que se restauran en SQL Managed Instance y detener el proceso si es necesario.

LRS no requiere que los archivos de copia de seguridad sigan una convención de nomenclatura determinada. Examina todos los archivos que se incluyen en Blob Storage y crea la cadena de copia de seguridad simplemente a partir de la lectura de los encabezados de archivo. Las bases de datos se encuentran en un estado de restauración durante el proceso de migración. Se restauran en modo [NORECOVERY](/sql/t-sql/statements/restore-statements-transact-sql#comparison-of-recovery-and-norecovery), por lo que no se pueden usar para leer ni escribir hasta que se complete el proceso de migración. 

Si va a migrar varias bases de datos, necesitará hacer lo siguiente:
 
- Colocar las copias de seguridad de cada base de datos en una carpeta independiente en Blob Storage.
- Iniciar LRS por separado para cada base de datos.
- Especificar diferentes rutas de acceso a las distintas carpetas de Blob Storage. 

Puede iniciar LRS en modo *autocompletar* o *continuo*. Cuando se inicia en modo autocompletar, la migración finalizará automáticamente cuando se haya restaurado el último de los archivos de copia de seguridad especificados. Cuando se inicia en modo continuo, el servicio restaura continuamente los nuevos archivos de copia de seguridad agregados y la migración solo se completa con la migración total manual. 

Se recomienda ejecutar la migración total manual después de haber realizado la copia de seguridad del final del registro y que esta copia se muestre como restaurada en SQL Managed Instance. En el paso final de la migración total, la base de datos pasará a estar en línea y disponible para la lectura y la escritura en SQL Managed Instance.

Una vez detenido LRS, ya sea automáticamente con autocompletar o manualmente con la migración total, no se puede reanudar el proceso de restauración de una base de datos que pasó a estar en línea en SQL Managed Instance. Para restaurar archivos de copia de seguridad adicionales una vez finalizada la migración con autocompletar o con la migración total, debe eliminar la base de datos. También debe restaurar la cadena completa de copia de seguridad desde cero reiniciando LRS.

:::image type="content" source="./media/log-replay-service-migrate/log-replay-service-conceptual.png" alt-text="Diagrama en el que se explican los pasos de orquestación del servicio de reproducción de registros para SQL Managed Instance." border="false":::
    
| Operación | Detalles |
| :----------------------------- | :------------------------- |
| **1. Realice la copia de seguridad de las bases de datos de SQL Server en Blob Storage**. | Realice copias de seguridad completas, diferenciales y de registros de SQL Server en un contenedor de Blob Storage mediante [Azcopy](../../storage/common/storage-use-azcopy-v10.md) o el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/). <br /><br />Use cualquier nombre de archivo. En LRS no hace falta seguir una convención de nomenclatura de archivos específica.<br /><br />Si migra varias bases de datos, necesitará una carpeta independiente para cada una de ellas. |
| **2. Inicie LRS en la nube**. | Puede reiniciar el servicio con una selección de cmdlets: PowerShell ([start-azsqlinstancedatabaselogreplay](/powershell/module/az.sql/start-azsqlinstancedatabaselogreplay)) o la CLI de Azure ([cmdlets az_sql_midb_log_replay_start](/cli/azure/sql/midb/log-replay#az_sql_midb_log_replay_start)). <br /><br /> Inicie LRS por separado para cada base de datos que apunte a una carpeta de copia de seguridad en Blob Storage. <br /><br /> Una vez iniciado el servicio, realizará copias de seguridad del contenedor de Blob Storage y comenzará a restaurarlas en SQL Managed Instance.<br /><br /> Si inició LRS en modo continuo, después de restaurar todas las copias de seguridad inicialmente cargadas, el servicio comprobará si se han cargado nuevos archivos en la carpeta. El servicio aplicará continuamente registros basados en la cadena de números de secuencia de registro (LSN) hasta que se detenga. |
| **2.1. Supervise el progreso de la operación**. | Puede supervisar el progreso de la operación de restauración con una selección de cmdlets: PowerShell ([get-azsqlinstancedatabaselogreplay](/powershell/module/az.sql/get-azsqlinstancedatabaselogreplay)) o la CLI de Azure ([cmdlets az_sql_midb_log_replay_show](/cli/azure/sql/midb/log-replay#az_sql_midb_log_replay_show)). |
| **2.2. Detenga la operación si es necesario**. | Si necesita detener el proceso de migración, dispone de varios cmdlets: PowerShell ([stop-azsqlinstancedatabaselogreplay](/powershell/module/az.sql/stop-azsqlinstancedatabaselogreplay)) o la CLI de Azure ([az_sql_midb_log_replay_stop](/cli/azure/sql/midb/log-replay#az_sql_midb_log_replay_stop)). <br /><br /> Al detener la operación se eliminará la base de datos que se está restaurando en SQL Managed Instance. Después de detener una operación, no se puede reanudar el servicio LRS para una base de datos. Deberá reiniciar el proceso de migración desde el principio. |
| **3. Realice la migración total a la nube cuando esté listo**. | Detenga la aplicación y la carga de trabajo. Realice la última copia de seguridad del final del registro y cárguela en Azure Blob Storage.<br /><br /> Complete la migración total iniciando una operación `complete` de LRS con una selección de cmdlets: PowerShell ([complete-azsqlinstancedatabaselogreplay](/powershell/module/az.sql/complete-azsqlinstancedatabaselogreplay)) o la CLI de Azure ([az_sql_midb_log_replay_complete](/cli/azure/sql/midb/log-replay#az_sql_midb_log_replay_complete)). Esta operación detendrá LRS y hará que la base de datos pase a estar en línea para la lectura y escritura en SQL Managed Instance.<br /><br /> Vuelva a apuntar la cadena de conexión de la aplicación de SQL Server a SQL Managed Instance. Deberá organizar este paso por sí mismo, ya sea a través de un cambio manual en la cadena de conexión de la aplicación o automáticamente (si la aplicación puede, por ejemplo, leer la cadena de conexión de una propiedad o una base de datos). |

## <a name="requirements-for-getting-started"></a>Requisitos para comenzar

### <a name="sql-server-side"></a>En lo que respecta a SQL Server
- SQL Server 2008-2019
- Copia de seguridad completa de las bases de datos (uno o varios archivos)
- Copia de seguridad diferencial (uno o varios archivos)
- Copia de seguridad de registros (sin dividir en el caso del archivo de registro de transacciones)
- `CHECKSUM` habilitado para copias de seguridad (obligatorio)

### <a name="azure-side"></a>En lo que respecta a Azure
- El módulo Az.SQL de PowerShell versión 2.16.0 o posterior ([instalado](https://www.powershellgallery.com/packages/Az.Sql/) o con acceso desde [Azure Cloud Shell](/azure/cloud-shell/))
- CLI de Azure versión 2.19.0 o posterior ([instalada](/cli/azure/install-azure-cli))
- Contenedor de Azure Blob Storage aprovisionado
- Token de seguridad de firma de acceso compartido (SAS) con permisos de lectura y enumeración generados para el contenedor de Blob Storage

### <a name="migration-of-multiple-databases"></a>Migración de varias bases de datos
Debe colocar los archivos de copia de seguridad de diferentes bases de datos en carpetas independientes de Blob Storage.

Inicie LRS por separado para cada base de datos, apuntando a la carpeta correspondiente en Blob Storage. LRS puede admitir hasta 100 procesos de restauración simultáneos por cada instancia administrada.

### <a name="azure-rbac-permissions"></a>Permisos de Azure RBAC
Para ejecutar LRS mediante los clientes proporcionados, se necesita uno de los siguientes roles de Azure:
- Rol de propietario de la suscripción
- Rol de [colaborador de instancia administrada de SQL](../../role-based-access-control/built-in-roles.md#sql-managed-instance-contributor)
- Rol personalizado con el siguiente permiso: `Microsoft.Sql/managedInstances/databases/*`

## <a name="best-practices"></a>Procedimientos recomendados

Se recomiendan las siguientes prácticas recomendadas:
- Ejecute [Data Migration Assistant](/sql/dma/dma-overview) para asegurarse de que las bases de datos están listas para la migración a SQL Managed Instance. 
- Divida las copias de seguridad completas y diferenciales en varios archivos, en lugar de usar uno solo.
- Habilite la compresión de copias de seguridad.
- Use Cloud Shell para ejecutar los scripts, ya que siempre se actualizarán con los últimos cmdlets publicados.
- Planee de forma que la migración se complete en un plazo de 47 horas después de iniciar LRS. Este período de gracia impide la instalación de revisiones de software administradas por el sistema.

> [!IMPORTANT]
> - La base de datos que se está restaurando mediante LRS no se puede usar mientras no haya finalizado el proceso de migración. 
> - LRS no admite el acceso de solo lectura a las bases de datos durante la migración.
> - Una vez finalizada la migración, termina el proceso de migración, porque LRS no admite la reanudación del proceso de restauración.

## <a name="steps-to-execute"></a>Pasos para ejecutar

### <a name="make-backups-of-sql-server"></a>Realización de copias de seguridad de SQL Server

Puede realizar copias de seguridad de SQL Server mediante una de estas dos opciones:

- Realice la copia de seguridad en el almacenamiento en disco local y, a continuación, cargue los archivos en Azure Blob Storage, si su entorno restringe las copias de seguridad directas a Blob Storage.
- Realice una copia de seguridad directamente en Blob Storage con la opción `TO URL` en T-SQL, si su entorno y los procedimientos de seguridad lo permiten. 

Establezca las bases de datos que quiere migrar en el modo de recuperación completa para permitir copias de seguridad de registros.

```SQL
-- To permit log backups, before the full database backup, modify the database to use the full recovery
USE master
ALTER DATABASE SampleDB
SET RECOVERY FULL
GO
```

Para realizar manualmente copias de seguridad completas, diferenciales y de registros de la base de datos en el almacenamiento local, use los siguientes scripts T-SQL de ejemplo. Asegúrese de que la opción `CHECKSUM` está habilitada, ya que se trata de un requisito obligatorio de LRS.

```SQL
-- Example of how to make a full database backup to the local disk
BACKUP DATABASE [SampleDB]
TO DISK='C:\BACKUP\SampleDB_full.bak'
WITH INIT, COMPRESSION, CHECKSUM
GO

-- Example of how to make a differential database backup to the local disk
BACKUP DATABASE [SampleDB]
TO DISK='C:\BACKUP\SampleDB_diff.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
GO

-- Example of how to make a transactional log backup to the local disk
BACKUP LOG [SampleDB]
TO DISK='C:\BACKUP\SampleDB_log.trn'
WITH COMPRESSION, CHECKSUM
GO
```

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Azure Blob Storage se utiliza como almacenamiento intermedio para los archivos de copia de seguridad entre SQL Server y SQL Managed Instance. Para crear una cuenta de almacenamiento y un contenedor de blobs dentro de la cuenta de almacenamiento, siga estos pasos:

1. [Crear una cuenta de almacenamiento](../../storage/common/storage-account-create.md?tabs=azure-portal).
2. [Crear un contenedor de blobs](../../storage/blobs/storage-quickstart-blobs-portal.md) dentro de la cuenta de almacenamiento.

### <a name="copy-backups-from-sql-server-to-blob-storage"></a>Creación de copias de seguridad de SQL Server a Blob Storage

Cuando migra bases de datos a una instancia administrada mediante LRS, puede usar los métodos siguientes para cargar las copias de seguridad en Blob Storage:
- Usar la funcionalidad [BACKUP TO URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) nativa de SQL Server
- Usar [Azcopy](../../storage/common/storage-use-azcopy-v10.md) o el [Explorador de Azure Storage](https://azure.microsoft.com/en-us/features/storage-explorer) para cargar las copias de seguridad en un contenedor de blobs
- Usar el Explorador de Storage en Azure Portal

### <a name="make-backups-from-sql-server-directly-to-blob-storage"></a>Creación de copias de seguridad de SQL Server directamente en Blob Storage
Si las directivas corporativas y de red lo permiten, una alternativa es crear las copias de seguridad de SQL Server directamente en Blob Storage mediante la opción [BACKUP TO URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) nativa de SQL Server. Si puede utilizar esta opción, no necesita realizar copias de seguridad en el almacenamiento local y cargarlas en Blob Storage.

Como primer paso, esta operación requiere que genere un token de autenticación de SAS para Blob Storage y que importe el token a SQL Server. El segundo paso consiste en crear copias de seguridad con la opción `TO URL` de T-SQL. Asegúrese de que todas las copias de seguridad se crean con la opción `CHEKSUM` habilitada.

A modo de referencia, el siguiente código de ejemplo realiza copias de seguridad en Blob Storage. En este ejemplo no se incluyen instrucciones sobre cómo importar el token de SAS. Puede encontrar instrucciones detalladas, incluidas indicaciones para generar e importar el token de SAS en SQL Server, en el tutorial [Usar el servicio Azure Blob Storage con SQL Server 2016](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016#1---create-stored-access-policy-and-shared-access-storage). 

```SQL
-- Example of how to make a full database backup to a URL
BACKUP DATABASE [SampleDB]
TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/SampleDB_full.bak'
WITH INIT, COMPRESSION, CHECKSUM
GO
-- Example of how to make a differential database backup to a URL
BACKUP DATABASE [SampleDB]
TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/SampleDB_diff.bak'  
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
GO

-- Example of how to make a transactional log backup to a URL
BACKUP LOG [SampleDB]
TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/SampleDB_log.trn'  
WITH COMPRESSION, CHECKSUM
```

### <a name="generate-a-blob-storage-sas-authentication-token-for-lrs"></a>Generación de un token de autenticación de SAS de Blob Storage para LRS

Azure Blob Storage se utiliza como almacenamiento intermedio para los archivos de copia de seguridad entre SQL Server y SQL Managed Instance. Debe generar un token de autenticación de SAS, solo con permisos de enumeración y lectura, para LRS. El token permitirá que LRS acceda a Blob Storage y use los archivos de copia de seguridad para restaurarlos en SQL Managed Instance. 

Siga estos pasos para generar el token:

1. Acceda al Explorador de Storage desde Azure Portal.
2. Expanda **Contenedores de blobs**.
3. Haga clic con el botón derecho en el contenedor de blobs y seleccione **Obtener firma de acceso compartido**.

   :::image type="content" source="./media/log-replay-service-migrate/lrs-sas-token-01.png" alt-text="Captura de pantalla que muestra las selecciones para generar un token de autenticación de S A S.":::

4. Seleccione el período de expiración del token. Asegúrese de que el token sea válido mientras dure la migración.
5. Seleccione la zona horaria del token (UTC o la hora local).
    
   > [!IMPORTANT]
   > Puede que la zona horaria del token y de la instancia administrada no coincidan. Asegúrese de que el token de SAS tenga la validez temporal adecuada teniendo en cuenta las zonas horarias. Si es posible, establezca la zona horaria en una hora anterior y posterior a la ventana de migración planeada.
6. Seleccione únicamente los permisos **Lectura** y **Lista**.

   > [!IMPORTANT]
   > No seleccione ningún otro permiso. Si lo hace, LRS no se iniciará. Este requisito de seguridad es así por naturaleza.
7. Seleccione **Crear**.

   :::image type="content" source="./media/log-replay-service-migrate/lrs-sas-token-02.png" alt-text="Captura de pantalla que muestra las selecciones de la expiración del token de S A S, la zona horaria y los permisos, junto con el botón Crear.":::

La autenticación de SAS se genera con la validez de tiempo que ha especificado. Necesitará la versión del URI del token, tal y como se muestra en la siguiente captura de pantalla.

:::image type="content" source="./media/log-replay-service-migrate/lrs-generated-uri-token.png" alt-text="Captura de pantalla que muestra un ejemplo de la versión de U R I de un token de S A S.":::

### <a name="copy-parameters-from-the-sas-token"></a>Copia de parámetros del token de SAS

Antes de usar el token de SAS para iniciar LRS, debe comprender su estructura. El URI del token de SAS generado consta de dos partes separadas por un signo de interrogación (`?`), tal como se muestra en este ejemplo:

:::image type="content" source="./media/log-replay-service-migrate/lrs-token-structure.png" alt-text="U R I de ejemplo de un token de S A S generado para el servicio de reproducción de registros." border="false":::

La primera parte, desde `https://` hasta el signo de interrogación (`?`), se usa en el parámetro `StorageContainerURI` que se suministra como entrada a LRS. Proporciona a LRS información sobre la carpeta donde se almacenan los archivos de copia de seguridad de base de datos.

La segunda parte, que empieza después del signo de interrogación (`?`) y llega hasta el final de la cadena, es el parámetro `StorageContainerSasToken`. Este es el token de autenticación firmado real y es válido durante el tiempo especificado. No es necesario que esta parte comience con `sp=` como en el ejemplo. Cada caso puede ser diferente.

Copie los parámetros de la manera siguiente:

1. Copie la primera parte del token, desde `https://` hasta el signo de interrogación (`?`). Úsela como parámetro `StorageContainerUri` en PowerShell o en la CLI de Azure para iniciar LRS.

   :::image type="content" source="./media/log-replay-service-migrate/lrs-token-uri-copy-part-01.png" alt-text="Captura de pantalla que muestra la copia de la primera parte del token.":::

2. Copie la segunda parte del token, empezando después del signo de interrogación (`?`) y hasta el final de la cadena. Úsela como parámetro `StorageContainerSasToken` en PowerShell o en la CLI de Azure para iniciar LRS.

   :::image type="content" source="./media/log-replay-service-migrate/lrs-token-uri-copy-part-02.png" alt-text="Captura de pantalla que muestra la copia de la segunda parte del token.":::
   
> [!NOTE]
> No incluya el signo de interrogación al copiar ninguna de las partes del token.

### <a name="log-in-to-azure-and-select-a-subscription"></a>Inicio de sesión en Azure y selección de la suscripción

Use el siguiente cmdlet de PowerShell para iniciar sesión en Azure:

```powershell
Login-AzAccount
```

Seleccione la suscripción en la que se encuentre la instancia administrada con el siguiente cmdlet de PowerShell:

```powershell
Select-AzSubscription -SubscriptionId <subscription ID>
```

## <a name="start-the-migration"></a>Inicio de la migración

La migración se inicia al iniciar LRS. Puede iniciar el servicio en modo autocompletar o continuo. 

Cuando se utiliza el modo autocompletar, la migración finalizará automáticamente cuando se haya restaurado el último de los archivos de copia de seguridad especificados. Esta opción requiere el comando de inicio para especificar el nombre de archivo del último archivo de copia de seguridad. 

Cuando se utiliza el modo continuo, el servicio seguirá restaurando continuamente los nuevos archivos de copia de seguridad que se agreguen. La migración solo finalizará cuando se haga la migración total manual. 

### <a name="start-lrs-in-autocomplete-mode"></a>Inicio de LRS en modo autocompletar

Para iniciar LRS en modo autocompletar, use los siguientes comandos de PowerShell o de la CLI de Azure. Especifique el nombre del último archivo de copia de seguridad con el parámetro `-LastBackupName`. Tras restaurar el último de los archivos de copia de seguridad especificados, el servicio iniciará automáticamente una migración total.

A continuación se muestra un ejemplo del inicio de LRS en modo autocompletar con PowerShell:

```PowerShell
Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "ResourceGroup01" `
    -InstanceName "ManagedInstance01" `
    -Name "ManagedDatabaseName" `
    -Collation "SQL_Latin1_General_CP1_CI_AS" `
    -StorageContainerUri "https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>" `
    -StorageContainerSasToken "sv=2019-02-02&ss=b&srt=sco&sp=rl&se=2023-12-02T00:09:14Z&st=2019-11-25T16:09:14Z&spr=https&sig=92kAe4QYmXaht%2Fgjocqwerqwer41s%3D" `
    -AutoCompleteRestore `
    -LastBackupName "last_backup.bak"
```

A continuación se muestra un ejemplo del inicio de LRS en modo autocompletar con la CLI de Azure:

```CLI
az sql midb log-replay start -g mygroup --mi myinstance -n mymanageddb -a --last-bn "backup.bak"
    --storage-uri "https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>"
    --storage-sas "sv=2019-02-02&ss=b&srt=sco&sp=rl&se=2023-12-02T00:09:14Z&st=2019-11-25T16:09:14Z&spr=https&sig=92kAe4QYmXaht%2Fgjocqwerqwer41s%3D"
```

### <a name="start-lrs-in-continuous-mode"></a>Inicio de LRS en modo continuo

A continuación se muestra un ejemplo del inicio de LRS en modo continuo con PowerShell:

```PowerShell
Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "ResourceGroup01" `
    -InstanceName "ManagedInstance01" `
    -Name "ManagedDatabaseName" `
    -Collation "SQL_Latin1_General_CP1_CI_AS" -StorageContainerUri "https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>" `
    -StorageContainerSasToken "sv=2019-02-02&ss=b&srt=sco&sp=rl&se=2023-12-02T00:09:14Z&st=2019-11-25T16:09:14Z&spr=https&sig=92kAe4QYmXaht%2Fgjocqwerqwer41s%3D"
```

A continuación se muestra un ejemplo del inicio de LRS en modo continuo con la CLI de Azure:

```CLI
az sql midb log-replay start -g mygroup --mi myinstance -n mymanageddb
    --storage-uri "https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>"
    --storage-sas "sv=2019-02-02&ss=b&srt=sco&sp=rl&se=2023-12-02T00:09:14Z&st=2019-11-25T16:09:14Z&spr=https&sig=92kAe4QYmXaht%2Fgjocqwerqwer41s%3D"
```

Los clientes de PowerShell y de la CLI para iniciar LRS en modo continuo son sincrónicos. Esto significa que esperarán a que la respuesta de API informe sobre si el trabajo se ha iniciado o no correctamente. 

Durante esta espera, el comando no devolverá el control al símbolo del sistema. Si está creando un script de la experiencia de migración y necesita que el comando de inicio de LRS devuelva el control inmediatamente para continuar con el resto del script, puede ejecutar PowerShell como trabajo en segundo plano con el modificador `-AsJob`. Por ejemplo:

```PowerShell
$lrsjob = Start-AzSqlInstanceDatabaseLogReplay <required parameters> -AsJob
```

Cuando se inicia un trabajo en segundo plano, se devuelve inmediatamente un objeto de trabajo, incluso aunque el trabajo tarde mucho en finalizar. Puede seguir trabajando en la sesión sin interrupción mientras se ejecuta el trabajo. Para más información sobre la ejecución de PowerShell como un trabajo en segundo plano, consulte la documentación sobre el [cmdlet Start-Job de PowerShell](/powershell/module/microsoft.powershell.core/start-job#description).

Del mismo modo, para iniciar un comando de la CLI de Azure en Linux como un proceso en segundo plano, use el signo de "y" comercial (`&`) al final del comando de inicio de LRS:

```CLI
az sql midb log-replay start <required parameters> &
```

> [!IMPORTANT]
> Después de iniciar LRS, todas las revisiones de software administradas por el sistema se detienen durante 47 horas. Tras este periodo, la siguiente revisión de software automatizada detendrá automáticamente LRS. Si esto sucede, no podrá reanudar la migración y deberá volver a iniciarla desde el principio. 

## <a name="monitor-the-migration-progress"></a>Supervisión del progreso de la migración

Para supervisar el progreso de la migración mediante PowerShell, use el siguiente comando:

```PowerShell
Get-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "ResourceGroup01" `
    -InstanceName "ManagedInstance01" `
    -Name "ManagedDatabaseName"
```

Para supervisar el progreso de la migración mediante la CLI de Azure, use el siguiente comando:

```CLI
az sql midb log-replay show -g mygroup --mi myinstance -n mymanageddb
```

## <a name="stop-the-migration"></a>Detención de la migración

Si necesita detener la migración, use los siguientes cmdlets. Al detener la migración, se elimina la base de datos de restauración en SQL Managed Instance, por lo que no es posible reanudar después la migración.

Para detener el proceso de migración mediante PowerShell, use el siguiente comando:

```PowerShell
Stop-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "ResourceGroup01" `
    -InstanceName "ManagedInstance01" `
    -Name "ManagedDatabaseName"
```

Para detener el proceso de migración mediante la CLI de Azure, use el siguiente comando:

```CLI
az sql midb log-replay stop -g mygroup --mi myinstance -n mymanageddb
```

## <a name="complete-the-migration-continuous-mode"></a>Finalización de la migración (modo continuo)

Si ha iniciado LRS en modo continuo, una vez que se haya asegurado de que se han restaurado todas las copias de seguridad, deberá iniciar la migración total para finalizar la migración. Tras completar la migración total, la base de datos se migrará y estará lista para el acceso de lectura y escritura.

Para finalizar el proceso de migración en modo continuo de LRS mediante PowerShell, use el siguiente comando:

```PowerShell
Complete-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "ResourceGroup01" `
-InstanceName "ManagedInstance01" `
-Name "ManagedDatabaseName" `
-LastBackupName "last_backup.bak"
```

Para finalizar el proceso de migración en modo continuo de LRS mediante la CLI de Azure, use el siguiente comando:

```CLI
az sql midb log-replay complete -g mygroup --mi myinstance -n mymanageddb --last-backup-name "backup.bak"
```

## <a name="functional-limitations"></a>Limitaciones funcionales

A continuación se indican las limitaciones funcionales de LRS:
- La base de datos que se está restaurando no se puede usar para el acceso de solo lectura durante el proceso de migración.
- Las revisiones de software administradas por el sistema se bloquean durante 47 horas después de iniciar LRS. Tras la expiración de esta ventana de tiempo, la siguiente actualización de software detendrá LRS. Después, deberá reiniciar LRS desde cero.
- LRS requiere que las copias de seguridad de las bases de datos de SQL Server se realicen con la opción `CHECKSUM` habilitada.
- El token de SAS que utilizará LRS debe generarse para todo el contenedor de Azure Blob Storage y solo debe tener permisos de lectura y enumeración.
- Los archivos de copia de seguridad de diferentes bases de datos se deben colocar en distintas carpetas de Blob Storage.
- LRS debe iniciarse por separado para cada base de datos que apunte a distintas carpetas con archivos de copia de seguridad en Blob Storage.
- LRS puede admitir hasta 100 procesos de restauración simultáneos por cada instancia administrada.

## <a name="troubleshooting"></a>Solución de problemas

Una vez que se inicia LRS, use el cmdlet de supervisión (`get-azsqlinstancedatabaselogreplay` o `az_sql_midb_log_replay_show`) para ver el estado de la operación. Si LRS no se inicia después de un tiempo y se produce un error, compruebe los problemas más comunes:

- ¿Existe una base de datos en SQL Managed Instance que tenga el mismo nombre que la que está intentando migrar desde SQL Server? Para resolver este conflicto, cambie el nombre de una de las bases de datos.
- ¿Se realizó la copia de seguridad de la base de datos en SQL Server con la opción `CHECKSUM`?
- ¿Los permisos en el token de SAS son solo de lectura y enumeración para LRS?
- ¿Ha copiado el token de SAS para LRS que figura tras el signo de interrogación (`?`), con un contenido similar al siguiente: `sv=2020-02-10...`? 
- ¿El tiempo de validez del token de SAS es aplicable a la ventana de tiempo de inicio y finalización de la migración? Podría haber discrepancias debido a las distintas zonas horarias que se usan en SQL Managed Instance y el token de SAS. Intente volver a generar el token de SAS ampliando la ventana de tiempo de validez del token antes y después de la fecha actual.
- ¿Se han escrito correctamente los nombres de la base de datos, del grupo de recursos y de la instancia administrada?
- Si inició LRS en modo autocompletar, ¿era válido el nombre del último archivo de copia de seguridad especificado?

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre la [migración de SQL Server a SQL Managed Instance](../migration-guides/managed-instance/sql-server-to-managed-instance-guide.md).
- Más información sobre las [diferencias entre SQL Server y SQL Managed Instance](transact-sql-tsql-differences-sql-server.md).
- Más información sobre los [procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs).
