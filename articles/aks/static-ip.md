---
title: Uso de una dirección IP estática con el equilibrador de carga
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear y usar una dirección IP estática con el equilibrador de carga de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 11/14/2020
ms.openlocfilehash: bb1e5691027a4bd86b57390e12259ac165ca9ed8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769524"
---
# <a name="use-a-static-public-ip-address-and-dns-label-with-the-azure-kubernetes-service-aks-load-balancer"></a>Uso de una dirección IP pública estática y una etiqueta DNS con el equilibrador de carga de Azure Kubernetes Service (AKS)

De forma predeterminada, la dirección IP pública asignada a un recurso de equilibrador de carga creado por un clúster de AKS solo es válida para la duración de ese recurso. Si elimina el servicio de Kubernetes, el equilibrador de carga asociado y la dirección IP también se eliminan. Si quiere asignar una dirección IP específica o conservar una dirección IP para los servicios de Kubernetes reimplementados, puede crear y usar una dirección IP pública estática.

En este artículo se muestra cómo crear una dirección IP pública estática y asignarla al servicio de Kubernetes.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

En este artículo se describe el uso de una dirección IP de SKU *Estándar* con un equilibrador de carga de SKU *Estándar*. Para más información, consulte [Tipos de direcciones IP y métodos de asignación en Azure][ip-sku].

## <a name="create-a-static-ip-address"></a>Crear una dirección IP estática

Cree una dirección IP pública estática con el comando [az network public ip create][az-network-public-ip-create]. A continuación se crea un recurso de IP estática denominado *myAKSPublicIP* en el grupo de recursos *myResourceGroup*:

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroup \
    --name myAKSPublicIP \
    --sku Standard \
    --allocation-method static
```

> [!NOTE]
> Si usa un equilibrador de carga de SKU *Básico* en el clúster de AKS, use *Basic* para el parámetro *SKU* al definir una dirección IP pública. Solo las direcciones IP de SKU *Básicas* funcionan con el equilibrador de carga de SKU *Básico* y solo las IP de SKU *Estándar* funcionan con los equilibradores de carga de SKU *Estándar*. 

Se muestra la dirección IP, como se indica en esta salida de ejemplo reducido:

```json
{
  "publicIp": {
    ...
    "ipAddress": "40.121.183.52",
    ...
  }
}
```

Después puede obtener la dirección IP pública mediante el comando [az network public-ip list][az-network-public-ip-list]. Especifique el nombre del grupo de recursos del nodo y la dirección IP pública creados y envíe una consulta para *ipAddress*, como se muestra en este ejemplo:

```azurecli-interactive
$ az network public-ip show --resource-group myResourceGroup --name myAKSPublicIP --query ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-using-the-static-ip-address"></a>Crear un servicio mediante la dirección IP estática

Antes de crear un servicio, compruebe que la identidad de clúster que usa el clúster de AKS tenga permisos delegados para el otro grupo de recursos. Por ejemplo:

```azurecli-interactive
az role assignment create \
    --assignee <Client ID> \
    --role "Network Contributor" \
    --scope /subscriptions/<subscription id>/resourceGroups/<resource group name>
```

> [!IMPORTANT]
> Si ha personalizado la dirección IP de salida, asegúrese de que la identidad del clúster tenga permisos tanto para la dirección IP pública de salida como para esta dirección IP pública de entrada.

Para crear un servicio *LoadBalancer* con la dirección IP pública estática, agregue la propiedad `loadBalancerIP` y el valor de la dirección IP pública estática al manifiesto YAML. Cree un archivo denominado `load-balancer-service.yaml` y cópielo en el siguiente código YAML. Indique su propia dirección IP pública que creó en el paso anterior. En el ejemplo siguiente también se establece el grupo de recursos denominado *myResourceGroup*. Indique su propio nombre del grupo de recursos.

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-resource-group: myResourceGroup
  name: azure-load-balancer
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

Cree el servicio y la implementación con el comando `kubectl apply`.

```console
kubectl apply -f load-balancer-service.yaml
```

## <a name="apply-a-dns-label-to-the-service"></a>Aplicación de una etiqueta DNS al servicio

Si el servicio usa una dirección IP pública dinámica o estática, puede usar la anotación de servicio `service.beta.kubernetes.io/azure-dns-label-name` para establecer una etiqueta DNS de acceso público. Esta acción publica un nombre de dominio completo para el servicio mediante los servidores DNS públicos de Azure y el dominio de nivel superior. El valor de la anotación debe ser único dentro de la ubicación de Azure, por lo que se recomienda usar una etiqueta suficientemente descriptiva.   

A continuación, Azure anexará automáticamente una subred predeterminada, como `<location>.cloudapp.azure.com` (donde location es la región seleccionada), al nombre que proporcione para crear el nombre DNS completo. Por ejemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-dns-label-name: myserviceuniquelabel
  name: azure-load-balancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

> [!NOTE] 
> Para publicar el servicio en su propio dominio, consulte [Azure DNS][azure-dns-zone] y el proyecto [external-dns][external-dns].

## <a name="troubleshoot"></a>Solución de problemas

Si la dirección IP estática definida en la propiedad *loadBalancerIP* del manifiesto de servicio de Kubernetes no existe o no se ha creado en el grupo de recursos del nodo y no se han configurado delegaciones adicionales, se produce un error en la creación del servicio del equilibrador de carga. Para solucionar este problema, revise los eventos de creación del servicio con el comando [kubectl describe][kubectl-describe]. Indique el nombre del servicio tal y como aparece en el manifiesto de YAML, como se muestra en este ejemplo:

```console
kubectl describe service azure-load-balancer
```

Se muestra información sobre el recurso de servicio de Kubernetes. Los *Eventos* al final de la salida de este ejemplo indican que *no se encontró la dirección IP proporcionada por el usuario*. En estos casos, compruebe que ha creado la dirección IP pública estática en el grupo de recursos del nodo y que la dirección IP especificada en el manifiesto de servicio de Kubernetes es correcta.

```
Name:                     azure-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=azure-load-balancer
Type:                     LoadBalancer
IP:                       10.0.18.125
IP:                       40.121.183.52
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32582/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                      Age               From                Message
  ----     ------                      ----              ----                -------
  Normal   CreatingLoadBalancer        7s (x2 over 22s)  service-controller  Creating load balancer
  Warning  CreatingLoadBalancerFailed  6s (x2 over 12s)  service-controller  Error creating load balancer (will retry): Failed to create load balancer for service default/azure-load-balancer: user supplied IP Address 40.121.183.52 was not found
```

## <a name="next-steps"></a>Pasos siguientes

Para un control adicional sobre el tráfico de red a las aplicaciones, puede ser conveniente [crear un controlador de entrada][aks-ingress-basic]. También puede [crear un controlador de entrada con una dirección IP pública estática][aks-static-ingress].

<!-- LINKS - External -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[azure-dns-zone]: https://azure.microsoft.com/services/dns/
[external-dns]: https://github.com/kubernetes-sigs/external-dns

<!-- LINKS - Internal -->
[aks-faq-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[az-network-public-ip-create]: /cli/azure/network/public-ip#az_network_public_ip_create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az_network_public_ip_list
[az-aks-show]: /cli/azure/aks#az_aks_show
[aks-ingress-basic]: ingress-basic.md
[aks-static-ingress]: ingress-static-ip.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ip-sku]: ../virtual-network/public-ip-addresses.md#sku
