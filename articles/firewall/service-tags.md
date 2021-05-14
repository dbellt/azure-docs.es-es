---
title: Información general de las etiquetas del servicio Azure Firewall
description: Una etiqueta de servicio representa un grupo de prefijos de direcciones IP que ayudan a reducir la complejidad de la creación de reglas de seguridad.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 4/5/2021
ms.author: victorh
ms.openlocfilehash: 3cc1e85a18eab1adb0a1dd8307a074cb43ba0c70
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529552"
---
# <a name="azure-firewall-service-tags"></a>Etiquetas de servicio de Azure Firewall

Una etiqueta de servicio representa un grupo de prefijos de direcciones IP que ayudan a reducir la complejidad de la creación de reglas de seguridad. No puede crear su propia etiqueta de servicio, ni especificar qué direcciones IP se incluyen dentro de una etiqueta. Microsoft administra los prefijos de direcciones que incluye la etiqueta de servicio y actualiza automáticamente esta a medida que las direcciones cambian.

Las etiquetas de servicio de Azure Firewall se pueden usar en el campo de destino de las reglas de red. Puede usarlas en lugar de direcciones IP específicas.

## <a name="supported-service-tags"></a>Etiquetas de servicio admitidas

Consulte [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md#available-service-tags) para obtener una lista de etiquetas de servicio que están disponibles para su uso en las reglas de red del firewall de Azure.

## <a name="configuration"></a>Configuración

Azure Firewall admite la configuración de etiquetas de servicio mediante PowerShell, la CLI de Azure o Azure Portal.

### <a name="configure-via-azure-powershell"></a>Configuración mediante Azure PowerShell

En este ejemplo, primero debemos obtener el contexto de la instancia de Azure Firewall creada anteriormente.

```Get the context to an existing Azure Firewall
$FirewallName = "AzureFirewall"
$ResourceGroup = "AzureFirewall-RG"
$azfirewall = Get-AzFirewall -Name $FirewallName -ResourceGroupName $ResourceGroup
```

A continuación, debemos crear una nueva regla.  Para origen o destino, puede especificar el valor de texto de la etiqueta de servicio que desea aprovechar, tal como se ha mencionado anteriormente en este artículo.

````Create new Network Rules using Service Tags
$rule = New-AzFirewallNetworkRule -Name "AllowSQL" -Description "Allow access to Azure Database as a Service (SQL, MySQL, PostgreSQL, Datawarehouse)" -SourceAddress "10.0.0.0/16" -DestinationAddress Sql -DestinationPort 1433 -Protocol TCP
$ruleCollection = New-AzFirewallNetworkRuleCollection -Name "Data Collection" -Priority 1000 -Rule $rule -ActionType Allow
````

A continuación, tenemos que actualizar la variable que contiene la definición de Azure Firewall con las nuevas reglas de red que hemos creado.

````Merge the new rules into our existing Azure Firewall variable
$azFirewall.NetworkRuleCollections.add($ruleCollection)
`````

Por último, debemos confirmar los cambios de la regla de red en la instancia de Azure Firewall de ejecución.
````Commit the changes to Azure
Set-AzFirewall -AzureFirewall $azfirewall
````

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las reglas de Azure Firewall, consulte [Lógica de procesamiento de reglas de Azure Firewall](rule-processing.md).
