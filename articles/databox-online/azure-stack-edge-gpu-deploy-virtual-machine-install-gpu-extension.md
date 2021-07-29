---
title: Instalación de la extensión de GPU en máquinas virtuales de GPU en el dispositivo de GPU de Azure Stack Edge Pro
description: Se describe cómo instalar extensiones de GPU en máquinas virtuales (VM) de GPU en un dispositivo GPU de Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 05/27/2021
ms.author: alkohli
ms.openlocfilehash: 76e1f80e1c6e1d977521724959db6ad36694f238
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110684127"
---
# <a name="install-gpu-extension-on-vms-for-your-azure-stack-edge-pro-gpu-device"></a>Instalación de la extensión de GPU en máquinas virtuales en el dispositivo de GPU de Azure Stack Edge Pro

[!INCLUDE [applies-to-GPU-and-pro-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-sku.md)]

En este artículo se describe cómo instalar la extensión de controlador de GPU para instalar los controladores de Nvidia adecuados en las máquinas virtuales de GPU que se ejecutan en el dispositivo de Azure Stack Edge. En el artículo se tratan los pasos de instalación de la extensión de GPU en máquinas virtuales Windows y Linux.


## <a name="prerequisites"></a>Prerrequisitos

Antes de instalar la extensión de GPU en las máquinas virtuales de GPU que se ejecutan en el dispositivo, asegúrese de que:

1. Tiene acceso a un dispositivo de Azure Stack Edge en el que ha implementado una o varias máquinas virtuales de GPU. Vea cómo [implementar una máquina virtual de GPU en el dispositivo](azure-stack-edge-gpu-deploy-gpu-virtual-machine.md).

    - Asegúrese de que el puerto habilitado para la red de proceso en el dispositivo está conectado y tiene acceso a Internet. Los controladores de GPU se descargan mediante el acceso a Internet.

        Este es un ejemplo en el que el puerto 2 estaba conectado a Internet y se usaba para habilitar la red de proceso. Si no se ha implementado Kubernetes en el entorno, puede omitir la dirección IP del nodo de Kubernetes y la asignación de direcciones IP del servicio externo.

        ![Habilitación de la configuración de proceso en el puerto conectado a Internet](media/azure-stack-edge-gpu-deploy-gpu-virtual-machine/enable-compute-network-1.png)
1. [Descargue las plantillas de extensión de GPU y los archivos de parámetros](https://aka.ms/ase-vm-templates) en la máquina cliente. Descomprímalos en un directorio que usará como directorio de trabajo.
1. Compruebe que el cliente que va a usar para acceder al dispositivo sigue conectado a Azure Resource Manager a través de Azure PowerShell. La conexión a Azure Resource Manager expira cada 1,5 horas o si se reinicia el dispositivo Azure Stack Edge. Si esto ocurre, los cmdlets que ejecute, devolverán mensajes de error porque ya no está conectado a Azure. Tiene que volver a iniciar sesión. Para consultar las instrucciones detalladas, vaya a [Conexión a Azure Resource Manager en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-connect-resource-manager.md).



## <a name="edit-parameters-file"></a>Edición del archivo de parámetros

En función del sistema operativo de la máquina virtual, podría instalar la extensión de GPU para Windows o para Linux.


### <a name="windows"></a>[Windows](#tab/windows)

Para implementar los controladores de GPU de Nvidia para una máquina virtual existente, edite el archivo de parámetros `addGPUExtWindowsVM.parameters.json` y, a continuación, implemente la plantilla `addGPUextensiontoVM.json`.

El archivo `addGPUExtWindowsVM.parameters.json` toma los parámetros siguientes:

```json
"parameters": { 
    "vmName": {
    "value": "<name of the VM>" 
    },
    "extensionName": {
    "value": "<name for the extension. Example: windowsGpu>" 
    },
    "publisher": {
    "value": "Microsoft.HpcCompute" 
    },
    "type": {
    "value": "NvidiaGpuDriverWindows" 
    },
    "typeHandlerVersion": {
    "value": "1.3" 
    },
    "settings": {
    "value": {
    "DriverURL" : "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
    "DriverCertificateUrl" : "https://go.microsoft.com/fwlink/?linkid=871664",
    "DriverType":"CUDA"
    }
    }
    }
```

### <a name="linux"></a>[Linux](#tab/linux)

Para implementar los controladores de GPU de Nvidia para una máquina virtual Linux existente, edite el archivo de parámetros y, a continuación, implemente la plantilla `addGPUextensiontoVM.json`. 

Si usa Ubuntu o Red Hat Enterprise Linux (RHEL), el archivo `addGPUExtLinuxVM.parameters.json` toma los parámetros siguientes:

```powershell
"parameters": { 
    "vmName": {
    "value": "<name of the VM>" 
    },
    "extensionName": {
    "value": "<name for the extension. Example: linuxGpu>" 
    },
    "publisher": {
    "value": "Microsoft.HpcCompute" 
    },
    "type": {
    "value": "NvidiaGpuDriverLinux" 
    },
    "typeHandlerVersion": {
    "value": "1.3" 
    },
    "settings": {
    }
    }
    }
```

Este es un archivo de parámetros de Ubuntu de ejemplo que se usó en este artículo:

```powershell
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "VM1" 
        },
        "extensionName": {
            "value": "gpuLinux" 
        },
        "publisher": {
            "value": "Microsoft.HpcCompute" 
        },
        "type": {
            "value": "NvidiaGpuDriverLinux" 
        },
        "typeHandlerVersion": {
            "value": "1.3" 
        },
        "settings": {
        }
    }
}
```

### <a name="gpu-vms-from-rhel-byos-images"></a>Máquinas virtuales de GPU a partir de imágenes BYOS de RHEL

Si creó la máquina virtual mediante una imagen de tipo "traiga su propia suscripción" de Red Hat Enterprise Linux (BYOS de RHEL), asegúrese de que:

- Ha seguido los pasos de [uso de la imagen BYOS de RHEL](azure-stack-edge-gpu-create-virtual-machine-image.md). 
- Después de crear la máquina virtual de GPU, regístrese y suscríbase a ella en el portal de clientes de Red Hat. Si la máquina virtual no está registrada correctamente, la instalación no continúa, ya que la máquina virtual no tiene derecho. Consulte [Registro y suscripción automática en un paso mediante el administrador de suscripciones de Red Hat.](https://access.redhat.com/solutions/253273) Este paso permite que el script de instalación descargue los paquetes pertinentes para el controlador de GPU.
- Puede instalar manualmente el paquete `vulkan-filesystem` o agregar el repositorio de CentOS7 a la lista de repositorios de Yum. Al instalar la extensión de GPU, el script de instalación busca un paquete `vulkan-filesystem` que se encuentra en el repositorio CentOS7 (para RHEL7). 

---

## <a name="deploy-template"></a>Implementar plantilla 

### <a name="windows"></a>[Windows](#tab/windows)

Implemente la plantilla `addGPUextensiontoVM.json`. Esta plantilla implementa la extensión en una máquina virtual existente. Ejecute el siguiente comando:

```powershell
$templateFile = "<Path to addGPUextensiontoVM.json>" 
$templateParameterFile = "<Path to addGPUExtWindowsVM.parameters.json>" 
$RGName = "<Name of your resource group>"
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Name for your deployment>"
```
> [!NOTE]
> La implementación de la extensión es un trabajo de larga duración y tarda unos 10 minutos en completarse.

Este es una salida de ejemplo:

```powershell
PS C:\WINDOWS\system32> "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json
PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addGPUExtWindowsVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "myasegpuvm1"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment3"

DeploymentName          : deployment3
ResourceGroupName       : myasegpuvm1
ProvisioningState       : Succeeded
Timestamp               : 12/16/2020 12:18:50 AM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM2
                          extensionName    String                     windowsgpuext
                          publisher        String                     Microsoft.HpcCompute
                          type             String                     NvidiaGpuDriverWindows
                          typeHandlerVersion  String                     1.3
                          settings         Object                     {
                            "DriverURL": "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
                            "DriverCertificateUrl": "https://go.microsoft.com/fwlink/?linkid=871664",
                            "DriverType": "CUDA"
                          }

Outputs                 :
DeploymentDebugLogLevel :
PS C:\WINDOWS\system32>
```

### <a name="linux"></a>[Linux](#tab/linux)

Implemente la plantilla `addGPUextensiontoVM.json`. Esta plantilla implementa la extensión en una máquina virtual existente. Ejecute el siguiente comando:

```powershell
$templateFile = "Path to addGPUextensiontoVM.json" 
$templateParameterFile = "Path to addGPUExtLinuxVM.parameters.json" 
$RGName = "<Name of your resource group>" 
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Name for your deployment>"
``` 

> [!NOTE]
> La implementación de la extensión es un trabajo de larga duración y tarda unos 10 minutos en completarse.

Este es una salida de ejemplo:

```powershell
Copyright (C) Microsoft Corporation. All rights reserved.
Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addGPUExtLinuxVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "rg2"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "delpoyment7"

DeploymentName          : delpoyment7
ResourceGroupName       : rg2
ProvisioningState       : Succeeded
Timestamp               : 12/10/2020 10:43:23 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM1
                          extensionName    String                     gpuLinux
                          publisher        String                     Microsoft.HpcCompute
                          type             String                     NvidiaGpuDriverLinux
                          typeHandlerVersion  String                     1.3
                          settings         Object                     {
                            "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
                            "PUBKEY_URL":
                          "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
                            "CUDA_ver": "10.0.130",
                            "InstallCUDA": "true"
                          }

Outputs                 :
DeploymentDebugLogLevel :
PS C:\WINDOWS\system32>
```
---

## <a name="track-deployment"></a>Seguimiento de la implementación

### <a name="windows"></a>[Windows](#tab/windows)

Para comprobar el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente: 

```powershell
Get-AzureRmVMExtension -ResourceGroupName <Name of resource group> -VMName <Name of VM> -Name <Name of the extension>
```
Este es una salida de ejemplo:

```powershell
PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName myasegpuvm1 -VMName VM2 -Name windowsgpuext

ResourceGroupName       : myasegpuvm1
VMName                  : VM2
Name                    : windowsgpuext
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.HpcCompute
ExtensionType           : NvidiaGpuDriverWindows
TypeHandlerVersion      : 1.3
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/myasegpuvm1/providers/Microsoft.Compute/virtualMachines/VM2/extensions/windowsgpuext
PublicSettings          : {
                            "DriverURL": "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
                            "DriverCertificateUrl": "https://go.microsoft.com/fwlink/?linkid=871664",
                            "DriverType": "CUDA"
                          }
ProtectedSettings       :
ProvisioningState       : Creating
Statuses                :
SubStatuses             :
AutoUpgradeMinorVersion : True
ForceUpdateTag          :

PS C:\WINDOWS\system32>
```

La salida de la ejecución de las extensiones se registra en el archivo siguiente: Consulte el archivo `C:\Packages\Plugins\Microsoft.HpcCompute.NvidiaGpuDriverWindows\1.3.0.0\Status` para realizar un seguimiento del estado de la instalación. 


Una instalación correcta se indica mediante el elemento `message` establecido como `Enable Extension` y el elemento `status` establecido como `success`.

```powershell
"status":  {
                       "formattedMessage":  {
                                                "message":  "Enable Extension",
                                                "lang":  "en"
                                            },
                       "name":  "NvidiaGpuDriverWindows",
                       "status":  "success",
```

### <a name="linux"></a>[Linux](#tab/linux)

Template Deployment es un trabajo de larga duración. Para comprobar el estado de implementación de las extensiones de una máquina virtual determinada, abra otra sesión de PowerShell (ejecutar como administrador). Ejecute el siguiente comando: 

```powershell
Get-AzureRmVMExtension -ResourceGroupName myResourceGroup -VMName <VM Name> -Name <Extension Name>
```
Este es una salida de ejemplo: 

```powershell
Copyright (C) Microsoft Corporation. All rights reserved.
Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName rg2 -VMName VM1 -Name gpulinux

ResourceGroupName       : rg2
VMName                  : VM1
Name                    : gpuLinux
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.HpcCompute
ExtensionType           : NvidiaGpuDriverLinux
TypeHandlerVersion      : 1.3
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/rg2/providers/Microsoft.Compute/virtualMachines/VM1/extensions/gpuLinux
PublicSettings          : {
                            "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
                            "PUBKEY_URL": "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
                            "CUDA_ver": "10.0.130",
                            "InstallCUDA": "true"
                          }
ProtectedSettings       :
ProvisioningState       : Creating
Statuses                :
SubStatuses             :
AutoUpgradeMinorVersion : True
ForceUpdateTag          :

PS C:\WINDOWS\system32>
```

> [!NOTE]
> Una vez completada la implementación, `ProvisioningState` cambia a `Succeeded`.

La salida de la ejecución de las extensiones se registra en el archivo siguiente:`/var/log/azure/nvidia-vmext-status`.

---

## <a name="verify-driver-installation"></a>Comprobación de la instalación del controlador

### <a name="windows"></a>[Windows](#tab/windows)

Inicie sesión en la máquina virtual y ejecute la utilidad de la línea de comandos nvidia-smi instalada con el controlador. El archivo `nvidia-smi.exe` se encuentra en `C:\Program Files\NVIDIA Corporation\NVSMI\nvidia-smi.exe`. Si no ve el archivo, es posible que la instalación del controlador siga ejecutándose en segundo plano. Espere 10 minutos y vuelva a comprobarlo.

Si el controlador está instalado, verá una salida similar a la del ejemplo siguiente: 

```powershell
PS C:\Users\Administrator> cd "C:\Program Files\NVIDIA Corporation\NVSMI"
PS C:\Program Files\NVIDIA Corporation\NVSMI> ls

    Directory: C:\Program Files\NVIDIA Corporation\NVSMI

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/26/2020  12:00 PM         849640 MCU.exe
-a----        2/26/2020  12:00 PM         443104 nvdebugdump.exe
-a----        2/25/2020   2:06 AM          81823 nvidia-smi.1.pdf
-a----        2/26/2020  12:01 PM         566880 nvidia-smi.exe
-a----        2/26/2020  12:01 PM         991344 nvml.dll

PS C:\Program Files\NVIDIA Corporation\NVSMI> .\nvidia-smi.exe
Wed Dec 16 00:35:51 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 442.50       Driver Version: 442.50       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla T4            TCC  | 0000503C:00:00.0 Off |                    0 |
| N/A   35C    P8    11W /  70W |      8MiB / 15205MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
PS C:\Program Files\NVIDIA Corporation\NVSMI>
```

Para más información, consulte [Extensión del controlador de GPU de NVIDIA para Windows](../virtual-machines/extensions/hpccompute-gpu-windows.md).

### <a name="linux"></a>[Linux](#tab/linux)

Siga estos pasos para comprobar la instalación del controlador:

1. Conéctese a la máquina virtual con GPU. Siga las instrucciones que se indican en [Conexión a una máquina virtual Linux](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#connect-to-a-linux-vm). 

    Este es una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> ssh -l Administrator 10.57.50.60
    Administrator@10.57.50.60's password:
    Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.0.0-1031-azure x86_64)
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
      System information as of Thu Dec 10 22:57:01 UTC 2020
    
      System load:  0.0                Processes:           133
      Usage of /:   24.8% of 28.90GB   Users logged in:     0
      Memory usage: 2%                 IP address for eth0: 10.57.50.60
      Swap usage:   0%
    
    249 packages can be updated.
    140 updates are security updates.
    
    Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.0.0-1031-azure x86_64)    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage    
      System information as of Thu Dec 10 22:57:01 UTC 2020    
      System load:  0.0                Processes:           133
      Usage of /:   24.8% of 28.90GB   Users logged in:     0
      Memory usage: 2%                 IP address for eth0: 10.57.50.60
      Swap usage:   0%
        
    249 packages can be updated.
    140 updates are security updates.
    
    New release '20.04.1 LTS' available.
    Run 'do-release-upgrade' to upgrade to it.
        
    *** System restart required ***
    Last login: Thu Dec 10 21:49:29 2020 from 10.90.24.23
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    Administrator@VM1:~$
    ```
2. Ejecute la utilidad de la línea de comandos nvidia-smi instalada con el controlador. Si el controlador se ha instalado correctamente, podrá ejecutar la utilidad y ver la siguiente salida:

    ```powershell
    Administrator@VM1:~$ nvidia-smi
    Thu Dec 10 22:58:46 2020
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 455.45.01    Driver Version: 455.45.01    CUDA Version: 11.1     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            Off  | 0000941F:00:00.0 Off |                    0 |
    | N/A   48C    P0    27W /  70W |      0MiB / 15109MiB |      5%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    Administrator@VM1:~$
    ```

Para más información, consulte [Extensión del controlador de GPU de NVIDIA para Linux](../virtual-machines/extensions/hpccompute-gpu-linux.md).

---


## <a name="remove-gpu-extension"></a>Eliminación de la extensión de GPU

Para eliminar la extensión de GPU, use el siguiente comando:

`Remove-AzureRmVMExtension -ResourceGroupName <Resource group name> -VMName <VM name> -Name <Extension name>`

Este es una salida de ejemplo:

```powershell
PS C:\azure-stack-edge-deploy-vms> Remove-AzureRmVMExtension -ResourceGroupName rgl -VMName WindowsVM -Name windowsgpuext
Virtual machine extension removal operation
This cmdlet will remove the specified virtual machine extension. Do you want to continue? [Y] Yes [N] No [S] Suspend [?] Help (default is "Y"): y
Requestld IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------    
          True                OK         OK
```


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo:

- [Administrar discos de máquinas virtuales](azure-stack-edge-gpu-manage-virtual-machine-disks-portal.md).
- [Administrar interfaces de red de máquina virtual](azure-stack-edge-gpu-manage-virtual-machine-network-interfaces-portal.md).
- [Administrar tamaños de máquina virtual](azure-stack-edge-gpu-manage-virtual-machine-resize-portal.md).