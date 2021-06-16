---
title: Intervalos de direcciones IP privadas de SNAT de Azure Firewall
description: Se pueden configurar los intervalos de direcciones IP para SNAT.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 04/14/2021
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6235e5ec34987c0a383ea776aabf0a00e345ce63
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693689"
---
# <a name="azure-firewall-snat-private-ip-address-ranges"></a>Intervalos de direcciones IP privadas de SNAT de Azure Firewall

Azure Firewall proporciona SNAT automática para todo el tráfico saliente a las IP públicas. De forma predeterminada, Azure Firewall no aplica SNAT con reglas de red cuando la dirección IP de destino es un intervalo de direcciones IP privadas, de acuerdo con la normativa [RFC 1918 de IANA](https://tools.ietf.org/html/rfc1918) o el espacio de direcciones compartidas según la normativa [RFC 6598 de IANA](https://tools.ietf.org/html/rfc6598). Las reglas de aplicación se aplican siempre mediante un [proxy transparente](https://wikipedia.org/wiki/Proxy_server#Transparent_proxy), independientemente de la dirección IP de destino.

Esta lógica funciona bien cuando se enruta el tráfico directamente a Internet. Sin embargo, si ha se habilitado [tunelización forzada](forced-tunneling.md), al tráfico enlazado a Internet se le aplica SNAT a una de las direcciones IP privadas del firewall en AzureFirewallSubnet, ocultando el origen del firewall local.

Si su organización usa un intervalo de direcciones IP públicas para las redes privadas, Azure Firewall aplicará SNAT al tráfico para una de las direcciones IP privadas de firewall en AzureFirewallSubnet. Sin embargo, puede configurar Azure Firewall de modo que **no** aplique SNAT al intervalo de direcciones IP públicas. Por ejemplo, para especificar una dirección IP individual, puede especificarla de la siguiente manera: `192.168.1.10`. Para especificar un intervalo de direcciones IP, puede especificarlo de la siguiente manera: `192.168.1.0/24`.

- Para configurar Azure Firewall en **nunca** aplicar SNAT, independientemente de la dirección IP de destino, use **0.0.0.0/0** como intervalo de direcciones IP privadas. Con esta configuración, Azure Firewall nunca puede enrutar el tráfico directamente a Internet. 

- Para configurar el firewall en **siempre** aplicar SNAT, independientemente de la dirección IP de destino, use **255.255.255.255/32** como intervalo de direcciones IP privadas.

> [!IMPORTANT]
> El intervalo de direcciones privadas que especifique solo se aplicará a las reglas de red. Actualmente, las reglas de aplicación siempre son SNAT.

> [!IMPORTANT]
> Si desea especificar sus propios intervalos de direcciones IP privadas y mantener los intervalos de direcciones RFC 1918 de IANA predeterminados, asegúrese de que la lista personalizada todavía incluye el intervalo de RFC 1918 de IANA. 

Puede configurar las direcciones IP privadas de SNAT mediante los métodos siguientes. Debe configurar las direcciones privadas de SNAT mediante el método adecuado para la configuración. Los firewalls asociados a una directiva de firewall deben especificar el intervalo de la directiva y no usar `AdditionalProperties`.


|Método            |Uso de reglas clásicas  |Uso de directiva de firewall  |
|---------|---------|---------|
|Azure Portal     | [Se admite](#classic-rules-3)| [Se admite](#firewall-policy-1)|
|Azure PowerShell     |[configurar `PrivateRange`](#classic-rules)|No se admite actualmente|
|Azure CLI|[configurar `--private-ranges`](#classic-rules-1)|No se admite actualmente|
|Plantilla ARM     |[configurar `AdditionalProperties` en la propiedad del firewall](#classic-rules-2)|[configurar `snat/privateRanges` en la directiva del firewall](#firewall-policy)|


## <a name="configure-snat-private-ip-address-ranges---azure-powershell"></a>Configuración de intervalos de direcciones IP privadas de SNAT: Azure PowerShell
### <a name="classic-rules"></a>Reglas clásicas

Puede usar Azure PowerShell a fin de especificar intervalos de direcciones IP privadas para el firewall.

> [!NOTE]
> La propiedad `PrivateRange` del firewall se omite para los firewalls asociados a una directiva de firewall. Debe usar la propiedad `SNAT` de `firewallPolicies`, tal y como se describe en [Configuración de intervalos de direcciones IP privadas de SNAT: plantilla de ARM](#firewall-policy).

#### <a name="new-firewall"></a>Nuevo firewall

En el caso de un nuevo firewall con reglas clásicas, el cmdlet de Azure PowerShell es:

```azurepowershell
$azFw = @{
    Name               = '<fw-name>'
    ResourceGroupName  = '<resourcegroup-name>'
    Location           = '<location>'
    VirtualNetworkName = '<vnet-name>'
    PublicIpName       = '<public-ip-name>'
    PrivateRange       = @("IANAPrivateRanges", "192.168.1.0/24", "192.168.1.10")
}

New-AzFirewall @azFw
```
> [!NOTE]
> La implementación de Azure Firewall con `New-AzFirewall` requiere que existan una red virtual y una dirección IP pública. Consulte [Implementación y configuración de Azure Firewall mediante Azure PowerShell](deploy-ps.md) para obtener una guía completa.

> [!NOTE]
> IANAPrivateRanges se expande a los valores predeterminados actuales en Azure Firewall mientras que los demás intervalos se agregan a él. Para mantener el valor predeterminado de IANAPrivateRanges en su especificación de intervalo privado, debe permanecer en la especificación de `PrivateRange`, tal como se muestra en los ejemplos siguientes.

Para más información, consulte [New-AzFirewall](/powershell/module/az.network/new-azfirewall).

#### <a name="existing-firewall"></a>Firewall existente

Para configurar un firewall existente con reglas clásicas, use los siguientes cmdlets de Azure PowerShell:

```azurepowershell
$azfw = Get-AzFirewall -Name '<fw-name>' -ResourceGroupName '<resourcegroup-name>'
$azfw.PrivateRange = @("IANAPrivateRanges","192.168.1.0/24", "192.168.1.10")
Set-AzFirewall -AzureFirewall $azfw
```

## <a name="configure-snat-private-ip-address-ranges---azure-cli"></a>Configuración de intervalos de direcciones IP privadas de SNAT: CLI de Azure
### <a name="classic-rules"></a>Reglas clásicas

Puede usar la CLI de Azure a fin de especificar intervalos de direcciones IP privadas para el firewall con reglas clásicas. 

#### <a name="new-firewall"></a>Nuevo firewall

En el caso de un nuevo firewall con reglas clásicas, el comando de la CLI de Azure es:

```azurecli-interactive
az network firewall create \
-n <fw-name> \
-g <resourcegroup-name> \
--private-ranges 192.168.1.0/24 192.168.1.10 IANAPrivateRanges
```

> [!NOTE]
> La implementación de Azure Firewall con el comando `az network firewall create` de la CLI de Azure requiere pasos de configuración adicionales para crear direcciones IP públicas y la configuración de IP. Consulte [Implementación y configuración de Azure Firewall mediante la CLI de Azure](deploy-cli.md) para obtener una guía completa.

> [!NOTE]
> IANAPrivateRanges se expande a los valores predeterminados actuales en Azure Firewall mientras que los demás intervalos se agregan a él. Para mantener el valor predeterminado de IANAPrivateRanges en su especificación de intervalo privado, debe permanecer en la especificación de `private-ranges`, tal como se muestra en los ejemplos siguientes.

#### <a name="existing-firewall"></a>Firewall existente

Para configurar un firewall existente con reglas clásicas, el comando de la CLI de Azure es:

```azurecli-interactive
az network firewall update \
-n <fw-name> \
-g <resourcegroup-name> \
--private-ranges 192.168.1.0/24 192.168.1.10 IANAPrivateRanges
```

## <a name="configure-snat-private-ip-address-ranges---arm-template"></a>Configuración de intervalos de direcciones IP privadas de SNAT: plantilla de ARM
### <a name="classic-rules"></a>Reglas clásicas

Para configurar SNAT durante la implementación de la plantilla de ARM, puede agregar lo siguiente a la propiedad `additionalProperties`:

```json
"additionalProperties": {
   "Network.SNAT.PrivateRanges": "IANAPrivateRanges , IPRange1, IPRange2"
},
```
### <a name="firewall-policy"></a>Directiva de firewall

Las instancias de Azure Firewall asociadas a una directiva de firewall admiten intervalos privados de SNAT desde la versión de API 2020-11-01. Actualmente, puede usar una plantilla para actualizar el intervalo privado de SNAT en la directiva de firewall. En el ejemplo siguiente se configura el firewall para que **siempre** sea tráfico de red SNAT:

```json
{ 

            "type": "Microsoft.Network/firewallPolicies", 
            "apiVersion": "2020-11-01", 
            "name": "[parameters('firewallPolicies_DatabasePolicy_name')]", 
            "location": "eastus", 
            "properties": { 
                "sku": { 
                    "tier": "Standard" 
                }, 
                "snat": { 
                    "privateRanges": [255.255.255.255/32] 
                } 
            } 
```

## <a name="configure-snat-private-ip-address-ranges---azure-portal"></a>Configuración de intervalos de direcciones IP privadas de SNAT: Azure Portal
### <a name="classic-rules"></a>Reglas clásicas

Puede usar Azure Portal a fin de especificar intervalos de direcciones IP privadas para el firewall.

1. Seleccione el grupo de recursos y, después, el firewall.
2. En la página **Información general**, **Intervalos IP privados**, seleccione el valor predeterminado **IANA RFC 1918**.

   Se abre la página **Editar prefijos IP privados**:

   :::image type="content" source="media/snat-private-range/private-ip.png" alt-text="Editar prefijos IP privados":::

1. De forma predeterminada, **IANAPrivateRanges** está configurado.
2. Edite los intervalos de direcciones IP privadas para su entorno y, después, seleccione **Guardar**.

### <a name="firewall-policy"></a>Directiva de firewall

1.  Seleccione el grupo de recursos y, después, la directiva de firewall.
2.  Seleccione **Intervalos de direcciones IP privadas (SNAT)** en la columna **Configuración**.

    De forma predeterminada, **Usar el comportamiento predeterminado de SNAT de la directiva de Azure Firewall** está seleccionado. 
3. Para personalizar la configuración de SNAT, desactive la casilla y, en **Perform SNAT** (Realizar SNAT), seleccione las condiciones para realizar la traducción SNAT para su entorno.
      :::image type="content" source="media/snat-private-range/private-ip-ranges-snat.png" alt-text="Intervalos de direcciones IP privadas (SNAT)":::


4.   Seleccione **Aplicar**.

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información sobre [Tunelización forzada de Azure Firewall](forced-tunneling.md).
