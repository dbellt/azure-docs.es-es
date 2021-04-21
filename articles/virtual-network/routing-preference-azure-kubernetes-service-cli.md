---
title: Configuración de preferencias de enrutamiento para una instancia de Azure Kubernetes Service mediante la CLI de Azure
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo configurar un clúster de AKS con preferencias de enrutamiento mediante la CLI de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2021
ms.author: mnayak
ms.openlocfilehash: 9eaad12e254150109498be0fac2f285f33a5965c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776612"
---
# <a name="configure-routing-preference-for-a-kubernetes-cluster-using-azure-cli"></a>Configuración de preferencias de enrutamiento de un clúster de Kubernetes mediante la CLI de Azure

En este artículo se muestra cómo configurar las preferencias de enrutamiento mediante la red del ISP (opción de **Internet**) para un clúster de Kubernetes mediante la CLI de Azure. La preferencia de enrutamiento se establece mediante la creación de una dirección IP pública de tipo preferencia de enrutamiento **Internet**** que se utiliza después al crear el clúster de AKS.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.49 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). En el siguiente ejemplo se crea un grupo de recursos en la región **Este de EE. UU.** de Azure:

```azurecli
  az group create --name myResourceGroup --location eastus
```
## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Cree una dirección IP pública con preferencia de enrutamiento de tipo **Internet** mediante el comando [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create).

El siguiente comando crea una nueva dirección IP pública con preferencia de enrutamiento de **Internet** en la región de Azure del **Este de EE. UU.** .

```azurecli
az network public-ip create \
--name MyRoutingPrefIP \
--resource-group MyResourceGroup \
--location eastus \
--ip-tags 'RoutingPreference=Internet' \
--sku STANDARD \
--allocation-method static \
--version IPv4
```
> [!NOTE]
>  Actualmente, la preferencia de enrutamiento solo admite direcciones IP públicas IPV4.

## <a name="get-the-id-of-public-ip-address"></a>Obtención del identificador de la dirección IP pública

El comando siguiente devuelve el identificador de la dirección IP pública que se creó en la sección anterior:
```azurecli
az network public-ip show \
--resource-group myResourceGroup \
--name myRoutingPrefIP \
--query id
```
## <a name="create-kubernetes-cluster-with-the-public-ip"></a>Creación de un clúster de Kubernetes con la dirección IP pública

El comando siguiente crea el clúster de AKS con la dirección IP pública que se creó en la sección anterior:

```azurecli
az aks create \
--resource-group MyResourceGroup \
--name MyAKSCluster \
--load-balancer-outbound-ips "Enter the public IP ID from previous step" --generate-ssh-key
```

>[!NOTE]
>El clúster de AKS tarda unos minutos en implementarse.

Para realizar la validación, busque la dirección IP pública creada en el paso anterior en Azure Portal; verá que la dirección IP está asociada con el equilibrador de carga asociado al clúster de Kubernetes, como se muestra a continuación:

 ![Dirección IP pública de preferencias de enrutamiento para Kubernetes](./media/routing-preference-azure-kubernetes-service-cli/routing-preference-azure-kubernetes-service.png)


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [preferencia de enrutamiento en direcciones IP públicas](routing-preference-overview.md). 
- [Configure la preferencia de enrutamiento de una VM mediante la CLI de Azure](configure-routing-preference-virtual-machine-cli.md).
