---
title: Habilitación del cifrado basado en host en Azure Kubernetes Service (AKS)
description: Aprenda a configurar un cifrado basado en host en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/03/2021
ms.openlocfilehash: 6942a3d445892faf0ea0570561eb06019e841e23
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443207"
---
# <a name="host-based-encryption-on-azure-kubernetes-service-aks-preview"></a>Cifrado basado en host en Azure Kubernetes Service (AKS) (versión preliminar)

Con el cifrado basado en host, los datos almacenados en el host de máquina virtual de las máquinas virtuales de los nodos de agente de AKS se cifran en reposo y se transmiten cifrados al servido Storage. Esto significa que los discos temporales se cifran en reposo con claves administradas por la plataforma. La memoria caché de los discos de datos y del sistema operativo se cifra en reposo con claves administradas por la plataforma o por el cliente, según el tipo de cifrado establecido en esos discos. De forma predeterminada, cuando se usa AKS, el sistema operativo y los discos de datos se cifran en reposo con claves administradas por la plataforma, lo que significa que las memorias caché de estos discos también se cifran de forma predeterminada en reposo con claves administradas por la plataforma.  Puede especificar sus propias claves administradas siguiendo [Traiga sus propias claves (BYOK) con discos de Azure en Azure Kubernetes Service (AKS)](azure-disk-customer-managed-keys.md). La memoria caché de estos discos también se cifrará con la clave que especifique en este paso.


## <a name="before-you-begin"></a>Antes de empezar

Esta característica solo se puede establecer durante la creación del clúster o en el momento de creación del grupo de nodos.

> [!NOTE]
> El cifrado basado en host está disponible en [regiones de Azure][supported-regions] que admiten el cifrado del lado servidor de discos administrados de Azure y solo con [tamaños de máquinas virtuales compatibles][supported-sizes] específicos.

### <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que tiene instalada la extensión de la CLI `aks-preview` versión v0.4.73 o posterior.
- Asegúrese de que tiene habilitada la marca de características `EnableEncryptionAtHostPreview` bajo `Microsoft.ContainerService`.

Debe habilitar la característica para su suscripción antes de usar la propiedad EncryptionAtHost para el clúster de Azure Kubernetes Service. Siga los pasos que se indican a continuación para habilitar la característica para su suscripción:

1. Ejecute el siguiente comando para registrar la característica para su suscripción

```azurecli-interactive
Register-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute"
```
2. Compruebe que el estado de registro es Registrado (tarda unos minutos) mediante el comando siguiente antes de probar la característica.

```azurecli-interactive
Get-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute"
```

### <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

Para crear un clúster de AKS que realice el cifrado basado en host, requiere la extensión de la CLI *aks-preview* más reciente. Instale la extensión de la CLI de Azure *aks-preview* con el comando [az extension add][az-extension-add] o busque las actualizaciones disponibles con el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Limitaciones

- Solo se puede habilitar en grupos de nodos nuevos.
- Solo se puede habilitar en [regiones de Azure][supported-regions] que admiten el cifrado del lado servidor de discos administrados de Azure y solo con [tamaños de máquinas virtuales compatibles][supported-sizes] específicos.
- Requiere un clúster de AKS y un grupo de nodos basado en Virtual Machine Scale Sets (VMSS) como *tipo de conjunto de máquinas virtuales*.

## <a name="use-host-based-encryption-on-new-clusters-preview"></a>Uso del cifrado basado en host en clústeres nuevos (versión preliminar)

Configure los nodos de agente de clúster para usar el cifrado basado en host cuando se cree el clúster. 

```azurecli-interactive
az aks create --name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --enable-encryption-at-host
```

Si quiere crear clústeres sin el cifrado basado en host, puede omitir el parámetro `--enable-encryption-at-host` para hacerlo.

## <a name="use-host-based-encryption-on-existing-clusters-preview"></a>Uso del cifrado basado en host en clústeres existentes (versión preliminar)

Puede habilitar el cifrado basado en host en clústeres existentes agregando un nuevo grupo de nodos al clúster. Configure un grupo de nodos nuevo para usar el cifrado basado en host mediante el parámetro `--enable-encryption-at-host`.

```azurecli
az aks nodepool add --name hostencrypt --cluster-name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --enable-encryption-at-host
```

Si quiere crear grupos de nodos nuevos sin la característica de cifrado basado en host, puede hacerlo omitiendo el parámetro `--enable-encryption-at-host`.

## <a name="next-steps"></a>Pasos siguientes

Revise las [prácticas recomendadas para la seguridad del clúster de AKS][best-practices-security]. Lea más sobre el [cifrado basado en host](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).


<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[supported-regions]: ../virtual-machines/disk-encryption.md#supported-regions
[supported-sizes]: ../virtual-machines/disk-encryption.md#supported-vm-sizes
[azure-cli-install]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
