---
title: Creación y configuración de un plan de Azure DDoS Protection mediante la CLI de Azure
description: Aprenda a crear un plan de DDoS Protection con la CLI de Azure
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.openlocfilehash: 98c71f3cf1c521c08d177acb89aad85301e61579
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103018"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-cli"></a>Inicio rápido: Creación y configuración de Azure DDoS Protection Estándar mediante la CLI de Azure

Introducción a Azure DDoS Protection Estándar mediante la CLI de Azure. 

Un plan de protección contra DDoS define un conjunto de redes virtuales que tiene habilitada la protección contra DDoS estándar en distintas suscripciones. Puede configurar un plan de protección contra DDoS para la organización y vincular redes virtuales de distintas suscripciones al mismo plan. 

En este inicio rápido, creará un plan de protección contra DDoS y lo vinculará a una red virtual. 

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- CLI de Azure instalada localmente o Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-ddos-protection-plan"></a>Creación de un plan de DDoS Protection

En Azure, puede asignar recursos relacionados a un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo.

Para crear un grupo de recursos, use [az group create](/cli/azure/group#az-group-create). En este ejemplo, se asignará el nombre _MyResourceGroup_ al grupo de recursos y se usará la ubicación _Este de EE. UU._ :

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus
```

Ahora, cree un plan de protección contra DDoS denominado _MyDdosProtectionPlan_:

```azurecli-interactive
az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

## <a name="enable-ddos-protection-for-a-virtual-network"></a>Habilitación de la protección contra DDoS en una red virtual

### <a name="enable-ddos-protection-for-a-new-virtual-network"></a>Habilitación de la protección contra DDoS para una red virtual nueva

Puede habilitar la protección contra DDoS al crear una red virtual. En este ejemplo, se asignará el nombre _MyVnet_ a la red virtual: 

```azurecli-interactive
az network vnet create \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --location eastus \
    --ddos-protection true
    --ddos-protection-plan MyDdosProtectionPlan
```

No puede mover una red virtual a otro grupo de recursos ni a otra suscripción si DDoS Standard está habilitado para la red virtual. Si tiene que mover una red virtual que tenga habilitado DDoS Standard, primero deshabilítelo, mueva la red virtual y, luego, habilite DDoS Standard. Después de eso, se restablecen los umbrales de directiva ajustados automáticamente para todas las direcciones IP públicas protegidas en la red virtual.

### <a name="enable-ddos-protection-for-an-existing-virtual-network"></a>Habilitación de la protección contra DDoS para una red virtual existente

Al [crear un plan de protección contra DDoS](#create-a-ddos-protection-plan), puede asociarle una o varias redes virtuales. Para agregar más de una red virtual, solo tiene que enumerar los nombres o los identificadores, separados por espacios. En este ejemplo se agregará _MyVnet_:

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus

az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
    --vnets MyVnet
```

Como alternativa, puede habilitar la protección contra DDoS para una red virtual concreta:

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --ddos-protection true
    --ddos-protection-plan MyDdosProtectionPlan
```

## <a name="validate-and-test"></a>Validación y pruebas

En primer lugar, compruebe los detalles del plan de protección contra DDoS:

```azurecli-interactive
az network ddos-protection show \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

Compruebe que el comando devuelve los detalles correctos del plan de protección contra DDoS.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede mantener los recursos para el tutorial siguiente. Si ya no lo necesita, elimine el grupo de recursos _MyResourceGroup_. Al eliminar el grupo de recursos, también elimina el plan de protección contra DDoS y todos sus recursos relacionados. 

Para eliminar el grupo de recursos, use [az group delete](/cli/azure/group#az_group_delete):

```azurecli-interactive
az group delete \
--name MyResourceGroup 
```

Actualice una red virtual concreta para deshabilitar la protección contra DDoS:

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --ddos-protection false
    --ddos-protection-plan ""
```

Si quiere eliminar un plan de protección contra DDoS, primero debe desasociar todas las redes virtuales que tenga. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo ver y configurar la telemetría del plan de protección contra DDoS, continúe con los tutoriales.

> [!div class="nextstepaction"]
> [Visualización y configuración de la telemetría de protección contra DDoS](telemetry.md)
