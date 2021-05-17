---
title: Implementación de un clúster administrado de Service Fabric con tipos de nodo sin estado
description: Aprenda a crear e implementar tipos de nodo sin estado en clústeres administrados de Service Fabric
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: a1ea4a9d37ce26ac90c2cbae0420e4dbf8797ee2
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109689400"
---
# <a name="deploy-a-service-fabric-managed-cluster-with-stateless-node-types"></a>Implementación de un clúster administrado de Service Fabric con tipos de nodo sin estado

Los tipos de nodo de Service Fabric vienen con una asunción inherente de que, en algún momento, se van a colocar servicios con estado en los nodos. Los tipos de nodo sin estado relajan esta asunción para un tipo de nodo. La relajación de esta asunción permite que los tipos de nodo sin estado se beneficien de operaciones de escalabilidad horizontal más rápidas mediante la eliminación de algunas de las restricciones sobre las operaciones de reparación y mantenimiento.

* Los tipos de nodo principales no se pueden configurar para que no tengan estado.
* Los tipos de nodo sin estado requieren una versión de API **2021-05-01** o posterior


Están disponibles plantillas de ejemplo: [Plantilla de tipos de nodos sin estado de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="enable-stateless-node-types-in-a-service-fabric-managed-cluster"></a>Habilitación de tipos de nodo sin estado en un clúster administrado de Service Fabric
Para establecer uno o más tipos de nodo como sin estado en un recurso de tipo de nodo, establezca la propiedad **isStateless** en **true**. Al implementar un clúster de Service Fabric con tipos de nodo sin estado, es necesario tener al menos un tipo de nodo principal que no sea sin estado en el clúster.

* El valor de apiVersion del recurso de clúster administrado de Service Fabric debe ser **2021-05-01** o posterior.

```json
     {
            "apiVersion": "[variables('sfApiVersion')]",
            "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
            "name": "[concat(parameters('clusterName'), '/', parameters('nodeTypeName'))]",
            "location": "[resourcegroup().location]",
            "dependsOn": [
              "[concat('Microsoft.ServiceFabric/managedclusters/', parameters('clusterName'))]"
            ],
            "properties": {
                "isStateless": true,
                "isPrimary": false,
                "vmImagePublisher": "[parameters('vmImagePublisher')]",
                "vmImageOffer": "[parameters('vmImageOffer')]",
                "vmImageSku": "[parameters('vmImageSku')]",
                "vmImageVersion": "[parameters('vmImageVersion')]",
                "vmSize": "[parameters('nodeTypeSize')]",
                "vmInstanceCount": "[parameters('nodeTypeVmInstanceCount')]",
                "dataDiskSizeGB": "[parameters('nodeTypeDataDiskSizeGB')]"
            }
        }
}
```

## <a name="configure-stateless-node-types-with-multiple-availability-zones"></a>Configuración de tipos de nodo sin estado con varias zonas de disponibilidad
Para configurar un tipo de nodo sin estado que abarque varias zonas de disponibilidad, siga [Implementación de un clúster de Azure Service Fabric en Availability Zones](.\service-fabric-cross-availability-zones.md). 

>[!NOTE]
> La propiedad de resistencia zonal debe establecerse en el nivel de clúster, y no se puede cambiar en su ubicación.

## <a name="migrate-to-using-stateless-node-types-in-a-cluster"></a>Migración para usar tipos de nodo sin estado en un clúster
En todos los escenarios de migración es necesario agregar un nuevo tipo de nodo sin estado. El tipo de nodo existente no se puede migrar para que sea sin estado. Puede agregar un nuevo tipo de nodo sin estado a un clúster administrado de Service Fabric existente y quitar los tipos de nodo originales del clúster. 

## <a name="next-steps"></a>Pasos siguientes 

Para más información acerca de los clústeres administrados de Service Fabric, consulte:

> [!div class="nextstepaction"]
> [Preguntas más frecuentes sobre los clústeres administrados de Service Fabric](./faq-managed-cluster.md)