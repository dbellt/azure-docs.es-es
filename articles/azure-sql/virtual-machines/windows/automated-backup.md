---
title: Automated Backup v2 para máquinas virtuales Azure con SQL Server 2016/2017 | Microsoft Docs
description: En este artículo se explica la característica Automated Backup para SQL Server 2016/2017 en máquinas virtuales que se ejecutan en Azure. Este artículo trata exactamente sobre las máquinas virtuales que usan Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: ebd23868-821c-475b-b867-06d4a2e310c7
ms.service: virtual-machines-sql
ms.subservice: backup
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/03/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 557d43829d8e235611d1e75cddd8f941e43433ee
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666212"
---
# <a name="automated-backup-v2-for-azure-virtual-machines-resource-manager"></a>Automated Backup v2 para Azure Virtual Machines (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [SQL Server 2014](automated-backup-sql-2014.md)
> * [SQL Server 2016 +](automated-backup.md)

La copia de seguridad automatizada v2 configura automáticamente la [copia de seguridad administrada en Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) para todas las bases de datos nuevas y existentes en una máquina virtual de Azure que ejecuta las ediciones Standard, Enterprise o Developer de SQL Server 2016 o versiones posteriores. Esto le permite configurar copias de seguridad de datos normales que utilizan el almacenamiento de blobs de Azure. Automated Backup v2 depende de la [Extensión del agente de IaaS (infraestructura como servicio) de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Requisitos previos
Para utilizar Automated Backup v2, revise los siguientes requisitos previos:

**Sistema operativo**:

- Windows Server 2012 R2 o superior

**Edición/versión de SQL Server**:

- SQL Server 2016 o posterior: Developer, Standard o Enterprise

> [!NOTE]
> Para SQL Server 2014, consulte [Copia de seguridad automatizada para SQL Server 2014](automated-backup-sql-2014.md).

**Configuración de base de datos**:

- Las bases de datos de _usuario_ de destino deben utilizar el modelo de recuperación completa. Las bases de datos del sistema no tienen que usar el modelo de recuperación completa. Aun así, sí debe usar el modelo de recuperación completa si necesita realizar copias de seguridad de registros para el modelo o para MSDB. Para obtener más información sobre el impacto del modelo de recuperación completa en las copias de seguridad, vea [Copia de seguridad en el modelo de recuperación completa](/previous-versions/sql/sql-server-2008-r2/ms190217(v=sql.105)). 
- La máquina virtual de SQL Server se ha registrado con la extensión Agente de IaaS de SQL en [modo de administración completa](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full). 
-  La copia de seguridad automatizada se basa en la [extensión de agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md) completa. Como tal, solo se admite en las bases de datos de destino de la instancia predeterminada o en una única instancia con nombre. Si no hay ninguna instancia predeterminada y hay varias instancias con nombre, se producirá un error en la extensión IaaS de SQL y la copia de seguridad automatizada no funcionará. 

## <a name="settings"></a>Configuración
En la siguiente tabla se describen las opciones que pueden configurarse para Automated Backup v2. Los pasos de configuración reales varían si usa Azure Portal o comandos de Windows PowerShell de Azure.

### <a name="basic-settings"></a>Configuración básica

| Configuración | Intervalo (valor predeterminado) | Descripción |
| --- | --- | --- |
| **Automated Backup** | Habilitar/deshabilitar (deshabilitado) | Habilita o deshabilita Automated Backup para una máquina virtual de Azure que ejecuta SQL Server 2016/2017 Developer, Standard o Enterprise. |
| **Período de retención** | 1-30 días (30 días) | El número de días para retener copias de seguridad. |
| **Storage Account** | Cuenta de almacenamiento de Azure | Una cuenta de almacenamiento de Azure que usar para almacenar archivos de Automated Backup en el almacenamiento de blobs. Se crea un contenedor en esta ubicación para guardar todos los archivos de copia de seguridad. La convención de nomenclatura de los archivos de copia de seguridad agrega la fecha, la hora y el GUID de base de datos. |
| **Cifrado** |Habilitar/deshabilitar (deshabilitado) | Habilita o deshabilita el cifrado. Cuando se habilita el cifrado, los certificados usados para restaurar la copia de seguridad se ubican en la cuenta de almacenamiento especificada. Usa el mismo contenedor **automatic backup** con la misma convención de nomenclatura. Si la contraseña cambia, se genera un nuevo certificado con esa contraseña, pero el certificado antiguo permanece para restaurar copias de seguridad anteriores. |
| **Contraseña** |Texto de contraseña | Una contraseña para claves de cifrado. Esta contraseña solo es necesaria si se habilita el cifrado. Para restaurar una copia de seguridad cifrada, debe disponer de la contraseña correcta y del certificado relacionado que se usó en el momento en el que se realizó la copia de seguridad. |

### <a name="advanced-settings"></a>Configuración avanzada

| Configuración | Intervalo (valor predeterminado) | Descripción |
| --- | --- | --- |
| **Copias de seguridad de bases de datos del sistema** | Habilitar/deshabilitar (deshabilitado) | Cuando se habilita, esta característica realiza también una copia de seguridad de las bases de datos del sistema: maestra, MSDB y modelo. Para las bases de datos MSDB y modelo, verifique que están en modo de recuperación completa si desea que se realicen copias de seguridad de registros. Nunca se realizan copias de seguridad de registros para las bases de datos maestras. Tampoco se realizan copias de seguridad para TempDB. |
| **Programación de copia de seguridad** | Manual/Automatizado (Automatizado) | De forma predeterminada, la programación de copias de seguridad se determina automáticamente en función del crecimiento de los registros. La programación manual de copias de seguridad permite al usuario especificar el período de tiempo para las copias de seguridad. En este caso, las copias de seguridad siempre se realizan con la frecuencia especificada y durante el período de tiempo especificado de un día determinado. |
| **Frecuencia de copia de seguridad completa** | Diariamente/semanalmente | Frecuencia de las copias de seguridad completas. En ambos casos, las copias de seguridad completas se inician durante el siguiente período de tiempo programado. Cuando se selecciona semanalmente, las copias de seguridad pueden extenderse varios días hasta que se realizan correctamente las copias de seguridad de todas las bases de datos. |
| **Hora de inicio de copia de seguridad completa** | 00:00 – 23:00 (01:00) | Hora de inicio de un día determinado durante el cual se pueden realizar copias de seguridad completas. |
| **Período de tiempo de copia de seguridad completa** | 1-23 horas (1 hora) | Duración del período de tiempo de un día determinado durante el cual se pueden realizar copias de seguridad completas. |
| **Frecuencia de copia de seguridad de registros** | 5-60 minutos (60 minutos) | Frecuencia de las copias de seguridad de registros. |

## <a name="understanding-full-backup-frequency"></a>Información de la frecuencia de copia de seguridad completa
Es importante conocer la diferencia entre las copias de seguridad completas diarias y semanales. Considere los dos escenarios de ejemplo siguientes.

### <a name="scenario-1-weekly-backups"></a>Escenario 1: Copias de seguridad semanales
Tiene una VM con SQL Server que contiene varias bases de datos de gran tamaño.

El lunes, se habilita Automated Backup v2 con las siguientes opciones:

- Programación de copia de seguridad: **Manual**
- Frecuencia de copia de seguridad completa: **Semanal**
- Hora de inicio de copia de seguridad completa: **01:00**
- Período de tiempo de copia de seguridad completa: **1 hora**

Esto significa que la siguiente ventana de copia de seguridad disponible es el martes a la 1:00 durante una hora. En ese momento, Automated Backup empieza a realizar copias de seguridad de las bases de datos una a una. En este escenario, las bases de datos son lo suficientemente grandes como para que primero se realicen las copias de seguridad completas de las bases de datos pares. No obstante, después de una hora no se realiza la copia de seguridad de todas las bases de datos.

Cuando esto ocurre, Automated Backup comienza la copia de seguridad de las demás bases de datos al día siguiente, es decir, el miércoles a la 1:00 durante una hora. Si no se realiza la copia de seguridad de todas las bases de datos a esa hora, vuelve a intentarlo al día siguiente a la misma hora. Se aplica este mismo comportamiento hasta que se realiza correctamente la copia de seguridad de todas las bases de datos.

Cuando vuelve a llegar el martes, Automated Backup empieza de nuevo a realizar la copia de seguridad de todas las bases de datos.

En este escenario se muestra que Automated Backup solo funciona durante el período de tiempo especificado y que la copia de seguridad de cada base de datos se realiza una vez a la semana. También se muestra que las copias de seguridad se pueden extender varios días en caso de que no sea posible completar todas las copias de seguridad en un solo día.

### <a name="scenario-2-daily-backups"></a>Escenario 2: Copias de seguridad diarias
Tiene una VM con SQL Server que contiene varias bases de datos de gran tamaño.

El lunes, se habilita Automated Backup v2 con las siguientes opciones:

- Programación de copia de seguridad: Manual
- Frecuencia de copia de seguridad completa: Diario
- Hora de inicio de copia de seguridad completa: 22:00
- Período de tiempo de copia de seguridad completa: 6 horas

Esto significa que la siguiente ventana de copia de seguridad disponible es el lunes a la 22:00 durante seis horas. En ese momento, Automated Backup empieza a realizar copias de seguridad de las bases de datos una a una.

Posteriormente, se vuelve a realizar la copia de seguridad completa de todas las bases de datos el martes a las 22:00 horas durante seis horas.

> [!IMPORTANT]
> Al programar copias de seguridad diarias, se recomienda programar un período de tiempo amplio para asegurarse de se puede realizar la copia de seguridad de todas las bases de datos durante ese período. Esto es especialmente importante en el caso de que tenga una gran cantidad de datos de los que realizar una copia de seguridad.

## <a name="configure-new-vms"></a>Configuración de máquinas virtuales nuevas

Use Azure Portal para configurar la opción Automated Backup v2 cuando cree una máquina virtual con SQL Server 2016 o 2017 en el modelo de implementación de Resource Manager.

En la pestaña **Configuración de SQL Server**, seleccione **Habilitar** en **Copia de seguridad automatizada**. La siguiente captura de pantalla de Azure Portal muestra la opción de configuración **Copia de seguridad automatizada de SQL**.

![Configuración de Automated Backup de SQL en Azure Portal](./media/automated-backup/automated-backup-blade.png)

> [!NOTE]
> Automated Backup v2 está deshabilitado de forma predeterminada.

## <a name="configure-existing-vms"></a>Configuración de máquinas virtuales existentes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

En el caso de las máquinas de virtuales con SQL Server existentes, vaya al [recurso máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) y seleccione **Copias de seguridad** para configurar las copias de seguridad automatizadas.

![Automated Backup de SQL para máquinas virtuales existentes](./media/automated-backup/sql-server-configuration.png)


Cuando termine, haga clic en el botón **Aplicar** de la parte inferior de página de configuración de **Copias de seguridad** para guardar los cambios.

Si habilita Automated Backup por primera vez, Azure configura el Agente de IaaS de SQL Server en segundo plano. Durante este tiempo, es posible que Azure Portal no muestre que se ha configurado Automated Backup. Espere unos minutos hasta que el agente se instale y configure. Después, Azure Portal mostrará la configuración nueva.

## <a name="configure-with-powershell"></a>Configuración con PowerShell

Puede usar PowerShell para configurar Automated Backup v2. Antes de comenzar:

- [Descargue e instale la última versión de Azure PowerShell](https://aka.ms/webpi-azps).
- Abra Windows PowerShell y asócielo a su cuenta con el comando **Connect-AzAccount**.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

### <a name="install-the-sql-server-iaas-extension"></a>Instalación de la extensión IaaS de SQL Server
Si se aprovisiona una máquina virtual con SQL Server desde Azure Portal, la extensión IaaS de SQL Server también debe estar instalada. Para determinar si está instalada para la VM, llame al comando **Get-AzVM** y examine la propiedad **Extensions**.

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions 
```

Si la extensión del agente IaaS de SQL Server está instalada, debe aparecer como "SqlIaaSAgent" o "SQLIaaSExtension". **ProvisioningState** para la extensión también debería aparecer como "Correcto". 

Si no está instalada o no se ha podido aprovisionar, puede instalarla con el comando siguiente. Además del grupo de recursos y del nombre de VM, también debe especificar la región ( **$region**) en que se encuentra dicha VM.

```powershell
$region = "EASTUS2"
Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region 
```

### <a name="verify-current-settings"></a><a id="verifysettings"></a> Verificación de la configuración actual
Si ha habilitado Automated Backup durante el aprovisionamiento, puede usar PowerShell para comprobar la configuración actual. Ejecute el comando **Get-AzVMSqlServerExtension** y examine la propiedad **AutoBackupSettings**:

```powershell
(Get-AzVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

Debería obtener una salida similar a la siguiente:

```
Enable                      : True
EnableEncryption            : False
RetentionPeriod             : 30
StorageUrl                  : https://test.blob.core.windows.net/
StorageAccessKey            :  
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : Manual
FullBackupFrequency         : WEEKLY
FullBackupStartTime         : 2
FullBackupWindowHours       : 2
LogBackupFrequency          : 60
```

Si la salida muestra que la opción **Habilitar** está establecida en **False**, tiene que habilitar Automated Backup. Lo bueno es que habilita y configura Automated Backup de la misma manera. Vea la sección siguiente para leer esta información.

> [!NOTE] 
> Si comprueba la configuración inmediatamente después de realizar un cambio, es posible que obtenga los valores de configuración anteriores. Espere unos minutos y compruebe la configuración de nuevo para asegurarse de que se hayan aplicado los cambios.

### <a name="configure-automated-backup-v2"></a>Configuración de Automated Backup v2
Puede usar PowerShell para habilitar Automated Backup, así como para modificar su configuración y comportamiento en cualquier momento. 

En primer lugar, seleccione o cree una cuenta de almacenamiento para los archivos de copia de seguridad. El script siguiente selecciona una cuenta de almacenamiento o la crea si no existe.

```powershell
$storage_accountname = "yourstorageaccount"
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region } 
```

> [!NOTE]
> Automated Backup no permite almacenar las copias de seguridad en Premium Storage, pero pueden realizar copias de seguridad de discos de VM que usan Premium Storage.

Luego use el comando **New-AzVMSqlServerAutoBackupConfig** para habilitar y configurar los valores de Copia de seguridad automatizada v2 para almacenar copias de seguridad en la cuenta de Azure Storage. En este ejemplo, las copias de seguridad están configuradas para que se conserven durante 10 días. Las copias de seguridad de las bases de datos del sistema están habilitadas. Las copias de seguridad completas están programadas semanalmente con un período de tiempo que empieza a las 20:00 horas y dura dos horas. Las copias de seguridad de registros están programadas para que se realicen cada 30 minutos. El segundo comando, **Set-AzVMSqlServerExtension**, actualiza la VM de Azure especificada con esta configuración.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname 
```

La instalación y configuración del agente de Iaas de SQL Server puede tardar algunos minutos. 

Para habilitar el cifrado, modifique el script anterior para pasar el parámetro **EnableEncryption** junto con una contraseña (cadena segura) para el parámetro **CertificatePassword**. El siguiente script habilita la configuración de Automated Backup en el ejemplo anterior y agrega cifrado.

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

Para confirmar que se ha aplicado la configuración, [verifique la configuración de Automated Backup](#verifysettings).

### <a name="disable-automated-backup"></a>Deshabilitar Automated Backup
Para deshabilitar Copia de seguridad automatizada, ejecute el mismo script sin el parámetro **-Enable** en el comando **New-AzVMSqlServerAutoBackupConfig**. La ausencia del parámetro **-Enable** indica al comando que deshabilite la característica. Al igual que la instalación, la deshabilitación de Automated Backup puede tardar algunos minutos.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>Script de ejemplo
El script siguiente proporciona un conjunto de variables que se pueden personalizar para habilitar y configurar Automated Backup para la VM. En su caso, debe personalizar el script en función de sus requisitos. Por ejemplo, debe realizar cambios si desea deshabilitar la copia de seguridad de bases de datos del sistema o habilitar el cifrado.

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = "Azure region name such as EASTUS2"
$storage_accountname = "storageaccountname"
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10
$backupscheduletype = "Manual"
$fullbackupfrequency = "Weekly"
$fullbackupstarthour = "20"
$fullbackupwindow = "2"
$logbackupfrequency = "30"

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL Server IaaS Extension 

Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType $backupscheduletype -FullBackupFrequency $fullbackupfrequency `
    -FullBackupStartHour $fullbackupstarthour -FullBackupWindowInHours $fullbackupwindow `
    -LogBackupFrequencyInMinutes $logbackupfrequency

# Apply the Automated Backup settings to the VM

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="monitoring"></a>Supervisión

Para supervisar Automated Backup en SQL Server 2016/2017, tiene dos opciones principales. Puesto que Automated Backup usa la característica Copia de seguridad de administrada SQL Server, las mismas técnicas de supervisión se aplican a ambos.

En primer lugar, puede sondear el estado mediante una llamada a [msdb.managed_backup.sp_get_backup_diagnostics](/sql/relational-databases/system-stored-procedures/managed-backup-sp-get-backup-diagnostics-transact-sql). O bien consultar la función con valores de tabla [msdb.managed_backup.fn_get_health_status](/sql/relational-databases/system-functions/managed-backup-fn-get-health-status-transact-sql).

Otra opción es aprovechar las ventajas de la característica integrada Correo electrónico de base de datos para las notificaciones.

1. Llame al procedimiento almacenado [msdb.managed_backup.sp_set_parameter](/sql/relational-databases/system-stored-procedures/managed-backup-sp-set-parameter-transact-sql) para asignar una dirección de correo electrónico al parámetro **SSMBackup2WANotificationEmailIds**. 
1. Habilite [SendGrid](../../../sendgrid-dotnet-how-to-send-email.md) para enviar los mensajes de correo electrónico desde la VM de Azure.
1. Utilice el nombre de usuario y el servidor SMTP para configurar Correo electrónico de base de datos. Puede configurar Correo electrónico de base de datos en SQL Server Management Studio o con comandos de Transact-SQL. Para obtener más información, consulte [Correo electrónico de base de datos](/sql/relational-databases/database-mail/database-mail).
1. [Configurar el Agente SQL Server para que use el Correo electrónico de base de datos](/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail).
1. Compruebe que el puerto SMTP está permitido tanto a través del firewall de VM local como del grupo de seguridad de red para la VM.

## <a name="next-steps"></a>Pasos siguientes
Automated Backup v2 configura Managed Backup en Azure Virtual Machines. Por lo tanto, es importante [revisar la documentación de la Copia de seguridad administrada](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) para comprender el comportamiento y las implicaciones.

Puede encontrar directrices adicionales sobre la copia de seguridad y la restauración para SQL Server en VM de Azure en el siguiente artículo: [Copias de seguridad y restauración para SQL Server en Azure Virtual Machines](backup-restore.md).

Para más información acerca de otras tareas de automatización disponibles, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Para más información sobre cómo ejecutar SQL Server en máquinas virtuales de Azure, consulte [Introducción a SQL Server en Azure máquinas virtuales de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).
