---
title: Uso del mantenimiento planeado para el clúster de Azure Kubernetes Service (AKS) (versión preliminar)
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre cómo usar el mantenimiento planeado en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/03/2021
ms.author: qpetraroia
author: qpetraroia
ms.openlocfilehash: f1e0822e77d8466b1b9796041fbdba53c3f9c91f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782916"
---
# <a name="use-planned-maintenance-to-schedule-maintenance-windows-for-your-azure-kubernetes-service-aks-cluster-preview"></a>Uso del mantenimiento planeado a fin de programar ventanas de mantenimiento para el clúster de Azure Kubernetes Service (AKS) (versión preliminar)

El clúster de AKS tiene un mantenimiento periódico que se realiza automáticamente. De manera predeterminada, este trabajo puede producirse en cualquier momento. El mantenimiento planeado le permite programar ventanas de mantenimiento semanales que actualizarán el plano de control, así como los pods del sistema Kube en una instancia de VMSS y minimizar el impacto en la carga de trabajo. Una vez programado, todo el mantenimiento se realizará durante la ventana seleccionada. Puede programar una o varias ventanas semanales en el clúster especificando un día o un intervalo de tiempo en un día concreto. Las ventanas de mantenimiento se configuran mediante la CLI de Azure.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="limitations"></a>Limitaciones

Al utilizar el mantenimiento planeado, se aplican las restricciones siguientes:

- AKS se reserva el derecho de interrumpir estas ventanas para las operaciones de mantenimiento no planeadas y reactivas que sean urgentes o críticas.
- Actualmente, la realización de operaciones de mantenimiento se considera *solo el mejor esfuerzo* y no se garantiza que se produzcan en una ventana especificada.
- Las actualizaciones no se pueden bloquear durante más de siete días.

### <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

También se necesita la versión 0.5.4 o posterior de la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="allow-maintenance-on-every-monday-at-100am-to-200am"></a>Permitir mantenimiento cada lunes de 1:00 a. m. a 2:00 a. m.

Para agregar una ventana de mantenimiento, puede usar el comando `az aks maintenanceconfiguration add`.

> [!IMPORTANT]
> Las ventanas de mantenimiento planeado se especifican en hora universal coordinada (UTC).

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday  --start-hour 1
```

En la salida de ejemplo siguiente se muestra la ventana de mantenimiento de 1:00 a. m. a 2:00 a. m. todos los lunes.

```json
{- Finished ..
  "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
  "name": "default",
  "notAllowedTime": null,
  "resourceGroup": "MyResourceGroup",
  "systemData": null,
  "timeInWeek": [
    {
      "day": "Monday",
      "hourSlots": [
        1
      ]
    }
  ],
  "type": null
}
```

Para permitir el mantenimiento a cualquier hora del día, omita el parámetro *start-hour*. Por ejemplo, el comando siguiente establece la ventana de mantenimiento del día completo todos los lunes:

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday
```

## <a name="add-a-maintenance-configuration-with-a-json-file"></a>Incorporación de una configuración de mantenimiento con un archivo JSON

También puede usar un archivo JSON para crear una ventana de mantenimiento en lugar de usar parámetros. Cree un archivo `test.json` con el siguiente contenido:

```json
  {
        "timeInWeek": [
          {
            "day": "Tuesday",
            "hour_slots": [
              1,
              2
            ]
          },
          {
            "day": "Wednesday",
            "hour_slots": [
              1,
              6
            ]
          }
        ],
        "notAllowedTime": [
          {
            "start": "2021-05-26T03:00:00Z",
            "end": "2021-05-30T12:00:00Z"
          }
        ]
}
```

El archivo JSON anterior especifica las ventanas de mantenimiento cada martes de 1:00 a. m. a 3:00 a. m. y todos los miércoles de 1:00 a. m. a 2:00 a. m. y de 6:00 a. m. a 7:00 a. m. También hay una excepción de *2021-05-26T03:00:00Z* a *2021-05-30T12:00:00Z*, donde no se permite el mantenimiento, incluso si se superpone a una ventana de mantenimiento. El comando siguiente agrega las ventanas de mantenimiento de `test.json`.

```azurecli-interactive
az aks maintenanceconfiguration add -g MyResourceGroup --cluster-name myAKSCluster --name default --config-file ./test.json
```

## <a name="update-an-existing-maintenance-window"></a>Actualización de una ventana de mantenimiento existente

Para actualizar una configuración de mantenimiento existente, use el comando `az aks maintenanceconfiguration update`.

```azurecli-interactive
az aks maintenanceconfiguration update -g MyResourceGroup --cluster-name myAKSCluster --name default --weekday Monday  --start-hour 1
```

## <a name="list-all-maintenance-windows-in-an-existing-cluster"></a>Enumeración de todas las ventanas de mantenimiento en un clúster existente

Para ver todas las ventanas de configuración de mantenimiento actuales en el clúster de AKS, use el comando `az aks maintenanceconfiguration list`.

```azurecli-interactive
az aks maintenanceconfiguration list -g MyResourceGroup --cluster-name myAKSCluster
```

En la salida siguiente, puede ver que hay dos ventanas de mantenimiento configuradas para myAKSCluster. Una ventana está para el lunes a la 1:00 a. m. y la otra para el viernes a las 4:00 a. m.

```json
[
  {
    "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
    "name": "default",
    "notAllowedTime": null,
    "resourceGroup": "MyResourceGroup",
    "systemData": null,
    "timeInWeek": [
      {
        "day": "Monday",
        "hourSlots": [
          1
        ]
      }
    ],
    "type": null
  },
  {
    "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/testConfiguration",
    "name": "testConfiguration",
    "notAllowedTime": null,
    "resourceGroup": "MyResourceGroup",
    "systemData": null,
    "timeInWeek": [
      {
        "day": "Friday",
        "hourSlots": [
          4
        ]
      }
    ],
    "type": null
  }
]
```

## <a name="show-a-specific-maintenance-configuration-window-in-an-aks-cluster"></a>Muestra de una ventana de configuración de mantenimiento específica en un clúster de AKS

Para ver una ventana de configuración de mantenimiento específica en el clúster de AKS, use el comando `az aks maintenanceconfiguration show`.

```azurecli-interactive
az aks maintenanceconfiguration show -g MyResourceGroup --cluster-name myAKSCluster --name default
```

En la salida de ejemplo siguiente se muestra la ventana de mantenimiento para *default*:

```json
{
  "id": "/subscriptions/<subscriptionID>/resourcegroups/MyResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/maintenanceConfigurations/default",
  "name": "default",
  "notAllowedTime": null,
  "resourceGroup": "MyResourceGroup",
  "systemData": null,
  "timeInWeek": [
    {
      "day": "Monday",
      "hourSlots": [
        1
      ]
    }
  ],
  "type": null
}
```

## <a name="delete-a-certain-maintenance-configuration-window-in-an-existing-aks-cluster"></a>Eliminación de una determinada ventana de configuración de mantenimiento en un clúster de AKS existente

Para eliminar una ventana de configuración de mantenimiento determinada en el clúster de AKS, use el comando `az aks maintenanceconfiguration delete`.

```azurecli-interactive
az aks maintenanceconfiguration delete -g MyResourceGroup --cluster-name myAKSCluster --name default
```

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a actualizar el clúster de AKS, vea [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].


<!-- LINKS - Internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-aks-install-cli]: /cli/azure/aks#az_aks_install_cli
[az-provider-register]: /cli/azure/provider#az_provider_register
[aks-upgrade]: upgrade-cluster.md
