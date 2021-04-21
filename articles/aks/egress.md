---
title: Uso de la dirección IP estática para el tráfico de salida
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear y usar una dirección IP pública estática para el tráfico de salida en un clúster de Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: 7cff3f5d66bb9872a0c949c6237150f8b69c9fa7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773016"
---
# <a name="use-a-static-public-ip-address-for-egress-traffic-with-a-basic-sku-load-balancer-in-azure-kubernetes-service-aks"></a>Usar una dirección IP pública estática para el tráfico de salida con un equilibrador de carga de la SKU *básico* en Azure Kubernetes Service (AKS)

De forma predeterminada, la dirección IP de salida desde un clúster de Azure Kubernetes Service (AKS) se asigna aleatoriamente. Esta configuración no es la más idónea cuando, por ejemplo, es necesario identificar una dirección IP para acceder a servicios externos. En su lugar, es posible que tenga que asignar una dirección IP estática que se va a agregar a una lista de permitidos de acceso al servicio.

En este artículo se muestra cómo crear y usar una dirección IP pública estática para el tráfico de salida en un clúster de Azure Kubernetes Service (AKS).

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que usa el equilibrador de carga básico de Azure.  Se recomienda usar el [equilibrador de carga estándar de Azure](../load-balancer/load-balancer-overview.md) y puede utilizar características más avanzadas para [controlar el tráfico de salida de AKS](./limit-egress-traffic.md).

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

> [!IMPORTANT]
> En este artículo se usa el equilibrador de carga de la SKU *básica* con un único grupo de nodos. Esta configuración no está disponible para varios grupos de nodos, ya que no se admite el equilibrador de carga de la SKU *básica* con varios grupos de nodos. Consulte [Uso de Standard Load Balancer en Azure Kubernetes Service (AKS)][slb] para más detalles sobre cómo usar el equilibrador de carga de la SKU *estándar*.

## <a name="egress-traffic-overview"></a>Descripción del tráfico de salida

El tráfico saliente de un clúster de AKS sigue las [convenciones de Azure Load Balancer][outbound-connections]. Antes de que se cree el primer servicio Kubernetes de tipo `LoadBalancer`, los nodos de agente en un clúster de AKS no forman parte de ningún grupo de Azure Load Balancer. En esta configuración, los nodos no tienen ninguna dirección IP pública de nivel de instancia. Azure traduce el flujo de salida a una dirección IP de origen pública que no es configurable o determinista.

Una vez que se ha creado un servicio Kubernetes de tipo `LoadBalancer`, los nodos de agente se agregan a un grupo Azure Load Balancer. Load Balancer básico elige un único servidor front-end para usarlo con los flujos de salida cuando hay varios servidores front-end de direcciones IP (públicas) candidatos para los flujos de salida. Esta selección no es configurable y el algoritmo de selección se debe considerar aleatorio. Esta dirección IP pública solo es válida para la duración de ese recurso. Si elimina el servicio Kubernetes LoadBalancer, el equilibrador de carga asociado y la dirección IP también se eliminan. Si quiere asignar una dirección IP específica o conservar una dirección IP para los servicios de Kubernetes reimplementados, puede crear y usar una dirección IP pública estática.

## <a name="create-a-static-public-ip"></a>Creación de una IP pública estática

Obtenga el nombre del grupo de recursos con el comando [az aks show][az-aks-show] y agregue el parámetro de consulta `--query nodeResourceGroup`. En este ejemplo se obtiene el grupo de recursos del nodo para el nombre de clúster de AKS *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

Cree una dirección IP pública estática con el comando [az network public ip create][az-network-public-ip-create]. Especifique el nombre del grupo de recursos del nodo que obtuvo en el comando anterior y, después, un nombre para el recurso de dirección IP, como *myAKSPublicIP*:

```azurecli-interactive
az network public-ip create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name myAKSPublicIP \
    --allocation-method static
```

Se muestra la dirección IP, como se indica en esta salida de ejemplo reducido:

```json
{
  "publicIp": {
    "dnsSettings": null,
    "etag": "W/\"6b6fb15c-5281-4f64-b332-8f68f46e1358\"",
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Network/publicIPAddresses/myAKSPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "40.121.183.52",
    [..]
  }
```

Después puede obtener la dirección IP pública mediante el comando [az network public-ip list][az-network-public-ip-list]. Especifique el nombre del grupo de recursos del nodo y, después, envíe una consulta para *ipAddress*, como se muestra en este ejemplo:

```azurecli-interactive
$ az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query [0].ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-with-the-static-ip"></a>Crear un servicio con la dirección IP estática

Para crear un servicio con la dirección IP pública estática, agregue la propiedad `loadBalancerIP` y el valor de la dirección IP pública estática al manifiesto de YAML. Cree un archivo denominado `egress-service.yaml` y cópielo en el siguiente código YAML. Indique su propia dirección IP pública que creó en el paso anterior.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-egress
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
```

Cree el servicio y la implementación con el comando `kubectl apply`.

```console
kubectl apply -f egress-service.yaml
```

Este servicio configura una nueva dirección IP de front-end en Azure Load Balancer. Si no tiene más direcciones IP configuradas, entonces ahora **todo** el tráfico de salida debería usar esta dirección. Cuando hay varias direcciones configuradas en Azure Load Balancer, cualquiera de estas direcciones IP públicas son candidatas para los flujos de salida y se selecciona una de forma aleatoria.

## <a name="verify-egress-address"></a>Comprobar la dirección de salida

Para comprobar que está usando la dirección IP pública estática, puede usar servicio de búsqueda DNS, como `checkip.dyndns.org`.

Iniciar y asociar un pod *Debian* básico:

```console
kubectl run -it --rm aks-ip --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
```

Para acceder a un sitio web desde dentro del contenedor, use `apt-get` para instalar `curl` en el contenedor.

```console
apt-get update && apt-get install curl -y
```

Use curl para tener acceso al sitio *checkip.dyndns.org*. Se muestra la dirección IP de salida, como se indica en esta salida de ejemplo: Esta dirección IP coincide con la dirección IP pública estática creada y definida para el servicio Azure Load Balancer:

```console
$ curl -s checkip.dyndns.org

<html><head><title>Current IP Check</title></head><body>Current IP Address: 40.121.183.52</body></html>
```

## <a name="next-steps"></a>Pasos siguientes

Para evitar mantener varias direcciones IP públicas en Azure Load Balancer, recomendamos que use un controlador de entrada. Los controladores de entrada proporcionan otras ventajas, como terminación SSL/TLS, compatibilidad con reescritura de URI y cifrado SSL/TLS ascendente. Para más información, consulte [Creación de un controlador de entrada en Azure Kubernetes Service (AKS)][ingress-aks-cluster].

<!-- LINKS - internal -->
[az-network-public-ip-create]: /cli/azure/network/public-ip#az_network_public_ip_create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az_network_public_ip_list
[az-aks-show]: /cli/azure/aks#az_aks_show
[azure-cli-install]: /cli/azure/install-azure-cli
[ingress-aks-cluster]: ./ingress-basic.md
[outbound-connections]: ../load-balancer/load-balancer-outbound-connections.md#scenarios
[public-ip-create]: /cli/azure/network/public-ip#az_network_public_ip_create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[slb]: load-balancer-standard.md
