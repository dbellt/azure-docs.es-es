---
title: 'Tutorial: Mejora del acceso de las aplicaciones web con Azure Application Gateway'
description: En este tutorial, obtendrá información sobre cómo crear una puerta de enlace de aplicaciones con redundancia de zona, escalabilidad automática y una dirección IP reservada con Azure PowerShell.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/08/2021
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 8196267ff7a71fb3910848fd0fef11a40a3c1c32
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107869948"
---
# <a name="tutorial-create-an-application-gateway-that-improves-web-application-access"></a>Tutorial: Creación de una puerta de enlace de aplicaciones que mejora el acceso de las aplicaciones web

Si es un administrador de TI interesado en mejorar el acceso a las aplicaciones web, puede optimizar el escalado de Application Gateway en función de la demanda de los clientes y abarcar varias zonas de disponibilidad. Para ello, este tutorial le ayudará a configurar las características de Azure Application Gateway: escalabilidad automática, redundancia de zona y direcciones IP virtuales (IP estática). Use los cmdlets de Azure PowerShell y el modelo de implementación de Azure Resource Manager para solucionar el problema.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un certificado autofirmado
> * Crear una red virtual de escalabilidad automática
> * Creación de una dirección IP pública reservada
> * Configurar una infraestructura para Application Gateway
> * Especificación de la escalabilidad automática
> * Creación de la puerta de enlace de aplicaciones
> * Prueba de la puerta de enlace de aplicaciones

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este tutorial, es necesario que ejecute una sesión administrativa de Azure PowerShell en el entorno local. Debe tener instalado el módulo Azure PowerShell 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Después de verificar la versión de PowerShell, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos en una de las ubicaciones disponibles.

```azurepowershell
$location = "East US 2"
$rg = "AppGW-rg"

#Create a new Resource Group
New-AzResourceGroup -Name $rg -Location $location
```

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Para su uso en producción, debe importar un certificado válido firmado por un proveedor de confianza. Para este tutorial, creará un certificado autofirmado mediante [New-SelfSignedCertificate](/powershell/module/pki/new-selfsignedcertificate). Puede usar [Export-PfxCertificate](/powershell/module/pki/export-pfxcertificate) con la huella digital que se devolvió al exportar un archivo pfx del certificado.

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

Debería ver algo parecido a este resultado:

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

Utilice la huella digital para crear el archivo pfx. Reemplace *\<password>* por una contraseña de su elección:

```powershell
$pwd = ConvertTo-SecureString -String "<password>" -Force -AsPlainText

Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```


## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con una subred específica para una instancia de Application Gateway con escalabilidad automática. Actualmente solo se puede implementar una puerta de enlace de aplicaciones con escalabilidad automática en cada subred específica.

```azurepowershell
#Create VNet with two subnets
$sub1 = New-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -AddressPrefix "10.0.0.0/24"
$sub2 = New-AzVirtualNetworkSubnetConfig -Name "BackendSubnet" -AddressPrefix "10.0.1.0/24"
$vnet = New-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg `
       -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $sub1, $sub2
```

## <a name="create-a-reserved-public-ip"></a>Creación de una dirección IP pública reservada

Especifique el método de asignación de PublicIPAddress como **Estático**. La dirección IP virtual de la puerta de enlace de aplicaciones con escalabilidad automática solo puede ser estática. No se admiten direcciones IP dinámicas. Se admite solo la SKU estándar PublicIpAddress.

```azurepowershell
#Create static public IP
$pip = New-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP" `
       -location $location -AllocationMethod Static -Sku Standard -Zone 1,2,3
```

## <a name="retrieve-details"></a>Recuperación de los detalles

Recupere los detalles del grupo de recursos, la subred y la dirección IP en un objeto local para crear los detalles de configuración de IP de Application Gateway.

```azurepowershell
$publicip = Get-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP"
$vnet = Get-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -VirtualNetwork $vnet
```

## <a name="create-web-apps"></a>Creación de aplicaciones web

Configure dos aplicaciones web para el grupo de back-end. Reemplace *\<site1-name>* y *\<site-2-name>* por nombres únicos del dominio `azurewebsites.net`.

```azurepowershell
New-AzAppServicePlan -ResourceGroupName $rg -Name "ASP-01"  -Location $location -Tier Basic `
   -NumberofWorkers 2 -WorkerSize Small
New-AzWebApp -ResourceGroupName $rg -Name <site1-name> -Location $location -AppServicePlan ASP-01
New-AzWebApp -ResourceGroupName $rg -Name <site2-name> -Location $location -AppServicePlan ASP-01
```

## <a name="configure-the-infrastructure"></a>Configuración de la infraestructura

Configure la dirección IP, la dirección IP de front-end, el grupo de servidores back-end, la configuración HTTP, el certificado, el puerto, el agente de escucha y la regla con un formato idéntico al de la instancia estándar de Application Gateway. La nueva SKU sigue el mismo modelo de objeto que la SKU estándar.

Reemplace los dos FQDN de la aplicación web (por ejemplo: `mywebapp.azurewebsites.net`) en la definición de la variable $pool.

```azurepowershell
$ipconfig = New-AzApplicationGatewayIPConfiguration -Name "IPConfig" -Subnet $gwSubnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIPCOnfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "Pool1" `
       -BackendIPAddresses <your first web app FQDN>, <your second web app FQDN>
$fp01 = New-AzApplicationGatewayFrontendPort -Name "SSLPort" -Port 443
$fp02 = New-AzApplicationGatewayFrontendPort -Name "HTTPPort" -Port 80

$securepfxpwd = ConvertTo-SecureString -String "Azure123456!" -AsPlainText -Force
$sslCert01 = New-AzApplicationGatewaySslCertificate -Name "SSLCert" -Password $securepfxpwd `
            -CertificateFile "c:\appgwcert.pfx"
$listener01 = New-AzApplicationGatewayHttpListener -Name "SSLListener" `
             -Protocol Https -FrontendIPConfiguration $fip -FrontendPort $fp01 -SslCertificate $sslCert01
$listener02 = New-AzApplicationGatewayHttpListener -Name "HTTPListener" `
             -Protocol Http -FrontendIPConfiguration $fip -FrontendPort $fp02

$setting = New-AzApplicationGatewayBackendHttpSettings -Name "BackendHttpSetting1" `
          -Port 80 -Protocol Http -CookieBasedAffinity Disabled -PickHostNameFromBackendAddress
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener01 -BackendAddressPool $pool
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule2" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener02 -BackendAddressPool $pool
```

## <a name="specify-autoscale"></a>Especificación de la escalabilidad automática

Ahora puede especificar la configuración de la escalabilidad automática para Application Gateway. 

   ```azurepowershell
   $autoscaleConfig = New-AzApplicationGatewayAutoscaleConfiguration -MinCapacity 2
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
   ```
En este modo, la puerta de enlace de aplicaciones escala automáticamente en función del patrón de tráfico de las aplicaciones.

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Cree la instancia de Application Gateway e incluya las zonas de redundancia y la configuración de escalabilidad automática.

```azurepowershell
$appgw = New-AzApplicationGateway -Name "AutoscalingAppGw" -Zone 1,2,3 `
  -ResourceGroupName $rg -Location $location -BackendAddressPools $pool `
  -BackendHttpSettingsCollection $setting -GatewayIpConfigurations $ipconfig `
  -FrontendIpConfigurations $fip -FrontendPorts $fp01, $fp02 `
  -HttpListeners $listener01, $listener02 -RequestRoutingRules $rule01, $rule02 `
  -Sku $sku -sslCertificates $sslCert01 -AutoscaleConfiguration $autoscaleConfig
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Use Get-AzureRmPublicIPAddress para obtener la dirección IP pública de la puerta de enlace de aplicaciones. Copie la dirección IP pública o el nombre DNS y péguelo en la barra de direcciones del explorador.

```azurepowershell
$pip = Get-AzPublicIPAddress -ResourceGroupName $rg -Name AppGwVIP
$pip.IpAddress
```


## <a name="clean-up-resources"></a>Limpieza de recursos

En primer lugar, explore los recursos que se crearon con Application Gateway. Después, cuando ya no se necesiten, puede usar el comando `Remove-AzResourceGroup` para quitar el grupo de recursos, la instancia de Application Gateway y todos los recursos relacionados.

`Remove-AzResourceGroup -Name $rg`

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Crear una puerta de enlace de aplicaciones con reglas de enrutamiento basadas en rutas de direcciones URL](./tutorial-url-route-powershell.md)
