---
title: RDP en los nodos de Windows Server de AKS
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear una conexión RDP con los nodos de Windows Server de clúster de Azure Kubernetes Service (AKS) para la solución de problemas y las tareas de mantenimiento.
services: container-service
ms.topic: article
ms.date: 06/04/2019
ms.openlocfilehash: 62f29c0550b858e34d888da61f1bd7fbd358f82d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782934"
---
# <a name="connect-with-rdp-to-azure-kubernetes-service-aks-cluster-windows-server-nodes-for-maintenance-or-troubleshooting"></a>Conexión con RDP a los nodos de Windows Server de clúster de Azure Kubernetes Service (AKS) para el mantenimiento o la solución de problemas

Durante el ciclo de vida del clúster de Azure Kubernetes Service (AKS), es posible que necesite acceder a un nodo de Windows Server de AKS. Este acceso podría ser para mantenimiento, recopilación de registros u otras operaciones de solución de problemas. Puede acceder a los nodos de Windows Server de AKS mediante conexión RDP. Como alternativa, si desea usar SSH para acceder a los nodos de Windows Server de AKS y tiene acceso al mismo par de claves que se usó durante la creación del clúster, puede seguir los pasos descritos en [Conexión SSH a los nodos de clúster de Azure Kubernetes Service (AKS)][ssh-steps]. Por motivos de seguridad, los nodos de AKS no están expuestos a Internet.

En este artículo se muestra cómo crear una conexión RDP con un nodo de AKS mediante sus direcciones IP privadas.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se presupone que ya tiene un clúster de AKS con un nodo de Windows Server. Si necesita un clúster de AKS, consulte el artículo sobre la [creación de un clúster de AKS con un contenedor de Windows mediante la CLI de Azure][aks-windows-cli]. Necesitará el nombre de usuario de administrador de Windows y la contraseña para el nodo de Windows Server cuyos problemas quiera solucionar. Si no los recuerda, puede restablecerlos con las instrucciones de [Restablecer los Servicios de Escritorio remoto o su contraseña de administrador en una máquina virtual Windows](/troubleshoot/azure/virtual-machines/reset-rdp). También necesitará un cliente RDP, como [Escritorio remoto de Microsoft][rdp-mac].

También es preciso que esté instalada y configurada la versión 2.0.61 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="deploy-a-virtual-machine-to-the-same-subnet-as-your-cluster"></a>Implementación de una máquina virtual en la misma subred que el clúster

Los nodos de Windows Server del clúster de AKS no tienen direcciones IP accesibles externamente. Para realizar una conexión RDP, puede implementar una máquina virtual con una dirección IP accesible públicamente en la misma subred que los nodos de Windows Server.

En el siguiente ejemplo se crea una máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup*.

En primer lugar, obtenga la subred que usa el grupo de nodos de Windows Server. Para obtener el identificador de la subred, se necesita el nombre de esta. Para obtener el nombre de la subred, se necesita el nombre de la red virtual. Obtenga el nombre de la red virtual mediante una consulta sobre su lista de redes al clúster. Para consultar el clúster, necesita su nombre. Puede obtener todos estos datos si ejecuta lo siguiente en Azure Cloud Shell:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
VNET_NAME=$(az network vnet list -g $CLUSTER_RG --query [0].name -o tsv)
SUBNET_NAME=$(az network vnet subnet list -g $CLUSTER_RG --vnet-name $VNET_NAME --query [0].name -o tsv)
SUBNET_ID=$(az network vnet subnet show -g $CLUSTER_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

Ahora que tiene el SUBNET_ID, ejecute el siguiente comando en la misma ventana de Azure Cloud Shell para crear la máquina virtual:

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2019datacenter \
    --admin-username azureuser \
    --admin-password myP@ssw0rd12 \
    --subnet $SUBNET_ID \
    --query publicIpAddress -o tsv
```

La salida del siguiente ejemplo muestra que la máquina virtual se ha creado correctamente y muestra su dirección IP pública.

```console
13.62.204.18
```

Registre la dirección IP pública de la máquina virtual, la usará en un paso posterior.

## <a name="allow-access-to-the-virtual-machine"></a>Permitir el acceso a la máquina virtual

De forma predeterminada, las subredes de los grupos de nodos de AKS están protegidas con grupos de seguridad de red. Para obtener acceso a la máquina virtual, tendrá que habilitar el acceso en el grupo de seguridad de red.

> [!NOTE]
> Los grupos de seguridad de red se controlan mediante el servicio AKS. El plano de control sobrescribirá en cualquier momento los cambios que realice en el grupo de seguridad de red.
>

En primer lugar, obtenga el grupo de recursos y el nombre del grupo de seguridad de red al que va a agregar la regla:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

A continuación, cree la regla del grupo de seguridad de red:

```azurecli-interactive
az network nsg rule create --name tempRDPAccess --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --priority 100 --destination-port-range 3389 --protocol Tcp --description "Temporary RDP access to Windows nodes"
```

## <a name="get-the-node-address"></a>Obtención de la dirección del nodo

Para administrar un clúster de Kubernetes, usará [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. Para instalar `kubectl` localmente, use el comando [az aks install-cli][az-aks-install-cli]:
    
```azurecli-interactive
az aks install-cli
```

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Enumere la dirección IP interna de los nodos de Windows Server con el comando [kubectl get][kubectl-get]:

```console
kubectl get nodes -o wide
```

La salida del ejemplo siguiente muestra las direcciones IP internas de todos los nodos del clúster, incluidos los nodos de Windows Server.

```console
$ kubectl get nodes -o wide
NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                    KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-42485177-vmss000000   Ready    agent   18h   v1.12.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS          4.15.0-1040-azure   docker://3.0.4
aksnpwin000000                      Ready    agent   13h   v1.12.7   10.240.0.67   <none>        Windows Server Datacenter   10.0.17763.437
```

Registre la dirección IP interna del nodo de Windows Server cuyos problemas desee solucionar, la usará en un paso posterior.

## <a name="connect-to-the-virtual-machine-and-node"></a>Conexión a la máquina virtual y al nodo

Conéctese a la dirección IP pública de la máquina virtual que creó antes mediante un cliente con conexión RDP, como [Escritorio remoto de Microsoft][rdp-mac].

![Imagen de la conexión a la máquina virtual mediante un cliente con conexión RDP](media/rdp/vm-rdp.png)

Una vez conectado a la máquina virtual, conéctese con la *dirección IP interna* del nodo de Windows Server cuyos problemas quiera solucionar con un cliente con conexión RDP desde la máquina virtual.

![Imagen de la conexión al nodo de Windows Server mediante un cliente con conexión RDP](media/rdp/node-rdp.png)

Ahora está conectado a su nodo de Windows Server.

![Imagen de la ventana de cmd en del nodo de Windows Server](media/rdp/node-session.png)

Ahora puede ejecutar los comandos de la solución de problemas en la ventana de *cmd*. Dado que los nodos de Windows Server usan Windows Server Core, no hay una interfaz gráfica de usuario completa ni otras herramientas de la interfaz gráfica de usuario cuando se conecta a un nodo de Windows Server mediante una conexión RDP.

## <a name="remove-rdp-access"></a>Eliminación de la conexión RDP

Cuando haya terminado, cierre la conexión RDP con el nodo de Windows Server y salga de la sesión de RDP para ir a la máquina virtual. Después de salir ambas sesiones de RDP, elimine la máquina virtual con el comando [az vm delete][az-vm-delete]:

```azurecli-interactive
az vm delete --resource-group myResourceGroup --name myVM
```

Y la regla del grupo de seguridad de red:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

```azurecli-interactive
az network nsg rule delete --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --name tempRDPAccess
```

## <a name="next-steps"></a>Pasos siguientes

Si necesita datos adicionales para la solución de problemas, puede [ver los registros del nodo maestro de Kubernetes][view-master-logs] o [Azure Monitor][azure-monitor-containers].

<!-- EXTERNAL LINKS -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[rdp-mac]: https://aka.ms/rdmac

<!-- INTERNAL LINKS -->
[aks-windows-cli]: windows-container-cli.md
[az-aks-install-cli]: /cli/azure/aks#az_aks_install_cli
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-vm-delete]: /cli/azure/vm#az_vm_delete
[azure-monitor-containers]: ../azure-monitor/containers/container-insights-overview.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ssh-steps]: ssh.md
[view-master-logs]: view-master-logs.md