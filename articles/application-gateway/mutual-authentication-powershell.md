---
title: Configuración de la autenticación mutua en Azure Application Gateway mediante PowerShell
description: Aprenda a configurar una instancia de Application Gateway para que tenga autenticación mutua mediante PowerShell.
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.topic: how-to
ms.date: 04/02/2021
ms.author: caya
ms.openlocfilehash: 95534760c09ca9e1f7f09d6079886216127c7eb0
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106230937"
---
# <a name="configure-mutual-authentication-with-application-gateway-through-powershell-preview"></a>Configuración de la autenticación mutua con Application Gateway mediante PowerShell (versión preliminar)
En este artículo se describe cómo usar PowerShell para configurar la autenticación mutua en Application Gateway. La autenticación mutua significa que Application Gateway autentica al cliente que envía la solicitud mediante el certificado de cliente que se carga en Application Gateway. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este artículo se requiere la versión 1.0.0 del módulo de Azure PowerShell, u otra posterior. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

## <a name="before-you-begin"></a>Antes de empezar

Para configurar la autenticación mutua con una instancia de Application Gateway, necesita cargar un certificado de cliente en la puerta de enlace. El certificado de cliente se utilizará para validar el certificado que el cliente va a presentar a Application Gateway. Para fines de prueba, puede usar un certificado autofirmado. Sin embargo, estos certificados no se recomiendan para las cargas de trabajo de producción, ya que son más difíciles de administrar y no son completamente seguros.

Para más información, en concreto, sobre qué tipo de certificados de cliente se pueden cargar, consulte [Introducción a la autenticación mutua con Application Gateway](./mutual-authentication-overview.md#certificates-supported-for-mutual-authentication).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En primer lugar, cree un grupo de recursos en la suscripción. 

```azurepowershell
$resourceGroup = New-AzResourceGroup -Name $rgname -Location $location -Tags @{ testtag = "APPGw tag"}
```
## <a name="create-a-virtual-network"></a>Creación de una red virtual

Implemente una red virtual para la instancia de Application Gateway en la que se va a implementar.

```azurepowershell
$gwSubnet = New-AzVirtualNetworkSubnetConfig -Name $gwSubnetName -AddressPrefix 10.0.0.0/24
$vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgname -Location $location -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgname
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name $gwSubnetName -VirtualNetwork $vnet
```

## <a name="create-a-public-ip"></a>Cree una dirección IP pública.

Cree una dirección IP pública para la instancia de Application Gateway. 

```azurepowershell
$publicip = New-AzPublicIpAddress -ResourceGroupName $rgname -name $publicIpName -location $location -AllocationMethod Static -sku Standard
```

## <a name="create-the-application-gateway-ip-configuration"></a>Creación de la configuración de IP de Application Gateway

Cree las configuraciones de IP y el puerto de front-end. 

```azurepowershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name $gipconfigname -Subnet $gwSubnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name $fipconfigName -PublicIPAddress $publicip
$port = New-AzApplicationGatewayFrontendPort -Name $frontendPortName  -Port 443
```

## <a name="configure-frontend-ssl"></a>Configuración de SSL de front-end 

Configure los certificados SSL para la instancia de Application Gateway.

```azurepowershell
$password = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force
$sslCertPath = $basedir + "/ScenarioTests/Data/ApplicationGatewaySslCert1.pfx"
$sslCert = New-AzApplicationGatewaySslCertificate -Name $sslCertName -CertificateFile $sslCertPath -Password $password
```

## <a name="configure-client-authentication"></a>Configuración de la autenticación de clientes 

Configure la autenticación de cliente en la instancia de Application Gateway. Para más información sobre cómo extraer cadenas de certificados de entidad de certificación de cliente de confianza para usarlas aquí, consulte [Extracción de cadenas de certificados de entidad de certificación de cliente de confianza](./mutual-authentication-certificate-management.md).

> [!IMPORTANT]
> Asegúrese de cargar toda la cadena de certificados de la entidad de certificación de cliente en un archivo y solo una cadena por archivo.  

> [!NOTE]
> Se recomienda usar TLS 1.2 con la autenticación mutua, ya que esta versión de TLS será obligatoria en el futuro. 

```azurepowershell
$clientCertFilePath = $basedir + "/ScenarioTests/Data/TrustedClientCertificate.cer"
$trustedClient01 = New-AzApplicationGatewayTrustedClientCertificate -Name $trustedClientCert01Name -CertificateFile $clientCertFilePath
$sslPolicy = New-AzApplicationGatewaySslPolicy -PolicyType Predefined -PolicyName "AppGwSslPolicy20170401S"
$clientAuthConfig = New-AzApplicationGatewayClientAuthConfiguration -VerifyClientCertIssuerDN
$sslProfile01 = New-AzApplicationGatewaySslProfile -Name $sslProfile01Name -SslPolicy $sslPolicy -ClientAuthConfiguration $clientAuthConfig -TrustedClientCertificates $trustedClient01
$listener = New-AzApplicationGatewayHttpListener -Name $listenerName -Protocol Https -SslCertificate $sslCert -FrontendIPConfiguration $fipconfig -FrontendPort $port -SslProfile $sslProfile01
```

## <a name="configure-the-backend-pool-and-settings"></a>Configuración de los valores y el grupo de back-end

Configure los valores y el grupo de back-end de la instancia de Application Gateway. También puede configurar el certificado raíz de confianza de back-end para el cifrado SSL de un extremo a otro.  

```azurepowershell
$certFilePath = $basedir + "/ScenarioTests/Data/ApplicationGatewayAuthCert.cer"
$trustedRoot = New-AzApplicationGatewayTrustedRootCertificate -Name $trustedRootCertName -CertificateFile $certFilePath
$pool = New-AzApplicationGatewayBackendAddressPool -Name $poolName -BackendIPAddresses www.microsoft.com, www.bing.com
$poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name $poolSettingName -Port 443 -Protocol Https -CookieBasedAffinity Enabled -PickHostNameFromBackendAddress -TrustedRootCertificate $trustedRoot
```

## <a name="configure-the-rule"></a>Configuración de la regla

Configure una regla en la instancia de Application Gateway.

```azurepowershell
$rule = New-AzApplicationGatewayRequestRoutingRule -Name $ruleName -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
```

## <a name="set-up-default-ssl-policy-for-future-listeners"></a>Configuración de la directiva SSL predeterminada para clientes de escucha futuros

Ha configurado una directiva SSL específica del cliente de escucha mientras configuraba la autenticación mutua. En este paso, puede establecer opcionalmente la directiva SSL predeterminada para los futuros clientes de escucha que cree. 

```azurepowershell
$sslPolicyGlobal = New-AzApplicationGatewaySslPolicy -PolicyType Predefined -PolicyName "AppGwSslPolicy20170401"
```

## <a name="create-the-application-gateway"></a>Creación de Application Gateway

Con todo lo que hemos creado anteriormente, implemente la instancia de Application Gateway.

```azurepowershell
$sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
$appgw = New-AzApplicationGateway -Name $appgwName -ResourceGroupName $rgname -Zone 1,2 -Location $location -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $port -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SslPolicy $sslPolicyGlobal -TrustedRootCertificate $trustedRoot -AutoscaleConfiguration $autoscaleConfig -TrustedClientCertificates $trustedClient01 -SslProfiles $sslProfile01 -SslCertificates $sslCert
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurepowershell
Remove-AzResourceGroup -Name $rgname
```

## <a name="renew-expired-client-ca-certificates"></a>Renovación de certificados de entidad de certificación de cliente expirados

En el caso de que el certificado de entidad de certificación de cliente haya expirado, puede actualizarlo en la puerta de enlace mediante los pasos siguientes: 

1. Inicio de sesión en Azure
    ```azurepowershell
    Connect-AzAccount
    Select-AzSubscription -Subscription "<sub name>"
    ```
2. Obtenga la configuración de Application Gateway.
    ```azurepowershell
    $gateway = Get-AzApplicationGateway -Name "<gateway-name>" -ResourceGroupName "<resource-group-name>"
    ```
3. Elimine el certificado de cliente de confianza de la puerta de enlace. 
    ```azurepowershell
    Remove-AzApplicationGatewayTrustedClientCertificate -Name "<name-of-client-certificate>" -ApplicationGateway $gateway
    ``` 
4. Agregue el nuevo certificado a la puerta de enlace. 
    ```azurepowershell
    Add-AzApplicationGatewayTrustedClientCertificate -ApplicationGateway $gateway -Name "<name-of-new-cert>" -CertificateFile "<path-to-certificate-file>"
    ```
5. Actualice la puerta de enlace con el nuevo certificado. 
    ```azurepowershell
    Set-AzApplicationGateway -ApplicationGateway $gateway
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Administrar el tráfico web con Application Gateway mediante la CLI de Azure](./tutorial-manage-web-traffic-cli.md)