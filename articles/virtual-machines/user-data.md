---
title: Datos de usuario en máquinas virtuales de Azure
description: Permite al cliente insertar scripts u otros metadatos en una máquina virtual de Azure en tiempo de aprovisionamiento.
services: virtual-machines-linux
author: ningk
manager: rogardle
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/30/2021
ms.author: ningk
ms.reviewer: azmetadata
ms.openlocfilehash: d149ead5cd45b55c846852f92342e6bbc56ba3cf
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109665324"
---
# <a name="user-data-for-azure-virtual-machine"></a>Datos de usuario en máquinas virtuales de Azure 

Los datos de usuario permiten pasar scripts o metadatos propios a la máquina virtual.

## <a name="what-is-user-data"></a>Qué son los "datos de usuario"

Los datos de usuario son un conjunto de scripts u otros metadatos que se van a insertar en una máquina virtual de Azure en tiempo de aprovisionamiento. Cualquier aplicación de la máquina virtual puede acceder a los datos de usuario desde Azure Instance Metadata Service (IMDS) después del aprovisionamiento. 

Los datos de usuario son una nueva versión de los [datos personalizados](./custom-data.md) y ofrecen ventajas adicionales:

* Los datos de usuario se pueden recuperar de Azure Instance Metadata Service (IMDS) después del aprovisionamiento.

* Los datos de usuario son persistentes. Estarán disponibles durante la vigencia de la máquina virtual.

* Los datos de usuario se pueden actualizar desde fuera de la máquina virtual, sin necesidad de detener ni reiniciar la máquina virtual.

* Los datos de usuario se pueden consultar mediante la API GET VM/VMSS con la opción $expand.

 Además, si los datos de usuario no se agregan en tiempo de aprovisionamiento, puede agregarlos después del aprovisionamiento.

**Advertencia de seguridad**

> [!WARNING]
> Los datos de usuario no se cifrarán y cualquier proceso de la máquina virtual puede consultarlos. No debe almacenar información confidencial en los datos de usuario.

Asegúrese de obtener la API de Azure Resource Manager más reciente para usar las nuevas características de datos de usuario. El contenido debe estar codificado en base64 antes de pasarlo a la API. El tamaño no puede ser superior a 64 KB.

## <a name="create-user-data-for-azure-vmvmss"></a>Creación de datos de usuario para VM/VMSS de Azure

**Adición de datos de usuario al crear una nueva máquina virtual**

Use [esta plantilla de Azure Resource Manager](https://aka.ms/ImdsUserDataArmTemplate) para crear una nueva máquina virtual con datos de usuario.
Si usa la API REST, en máquinas virtuales únicas, agregue "UserData" a la sección "properties" con la solicitud PUT para crear la máquina virtual.

```json
{
  "name": "testVM",
  "location": "West US",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_A1"
    },
    "storageProfile": {
      "osDisk": {
        "osType": "Windows",
        "name": "osDisk",
        "createOption": "Attach",
        "vhd": {
          "uri": "http://myaccount.blob.core.windows.net/container/directory/blob.vhd"
        }
      }
    },
    "userData": "c2FtcGxlIHVzZXJEYXRh",
    "networkProfile": { "networkInterfaces" : [ { "name" : "nic1" } ] },
  }
}
```

**Adición de datos de usuario al crear un nuevo conjunto de escalado de máquinas virtuales**

Si usa la API REST, agregue "UserData" a la sección "virtualMachineProfile" con la solicitud PUT al crear el conjunto de escalado de máquinas virtuales.
```json
{
  "location": "West US",
  "sku": {
    "name": "Standard_A1",
    "capacity": 1
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
      "userData": "VXNlckRhdGE=",
      "osProfile": {
        "computerNamePrefix": "TestVM",
        "adminUsername": "TestUserName",
        "windowsConfiguration": {
          "provisionVMAgent": true,
          "timeZone": "Dateline Standard Time"
        }
      },
      "storageProfile": {
        "osDisk": {
          "createOption": "FromImage",
          "caching": "ReadOnly"
        },
        "imageReference": {
          "publisher": "publisher",
          "offer": "offer",
          "sku": "sku",
          "version": "1.2.3"
        }
      },
      "networkProfile": {"networkInterfaceConfigurations":[{"name":"nicconfig1","properties":{"ipConfigurations":[{"name":"ip1","properties":{"subnet":{"id":"vmssSubnet0"}}}]}}]},
      "diagnosticsProfile": {
        "bootDiagnostics": {
          "enabled": true,
          "storageUri": "https://crputest.blob.core.windows.net"
        }
      }
    },
    "provisioningState": 0,
    "overprovision": false,
    "uniqueId": "00000000-0000-0000-0000-000000000000"
  }
}
```


## <a name="retrieving-user-data"></a>Recuperación de datos de usuario

Las aplicaciones que se ejecutan dentro de la máquina virtual pueden recuperar datos de usuario del punto de conexión de IMDS. Para obtener más información, consulte [este código de ejemplo de IMDS](./linux/instance-metadata-service.md?tabs=linux#get-user-data).

Los clientes pueden recuperar el valor existente de los datos de usuario mediante la API REST con el punto de conexión \$expand=userData (el cuerpo de la solicitud se puede dejar en blanco).

Máquinas virtuales únicas:

`GET "/subscriptions/{guid}/resourceGroups/{RGName}/providers/Microsoft.Compute/virtualMachines/{VMName}?$expand=userData"`

Conjunto de escalado de máquinas virtuales:

`GET "/subscriptions/{guid}/resourceGroups/{RGName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMSSName}?$expand=userData"`

Máquina virtual del conjunto de escalado de máquinas virtuales:

` GET "/subscriptions/{guid}/resourceGroups/{RGName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMSSName}/virtualmachines/{vmss instance id}?$expand=userData" `

## <a name="updating-user-data"></a>Actualización de datos de usuario

Con la API REST, puede usar una solicitud PUT o PATCH normal para actualizar los datos de usuario. Los datos de usuario se actualizarán sin necesidad de detener ni reiniciar la máquina virtual.

`PUT
"/subscriptions/{guid}/resourceGroups/{RGName}/providers/Microsoft.Compute/ virtualMachines/{VMName}
`

`PATCH
"/subscriptions/{guid}/resourceGroups/{RGName}/providers/Microsoft.Compute/ virtualMachines/{VMName}
`

El elemento VM.Properties de estas solicitudes debe contener el campo UserData deseado, como se muestra a continuación:

```json
"properties": {
        "hardwareProfile": {
          "vmSize": "Standard_D1_v2"
        },
        "storageProfile": {
          "imageReference": {
            "sku": "2016-Datacenter",
            "publisher": "MicrosoftWindowsServer",
            "version": "latest",
            "offer": "WindowsServer"
          },
          "osDisk": {
            "caching": "ReadWrite",
            "managedDisk": {
              "storageAccountType": "Standard_LRS"
            },
            "name": "vmOSdisk",
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/{existing-nic-name}",
              "properties": {
                "primary": true
              }
            }
          ]
        },
        "osProfile": {
          "adminUsername": "{your-username}",
          "computerName": "{vm-name}",
          "adminPassword": "{your-password}"
        },
        "diagnosticsProfile": {
          "bootDiagnostics": {
            "storageUri": "http://{existing-storage-account-name}.blob.core.windows.net",
            "enabled": true
          }
        },
        "userData": "U29tZSBDdXN0b20gRGF0YQ=="
      } 
```
> [!NOTE]
> Si en este caso pasa una cadena vacía para "userData", los datos de usuario se eliminarán.

## <a name="user-data-and-custom-data"></a>Datos de usuario y datos personalizados

Los datos personalizados continuarán funcionando como lo hacen ahora. Tenga en cuenta que no es posible recuperar datos personalizados de IMDS. 

## <a name="adding-user-data-to-an-existing-vm"></a>Adición de datos de usuario a una máquina virtual existente 

Si ya tiene una máquina virtual o un conjunto de escalado de máquinas virtuales sin datos de usuario, puede agregarle datos de usuario mediante los comandos de actualización, como se describe en la sección ["Actualización de los datos de usuario"](#updating-user-data). Asegúrese de actualizar a la versión más reciente de la API de Azure Resource Manager.

## <a name="next-steps"></a>Pasos siguientes 
 
Pruebe [Azure Instance Metadata Service](./linux/instance-metadata-service.md), aprenda a obtener los metadatos y los datos de usuario de la instancia de máquina virtual desde su punto de conexión. 
