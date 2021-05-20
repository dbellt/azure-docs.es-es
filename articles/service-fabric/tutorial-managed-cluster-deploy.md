---
title: Implementación de un clúster administrado de Service Fabric
description: En este tutorial, implementará un clúster administrado de Service Fabric para pruebas.
ms.topic: tutorial
ms.date: 5/10/2021
ms.custom: references_regions
ms.openlocfilehash: 48332bd8115a48497157b846f02db49c96e61001
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685790"
---
# <a name="tutorial-deploy-a-service-fabric-managed-cluster"></a>Tutorial: Implementación de un clúster administrado de Service Fabric

En esta serie de tutoriales, trataremos de lo siguiente:

> [!div class="checklist"]
> * Implementación de un clúster administrado de Service Fabric 
> * [Escalado horizontal de un clúster administrado de Service Fabric](tutorial-managed-cluster-scale.md)
> * [Incorporación y eliminación de nodos de un clúster administrado de Service Fabric](tutorial-managed-cluster-add-remove-node-type.md)
> * [Implementación de una aplicación en un clúster administrado de Service Fabric](tutorial-managed-cluster-deploy-app.md)

En esta parte de la serie se explica lo siguiente:

> [!div class="checklist"]
> * Conexión a la cuenta de Azure
> * Creación de un grupo de recursos
> * Implementación de un clúster administrado de Service Fabric
> * Incorporación de un tipo de nodo principal al clúster

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si aún no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Instalación de [Service Fabric SDK y del módulo de PowerShell](service-fabric-get-started.md).

* Instale [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) (o posterior).

## <a name="connect-to-your-azure-account"></a>Conexión a la cuenta de Azure

Reemplace `<your-subscription>` por la cadena de suscripción de su cuenta de Azure y conéctese:

```powershell
Login-AzAccount
Set-AzContext -SubscriptionId <your-subscription>

```

## <a name="create-a-new-resource-group"></a>Creación de un grupo de recursos

A continuación, cree el grupo de recursos para el clúster administrado de Service Fabric y reemplace `<your-rg>` y `<location>` por el nombre y la ubicación del grupo que desee.

> [!NOTE]
> En las regiones admitidas en la versión preliminar pública se incluyen `centraluseuap`, `eastus2euap`, `eastasia`, `northeurope`, `westcentralus` y `eastus2`.

```powershell
$resourceGroup = "myResourceGroup"
$location = "EastUS2"

New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="deploy-a-service-fabric-managed-cluster"></a>Implementación de un clúster administrado de Service Fabric

### <a name="create-a-service-fabric-managed-cluster"></a>Creación de un clúster administrado de Service Fabric

En este paso, creará un clúster administrado de Service Fabric con el comando New-AzServiceFabricManagedCluster de PowerShell. En el ejemplo siguiente se crea un clúster denominado myCluster en el grupo de recursos denominado myResourceGroup. Este grupo de recursos se creó en el paso anterior en la región eastus2.

En este paso, proporcione sus propios valores para los siguientes parámetros:

* **Nombre del clúster**: escriba un nombre único para el clúster, como *miclustersf*.
* **Contraseña de administrador**: escriba una contraseña para el administrador que se va a usar para RDP en las máquinas virtuales subyacentes del clúster.
* **Huella digital de certificado de cliente**: Proporcione la huella digital del certificado de cliente que quiere usar para obtener acceso al clúster. Si no tiene un certificado, siga los pasos que se indican en [Establecimiento y recuperación de un certificado](../key-vault/certificates/quick-create-portal.md) para crear un certificado autofirmado.
* **SKU de clúster**: especifique el tipo de [clúster administrado de Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus) que se va a implementar. Los clústeres de SKU *básicos* solo están diseñados para implementaciones de prueba y no permiten la incorporación o eliminación de tipos de nodo.

```powershell
$clusterName = "<unique cluster name>"
$password = "Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$clientThumbprint = "<certificate thumbprint>"
$clusterSku = "Standard"

New-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroup -Location $location -ClusterName $clusterName -ClientCertThumbprint $clientThumbprint -ClientCertIsAdmin -AdminPassword $password -Sku $clusterSKU -Verbose
```

### <a name="add-a-primary-node-type-to-the-service-fabric-managed-cluster"></a>Incorporación de un tipo de nodo principal al clúster administrado de Service Fabric

En este paso, agregará un tipo de nodo principal al clúster que acaba de crear. Cada clúster de Service Fabric debe tener como mínimo un tipo de nodo principal.

En este paso, proporcione sus propios valores para los siguientes parámetros:

* **Nombre del tipo de nodo**: escriba un nombre único para el tipo de nodo que se va a agregar al clúster, como "NT1".

> [!NOTE]
> Si el tipo de nodo que se va a agregar es el primero o el único tipo de nodo del clúster, se debe usar la propiedad Primary.

```powershell
$nodeType1Name = "NT1"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeType1Name -Primary -InstanceCount 5
```

Este comando puede tardar varios minutos en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

### <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Una vez finalizada la implementación, busque el valor de Service Fabric Explorer en la página de información general sobre recursos del clúster administrado de Service Fabric en el portal. Cuando se le solicite para un certificado, utilice el certificado para el que se proporcionó la huella digital de cliente en el comando de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

En este paso hemos creado e implementado el primer clúster administrado de Service Fabric. Para más información sobre cómo escalar un clúster, consulte:

> [!div class="nextstepaction"]
> [Escalado horizontal de un clúster administrado de Service Fabric](tutorial-managed-cluster-scale.md)
