---
title: Extensión de Windows de instantánea de máquina virtual para Azure Backup
description: Realice una copia de seguridad coherente con la aplicación de la máquina virtual desde Azure Backup mediante la extensión de instantánea de máquina virtual
services: backup, virtual-machines
documentationcenter: ''
author: trinadhkotturu
manager: gwallace
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: article
ms.date: 10/15/2020
ms.author: trinadhk
ms.openlocfilehash: 6dc2fb12ebd166f62f04a1ecb9833edaad18f539
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110789617"
---
# <a name="vm-snapshot-windows-extension-for-azure-backup"></a>Extensión de Windows de instantánea de máquina virtual para Azure Backup

Azure Backup proporciona compatibilidad para realizar copias de seguridad de cargas de trabajo desde el entorno local a la nube y para realizar copias de seguridad de recursos en la nube en el almacén de Recovery Services. Azure Backup usa la extensión de instantánea de máquina virtual para realizar una copia de seguridad coherente con la aplicación de la máquina virtual de Azure sin necesidad de apagar la máquina virtual. Microsoft publica la extensión de instantánea de máquina virtual y es compatible con esta como parte del servicio de Azure Backup. Azure Backup instalará la extensión como parte de la primera copia de seguridad programada que se desencadene tras habilitar la copia de seguridad. En este documento se especifican las plataformas compatibles, configuraciones y opciones de implementación de la extensión de instantánea de máquina virtual.

La extensión VMSnapshot solo aparece en Azure Portal para las máquinas virtuales no administradas.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo
Para obtener una lista de sistemas operativos compatibles, vea [Sistemas operativos compatibles con Azure Backup](../../backup/backup-azure-arm-vms-prepare.md#before-you-start).

## <a name="extension-schema"></a>Esquema de extensión

El siguiente JSON muestra el esquema para la extensión de instantánea de máquina virtual. La extensión requiere el identificador de tarea (que identifica el trabajo de copia de seguridad que ha desencadenado la instantánea en la máquina virtual), el URI del blob de estado (donde se escribe el estado de la operación de instantánea), la hora de inicio programada de la instantánea, el URI del blob de registros (donde se escriben los registros correspondientes a la tarea de instantánea, objstr), la representación de discos de máquina virtual y los metadatos.  Como estas opciones deben tratarse como datos confidenciales, deben almacenarse en una configuración protegida. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino. Tenga en cuenta que se recomienda pasar estas opciones desde el servicio de Azure Backup únicamente como parte de un trabajo de copia de seguridad.

```json
{
  "type": "extensions",
  "name": "VMSnapshot",
  "location":"<myLocation>",
  "properties": {
    "publisher": "Microsoft.Azure.RecoveryServices",
    "type": "VMSnapshot",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "locale":"<location>",
      "taskId":"<taskId used by Azure Backup service to communicate with extension>",
      "commandToExecute": "snapshot",
      "commandStartTimeUTCTicks": "<scheduled start time of the snapshot task>",
      "vmType": "microsoft.compute/virtualmachines"
    },
    "protectedSettings": {
      "objectStr": "<blob SAS uri representation of VM sent by Azure Backup service to extension>",
      "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
      "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
    }
  }
}
```

### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| taskId | e07354cf-041e-4370-929f-25a319ce8933_1 | string |
| commandStartTimeUTCTicks | 6.36458E+17 | string |
| locale | es-es | string |
| objectStr | Codificación de la matriz de URI de SAS: "blobSASUri": ["https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc201652903941.vhd?sv=2014-02-14&sr=b&sig=TywkROXL1zvhXcLujtCut8g3jTpgbE6JpSWRLZxAdtA%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/vhds\/vmwin1404ltsc-20160629-122418.vhd?sv=2014-02-14&sr=b&sig=5S0A6YDWvVwqPAkzWXVy%2BS%2FqMwzFMbamT5upwx05v8Q%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna8461.blob.core.windows.net\/bootdiagnostics-vmwintu1-deb58392-ed5e-48be-9228-ff681b0cd3ee\/vmubuntu1404ltsc-20160629-122541.vhd?sv=2014-02-14&sr=b&sig=X0Me2djByksBBMVXMGIUrcycvhQSfjYvqKLeRA7nBD4%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc-20160701-163922.vhd?sv=2014-02-14&sr=b&sig=oXvtK2IXCNqWv7fpjc7TAzFDpc1GoXtT7r%2BC%2BNIAork%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw", "https:\/\/sopattna5365.blob.core.windows.net\/vhds\/vmwin1404ltsc-20170705-124311.vhd?sv=2014-02-14&sr=b&sig=ZUM9d28Mvvm%2FfrhJ71TFZh0Ni90m38bBs3zMl%2FQ9rs0%3D&st=2017-11-09T14%3A23%3A28Z&se=2017-11-09T17%3A38%3A28Z&sp=rw"] | string |
| logsBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | string |
| statusBlobUri | https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw | string |



## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Aun así, la manera recomendada de agregar una extensión de instantánea de máquina virtual a una máquina virtual consiste en habilitar la copia de seguridad en la máquina virtual. Esto puede conseguirse mediante una plantilla de Resource Manager.  Encontrará una plantilla de Resource Manager de ejemplo que habilita la copia de seguridad en una máquina virtual en la [Galería de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-backup-vms/).


## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

Puede usarse la CLI de Azure para habilitar la copia de seguridad en una máquina virtual. Tras habilitar la copia de seguridad, el primer trabajo de copia de seguridad programado instalará la extensión de instantánea de máquina virtual en la máquina virtual.

```azurecli
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm myVM \
    --policy-name DefaultPolicy
```

## <a name="azure-powershell-deployment"></a>Implementación de Azure PowerShell

Puede usarse Azure PowerShell para habilitar la copia de seguridad en una máquina virtual. Una vez configurada la copia de seguridad, el primer trabajo de copia de seguridad programado instalará la extensión de instantánea de máquina virtual en la máquina virtual.

```azurepowershell
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "myResourceGroup" -Name "myRecoveryServicesVault"
$pol = Get-AzRecoveryServicesBackupProtectionPolicy Name DefaultPolicy -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "myVM" -ResourceGroupName "myVMResourceGroup" -VaultId $targetVault.ID
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

El resultado de la ejecución de las extensiones se registra en el archivo siguiente:

```
C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot
```

### <a name="error-codes-and-their-meanings"></a>Códigos de error y su significado

Encontrará información para solucionar problemas en la guía de [Solución de problemas de copia de seguridad de máquinas virtuales de Azure](../../backup/backup-azure-vms-troubleshoot.md).

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
