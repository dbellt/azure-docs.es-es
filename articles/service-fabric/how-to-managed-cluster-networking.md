---
title: Configuración de la red para clústeres administrados de Service Fabric
description: Aprenda a configurar el clúster administrado de Service Fabric para las reglas de grupos de seguridad de red, el acceso a puertos RDP, las reglas de equilibrio de carga, etc.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 67bcdccbd3a54fc0e05b2516aaf5633ddddb1f00
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110060983"
---
# <a name="configure-network-settings-for-service-fabric-managed-clusters"></a>Configuración de la red para clústeres administrados de Service Fabric

Los clústeres administrados de Service Fabric se crean con una configuración de red predeterminada. Esta configuración consta de reglas obligatorias para la funcionalidad esencial del clúster y algunas reglas opcionales, como permitir de manera predeterminada todo el tráfico de salida, que están diseñadas para facilitar la configuración del cliente.

Además de la configuración de red predeterminada, puede modificar las reglas de red para que se adapten a las necesidades de su escenario.

## <a name="nsg-rules-guidance"></a>Orientaciones sobre las reglas del grupo de seguridad de red

Tenga en cuenta estas consideraciones al crear nuevas reglas del grupo de seguridad de red para el clúster administrado.

* Los clústeres administrados de Service Fabric reservan el intervalo de prioridad de la regla de grupo de seguridad de red del 0 al 999 para la funcionalidad esencial. No se pueden crear reglas de grupos de seguridad de red personalizadas con un valor de prioridad inferior a 1000.
* Los clústeres administrados de Service Fabric reservan el intervalo de prioridad de 3001 a 4000 para crear reglas de grupo de seguridad de red opcionales. Estas reglas se agregan automáticamente para que las configuraciones sean rápidas y sencillas. Para invalidar estas reglas puede agregar reglas de grupo de seguridad de red personalizadas en el intervalo de prioridad de 1000 a 3000.
* Las reglas de grupo de seguridad de red personalizadas deben tener una prioridad en el intervalo de 1000 a 3000.

## <a name="apply-nsg-rules"></a>Aplicación de las reglas de grupo de seguridad de red

Con los clústeres clásicos (no administrados) de Service Fabric, debe declarar y administrar un recurso *Microsoft.Network/networkSecurityGroups* independiente para [aplicar las reglas del grupo de seguridad de red al clúster](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.servicefabric/service-fabric-secure-nsg-cluster-65-node-3-nodetype). Los clústeres administrados de Service Fabric le permiten asignar reglas de grupo de seguridad de red directamente en el recurso de clúster de la plantilla de implementación.

Use la propiedad [networkSecurityRules](/azure/templates/microsoft.servicefabric/managedclusters#managedclusterproperties-object) del recurso *Microsoft.ServiceFabric/managedclusters* (versión `2021-05-01` o posterior) para asignar reglas de grupo de seguridad de red. Por ejemplo:

```json
            "apiVersion": "2021-05-01",
            "type": "Microsoft.ServiceFabric/managedclusters",
            ...
            "properties": {
                ...
                "networkSecurityRules" : [
                    {
                        "name": "AllowCustomers",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "Internet",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33000-33499",
                        "access": "Allow",
                        "priority": 2001,
                        "direction": "Inbound"
                    },
                    {
                        "name": "AllowARM",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "AzureResourceManager",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33500-33699",
                        "access": "Allow",
                        "priority": 2002,
                        "direction": "Inbound"
                    },
                    {
                        "name": "DenyCustomers",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "Internet",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33700-33799",
                        "access": "Deny",
                        "priority": 2003,
                        "direction": "Outbound"
                    },
                    {
                        "name": "DenyRDP",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "*",
                        "destinationAddressPrefix": "VirtualNetwork",
                        "destinationPortRange": "3389",
                        "access": "Deny",
                        "priority": 2004,
                        "direction": "Inbound",
                        "description": "Override for optional SFMC_AllowRdpPort rule. This is required in tests to avoid Sev2 incident for security policy violation."
                    }
                ],
                "fabricSettings": [
                ...
                ]
            }
```

## <a name="rdp-ports"></a>Puertos RDP

Los clústeres administrados de Service Fabric no permiten el acceso a los puertos RDP de forma predeterminada. Puede abrir puertos RDP en Internet estableciendo la siguiente propiedad en un recurso de clúster administrado de Service Fabric.

```json
"allowRDPAccess": true
```

Si la propiedad allowRDPAccess está establecida en true, se agregará la siguiente regla de grupo de seguridad de red a la implementación del clúster.

```json
{
    "name": "SFMC_AllowRdpPort",
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "Optional rule to open RDP ports.",
        "protocol": "tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 3002,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRange": "3389"
    }
}
```

## <a name="clientconnection-and-httpgatewayconnection-ports"></a>Puertos ClientConnection y HttpGatewayConnection

### <a name="nsg-rule-sfmc_allowservicefabricgatewaytosfrp"></a>Regla de grupo de seguridad de red: SFMC_AllowServiceFabricGatewayToSFRP

Se agrega una regla de grupo de seguridad de red predeterminada para permitir que el proveedor de recursos de Service Fabric tenga acceso a los puertos clientConnectionPort y httpGatewayConnectionPort del clúster. Esta regla permite el acceso a los puertos a través de la etiqueta de servicio "ServiceFabric".

>[!NOTE]
>Esta regla se agrega siempre y no se puede invalidar.

```json
{
    "name": "SFMC_AllowServiceFabricGatewayToSFRP",
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "This is required rule to allow SFRP to connect to the cluster. This rule cannot be overridden.",
        "protocol": "TCP",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "ServiceFabric",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 500,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRanges": [
            "19000",
            "19080"
        ]
    }
}
```

### <a name="nsg-rule-sfmc_allowservicefabricgatewayports"></a>Regla de grupo de seguridad de red: SFMC_AllowServiceFabricGatewayPorts

Se trata de una regla de grupo de seguridad de red opcional para permitir el acceso a clientConnectionPort y httpGatewayPort desde Internet. Esta regla permite a los clientes tener acceso a SFX, conectarse al clúster mediante PowerShell y usar puntos de conexión de API del clúster de Service Fabric desde fuera.

>[!NOTE]
>Esta regla no se agregará si hay una regla personalizada con los mismos valores de acceso, dirección y protocolo para el mismo puerto. Puede invalidar esta regla con reglas de grupo de seguridad de red personalizadas.

```json
{
    "name": "SFMC_AllowServiceFabricGatewayPorts",
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "Optional rule to open SF cluster gateway ports. To override add a custom NSG rule for gateway ports in priority range 1000-3000.",
        "protocol": "tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 3001,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRanges": [
            "19000",
            "19080"
        ]
    }
}
```

## <a name="load-balancer-ports"></a>Puertos del equilibrador de carga

Los clústeres administrados de Service Fabric crean una regla de grupo de seguridad de red en el intervalo de prioridad predeterminado de todos los puertos del equilibrador de carga (LB) configurados en la sección "loadBalancingRules" en las propiedades de *ManagedCluster*. Esta regla abre los puertos del equilibrador de carga para el tráfico entrante desde Internet.

>[!NOTE]
>Esta regla se agrega en el intervalo de prioridad opcional y se puede invalidar mediante la adición de reglas de grupo de seguridad de red personalizadas.

```json
{
    "name": "SFMC_AllowLoadBalancedPorts",
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "Optional rule to open LB ports",
        "protocol": "*",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 3003,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRanges": [
        "80", "8080", "4343"
        ]
    }
}
```

## <a name="load-balancer-probes"></a>Sondeos del equilibrador de carga

Los clústeres administrados de Service Fabric crean automáticamente sondeos del equilibrador de carga para los puertos de Fabric Gateway, así como para todos los puertos configurados en la sección "loadBalancingRules" de las propiedades del clúster administrado.

```json
{
  "value": [
    {
        "name": "FabricTcpGateway",
        "properties": {
            "provisioningState": "Succeeded",
            "protocol": "Tcp",
            "port": 19000,
            "intervalInSeconds": 5,
            "numberOfProbes": 2,
            "loadBalancingRules": [
                {
                    "id": "<>"
                }
            ]
        },
        "type": "Microsoft.Network/loadBalancers/probes"
    },
    {
        "name": "FabricHttpGateway",
        "properties": {
            "provisioningState": "Succeeded",
            "protocol": "Tcp",
            "port": 19080,
            "intervalInSeconds": 5,
            "numberOfProbes": 2,
            "loadBalancingRules": [
                {
                    "id": "<>"
                }
            ]
        },
        "type": "Microsoft.Network/loadBalancers/probes"
    },
    {
        "name": "probe1_tcp_8080",
        "properties": {
            "provisioningState": "Succeeded",
            "protocol": "Tcp",
            "port": 8080,
            "intervalInSeconds": 5,
            "numberOfProbes": 2,
            "loadBalancingRules": [
            {
                "id": "<>"
            }
        ]
      },
      "type": "Microsoft.Network/loadBalancers/probes"
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

[Opciones de configuración del clúster administrado de Service Fabric](how-to-managed-cluster-configuration.md)

[Información general de los clústeres administrados de Service Fabric](overview-managed-cluster.md)
