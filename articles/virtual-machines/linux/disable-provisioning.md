---
title: Deshabilitar o quitar los agentes de aprovisionamiento
description: Obtenga información sobre cómo deshabilitar o quitar los agentes de aprovisionamiento en las máquinas virtuales y las imágenes de Linux.
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: c70b02bdc554c723f53ad5f8c0d36c5eca87811e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774375"
---
# <a name="disable-or-remove-the-linux-agent-from-vms-and-images"></a>Deshabilitar o quitar el agente Linux de las máquinas virtuales y las imágenes

Antes de quitar el agente Linux, debe saber qué es lo que no podrá hacer la VM después de quitarlo.

Las [extensiones](../extensions/overview.md) de máquina virtual de Azure son pequeñas aplicaciones que realizan tareas de automatización y configuración posteriores a la implementación en máquinas virtuales de Azure. El plano de control de Azure instala y administra las extensiones. El trabajo del [Agente Linux de Azure](../extensions/agent-linux.md) es procesar los comandos de extensión de plataforma y garantizar el estado correcto de la extensión dentro de la máquina virtual.

La plataforma de Azure hospeda numerosas extensiones, que abarcan aplicaciones de configuración, supervisión, seguridad y utilidad de máquinas virtuales. Hay una gran variedad de extensiones de primera y de terceros. A continuación, se incluyen ejemplos de escenarios clave para los que se usan las extensiones:
* Compatibilidad con servicios de Azure propios, como Azure Backup, supervisión,Disk Encryption, seguridad, replicación de sitios, etc.
* Restablecimiento de contraseña o SSH
* Configuración de la VM: ejecución de scripts personalizados, instalación de Chef, agentes Puppet, etc.
* Productos de terceros, como productos antivirus, herramientas de vulnerabilidad de VM, herramientas de supervisión de aplicaciones y VM.
* Las extensiones se pueden incluir con una nueva implementación de máquina virtual. Por ejemplo, puede formar parte de una implementación más amplia que configure aplicaciones en el aprovisionamiento de máquinas virtuales o se ejecute en cualquier sistema operado por extensiones admitido tras la implementación.

## <a name="disabling-extension-processing"></a>Deshabilitar el procesamiento de la extensión

Hay varias maneras de deshabilitar el procesamiento de la extensión, en función de sus necesidades, pero antes de continuar, **debe** quitar todas las extensiones implementadas en la VM, por ejemplo, con la CLI de Azure, puede [enumerar](/cli/azure/vm/extension#az_vm_extension_list) y [eliminar](/cli/azure/vm/extension#az_vm_extension_delete):

```azurecli
az vm extension delete -g MyResourceGroup --vm-name MyVm -n extension_name
```
> [!Note]
> 
> Si no hace lo anterior, la plataforma intentará enviar la configuración de la extensión y el tiempo de espera después de 40 min.

### <a name="disable-at-the-control-plane"></a>Deshabilitar en el plano de control
Si no está seguro de si necesitará las extensiones en el futuro, puede dejar el agente Linux instalado en la VM y, a continuación, deshabilitar la capacidad de procesamiento de la extensión de la plataforma. Esta opción está disponible en la versión de la API `Microsoft.Compute` `2018-06-01` o superior y no depende de la versión del agente Linux instalada.

```azurecli
az vm update -g <resourceGroup> -n <vmName> --set osProfile.allowExtensionOperations=false
```
Puede volver a habilitar fácilmente este procesamiento de extensión desde la plataforma, con el comando anterior, pero establézcalo en "true".

## <a name="remove-the-linux-agent-from-a-running-vm"></a>Quitar el agente Linux de una VM en ejecución

Asegúrese de que haber **quitado** todas las extensiones existentes de la VM antes, como se indicó anteriormente.

### <a name="step-1-remove-the-azure-linux-agent"></a>Paso 1: Quitar el agente Linux de Azure

Si solo quita el agente Linux, y no los artefactos de configuración asociados, puede volver a instalarlo después. Ejecute uno de los siguientes, como raíz, para quitar el agente Linux de Azure:

#### <a name="for-ubuntu-1804"></a>Para Ubuntu >=18.04
```bash
apt -y remove walinuxagent
```

#### <a name="for-redhat--77"></a>Para Redhat >= 7.7
```bash
yum -y remove WALinuxAgent
```

#### <a name="for-suse"></a>Para SUSE
```bash
zypper --non-interactive remove python-azure-agent
```

### <a name="step-2-optional-remove-the-azure-linux-agent-artifacts"></a>Paso 2: (Opcional) Quitar los artefactos del agente Linux de Azure
> [!IMPORTANT] 
>
> Puede quitar todos los artefactos asociados del agente Linux, pero esto significa que no podrá volver a instalarlos. Por lo tanto, se recomienda encarecidamente primero deshabilitar el agente Linux y quitar el agente Linux solo con lo anterior. 

Si sabe que nunca volverá a instalar el agente Linux, puede ejecutar lo siguiente:

#### <a name="for-ubuntu-1804"></a>Para Ubuntu >=18.04
```bash
apt -y purge walinuxagent
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-redhat--77"></a>Para Redhat >= 7.7
```bash
yum -y remove WALinuxAgent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-suse"></a>Para SUSE
```bash
zypper --non-interactive remove python-azure-agent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

## <a name="preparing-an-image-without-the-linux-agent"></a>Preparación de una imagen sin el agente Linux
Si tiene una imagen que contiene cloud-init y quiere quitar el agente de Linux, pero todavía aprovisiona con cloud-init, ejecute los pasos del paso 2 (y, opcionalmente, el paso 3) como raíz para quitar el agente Linux de Azure y lo siguiente quitará la configuración de cloud-init y los datos en caché, y preparará la máquina para crear una imagen personalizada.

```bash
cloud-init clean --logs --seed 
```

## <a name="deprovision-and-create-an-image"></a>Desaprovisionamiento y creación de una imagen
El agente Linux tiene la capacidad de limpiar algunos de los metadatos de imagen existentes, con el paso "waagent-deprovision + user". Sin embargo, una vez que se ha quitado, deberá realizar acciones como la que se muestra a continuación y quitar cualquier otro dato confidencial de este.

- Quitar todas las claves de host de ssh existentes

   ```bash
   rm /etc/ssh/ssh_host_*key*
   ```
- Eliminar la cuenta de administrador

   ```bash
   touch /var/run/utmp
   userdel -f -r <admin_user_account>
   ```
- Eliminar la contraseña raíz

   ```bash
   passwd -d root
   ```

Una vez que haya completado lo anterior, puede crear la imagen personalizada mediante la CLI de Azure.


**Creación de una imagen administrada normal**
```azurecli
az vm deallocate -g <resource_group> -n <vm_name>
az vm generalize -g <resource_group> -n <vm_name>
az image create -g <resource_group> -n <image_name> --source <vm_name>
```

**Creación de una versión de imagen en una instancia de Shared Image Gallery**

```azurecli
az sig image-version create \
    -g $sigResourceGroup 
    --gallery-name $sigName 
    --gallery-image-definition $imageDefName 
    --gallery-image-version 1.0.0 
    --managed-image /subscriptions/00000000-0000-0000-0000-00000000xxxx/resourceGroups/imageGroups/providers/images/MyManagedImage
```
### <a name="creating-a-vm-from-an-image-that-does-not-contain-a-linux-agent"></a>Creación de una VM a partir de una imagen que no contiene un agente Linux
Al crear la VM a partir de la imagen sin agente Linux, debe asegurarse de que la configuración de implementación de la VM indica que las extensiones no se admiten en esta VM.

> [!NOTE] 
> 
> Si no hace lo anterior, la plataforma intentará enviar la configuración de la extensión y el tiempo de espera después de 40 min.

Para implementar la VM con extensiones deshabilitadas, puede usar la CLI de Azure con [--enable-Agent](/cli/azure/vm#az_vm_create).

```azurecli
az vm create \
    --resource-group $resourceGroup \
    --name $prodVmName \
    --image RedHat:RHEL:8.1-ci:latest \
    --admin-username azadmin \
    --ssh-key-value "$sshPubkeyPath" \
    --enable-agent false
```

También puede hacerlo con las plantillas de Azure Resource Manager (ARM), al establecer `"provisionVMAgent": false,`.

```json
"osProfile": {
    "computerName": "[parameters('virtualMachineName')]",
    "adminUsername": "[parameters('adminUsername')]",
    "linuxConfiguration": {
        "disablePasswordAuthentication": "true",
        "provisionVMAgent": false,
        "ssh": {
            "publicKeys": [
                {
                    "path": "[concat('/home/', parameters('adminUsername'), '/.ssh/authorized_keys')]",
                    "keyData": "[parameters('adminPublicKey')]"
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Aprovisionamiento de Linux](provisioning.md).
