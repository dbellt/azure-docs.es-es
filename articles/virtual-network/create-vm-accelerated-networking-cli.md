---
title: Creación de una máquina virtual Azure con Accelerated Networking mediante la CLI de Azure
description: Aprenda a crear una máquina virtual Linux con redes aceleradas habilitadas.
services: virtual-network
documentationcenter: na
author: gsilva5
manager: gedegrac
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/10/2019
ms.author: gsilva
ms.custom: ''
ms.openlocfilehash: 4e67ecdb25a64c7e61eb54391a6f09f5f68df435
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755274"
---
# <a name="create-a-linux-virtual-machine-with-accelerated-networking-using-azure-cli"></a>Creación de una máquina virtual Linux con Accelerated Networking con la CLI de Azure

En este tutorial, obtendrá información sobre cómo crear una máquina virtual Linux con Accelerated Networking. Para crear una máquina virtual Windows con Accelerated Networking, consulte [Creación de una máquina virtual Windows con Accelerated Networking](create-vm-accelerated-networking-powershell.md). Accelerated Networking habilita la virtualización de E/S de raíz única (SR-IOV) en una máquina virtual (VM), lo que mejora significativamente su rendimiento en la red. Este método de alto rendimiento omite el host de la ruta de acceso de datos, lo que reduce la latencia, la inestabilidad y la utilización de la CPU para usarse con las cargas de trabajo de red más exigentes en los tipos de máquina virtual admitidos. En la siguiente imagen, se muestra la comunicación entre dos máquinas virtuales (VM) con y sin Accelerated Networking:

![De comparación](./media/create-vm-accelerated-networking/accelerated-networking.png)

Sin Accelerated Networking, todo el tráfico de red de entrada y salida de la máquina virtual tiene que atravesar el host y el conmutador virtual. El conmutador virtual se encarga de toda la aplicación de directivas, como grupos de seguridad de red, listas de control de acceso, aislamiento y otros servicios virtualizados de red, al tráfico de red. Para más información sobre los conmutadores virtuales, lea el artículo sobre [virtualización de red y conmutador virtual de Hyper-V](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj134230(v=ws.11)).

Con Accelerated Networking, el tráfico de red llega a la interfaz de red (NIC) de la máquina virtual y se reenvía después a la máquina virtual. Todas las directivas de red que el conmutador virtual aplica se descargan y aplican en el hardware. La aplicación de directivas en hardware permite que la NIC reenvíe el tráfico de red directamente a la máquina virtual, pasando por alto el host y el conmutador virtual, al mismo tiempo que se mantienen todas las directivas aplicadas en el host.

Las ventajas de Accelerated Networking solo se aplican a la máquina virtual donde esté habilitado. Para obtener resultados óptimos, lo ideal es habilitar esta característica en al menos dos máquinas virtuales conectadas a la misma instancia de Azure Virtual Network (VNet). Al comunicarse entre redes virtuales o conectarse de forma local, esta característica tiene un efecto mínimo sobre la latencia total.

## <a name="benefits"></a>Ventajas
* **Menor latencia/Más paquetes por segundo (pps):** al quitarse el conmutador virtual de la ruta de acceso de datos, se elimina el tiempo que los paquetes pasan en el host para el procesamiento de las directivas y se aumenta el número de paquetes que se pueden procesar dentro de la máquina virtual.
* **Inestabilidad reducida:** el procesamiento del conmutador virtual depende de la cantidad de directivas que deben aplicarse y la carga de trabajo de la CPU que se encarga del procesamiento. Al descargarse la aplicación de directivas en el hardware, se elimina esa variabilidad, ya que los paquetes se entregan directamente a la máquina virtual y se elimina el host de la comunicación de la máquina virtual, así como todas las interrupciones de software y los cambios de contexto.
* **Disminución de la utilización de la CPU:** el pasar por alto el conmutador virtual en el host conlleva una disminución de la utilización de la CPU para procesar el tráfico de red.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Se admiten las siguientes distribuciones de fábrica desde la galería de Azure: 
* **Ubuntu 14.04 con el kernel linux-azure**
* **Ubuntu 16.04 o posterior** 
* **SLES12 SP3 o posterior** 
* **RHEL 7.4 o posterior**
* **CentOS 7.4 o posterior**
* **CoreOS Linux**
* **Debian "Stretch" con kernel backports, Debian "Buster" o posterior**
* **Oracle Linux 7.4 y posterior con Red Hat Compatible Kernel (RHCK)**
* **Oracle Linux 7.5 y posterior con UEK, versión 5**
* **FreeBSD 10.4, 11.1 y 12.0 o posterior**

## <a name="limitations-and-constraints"></a>Limitaciones y restricciones

### <a name="supported-vm-instances"></a>Instancias de máquina virtual admitidas
Accelerated Networking se admite con la mayoría de los tamaños de instancia de uso general y optimizados para procesos de dos o más vCPU. En instancias que admiten hyperthreading, las redes aceleradas se admiten en instancias de máquina virtual con cuatro o más vCPU. 

La compatibilidad con Accelerated Networking se puede encontrar en la documentación de los [tamaños de máquina virtual](../virtual-machines/sizes.md) individuales. 

### <a name="custom-images"></a>Custom Images
Si usa una imagen personalizada compatible con redes aceleradas, asegúrese de disponer de los controladores necesarios para trabajar con las NIC ConnectX-3 y ConnectX-4 Lx de Mellanox en Azure.

### <a name="regions"></a>Regions
Está disponible en todas las regiones públicas de Azure y en las nubes de Azure Government.

<!-- ### Network interface creation 
Accelerated networking can only be enabled for a new NIC. It cannot be enabled for an existing NIC.
removed per issue https://github.com/MicrosoftDocs/azure-docs/issues/9772 -->
### <a name="enabling-accelerated-networking-on-a-running-vm"></a>Habilitación de Accelerated Networking en una máquina virtual en ejecución
Un tamaño de máquina virtual admitido sin tener habilitado Accelerated Networking solo puede tener la característica habilitada cuando se detiene o se desasigna la máquina virtual.  
### <a name="deployment-through-azure-resource-manager"></a>Implementación mediante Azure Resource Manager
Las máquinas virtuales (clásicas) no se pueden implementar con Accelerated Networking.

## <a name="create-a-linux-vm-with-azure-accelerated-networking"></a>Creación de una máquina virtual Linux con Azure Accelerated Networking
## <a name="portal-creation"></a>Creación del portal
Aunque en este artículo se explica cómo crear una máquina virtual con redes aceleradas mediante la CLI de Azure, también puede [crear una máquina virtual con redes aceleradas mediante Azure Portal](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Al crear una máquina virtual en el portal, en la hoja **Crear una máquina virtual**, seleccione la pestaña **Redes**.  En esta pestaña hay una opción para **Redes aceleradas**.  Si ha seleccionado un [sistema operativo compatible](#supported-operating-systems) y un [tamaño de máquina virtual](#supported-vm-instances), esta opción se rellena automáticamente como "Activado".  Si no, se rellena la opción "Desactivado" para Redes aceleradas y se proporciona al usuario un motivo de por qué no están habilitadas.   

* *Nota:* Solo los sistemas operativos compatibles se pueden habilitar a través del portal.  Si usa una imagen personalizada y esta es compatible con Redes aceleradas, cree la máquina virtual mediante la CLI o PowerShell. 

Una vez creada la máquina virtual, puede confirmar que la opción Redes aceleradas está habilitada si sigue las instrucciones que se indican en [Confirmar que las redes aceleradas están habilitadas](#confirm-that-accelerated-networking-is-enabled).

## <a name="cli-creation"></a>Creación de la CLI
### <a name="create-a-virtual-network"></a>Creación de una red virtual

Instale la última versión de la [CLI de Azure](/cli/azure/install-azure-cli) e inicie sesión en una cuenta de Azure con [az login](/cli/azure/reference-index). En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *myNic* y *myVM*.

Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *centralus*:

```azurecli
az group create --name myResourceGroup --location centralus
```

Seleccione una región de Linux admitida enumerada en [Linux accelerated networking](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview) (Accelerated Networking para Linux).

Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En el ejemplo siguiente se crea una red virtual denominada *myVnet* con una subred:

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red
Cree un grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg). En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNetworkSecurityGroup*:

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

El grupo de seguridad de red contiene varias reglas predeterminadas, una de las cuales deshabilita todo el acceso entrante desde Internet. Abra un puerto para permitir el acceso SSH a la máquina virtual con [az network nsg rule create](/cli/azure/network/nsg/rule):

```azurecli
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name Allow-SSH-Internet \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 22
```

### <a name="create-a-network-interface-with-accelerated-networking"></a>Creación de una interfaz de red con Accelerated Networking

Cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip). No se requiere una dirección IP pública si no tiene pensado acceder a la máquina virtual desde Internet, pero, para completar los pasos de este artículo, sí es necesaria.

```azurecli
az network public-ip create \
    --name myPublicIp \
    --resource-group myResourceGroup
```

Cree una interfaz de red con [az network nic create](/cli/azure/network/nic) con Accelerated Networking habilitado. En el ejemplo siguiente se crea una interfaz de red denominada *myNic* en la subred *mySubnet* de la red virtual *myVnet* y asocia el grupo de seguridad de red  *myNetworkSecurityGroup* a la interfaz de red:

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --accelerated-networking true \
    --public-ip-address myPublicIp \
    --network-security-group myNetworkSecurityGroup
```

### <a name="create-a-vm-and-attach-the-nic"></a>Creación de una máquina virtual y conexión de NIC
Cuando cree la máquina virtual, especifique las NIC creadas con `--nics`. Seleccione un tamaño y una distribución enumerados en [Linux accelerated networking](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview) (Accelerated Networking para Linux). 

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM* con la imagen de UbuntuTLS y un tamaño que admite Accelerated Neworking (*Standard_DS4_v2*):

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS4_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic
```

Para obtener una lista de todos los tamaños de máquinas virtuales y las características, consulte [Tamaños de máquina virtual Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Una vez creada la máquina virtual, se devuelve una salida similar a la del siguiente ejemplo. Anote el valor de **publicIpAddress**. Esta dirección se utiliza para acceder a la máquina virtual en los pasos posteriores.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "centralus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "192.168.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

### <a name="confirm-that-accelerated-networking-is-enabled"></a>Confirmación de que Accelerated Networking esté habilitado

Use el siguiente comando para crear una sesión de SSH con la máquina virtual. Reemplace `<your-public-ip-address>` con la dirección IP pública asignada a la máquina virtual que ha creado y reemplace *azureuser* si usó un valor diferente para `--admin-username` cuando creó la máquina virtual.

```bash
ssh azureuser@<your-public-ip-address>
```

En el shell de Bash, escriba `uname -r` y confirme que la versión de kernel es una de las siguientes o una posterior:

* **Ubuntu 16.04**: 4.11.0-1013
* **SLES SP3**: 4.4.92-6.18
* **RHEL**: 3.10.0-693
* **CentOS**: 3.10.0-693


Con el comando `lspci`, confirme que el dispositivo Mellanox VF se expone a la máquina virtual. La salida devuelta será similar a la siguiente:

```output
0000:00:00.0 Host bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX Host bridge (AGP disabled) (rev 03)
0000:00:07.0 ISA bridge: Intel Corporation 82371AB/EB/MB PIIX4 ISA (rev 01)
0000:00:07.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
0000:00:07.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 02)
0000:00:08.0 VGA compatible controller: Microsoft Corporation Hyper-V virtual VGA
0001:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
```

Compruebe la actividad en la función virtual con el comando `ethtool -S eth0 | grep vf_`. Si recibe un resultado similar al siguiente ejemplo de salida, Accelerated Networking red está habilitado y en funcionamiento.

```output
vf_rx_packets: 992956
vf_rx_bytes: 2749784180
vf_tx_packets: 2656684
vf_tx_bytes: 1099443970
vf_tx_dropped: 0
```
Las redes aceleradas ya están habilitadas para su máquina virtual.

## <a name="handle-dynamic-binding-and-revocation-of-virtual-function"></a>Control de la revocación y el enlace dinámicos de la función virtual 
Las aplicaciones deben ejecutarse a través del NIC sintético que se expone en la máquina virtual. Si la aplicación se ejecuta directamente sobre el VF de NIC, no recibirá **todos** los paquetes destinados a la máquina virtual, ya que algunos paquetes se mostrarán sobre la interfaz sintética.
Si ejecuta una aplicación a través del NIC sintético, garantiza que la aplicación recibe **todos** los paquetes que están destinados a él. También se asegura de que la aplicación sigue ejecutándose, incluso si se revoca el VF cuando el host está en mantenimiento. El enlace de aplicaciones al NIC sintético es un requisito **obligatorio** para todas las aplicaciones que usan **redes aceleradas**.

## <a name="enable-accelerated-networking-on-existing-vms"></a>Habilitación de Accelerated Networking en máquinas virtuales existentes
Si ha creado una VM sin Accelerated Networking, es posible habilitar esta característica en una VM existente.  Para admitir Accelerated Networking, la VM debe cumplir los siguientes requisitos previos que también se han descrito anteriormente:

* La VM debe tener un tamaño admitido para Accelerated Networking.
* La VM debe ser una imagen admitida de la galería de Azure (y la versión de kernel de Linux).
* Todas las máquinas virtuales de un conjunto de disponibilidad o VMSS se deben detener o desasignar antes de habilitar Accelerated Networking en cualquier NIC.

### <a name="individual-vms--vms-in-an-availability-set"></a>VM individuales y VM de un conjunto de disponibilidad
En primer lugar, detenga o desasigne la VM o, en el caso de un conjunto de disponibilidad, todas las VM del conjunto:

```azurecli
az vm deallocate \
    --resource-group myResourceGroup \
    --name myVM
```

Es importante tener en cuenta que, si la máquina virtual se creó de forma individual, sin un conjunto de disponibilidad, solo es necesario detener o desasignar esa máquina virtual individual para habilitar Accelerated Networking.  Si la máquina virtual se creó con un conjunto de disponibilidad, será necesario detener o desasignar todas las máquinas virtuales contenidas en el conjunto de disponibilidad antes de habilitar Accelerated Networking en cualquiera de las NIC. 

Una vez detenida, habilite Accelerated Networking en la NIC de la máquina virtual:

```azurecli
az network nic update \
    --name myNic \
    --resource-group myResourceGroup \
    --accelerated-networking true
```

Reinicie la máquina virtual o, en el caso de un conjunto de disponibilidad, todas las máquinas virtuales del conjunto, y confirme que Accelerated Networking está habilitado: 

```azurecli
az vm start --resource-group myResourceGroup \
    --name myVM
```

### <a name="vmss"></a>VMSS
VMSS es ligeramente diferente, pero sigue el mismo flujo de trabajo.  En primer lugar, detenga las VM:

```azurecli
az vmss deallocate \
    --name myvmss \
    --resource-group myrg
```

Una vez detenidas las VM, actualice la propiedad Accelerated Networking bajo la interfaz de red:

```azurecli
az vmss update --name myvmss \
    --resource-group myrg \
    --set virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].enableAcceleratedNetworking=true
```

Tenga en cuenta que un VMSS tiene actualizaciones de VM que aplican actualizaciones mediante tres valores diferentes: automático, gradual y manual.  En estas instrucciones, la directiva se establece en automático para que el VMSS recoja los cambios inmediatamente después de reiniciarse.  Para establecerla en automático para que los cambios se recojan inmediatamente: 

```azurecli
az vmss update \
    --name myvmss \
    --resource-group myrg \
    --set upgradePolicy.mode="automatic"
```

Por último, reinicie el VMSS:

```azurecli
az vmss start \
    --name myvmss \
    --resource-group myrg
```

Después de reiniciar, espere a que las actualizaciones finalicen y la función virtual aparecerá dentro de la VM.  (Asegúrese de que usa un tamaño admitido de sistema operativo y VM).

### <a name="resizing-existing-vms-with-accelerated-networking"></a>Cambio del tamaño de las VM existentes con Accelerated Networking

Las VM con Accelerated Networking habilitado solo pueden cambiar al tamaño de VM que admitan Accelerated Networking.  

Una VM con Accelerated Networking habilitado no puede cambiar al tamaño de una instancia de VM que no admita Accelerated Networking mediante la operación de cambio de tamaño.  Para cambiar el tamaño de una de estas VM, debe hacer lo siguiente: 

* Detener o desasignar la VM o, si se trata de un conjunto de disponibilidad o VMSS, detener o desasignar todas las VM del conjunto o VMSS.
* Accelerated Networking debe estar deshabilitado en la NIC de la VM o, si está en un conjunto de disponibilidad o VMSS, de todas las VM del conjunto o VMSS.
* Una vez deshabilitadas, la máquina virtual, el conjunto de disponibilidad o el VMSS se pueden mover a un nuevo tamaño que no admita redes aceleradas y reiniciarse.
