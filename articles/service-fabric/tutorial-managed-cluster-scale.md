---
title: Escalado horizontal de un clúster administrado de Service Fabric
description: En este tutorial aprenderá a escalar horizontalmente un tipo de nodo de un clúster administrado de Service Fabric.
ms.topic: tutorial
ms.date: 5/10/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9e2bd57925ddb78dcfe23742b35c1490584558f8
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666884"
---
# <a name="tutorial-scale-out-a-service-fabric-managed-cluster"></a>Tutorial: Escalado horizontal de un clúster administrado de Service Fabric

En esta serie de tutoriales, trataremos de lo siguiente:

> [!div class="checklist"]
> * [Implementación de un clúster administrado de Service Fabric.](tutorial-managed-cluster-deploy.md)
> * Escalado horizontal de un clúster administrado de Service Fabric
> * [Incorporación y eliminación de tipos de nodo en un clúster administrado de Service Fabric](tutorial-managed-cluster-add-remove-node-type.md)
> * [Implementación de una aplicación en un clúster administrado de Service Fabric](tutorial-managed-cluster-deploy-app.md)

En esta parte de la serie se explica lo siguiente:

> [!div class="checklist"]
> * Escalado de un nodo de clúster administrado de Service Fabric

## <a name="prerequisites"></a>Requisitos previos

* Un clúster administrado de Service Fabric (consulte [*Implementación de un clúster administrado*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) o posterior (consulte [*Instalación de Azure PowerShell*](/powershell/azure/install-az-ps)).

## <a name="scale-a-service-fabric-managed-cluster"></a>Escalado de un clúster administrado de Service Fabric
Cambie el recuento de instancias para aumentar o disminuir el número de nodos del tipo de nodo que desea escalar. Puede buscar los nombres de tipo de nodo en la plantilla de Azure Resource Manager (plantilla de ARM) de la implementación del clúster o en Service Fabric Explorer.  

> [!NOTE]
> Si el tipo de nodo es principal, no podrá desplazarse por debajo de 3 nodos para un clúster de SKU básico y de 5 nodos, para un clúster de SKU estándar.

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "FE"
$instanceCount = "7"

Set-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -name $nodeTypeName -InstanceCount $instanceCount -Verbose
```

El clúster comenzará a actualizarse automáticamente y, después de unos minutos, verá los nodos adicionales.

## <a name="next-steps"></a>Pasos siguientes

En este paso, hemos escalado un tipo de nodo en un clúster administrado de Service Fabric. Para más información sobre cómo agregar y eliminar tipos de nodo, consulte:

> [!div class="nextstepaction"]
> [Incorporación y eliminación de tipos de nodo en un clúster administrado de Service Fabric](tutorial-managed-cluster-add-remove-node-type.md)
