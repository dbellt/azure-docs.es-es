---
title: Creación de una regla de Azure Load Balancer para un clúster
description: Configure una instancia de Azure Load Balancer para abrir puertos para el clúster de Azure Service Fabric.
ms.topic: conceptual
ms.date: 12/06/2017
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0e3d3d065d27268085721d3080c4fc21c407261a
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671346"
---
# <a name="open-ports-for-a-service-fabric-cluster"></a>Abrir puertos para un clúster de Service Fabric

El equilibrador de carga que se implementa con el clúster de Azure Service Fabric dirige el tráfico a la aplicación que se ejecuta en un nodo. Si cambia la aplicación para usar otro puerto, debe exponer ese puerto (o enrutar un puerto diferente) en Azure Load Balancer.

Al implementar el clúster de Service Fabric en Azure, se crea automáticamente un equilibrador de carga. Si no tiene un equilibrador de carga, consulte [Creación de un equilibrador de carga orientado a Internet](../load-balancer/quickstart-load-balancer-standard-public-portal.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-service-fabric"></a>Configurar Service Fabric

El archivo de configuración **ServiceManifest.xml** de la aplicación de Service Fabric define los puntos de conexión que usará la aplicación. Después de que se haya actualizado el archivo de configuración para definir un punto de conexión, el equilibrador de carga se debe actualizar para exponer ese puerto (u otro). Para obtener más información sobre cómo crear el punto de conexión de Service Fabric, consulte [Especificación de los puntos de conexión](service-fabric-service-manifest-resources.md).

## <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla de equilibrador de carga abre un puerto con conexión a Internet y reenvía el tráfico al puerto del nodo interno usado por la aplicación. Si no tiene un equilibrador de carga, consulte [Creación de un equilibrador de carga orientado a Internet](../load-balancer/quickstart-load-balancer-standard-public-portal.md).

Para crear una regla de equilibrador de carga, debe recopilar la información siguiente:

- Nombre del equilibrador de carga.
- Grupo de recursos del clúster de Service Fabric y el equilibrador de carga.
- Puerto externo.
- Puerto interno.

## <a name="azure-cli"></a>Azure CLI
Solo hace falta un comando para crear una regla de equilibrador de carga con la **CLI de Azure**. Tiene que saber el nombre del grupo de recursos y del equilibrador de carga para crear una regla.

>[!NOTE]
>Si necesita determinar el nombre del equilibrador de carga, use este comando para obtener rápidamente una lista de todos los equilibradores de carga y los grupos de recursos asociados.
>
>`az network lb list --query "[].{ResourceGroup: resourceGroup, Name: name}"`
>


```azurecli
az network lb rule create --backend-port 40000 --frontend-port 39999 --protocol Tcp --lb-name LB-svcfab3 -g svcfab_cli -n my-app-rule
```

El comando de la CLI de Azure tiene unos parámetros que se describen en la tabla siguiente:

| Parámetro | Descripción |
| --------- | ----------- |
| `--backend-port`  | El puerto en que escucha la aplicación de Service Fabric. |
| `--frontend-port` | El puerto que expone el equilibrador de carga para las conexiones externas. |
| `-lb-name` | El nombre del equilibrador de carga que se cambiará. |
| `-g`       | El grupo de recursos que tiene el clúster de Service Fabric y el equilibrador de carga. |
| `-n`       | El nombre deseado de la regla. |


>[!NOTE]
>Para obtener más información sobre cómo crear un equilibrador de carga con la CLI de Azure, consulte [Creación de un equilibrador de carga interno mediante la CLI de Azure](../load-balancer/quickstart-load-balancer-standard-internal-cli.md).

## <a name="powershell"></a>PowerShell

PowerShell es un poco más complicado que la CLI de Azure. Siga estos pasos conceptuales para crear una regla:

1. Obtenga el equilibrador de carga de Azure.
2. Cree una regla.
3. Agregue la regla al equilibrador de carga.
4. Actualice el equilibrador de carga.

>[!NOTE]
>Si necesita determinar el nombre del equilibrador de carga, use este comando para obtener rápidamente una lista de todos los equilibradores de carga y los grupos de recursos asociados.
>
>`Get-AzLoadBalancer | Select Name, ResourceGroupName`

```powershell
# Get the load balancer
$lb = Get-AzLoadBalancer -Name LB-svcfab3 -ResourceGroupName svcfab_cli

# Create the rule based on information from the load balancer.
$lbrule = New-AzLoadBalancerRuleConfig -Name my-app-rule7 -Protocol Tcp -FrontendPort 39990 -BackendPort 40009 `
                                            -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
                                            -BackendAddressPool  $lb.BackendAddressPools[0] `
                                            -Probe $lb.Probes[0]

# Add the rule to the load balancer
$lb.LoadBalancingRules.Add($lbrule)

# Update the load balancer on Azure
$lb | Set-AzLoadBalancer
```

Con respecto al comando `New-AzLoadBalancerRuleConfig`, `-FrontendPort` representa el puerto que el equilibrador de carga expone para las conexiones externas, y `-BackendPort` representa el puerto en que está escuchando la aplicación de Service Fabric.

>[!NOTE]
>Para obtener más información sobre cómo crear un equilibrador de carga con PowerShell, consulte [Creación de un equilibrador de carga orientado a Internet en Resource Manager mediante PowerShell](../load-balancer/quickstart-load-balancer-standard-internal-powershell.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [las redes en Service Fabric](service-fabric-patterns-networking.md).
