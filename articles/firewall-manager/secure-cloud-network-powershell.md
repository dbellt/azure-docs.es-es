---
title: 'Tutorial: Protección del centro de conectividad virtual mediante Azure PowerShell'
description: En este artículo se describe cómo crear una red de Azure Virtual WAN en una región con Azure Firewall habilitado en el centro de conectividad.
services: firewall-manager
author: jomore
ms.topic: tutorial
ms.service: firewall-manager
ms.date: 10/22/2020
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9161eee3fe892092d06080a3a5ce1e11c4fa1764
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701898"
---
# <a name="tutorial-secure-your-virtual-hub-using-azure-powershell"></a>Tutorial: Protección del centro de conectividad virtual mediante Azure PowerShell

En este tutorial, creará una instancia de Virtual WAN con un centro de conectividad virtual en una región e implementará una instancia de Azure Firewall en dicho centro para proteger la conectividad. En este ejemplo, se muestra la conectividad segura entre redes virtuales. El tráfico entre las redes virtuales y las ramas de sitio a sitio, de punto a sitio o de ExpressRoute es compatible también con el centro de conectividad seguro virtual.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Implementar la WAN virtual.
> * Implementar Azure Firewall y configurar el enrutamiento personalizado.
> * Comprobación de la conectividad

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- PowerShell 7

   En este tutorial, es preciso ejecutar Azure PowerShell de forma local en PowerShell 7. Para instalar PowerShell 7, consulte [Migración de Windows PowerShell 5.1 a PowerShell 7](/powershell/scripting/install/migrating-from-windows-powershell-51-to-powershell-7?view=powershell-7&preserve-view=true).
- Az.Network, versión 3.2.0

    Si tiene la versión 3.4.0 o posterior de Az.Network, deberá cambiar a una versión anterior para usar algunos de los comandos de este tutorial. Puede comprobar la versión del módulo Az.Network con el comando `Get-InstalledModule -Name Az.Network`. Para desinstalar el módulo Az.Network, ejecute `Uninstall-Module -name az.network`. Para instalar el módulo Az.Network 3.2.0, ejecute `Install-Module az.network -RequiredVersion 3.2.0 -force`.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="initial-virtual-wan-deployment"></a>Implementación inicial de Virtual WAN

Un primer paso es establecer algunas variables y crear el grupo de recursos, la instancia de WAN virtual y el centro de conectividad virtual:

```azurepowershell
# Variable definition
$RG = "vwan-rg"
$Location = "westeurope"
$VwanName = "vwan"
$HubName =  "hub1"
# Create Resource Group, Virtual WAN and Virtual Hub
New-AzResourceGroup -Name $RG -Location $Location
$Vwan = New-AzVirtualWan -Name $VwanName -ResourceGroupName $RG -Location $Location -AllowVnetToVnetTraffic -AllowBranchToBranchTraffic -VirtualWANType "Standard"
$Hub = New-AzVirtualHub -Name $HubName -ResourceGroupName $RG -VirtualWan $Vwan -Location $Location -AddressPrefix "192.168.1.0/24" -Sku "Standard"
```

Cree dos redes virtuales y conéctelas al centro de conectividad como sus radios:

```azurepowershell
# Create Virtual Network
$Spoke1 = New-AzVirtualNetwork -Name "spoke1" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.1.0/24"
$Spoke2 = New-AzVirtualNetwork -Name "spoke2" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.2.0/24"
# Connect Virtual Network to Virtual WAN
$Spoke1Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RemoteVirtualNetwork $Spoke1
$Spoke2Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RemoteVirtualNetwork $Spoke2
```

Llegados a este punto, tiene una red Virtual WAN totalmente funcional que proporciona conectividad universal. Para mejorarla con seguridad, debe implementar una instancia de Azure Firewall en cada centro de conectividad virtual. Para administrar de forma eficaz la instancia de Azure Firewall de WAN virtual, se pueden usar directivas de firewall. Por lo tanto, en este ejemplo también se crea una directiva de firewall:

```azurepowershell
# New Firewall Policy
$FWPolicy = New-AzFirewallPolicy -Name "VwanFwPolicy" -ResourceGroupName $RG -Location $Location
# New Firewall Public IP
$AzFWPIPs = New-AzFirewallHubPublicIpAddress -Count 1
$AzFWHubIPs = New-AzFirewallHubIpAddress -PublicIP $AzFWPIPs
# New Firewall
$AzFW = New-AzFirewall -Name "azfw1" -ResourceGroupName $RG -Location $Location `
            -VirtualHubId $Hub.Id -FirewallPolicyId $FWPolicy.Id `
            -Sku AZFW_Hub -HubIPAddress $AzFWHubIPs
```

La habilitación del registro de Azure Firewall a Azure Monitor es opcional, pero, en este ejemplo, se usan los registros de firewall para demostrar que el tráfico atraviesa el firewall:

```azurepowershell
# Optionally, enable looging of Azure Firewall to Azure Monitor
$LogWSName = "vwan-" + (Get-Random -Maximum 99999) + "-" + $RG
$LogWS = New-AzOperationalInsightsWorkspace -Location $Location -Name $LogWSName -Sku Standard -ResourceGroupName $RG
Set-AzDiagnosticSetting -ResourceId $AzFW.Id -Enabled $True -Category AzureFirewallApplicationRule, AzureFirewallNetworkRule -WorkspaceId $LogWS.ResourceId
```

## <a name="deploy-azure-firewall-and-configure-custom-routing"></a>Implementar Azure Firewall y configurar el enrutamiento personalizado.

Ahora tiene una instancia de Azure Firewall en el centro de conectividad, pero todavía necesita modificar el enrutamiento para que la red Virtual WAN envíe el tráfico que procede de las redes virtuales y de las ramas a través del firewall. Esto se hace en dos pasos:

1. Configure todas las conexiones de red virtual (y las conexiones de rama, si las hubiera) que se propagarán a la tabla de rutas `None`. El efecto de esta configuración es que otras redes virtuales y ramas no aprenderán sus prefijos, por lo que no hay un enrutamiento para llegar a ellas.
1. Ahora puede insertar rutas estáticas en la tabla de enrutamiento `Default` (donde todas las redes virtuales y ramas están asociadas de forma predeterminada), para que todo el tráfico se envíe a Azure Firewall.

> [!NOTE]
> Esta es la configuración que se implementa al proteger la conectividad de Azure Portal con Azure Firewall Manager.

Comience con el primer paso, para configurar las conexiones de red virtual de forma que se propaguen a la tabla de rutas `None`:

```azurepowershell
# Configure Virtual Network connections in hub to propagate to None
$VnetRoutingConfig = $Spoke1Connection.RoutingConfiguration    # We take $Spoke1Connection as baseline for the future vnet config, all vnets will have an identical config
$NoneRT = Get-AzVhubRouteTable -ResourceGroupName $RG -HubName $HubName -Name "noneRouteTable"
$NewPropRT = @{}
$NewPropRT.Add('Id', $NoneRT.id)
$PropRTList = @()
$PropRTList += $NewPropRT
$VnetRoutingConfig.PropagatedRouteTables.Ids = $PropRTList
$VnetRoutingConfig.PropagatedRouteTables.Labels = @()
$Spoke1Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RoutingConfiguration $VnetRoutingConfig
$Spoke2Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RoutingConfiguration $VnetRoutingConfig
```

Ahora puede continuar con el segundo paso, para agregar las rutas estáticas a la tabla de rutas `Default`. En este ejemplo, aplicará la configuración predeterminada que Azure Firewall Manager generaría al proteger la conectividad en una red Virtual WAN, pero puede cambiar la lista de prefijos de la ruta estática para ajustarse a sus requisitos específicos:

```azurepowershell
# Create static routes in default Route table
$AzFWId = $(Get-AzVirtualHub -ResourceGroupName $RG -name  $HubName).AzureFirewall.Id
$AzFWRoute = New-AzVHubRoute -Name "private-traffic" -Destination @("0.0.0.0/0", "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16") -DestinationType "CIDR" -NextHop $AzFWId -NextHopType "ResourceId"
$DefaultRT = Update-AzVHubRouteTable -Name "defaultRouteTable" -ResourceGroupName $RG -VirtualHubName  $HubName -Route @($AzFWRoute)
```

## <a name="test-connectivity"></a>Comprobación de la conectividad

Ahora tiene un centro de conectividad seguro totalmente operativo. Para probar la conectividad, necesita una máquina virtual en cada red virtual de radio conectada al centro:

```azurepowershell
# Create VMs in spokes for testing
$VMLocalAdminUser = "lab-user"
$VMLocalAdminSecurePassword = ConvertTo-SecureString -AsPlainText -Force
$VMCredential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
$VMSize = "Standard_B2ms"
# Spoke1
$Spoke1 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke1"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke1 -AddressPrefix "10.1.1.0/26"
$Spoke1 | Set-AzVirtualNetwork
$VM1 = New-AzVM -Name "spoke1-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke1" -SubnetName "vm" -PublicIpAddressName "spoke1-pip"
$NIC1 = Get-AzNetworkInterface -ResourceId $($VM1.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke1VMPrivateIP = $NIC1.IpConfigurations[0].PrivateIpAddress
$Spoke1VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke1-pip")
# Spoke2
$Spoke2 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke2"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke2 -AddressPrefix "10.1.2.0/26"
$Spoke2 | Set-AzVirtualNetwork
$VM2 = New-AzVM -Name "spoke2-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke2" -SubnetName "vm" -PublicIpAddressName "spoke2-pip"
$NIC2 = Get-AzNetworkInterface -ResourceId $($VM2.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke2VMPrivateIP = $NIC2.IpConfigurations[0].PrivateIpAddress
$Spoke2VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke2-pip")
```

La configuración predeterminada de la directiva de firewall es eliminar todo. Por lo tanto, debe configurar algunas reglas. Comience con las reglas DNAT, de modo que se pueda acceder a las máquinas virtuales de prueba a través de la dirección IP pública del firewall:

```azurepowershell
# Adding DNAT rules for virtual machines in the spokes
$AzFWPublicAddress = $AzFW.HubIPAddresses.PublicIPs.Addresses[0].Address
$NATRuleSpoke1 = New-AzFirewallPolicyNatRule -Name "Spoke1SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10001 `
        -TranslatedAddress $Spoke1VMPrivateIP -TranslatedPort 22
$NATRuleSpoke2 = New-AzFirewallPolicyNatRule -Name "Spoke2SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10002 `
        -TranslatedAddress $Spoke2VMPrivateIP -TranslatedPort 22
$NATCollection = New-AzFirewallPolicyNatRuleCollection -Name "SSH" -Priority 100 `
        -Rule @($NATRuleSpoke1, $NATRuleSpoke2) -ActionType "Dnat"
$NATGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "NAT" -Priority 100 -RuleCollection $NATCollection -FirewallPolicyObject $FWPolicy
```

Ahora puede configurar algunas reglas de ejemplo. Defina una regla de red que permita el tráfico SSH, más una regla de aplicación que permita el acceso al nombre de dominio completo `ifconfig.co` a través de Internet. Esta dirección URL devuelve la dirección IP de origen que se ve en la solicitud HTTP:

```azurepowershell
# Add Network Rule
$SSHRule = New-AzFirewallPolicyNetworkRule -Name PermitSSH -Protocol TCP `
        -SourceAddress "10.0.0.0/8" -DestinationAddress "10.0.0.0/8" -DestinationPort 22
$NetCollection = New-AzFirewallPolicyFilterRuleCollection -Name "Management" -Priority 100 -ActionType Allow -Rule $SSHRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "Management" -Priority 200 -RuleCollection $NetCollection -FirewallPolicyObject $FWPolicy
# Add Application Rul
$ifconfigRule = New-AzFirewallPolicyApplicationRule -Name PermitIfconfig -SourceAddress "10.0.0.0/8" -TargetFqdn "ifconfig.co" -Protocol "http:80","https:443"
$AppCollection = New-AzFirewallPolicyFilterRuleCollection -Name "TargetURLs" -Priority 300 -ActionType Allow -Rule $ifconfigRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "TargetURLs" -Priority 300 -RuleCollection $AppCollection -FirewallPolicyObject $FWPolicy
```

Antes de enviar realmente ningún tráfico, puede inspeccionar las rutas efectivas de las máquinas virtuales. Estas deben contener los prefijos aprendidos de la red Virtual WAN (`0.0.0.0/0` más RFC1918), pero no el prefijo del otro radio:

```azurepowershell
# Check effective routes in the VM NIC in spoke 1
# Note that 10.1.2.0/24 (the prefix for spoke2) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC1.Name | ft
# Check effective routes in the VM NIC in spoke 2
# Note that 10.1.1.0/24 (the prefix for spoke1) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC2.Name | ft
```

Ahora, genere tráfico de una máquina virtual a otra y compruebe que se ha eliminado en Azure Firewall. En los siguientes comandos de SSH, debe aceptar las huellas digitales de las máquinas virtuales y proporcionar la contraseña que definió al crear estas. En este ejemplo, va a enviar cinco paquetes de solicitudes de eco ICMP de la máquina virtual de spoke1 a spoke2, más un intento de conexión TCP en el puerto 22 mediante la utilidad de Linux `nc` (con las marcas `-vz` simplemente se envía una solicitud de conexión y se muestra el resultado). Verá que la operación de hacer ping ha dado error y que el intento de conexión TCP en el puerto 22 se ha realizado correctamente, ya que así lo permite la regla de red que configuró anteriormente:

```azurepowershell
# Connect to one VM and ping the other. It shouldnt work, because the firewall should drop the traffic, since no rule for ICMP is configured
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "ping $Spoke2VMPrivateIP -c 5"
# Connect to one VM and send a TCP request on port 22 to the other. It should work, because the firewall is configured to allow SSH traffic (port 22)
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "nc -vz $Spoke2VMPrivateIP 22"
```

También puede comprobar el tráfico de Internet. Las solicitudes HTTP al FQDN permitido en la directiva de firewall mediante la utilidad `curl` (`ifconfig.co`) funcionarán, pero las solicitudes HTTP a cualquier otro destino darán error (en este ejemplo, se prueba esto con `bing.com`):

```azurepowershell
# This HTTP request should succeed, since it is allowed in an app rule in the AzFW, and return the public IP of the FW
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 ifconfig.co"
# This HTTP request should fail, since the FQDN bing.com is not in any app rule in the firewall policy
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 bing.com"
```

La manera más sencilla de confirmar que el firewall elimina los paquetes es comprobar los registros. Dado que configuró Azure Firewall para enviar registros a Azure Monitor, puede usar el lenguaje de consulta de Kusto para recuperar los registros pertinentes de Azure Monitor:

> [!NOTE]
> Los registros pueden tardar alrededor de un minuto en aparecer para su envío a Azure Monitor.

```azurepowershell
# Getting Azure Firewall network rule Logs
$LogWS = Get-AzOperationalInsightsWorkspace -ResourceGroupName $RG
$LogQuery = 'AzureDiagnostics 
| where Category == "AzureFirewallNetworkRule" 
| where TimeGenerated >= ago(5m) 
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int * 
| parse msg_s with * ". Action: " Action1a 
| parse msg_s with * " was " Action1b " to " NatDestination 
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2 
| extend SourcePort = tostring(SourcePortInt),TargetPort = tostring(TargetPortInt) 
| extend Action = case(Action1a == "", case(Action1b == "",Action2,Action1b), Action1a),Protocol = case(Protocol == "", Protocol2, Protocol),SourceIP = case(SourceIP == "", SourceIP2, SourceIP),TargetIP = case(TargetIP == "", TargetIP2, TargetIP),SourcePort = case(SourcePort == "", "N/A", SourcePort),TargetPort = case(TargetPort == "", "N/A", TargetPort),NatDestination = case(NatDestination == "", "N/A", NatDestination) 
| project TimeGenerated, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action, NatDestination, Resource 
| take 25 '
$(Invoke-AzOperationalInsightsQuery -Workspace $LogWS -Query $LogQuery).Results | ft
```

En el comando anterior, verá diferentes entradas:

* La conexión SSH a la que se ha aplicado reglas DNAT
* Los paquetes ICMP eliminados entre las máquinas virtuales de los radios (10.1.1.4 y 10.1.2.4)
* Las conexiones SSH permitidas entre las máquinas virtuales de los radios

Aquí se muestra una salida de ejemplo generada por el comando anterior:

```
TimeGenerated            Protocol    SourceIP       SourcePort TargetIP      TargetPort Action  NatDestination Resource
-------------            --------    --------       ---------- --------      ---------- ------  -------------- --------
2020-10-04T20:53:02.41Z  TCP         109.125.122.99 62281      51.105.224.44 10001      DNAT'ed 10.1.1.4:22    AZFW1
2020-10-04T20:53:07.045Z TCP         10.1.1.4       35932      10.1.2.4      22         Allow   N/A            AZFW1
2020-10-04T20:53:50.119Z TCP         109.125.122.99 62293      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:52:47.475Z TCP         109.125.122.99 62273      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:04.682Z TCP         109.125.122.99 62200      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:17.031Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:18.049Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:19.075Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:20.097Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:21.121Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:52:52.356Z TCP         10.1.1.4       53748      10.1.2.4      22         Allow   N/A            AZFW1
```

Si quiere ver los registros de las reglas de aplicación (que describen las conexiones HTTP permitidas y denegadas) o cambiar la forma en que se muestran los registros, puede probar con otras consultas de KQL. Puede encontrar algunos ejemplos en [Registros de Azure Monitor para Azure Firewall](../firewall/firewall-workbook.md).


## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el entorno de prueba, puede quitar el grupo de recursos con todos los objetos que contiene:

```azurepowershell
# Delete resource group and all contained resources
Remove-AzResourceGroup -Name $RG
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los asociados de seguridad de confianza](trusted-security-partners.md)
