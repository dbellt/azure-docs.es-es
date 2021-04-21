---
title: Azure Backup para SQL Server que se ejecuta en la máquina virtual de Azure
description: En este artículo, obtendrá información sobre cómo registrar Azure Backup en una instancia de SQL Server que se ejecuta en una máquina virtual de Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: v-amallick
ms.author: v-amallick
ms.collection: windows
ms.date: 07/05/2019
ms.openlocfilehash: c10be941206dd60887c9d82025506d1ea15c51a2
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517256"
---
# <a name="azure-backup-for-sql-server-running-in-azure-vm"></a>Azure Backup para SQL Server que se ejecuta en la máquina virtual de Azure

Azure Backup, entre otras ofertas, admite la realización de copias de seguridad de cargas de trabajo, por ejemplo de una instancia de SQL Server que se ejecuta en máquinas virtuales Azure. Como la aplicación SQL se ejecuta dentro de una máquina virtual de Azure, el servicio de copia de seguridad necesita permiso para acceder a la aplicación y capturar los detalles necesarios.
Para ello, Azure Backup instala la extensión **AzureBackupWindowsWindowsWorkload** en la máquina virtual, en la que se ejecuta SQL Server, durante el proceso de registro iniciado por el usuario.

## <a name="prerequisites"></a>Prerrequisitos

Para obtener la lista de escenarios admitidos, consulte la [matriz de compatibilidad](../../backup/sql-support-matrix.md#scenario-support) admitida con Azure Backup.

## <a name="network-connectivity"></a>Conectividad de red

Azure Backup admite las etiquetas del grupo de seguridad de red, implementando un servidor proxy o rangos IP listados; para más detalles sobre cada uno de los métodos, consulte este [artículo](../../backup/backup-sql-server-database-azure-vms.md#establish-network-connectivity).

## <a name="extension-schema"></a>Esquema de extensión

Los valores de propiedad y del esquema de extensión son los valores de configuración (configuración del entorno de ejecución) que está pasando el servicio a la API de CRP. Estos valores de configuración se usan durante el registro y la actualización. La extensión **AzureBackupWindowsWorkload** también usa este esquema. El esquema está preestablecido; se puede agregar un nuevo parámetro en el campo objectStr.

  ```json
      "runtimeSettings": [{
      "handlerSettings": {
      "protectedSettingsCertThumbprint": "",
      "protectedSettings": {
      "objectStr": "",
      "logsBlobUri": "",
      "statusBlobUri": ""
      }
      },
      "publicSettings": {
      "locale": "en-us",
      "taskId": "1c0ae461-9d3b-418c-a505-bb31dfe2095d",
      "objectStr": "",
      "commandStartTimeUTCTicks": "636295005824665976",
      "vmType": "vmType"
      }
      }]
      }
  ```

En el siguiente JSON, se muestra el esquema para la extensión WorkloadBackup.  

  ```json
  {
    "type": "extensions",
    "name": "WorkloadBackup",
    "location":"<myLocation>",
    "properties": {
      "publisher": "Microsoft.RecoveryServices",
      "type": "AzureBackupWindowsWorkload",
      "typeHandlerVersion": "1.1",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "locale":"<location>",
        "taskId":"<TaskId used by Azure Backup service to communicate with extension>",

        "objectStr": "<The configuration passed by Azure Backup service to extension>",

        "commandStartTimeUTCTicks": "<Scheduled start time of registration or upgrade task>",
        "vmType": "<Type of VM where registration got triggered Eg. Compute or ClassicCompute>"
      },
      "protectedSettings": {
        "objectStr": "<The sensitive configuration passed by Azure Backup service to extension>",
        "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
        "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
      }
    }
  }
  ```

### <a name="property-values"></a>Valores de propiedad

Nombre | Valor o ejemplo | Tipo de datos
 --- | --- | ---
locale | es-es  |  string
taskId | "1c0ae461-9d3b-418c-a505-bb31dfe2095d"  | string
objectStr <br/> (publicSettings)  | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ==" | string
commandStartTimeUTCTicks | "636967192566036845"  | string
vmType  | "microsoft.compute/virtualmachines"  | string
objectStr <br/> (protectedSettings) | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ==" | string
logsBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | string
statusBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | string

## <a name="template-deployment"></a>Implementación de plantilla

Recomendamos agregar la extensión AzureBackupWindowsWorkload a una máquina virtual mediante la habilitación la copia de seguridad de SQL Server en la máquina virtual. Esto puede lograrse mediante la [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) diseñada para automatizar la copia de seguridad en una máquina virtual de SQL Server.

## <a name="powershell-deployment"></a>Implementación de PowerShell

Para ello, debe "registrar" la máquina virtual de Azure que contiene la aplicación de SQL con un almacén de Recovery Services. Durante el registro, la extensión AzureBackupWindowsWorkload se instala en la máquina virtual. Use el cmdlet  [Register-AzRecoveryServicesBackupContainerPS](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer) para registrar la máquina virtual.

```powershell
$myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

El comando devolverá un **contenedor de copia de seguridad** de este recurso y el estado se **registrará**.

## <a name="next-steps"></a>Pasos siguientes

- [Más información ](../../backup/backup-sql-server-azure-troubleshoot.md)sobre las directrices para la solución de problemas de copias de seguridad de máquinas virtuales de Azure SQL Server
- [Preguntas comunes](../../backup/faq-backup-sql-server.yml) sobre la copia de seguridad de bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure con el servicio Azure Backup.
