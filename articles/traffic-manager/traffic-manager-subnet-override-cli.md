---
title: Invalidación de la subred de Azure Traffic Manager mediante la CLI de Azure | Microsoft Docs
description: Este artículo lo ayudará a entender cómo la invalidación de la subred de Traffic Manager se puede usar para reemplazar el método de enrutamiento de un perfil de Traffic Manager para dirigir el tráfico a un punto de conexión en función de la dirección IP del usuario final a través de un intervalo IP predefinido a las asignaciones de punto de conexión.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: fab167884d9060edc4f626d3ee05fa0b23389d92
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767806"
---
# <a name="traffic-manager-subnet-override-using-azure-cli"></a>Invalidación de la subred de Traffic Manager mediante la CLI de Azure

La invalidación de la subred de Traffic Manager le permite modificar el método de enrutamiento de un perfil.  La incorporación de una invalidación dirigirá el tráfico en función de la dirección IP del usuario final con un intervalo IP predefinido a una asignación de punto de conexión. 

## <a name="how-subnet-override-works"></a>Funcionamiento de la invalidación de subred

Cuando se agregan invalidaciones de subred a un perfil de Traffic Manager, Traffic Manager comprobará primero si existe una invalidación de subred para la dirección IP del usuario final. Si se encuentra una, la consulta de DNS del usuario se dirigirá al punto de conexión correspondiente.  Si no se encuentra una asignación, Traffic Manager revertirá al método de enrutamiento original del perfil. 

Los intervalos de direcciones IP se pueden especificar como intervalos CIDR (por ejemplo, 1.2.3.0/24) o como intervalos de direcciones (por ejemplo, 1.2.3.4-5.6.7.8). Los intervalos IP asociados a cada punto de conexión deben ser únicos para ese punto de conexión. Cualquier superposición de intervalos IP entre distintos puntos de conexión hará Traffic Manager rechace el perfil.

Hay dos tipos de perfiles de enrutamiento que admiten invalidaciones de subred:

* **Geográfico**: si Traffic Manager encuentra una invalidación de subred para la dirección IP de la consulta de DNS, enrutará la consulta al punto de conexión, independientemente del estado del punto de conexión.
* **Rendimiento**: si Traffic Manager encuentra una invalidación de subred para la dirección IP de la consulta de DNS, solo enrutará el tráfico al punto de conexión si tiene un estado correcto.  Traffic Manager volverá a la heurística de enrutamiento del rendimiento si el punto de conexión de invalidación de subred no tiene un estado correcto.

## <a name="create-a-traffic-manager-subnet-override"></a>Creación de una invalidación de subred de Traffic Manager

Para crear una invalidación de subred de Traffic Manager, puede usar la CLI de Azure para agregar las subredes para la invalidación al punto de conexión de Traffic Manager.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.28 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="update-the-traffic-manager-endpoint-with-subnet-override"></a>Actualice el punto de conexión de Traffic Manager con la invalidación de subred.
Use la CLI de Azure para actualizar el punto de conexión con [az network traffic-manager endpoint update](/cli/azure/network/traffic-manager/endpoint#az_network_traffic_manager_endpoint_update).

```azurecli-interactive
### Add a range of IPs ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 1.2.3.4-5.6.7.8 \
    --type AzureEndpoints

### Add a subnet ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 9.10.11.0:24 \
    --type AzureEndpoints
```

Puede quitar los intervalos de direcciones IP si ejecuta [az network traffic-manager endpoint update](/cli/azure/network/traffic-manager/endpoint#az_network_traffic_manager_endpoint_update) con la opción **--remove**.

```azurecli-interactive
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --remove subnets \
    --type AzureEndpoints
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los [métodos de enrutamiento del tráfico](traffic-manager-routing-methods.md) de Traffic Manager.

Obtenga más información sobre el [método de enrutamiento de tráfico de subred](./traffic-manager-routing-methods.md#subnet-traffic-routing-method).