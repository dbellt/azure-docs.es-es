---
title: Intervalos de direcciones IP autorizadas por el servidor de API en Azure Kubernetes Service (AKS)
description: Aprenda a proteger el clúster mediante un intervalo de direcciones IP para acceder al servidor de API en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.openlocfilehash: e97ed38d3491ace658a92e2abfbc7cd417ca9cf1
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110100565"
---
# <a name="secure-access-to-the-api-server-using-authorized-ip-address-ranges-in-azure-kubernetes-service-aks"></a>Protección del acceso al servidor de API con intervalos de direcciones IP autorizadas en Azure Kubernetes Service (AKS)

En Kubernetes, el servidor de API recibe solicitudes para realizar acciones en el clúster, como crear recursos o reducir el número de nodos. El servidor de API es la manera centralizada de interactuar con un clúster y administrarlo. Para mejorar la seguridad del clúster y minimizar los ataques, el servidor de API solo debe ser accesible desde un conjunto limitado de intervalos de direcciones IP.

En este artículo se muestra cómo usar los intervalos IP autorizados por el servidor de API para limitar qué direcciones IP y CIDR pueden acceder al plano de control.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se muestra cómo utilizar la CLI de Azure para crear un clúster de AKS.

Es preciso que esté instalada y configurada la versión 2.0.76 de la CLI de Azure, o cualquier otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

### <a name="limitations"></a>Limitaciones

La característica de intervalos IP autorizados por el servidor de la API tiene las siguientes limitaciones:
- En los clústeres creados después de que los intervalos de direcciones IP autorizados por el servidor de API se hayan pasado de la versión preliminar en octubre de 2019, los intervalos de direcciones IP autorizados del servidor de API solo se admiten en el equilibrador de carga de SKU *Estándar*. Los clústeres existentes con la SKU *Básica* del equilibrador de carga y los intervalos IP autorizados del servidor de API seguirán funcionando tal cual, pero no se podrán migrar a un equilibrador de carga de SKU *Estándar*. Los clústeres existentes también seguirán funcionando si se actualiza la versión de Kubernetes o el plano de control. Los intervalos de direcciones IP autorizados del servidor de API no se admiten para los clústeres privados.
- Esta característica no es compatible con los clústeres que usan la [dirección IP pública por nodo](use-multiple-node-pools.md#assign-a-public-ip-per-node-for-your-node-pools).

## <a name="overview-of-api-server-authorized-ip-ranges"></a>Información general sobre los intervalos de direcciones IP autorizadas por el servidor de API

El servidor de API de Kubernetes es el modo en el que se exponen las API de Kubernetes subyacentes. Este componente proporciona la interacción de las herramientas de administración, como `kubectl` o el panel de Kubernetes. AKS proporciona un plano de control de inquilino único con un servidor de API dedicado, etc. El servidor de API se asigna de forma predeterminada a una dirección IP pública, y el usuario debe regular el acceso mediante el control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) o RBAC de Azure.

Para proteger el acceso al panel de control de AKS y al servidor de API (que, de lo contrario, serían accesibles al público), puede habilitar y usar intervalos de direcciones IP autorizadas. De esta manera, solo se permite la comunicación con el servidor de API a los intervalos de direcciones IP definidas. Las solicitudes al servidor de API procedentes de direcciones IP que no formen parte de este intervalo se bloquean. Siga usando RBAC de Kubernetes o RBAC de Azure para autorizar a los usuarios y las acciones que soliciten.

Para más información sobre el servidor de API y otros componentes del clúster, consulte los [conceptos clave de Kubernetes para AKS][concepts-clusters-workloads].

## <a name="create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled"></a>Creación de un clúster de AKS con la opción de intervalos IP autorizados por el servidor de API habilitada

Cree un clúster con el comando [az aks create][az-aks-create] y especifique el parámetro *`--api-server-authorized-ip-ranges`* para proporcionar una lista de intervalos IP autorizados. Estos intervalos IP suelen ser intervalos de direcciones que usan las redes locales o IP públicas. Al especificar un intervalo de CIDR, comience por la primera dirección IP del intervalo. Por ejemplo, *137.117.106.90/29* es un intervalo válido, pero asegúrese de especificar la primera dirección IP del intervalo, por ejemplo, *137.117.106.88/29*.

> [!IMPORTANT]
> De forma predeterminada, el clúster usa el [equilibrador de carga de SKU estándar][standard-sku-lb], que se puede usar para configurar la puerta de enlace de salida. Cuando se habilitan intervalos IP autorizados por el servidor de API durante la creación del clúster, además de los intervalos especificados también se permite de forma predeterminada la dirección IP pública del clúster. Si especifica *""* o ningún valor para *`--api-server-authorized-ip-ranges`* , se deshabilitarán los intervalos IP autorizados por el servidor de API. Tenga en cuenta que si usa PowerShell, debe usar *`--api-server-authorized-ip-ranges=""`* (con el signo igual) para evitar problemas de análisis.

En el siguiente ejemplo se crea un clúster de nodo único denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*, con la opción de intervalos IP autorizados por el servidor de API habilitada. Los intervalos de direcciones IP permitidos son *73.140.245.0/24*:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --generate-ssh-keys
```

> [!NOTE]
> Debe agregar estos intervalos a una lista de permitidos:
> - La dirección IP pública del firewall
> - Cualquier intervalo que represente redes desde las que se va a administrar el clúster
>
> El límite superior para el número de intervalos IP que se puede especificar es de 200.
>
> Las reglas pueden tardar hasta 2 minutos en propagarse. Espere ese tiempo al probar la conexión.

### <a name="specify-the-outbound-ips-for-the-standard-sku-load-balancer"></a>Especificación de las direcciones IP de salida para el equilibrador de carga de SKU estándar

Al crear un clúster de AKS, si especifica direcciones IP o prefijos de salida para el clúster, estas direcciones o prefijos también se permiten. Por ejemplo:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 73.140.245.0/24 \
    --load-balancer-outbound-ips <publicIpId1>,<publicIpId2> \
    --generate-ssh-keys
```

En el ejemplo anterior, todas las direcciones IP proporcionadas en el parámetro *`--load-balancer-outbound-ip-prefixes`* están permitidas, junto con las direcciones IP en el parámetro *`--api-server-authorized-ip-ranges`* .

También puede especificar el parámetro *`--load-balancer-outbound-ip-prefixes`* para permitir prefijos de direcciones IP del equilibrador de carga de salida.

### <a name="allow-only-the-outbound-public-ip-of-the-standard-sku-load-balancer"></a>Permiso solo para la dirección IP pública de salida del equilibrador de carga de SKU estándar

Cuando se habilitan intervalos IP autorizados por el servidor de API durante la creación del clúster, además de los intervalos especificados también se permite la dirección IP pública de salida para el equilibrador de carga de SKU estándar para el clúster. Para permitir solo la dirección IP pública de salida del equilibrador de carga de SKU estándar, use *0.0.0.0/32* al especificar el parámetro *`--api-server-authorized-ip-ranges`* .

En el ejemplo siguiente, solo se permite la dirección IP pública de salida del equilibrador de carga de SKU estándar y solo se puede acceder al servidor de API desde los nodos del clúster.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --api-server-authorized-ip-ranges 0.0.0.0/32 \
    --generate-ssh-keys
```

## <a name="update-a-clusters-api-server-authorized-ip-ranges"></a>Actualización de los intervalos IP autorizados por el servidor de API de un clúster

Para actualizar los intervalos IP autorizados por el servidor de API en un clúster existente, use el comando [az aks update][az-aks-update] y los parámetros *`--api-server-authorized-ip-ranges`* ,--load-balancer-outbound-ip-prefixes *, *`--load-balancer-outbound-ips`* y or--load-balancer-outbound-ip-prefixes*.

En el siguiente ejemplo se actualizan los intervalos IP autorizados por el servidor de API en el clúster denominado *myAKSCluster* del grupo de recursos denominado *myResourceGroup*. El intervalo de direcciones IP que se va a autorizar es *73.140.245.0/24*:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges  73.140.245.0/24
```

También puede usar *0.0.0.0/32* al especificar el parámetro *`--api-server-authorized-ip-ranges`* para permitir solo la dirección IP pública del equilibrador de carga de SKU estándar.

## <a name="disable-authorized-ip-ranges"></a>Deshabilitación de los intervalos IP autorizados

Para deshabilitar los intervalos IP autorizados, use [az aks update][az-aks-update] y especifique un intervalo vacío para deshabilitar los intervalos IP autorizados del servidor de API. Por ejemplo:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --api-server-authorized-ip-ranges ""
```

## <a name="find-existing-authorized-ip-ranges"></a>Búsqueda de intervalos IP autorizados existentes

Para buscar los intervalos IP que se han autorizado, use [az aks show][az-aks-show] y especifique el nombre del clúster y el grupo de recursos. Por ejemplo:

```azurecli-interactive
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query apiServerAccessProfile.authorizedIpRanges'
```

## <a name="update-disable-and-find-authorized-ip-ranges-using-azure-portal"></a>Actualización, deshabilitación y búsqueda de intervalos IP autorizados mediante Azure Portal

Las operaciones anteriores de agregar, actualizar, buscar y deshabilitar intervalos IP autorizados también se pueden realizar en Azure Portal. Para acceder a ellas, vaya a **Redes** en **Configuración** en la hoja de menús del recurso de clúster.

:::image type="content" source="media/api-server-authorized-ip-ranges/ip-ranges-specified.PNG" alt-text="En un explorador, se muestra la página de Azure Portal de configuración de la red del recurso del clúster. Se resaltan las opciones para establecer el intervalo de direcciones IP especificado y de intervalos IP especificados.":::

## <a name="how-to-find-my-ip-to-include-in---api-server-authorized-ip-ranges"></a>¿Cómo se encuentra la dirección IP que se va a incluir en `--api-server-authorized-ip-ranges`?

Debe agregar las máquinas de desarrollo, las herramientas o las direcciones IP de automatización a la lista de clústeres de AKS de intervalos IP aprobados para tener acceso al servidor de API desde allí. 

Otra opción consiste en configurar una JumpBox con las herramientas necesarias en una subred independiente de la red virtual del firewall. Esto supone que el entorno tiene un firewall con la red correspondiente y que ha agregado las direcciones IP del firewall a los intervalos autorizados. Del mismo modo, si se ha forzado la tunelización de la subred AKS a la subred del firewall, el hecho de tener el jumpbox en la subred del clúster también es correcto.

Agregue otra dirección IP a los intervalos autorizados con el siguiente comando.

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)
# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32
```

>> [!NOTE]
> El ejemplo anterior agrega los intervalos IP autorizados de servidor de API en el clúster. Para deshabilitar los intervalos IP autorizados, use az aks update y especifique un intervalo vacío "". 

Otra opción consiste en usar el comando siguiente en los sistemas de Windows para obtener la dirección IPv4 pública, o bien puede usar los pasos descritos en [Buscar su dirección IP](https://support.microsoft.com/en-gb/help/4026518/windows-10-find-your-ip-address).

```azurepowershell-interactive
Invoke-RestMethod http://ipinfo.io/json | Select -exp ip
```

Para encontrar esta dirección, busque "cuál es mi dirección IP" en un explorador de Internet.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha habilitado intervalos de direcciones IP autorizadas por el servidor de API. Este enfoque forma parte del modo de ejecución segura de un clúster de AKS.

Para más información, consulte [Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)][concepts-security] y [Procedimientos recomendados para la seguridad de clústeres y las actualizaciones en AKS][operator-best-practices-cluster-security].

<!-- LINKS - external -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[dev-spaces-ranges]: ../dev-spaces/configure-networking.md#aks-cluster-network-requirements
[kubenet]: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet

<!-- LINKS - internal -->
[az-aks-update]: /cli/azure/aks#az_aks_update
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-network-public-ip-list]: /cli/azure/network/public-ip#az_network_public_ip_list
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[concepts-security]: concepts-security.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[route-tables]: ../virtual-network/manage-route-table.md
[standard-sku-lb]: load-balancer-standard.md
