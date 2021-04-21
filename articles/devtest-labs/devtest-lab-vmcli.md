---
title: Creación y administración de máquinas virtuales en DevTest Labs con la CLI de Azure
description: Aprenda a usar Azure DevTest Labs para crear y administrar máquinas virtuales con la CLI de Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 95e0add8ce14e47c609b1ae951673c261316293f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763548"
---
# <a name="create-and-manage-virtual-machines-with-devtest-labs-using-the-azure-cli"></a>Creación y administración de máquinas virtuales con DevTest Labs mediante la CLI de Azure
Este inicio rápido le ayudará a crear, iniciar, actualizar y limpiar una máquina de desarrollo en el laboratorio, así como conectarse a ella. 

Antes de empezar:

* Si no se ha creado un laboratorio, encontrará instrucciones [aquí](devtest-lab-create-lab.md).

* [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Para empezar, ejecute az login para crear una conexión con Azure. 

## <a name="create-and-verify-the-virtual-machine"></a>Creación y comprobación de la máquina virtual 
Antes de ejecutar los comandos relacionados de DevTest Labs, establezca el contexto de Azure adecuado mediante el comando `az account set`:

```azurecli
az account set --subscription 11111111-1111-1111-1111-111111111111
```

El comando para crear una máquina virtual es `az lab vm create`. Solo se necesita el grupo de recursos para el laboratorio, el nombre del laboratorio y el nombre de la máquina virtual. El resto de los argumentos cambian según el tipo de máquina virtual.

El siguiente comando crea una imagen basada en Windows desde Azure Marketplace. El nombre de la imagen es el mismo que vería al crear una máquina virtual mediante Azure Portal. 

```azurecli
az lab vm create --resource-group DtlResourceGroup --lab-name MyLab --name 'MyTestVm' --image "Visual Studio Community 2017 on Windows Server 2016 (x64)" --image-type gallery --size 'Standard_D2s_v3' --admin-username 'AdminUser' --admin-password 'Password1!'
```

El siguiente comando crea una máquina virtual basada en una imagen personalizada disponible en el laboratorio:

```azurecli
az lab vm create --resource-group DtlResourceGroup --lab-name MyLab --name 'MyTestVm' --image "My Custom Image" --image-type custom --size 'Standard_D2s_v3' --admin-username 'AdminUser' --admin-password 'Password1!'
```

El argumento **image-type** ha cambiado de **gallery** a **custom**. El nombre de la imagen coincide con lo que se ve si fuera a crear la máquina virtual en Azure Portal.

El siguiente comando crea una máquina virtual desde una imagen de Marketplace mediante autenticación ssh:

```azurecli
az lab vm create --lab-name sampleLabName --resource-group sampleLabResourceGroup --name sampleVMName --image "Ubuntu Server 16.04 LTS" --image-type gallery --size Standard_DS1_v2 --authentication-type  ssh --generate-ssh-keys --ip-configuration public 
```

También puede crear máquinas virtuales basadas en las fórmulas estableciendo el parámetro **image-type** en **formula**. Si tiene que elegir una red virtual específica para la máquina virtual, use los parámetros **vnet-name** y **subnet**. Para obtener más información, consulte[az lab vm create](/cli/azure/lab/vm#az_lab_vm_create).

## <a name="verify-that-the-vm-is-available"></a>Compruebe que la máquina virtual esté disponible.
Use el comando `az lab vm show` para comprobar que la máquina virtual está disponible antes de iniciarla y conectarse a ella. 

```azurecli
az lab vm show --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup --expand 'properties($expand=ComputeVm,NetworkInterface)' --query '{status: computeVm.statuses[0].displayStatus, fqdn: fqdn, ipAddress: networkInterface.publicIpAddress}'
```
```json
{
  "fqdn": "lisalabvm.southcentralus.cloudapp.azure.com",
  "ipAddress": "13.85.228.112",
  "status": "Provisioning succeeded"
}
```

## <a name="start-and-connect-to-the-virtual-machine"></a>Inicio y conexión a la máquina virtual
El siguiente comando de ejemplo inicia una máquina virtual:

```azurecli
az lab vm start --lab-name sampleLabName --name sampleVMName --resource-group sampleLabResourceGroup
```

Conéctese a una máquina virtual: [SSH](../virtual-machines/linux/mac-create-ssh-keys.md) o [Escritorio remoto](../virtual-machines/windows/connect-logon.md).
```bash
ssh userName@ipAddressOrfqdn 
```

## <a name="update-the-virtual-machine"></a>Actualización de la máquina virtual
El siguiente comando de ejemplo aplica artefactos a una máquina virtual:

```azurecli
az lab vm apply-artifacts --lab-name  sampleLabName --name sampleVMName  --resource-group sampleResourceGroup  --artifacts @/artifacts.json
```

```json
[
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-java",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-install-nodejs",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-apt-package",
    "parameters": [
      {
        "name": "packages",
        "value": "abcd"
      },
      {
        "name": "update",
        "value": "true"
      },
      {
        "name": "options",
        "value": ""
      }
    ]
  } 
]
```

### <a name="list-artifacts-available-in-the-lab"></a>Enumeración de los artefactos disponibles en el laboratorio

Para enumerar los artefactos disponibles en una máquina virtual de un laboratorio, ejecute los siguientes comandos.

**Cloud Shell - PowerShell**: observe el uso del acento grave (\`) delante de $ en $expand (por ejemplo, `$expand):

```azurecli-interactive
az lab vm show --resource-group <resourcegroupname> --lab-name <labname> --name <vmname> --expand "properties(`$expand=artifacts)" --query "artifacts[].{artifactId: artifactId, status: status}"
```

**Cloud Shell - Bash**: observe el uso del carácter de barra (\\) delante de $ en el comando. 

```azurecli-interactive
az lab vm show --resource-group <resourcegroupname> --lab-name <labname> --name <vmname> --expand "properties(\$expand=artifacts)" --query "artifacts[].{artifactId: artifactId, status: status}"
```

Salida del ejemplo: 

```json
[
  {
    "artifactId": "/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DevTestLab/labs/<lab name>/artifactSources/public repo/artifacts/windows-7zip",
    "status": "Succeeded"
  }
]
```

## <a name="stop-and-delete-the-virtual-machine"></a>Detención y eliminación de la máquina virtual    
El siguiente comando de ejemplo detiene una máquina virtual.

```azurecli
az lab vm stop --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

Elimine una máquina virtual.
```azurecli
az lab vm delete --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes
Consulte el siguiente contenido: [Documentación de la CLI de Azure para Azure DevTest Labs](/cli/azure/lab). 
