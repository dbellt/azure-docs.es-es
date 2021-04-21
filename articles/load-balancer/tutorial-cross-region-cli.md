---
title: 'Tutorial: Creación de un equilibrador de carga entre regiones mediante la CLI de Azure'
titleSuffix: Azure Load Balancer
description: Comience a trabajar con este tutorial para la implementación de Azure Load Balancer entre regiones con la CLI de Azure.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 03/04/2021
ms.openlocfilehash: ca4134ff25dc9915f256b5a7bdd9404021b60a8e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791920"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-azure-cli"></a>Tutorial: Creación de una instancia de Azure Load Balancer entre regiones mediante la CLI de Azure

Un equilibrador de carga entre regiones garantiza que un servicio está disponible globalmente en varias regiones de Azure. Si se produce un error en una región, el tráfico se enruta al siguiente equilibrador de carga regional correcto más cercano.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un equilibrador de carga entre regiones.
> * Cree una regla de equilibrador de carga.
> * Crear un grupo de back-end que contenga dos equilibradores de carga regionales.
> * Probar el equilibrador de carga.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure.
- Dos instancias de Azure Load Balancer con SKU **Estándar** con grupos de back-end implementados en dos regiones de Azure diferentes.
    - Para obtener información sobre cómo crear un equilibrador de carga estándar regional y máquinas virtuales para los grupos de back-end, consulte [Inicio rápido: Creación de un equilibrador de carga público para equilibrar la carga de las VM con la CLI de Azure](quickstart-load-balancer-standard-public-cli.md).
        - Anexe **-R1** y **-R2** al nombre de los equilibradores de carga y las máquinas virtuales de cada región. 
- La CLI de Azure instalada localmente o Azure Cloud Shell.

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="sign-in-to-azure-cli"></a>Inicio de sesión en la CLI de Azure

Inicie sesión en la CLI de Azure:

```azurecli-interactive
az login
```

## <a name="create-cross-region-load-balancer"></a>Creación de un equilibrador de carga entre regiones

En esta sección va a crear un equilibrador de carga entre regiones, una dirección IP pública y una regla de equilibrio de carga.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

* Con el nombre **myResourceGroupLB-CR**.
* En la ubicación **westus**.

```azurecli-interactive
  az group create \
    --name myResourceGroupLB-CR \
    --location westus
```

### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga entre regiones con [az network cross-region-lb create](/cli/azure/network/cross-region-lb#az_network_cross_region_lb_create):

* Llamado **myLoadBalancer-CR**.
* Un grupo de front-end llamado **myFrontEnd-CR**.
* Un grupo de back-end llamado **myBackEndPool-CR**.

```azurecli-interactive
  az network cross-region-lb create \
    --name myLoadBalancer-CR \
    --resource-group myResourceGroupLB-CR \
    --frontend-ip-name myFrontEnd-CR \
    --backend-pool-name myBackEndPool-CR     
```

### <a name="create-the-load-balancer-rule"></a>Creación de la regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP del front-end para el tráfico entrante.
* El grupo de IP de back-end para recibir el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla del equilibrador de carga con [az network cross-region-lb rule create](/cli/azure/network/cross-region-lb/rule#az_network_cross_region_lb_rule_create):

* Llamada **myHTTPRule-CR**.
* Que escuche en el **puerto 80** en el grupo de front-end **myFrontEnd-CR**.
* Que envíe el tráfico de red con equilibrio de carga al grupo de direcciones de back-end **myBackEndPool-CR** mediante el **puerto 80**. 
* Protocolo **TCP**.

```azurecli-interactive
  az network cross-region-lb rule create \
    --backend-port 80 \
    --frontend-port 80 \
    --lb-name myLoadBalancer-CR \
    --name myHTTPRule-CR \
    --protocol tcp \
    --resource-group myResourceGroupLB-CR \
    --backend-pool-name myBackEndPool-CR \
    --frontend-ip-name myFrontEnd-CR
```

## <a name="create-backend-pool"></a>Creación de un grupo de back-end

En esta sección va a agregar dos equilibradores de carga estándar regionales al grupo de back-end del equilibrador de carga entre regiones.

> [!IMPORTANT]
> Para completar estos pasos, asegúrese de que se han implementado en la suscripción dos equilibradores de carga regionales con grupos de back-end.  Para más información, consulte **[Inicio rápido: Creación de un equilibrador de carga público para equilibrar la carga de las VM con la CLI de Azure](quickstart-load-balancer-standard-public-cli.md)** .

### <a name="add-the-regional-frontends-to-load-balancer"></a>Adición de los servidores front-end regionales al equilibrador de carga

En esta sección, colocará los identificadores de recurso de dos servidores front-end de equilibradores de carga regionales en variables.  A continuación, usará las variables para agregar los front-ends al grupo de direcciones de back-end del equilibrador de carga entre regiones.

Recupere los identificadores de recurso con [az network lb frontend-ip show](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_show).

Use [az network cross-region-lb address-pool address add](/cli/azure/network/cross-region-lb/address-pool/address#az_network_cross_region_lb_address_pool_address_add) para agregar los front-ends que colocó en variables en el grupo de back-end del equilibrador de carga entre regiones:

```azurecli-interactive
  region1id=$(az network lb frontend-ip show \
    --lb-name myLoadBalancer-R1 \
    --name myFrontEnd-R1 \
    --resource-group CreatePubLBQS-rg-r1 \
    --query id \
    --output tsv)

  az network cross-region-lb address-pool address add \
    --frontend-ip-address $region1id \
    --lb-name myLoadBalancer-CR \
    --name myFrontEnd-R1 \
    --pool-name myBackEndPool-CR \
    --resource-group myResourceGroupLB-CR

  region2id=$(az network lb frontend-ip show \
    --lb-name myLoadBalancer-R2 \
    --name myFrontEnd-R2 \
    --resource-group CreatePubLBQS-rg-r2 \
    --query id \
    --output tsv)
  
  az network cross-region-lb address-pool address add \
    --frontend-ip-address $region2id \
    --lb-name myLoadBalancer-CR \
    --name myFrontEnd-R2 \
    --pool-name myBackEndPool-CR \
    --resource-group myResourceGroupLB-CR
```

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

En esta sección va a probar el equilibrador de carga entre regiones. Se conectará a la dirección IP pública en un explorador web.  Detendrá las máquinas virtuales de uno de los grupos de back-end del equilibrador de carga regional y observará la conmutación por error.

1. Para obtener la dirección IP pública del equilibrador de carga, use [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show):

    ```azurecli-interactive
      az network public-ip show \
        --resource-group myResourceGroupLB-CR \
        --name PublicIPmyLoadBalancer-CR \
        --query ipAddress \
        --output tsv
    ```
2. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

3. Detenga las máquinas virtuales del grupo de back-end de uno de los equilibradores de carga regionales.

4. Actualice el explorador web y observe la conmutación por error de la conexión al otro equilibrador de carga regional.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use el comando [az group delete](/cli/azure/group#az_group_delete) para quitar el grupo de recursos, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete \
    --name myResourceGroupLB-CR
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:

* Creado un equilibrador de carga entre regiones.
* Creado una regla de equilibrio de carga.
* Agregado equilibradores de carga regionales al grupo de back-end del equilibrador de carga entre regiones.
* Probó el equilibrador de carga.

Avance al siguiente artículo para obtener información sobre...
> [!div class="nextstepaction"]
> [Equilibrio de carga de máquinas virtuales en distintas zonas de disponibilidad](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
