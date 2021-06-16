---
title: Configuración de VPN en el dispositivo de Azure Stack Edge Pro R mediante Azure PowerShell
description: Describe cómo configurar una VPN en el dispositivo de Azure Stack Edge Pro R mediante un script de Azure PowerShell para crear recursos de Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/23/2020
ms.author: alkohli
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 750fd8e64c1c42b6b6452ef20041c2b34d5ef287
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701192"
---
# <a name="configure-vpn-on-your-azure-stack-edge-pro-r-device-via-azure-powershell"></a>Configuración de una VPN en el dispositivo de Azure Stack Edge Pro R mediante Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

La opción de VPN proporciona una segunda capa de cifrado para los datos en movimiento a través de *TLS* desde el dispositivo de Azure Stack Edge Pro R a Azure. Puede configurar una VPN en el dispositivo de Azure Stack Edge Pro R a través de Azure Portal o Azure PowerShell.

En este artículo se describen los pasos necesarios para configurar una VPN en el dispositivo de Azure Stack Edge Pro R mediante Azure PowerShell para crear la configuración en la nube.

## <a name="about-vpn-setup"></a>Acerca de la configuración de VPN

Una conexión VPN entre locales consta de una puerta de enlace de VPN de Azure, un dispositivo VPN local y un túnel VPN S2S de IPsec que conecta los dos. El flujo de trabajo típico incluye los siguientes pasos:

- Configurar los requisitos previos.
- Configurar los recursos necesarios en Azure.
    - Crear y configurar una instancia de Azure VPN Gateway (puerta de enlace de red virtual).
    - Crear y configurar una puerta de enlace de red local de Azure que representa la red local y el dispositivo VPN.
    - Crear y configurar una conexión VPN de Azure entre Azure VPN Gateway y la puerta de enlace de red local.
    - Configurar Azure Firewall y agregar reglas de red y aplicación.
    - Crear tablas de enrutamiento de Azure y agregar rutas.
- Configurar una VPN en la interfaz de usuario web local del dispositivo. Configure el dispositivo VPN local representado por la puerta de enlace de red local para establecer el túnel VPN S2S real con Azure VPN Gateway.

En las secciones siguientes se proporcionan los pasos detallados.

## <a name="configure-prerequisites"></a>Configuración de los requisitos previos

1. Debe tener acceso a un dispositivo Azure Stack Edge Pro R que se instale según las instrucciones incluidas en el artículo sobre cómo [instalar el dispositivo Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-install.md). Este dispositivo actuará como el dispositivo VPN local para crear la conexión VPN con Azure. 

2. El dispositivo VPN debe tener una dirección IP pública estática (externa). Esta dirección no debe ser una NAT.

3. Debe tener acceso a una suscripción de Azure válida habilitada para el servicio Azure Stack Edge en Azure. Use esta suscripción para crear un recurso correspondiente en Azure a fin de administrar el dispositivo de Azure Stack Edge Pro R.  

4. Tiene acceso a un cliente Windows que usará para tener acceso al dispositivo de Azure Stack Edge Pro R. Usará este cliente para crear mediante programación la configuración en la nube.

    1. Para instalar la versión necesaria de PowerShell en el cliente Windows, ejecute los siguientes comandos:

        ```azurepowershell
        Install-Module -Name Az -AllowClobber -Scope CurrentUser 
        Import-Module Az.Accounts
        ```
    2. Para conectarse a su cuenta y suscripción de Azure, ejecute los siguientes comandos:

        ```azurepowershell
        Connect-AzAccount 
        Set-AzContext -Subscription "<Your subscription name>"
        ```
        Proporcione el nombre de la suscripción de Azure que usa con el dispositivo de Azure Stack Edge Pro R para configurar la VPN.

    3. [Descargue el script](https://aka.ms/ase-vpn-deployment) necesario para crear la configuración en la nube. Este script le permitirá hacer lo siguiente:
        
        - Crear una instancia de Azure Virtual Network y las subredes siguientes: *GatewaySubnet* y *AzureFirewallSubnet*.
        - Crear y configurar una instancia de Azure VPN Gateway.
        - Crear y configurar una puerta de enlace de red local de Azure.
        - Crear y configurar una conexión VPN de Azure entre la instancia de Azure VPN Gateway y la puerta de enlace de red local.
        - Crear una instancia de Azure Firewall y agregar reglas de red y aplicación.
        - Crear una tabla de enrutamiento de Azure y agregarle rutas.


## <a name="use-the-script"></a>Uso del script

En primer lugar, modifique el archivo `parameters.json` para escribir los parámetros. A continuación, ejecute el script con el archivo JSON modificado.

En las siguientes secciones se explican cada uno de estos pasos.

### <a name="download-service-tags-file"></a>Descarga del archivo de etiquetas de servicio

Es posible que ya tenga un archivo `ServiceTags.json` en la carpeta donde descargó el script. Si no es así, puede descargar el archivo de etiquetas de servicio.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Modificación del archivo de parámetros

El primer paso sería modificar el archivo `parameters.json` y guardar los cambios. 


Para los recursos de Azure que cree, proporcionará los siguientes nombres:

|Nombre de parámetro  |Descripción  |
|---------|---------|
|virtualNetworks_vnet_name    | Nombre de Azure Virtual Network        |
|azureFirewalls_firewall_name     | Nombre de Azure Firewall        |
|routeTables_routetable_name     | Nombre de tabla de rutas de Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Nombre de dirección IP pública de la puerta de enlace de red virtual       |
|virtualNetworkGateways_VNGW_name    | Nombre de Azure VPN Gateway (puerta de enlace de red virtual)        |
|publicIPAddresses_firewall_public_ip_name     | Nombre de dirección IP pública de su instancia de Azure Firewall         |
|localNetworkGateways_LNGW_name    |Nombre de puerta de enlace de red local de Azure          |
|connections_vngw_lngw_name    | Nombre de la conexión VPN de Azure. Esta es la conexión entre la puerta de enlace de red virtual y la puerta de enlace de red local.       |
|ubicación     |Esta es la región en la que desea crear la red virtual. Seleccione la misma región que la asociada al dispositivo.         |

Las siguientes direcciones IP y espacios de direcciones corresponden a los recursos de Azure creados, entre los que se incluyen la red virtual y las subredes asociadas (predeterminada, de firewall y GatewaySubnet).

|Nombre de parámetro  |Descripción  |
|---------|---------|
|VnetIPv4AddressSpace    | Este es el espacio de direcciones asociado a la red virtual.       |
|DefaultSubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `Default` de la red virtual.         |
|FirewallSubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `Firewall` de la red virtual.          |
|GatewaySubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `GatewaySubnet` de la red virtual.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Esta es la dirección IP que se reserva para la comunicación BGP y se basa en el espacio de direcciones asociado a la subred `GatewaySubnet` de la red virtual.          |

Las siguientes direcciones IP y espacios de direcciones pertenecen a la red local (en la que está implementado el dispositivo de Azure Stack Edge Pro R).

|Nombre de parámetro  |Descripción  |
|---------|---------|
|CustomerNetworkAddressSpace    |  Este es el espacio de direcciones para la dirección IP privada.       |
|CustomerPublicNetworkAddressSpace    |  Este es el espacio de direcciones para la dirección IP pública.       |
|DbeIOTNetworkAddressSpace    |Esta dirección IP está reservada para el servicio IoT. No cambie este parámetro.        |
|AzureVPNsharedKey    |Esta clave compartida se usa durante la creación del recurso de conexión VPN de Azure. Esta clave también se proporciona como secreto compartido de VPN durante la configuración de la VPN de la interfaz de usuario web local.         |
|DBE-Gateway-ipaddress   | Dirección IP pública de un dispositivo de Azure Stack Edge Pro R. Esta puede no ser conocida y puede ejecutar el script con una dirección IP de marcador de posición. Edite la puerta de enlace de red local más adelante con la dirección IP real.     |

#### <a name="caveats-to-keep-in-mind"></a>Salvedades que debe tener en cuenta:

- No tendrá la dirección IP del dispositivo de Azure Stack Edge Pro R. Puede usar una dirección IP de marcador de posición para crear el recurso y, posteriormente, modificar la puerta de enlace de red local de Azure para asignar la dirección IP del dispositivo real y el espacio de direcciones de la red local del dispositivo.
- En función de la dirección de IETF en la autoridad de asignación de números de Internet (IANA), use cualquier subred de 172.16.x.y a 172.24.z.a. Los intervalos de direcciones IPv4 172.24 están reservados para la red de Azure.

### <a name="run-the-script"></a>Ejecute el script.

Siga estos pasos para usar el archivo `parameters.json` modificado y ejecutar el script para crear recursos de Azure.

1. Ejecute PowerShell como administrador.

2. Cambie al directorio en el que se encuentra el script.

3. Ejecute el script.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    A continuación se muestra una salida de ejemplo.

    `.\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "ServiceTags_Public_20191216.json"  -ResourceGroupName "devtestrg4" -AzureDeploymentName "dbetestdeployment20" -NetworkRuleCollectionName "testnrc20" -Priority 115 -AppRuleCollectionName "testarc20"`

    El script tarda 90 minutos aproximadamente en ejecutarse. Una vez completado el script, se genera un registro de implementación en la misma carpeta en la que reside el script.


## <a name="verify-the-azure-resources"></a>Comprobación de los recursos de Azure

Una vez que haya ejecutado correctamente el script, compruebe que todos los recursos se crearon en Azure.

Después, configurará la VPN en la interfaz de usuario web local del dispositivo.


## <a name="vpn-configuration-on-the-device"></a>Configuración de VPN en el dispositivo

[!INCLUDE [azure-stack-edge-gateway-configure-vpn-local-ui](../../includes/azure-stack-edge-gateway-configure-vpn-local-ui.md)]


## <a name="verify-vpn"></a>Comprobación de VPN

[!INCLUDE [azure-stack-edge-gateway-verify-vpn](../../includes/azure-stack-edge-gateway-verify-vpn.md)]

## <a name="validate-data-transfer-through-vpn"></a>Validación de la transferencia de datos a través de VPN

Para confirmar que la VPN funciona, copie los datos en un recurso compartido SMB. Siga los pasos descritos en [Agregar un recurso compartido](azure-stack-edge-gpu-manage-shares.md#add-a-share) en el dispositivo de Azure Stack Edge Pro R. 

1. Copie un archivo, por ejemplo \data\pictures\waterfall.jpg en el recurso compartido SMB que montó en el sistema cliente. 
2. Compruebe que este archivo se muestra en la cuenta de almacenamiento de la nube.

Para validar que los datos se pasan a través de VPN:

1. Abra el recurso de conexión presente en el grupo de recursos. 

2. Compruebe el valor de datos de entrada y datos de salida.
 
3. Abra la instancia de Virtual Network Gateway en el grupo de recursos. Vea los gráficos de **Total de entrada del túnel** y **Total de salida del túnel**.
 
## <a name="debug-issues"></a>Depuración de problemas

Para depurar cualquier problema, use los siguientes comandos:

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

A continuación se muestra la salida de ejemplo:


```azurepowershell
PS C:\Projects\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "aseprorvpnrg14_deployment" -ResourceGroupName "aseprorvpnrg14"


DeploymentName          : aseprorvpnrg14_deployment
ResourceGroupName       : aseprorvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 10/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     aseprorvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     aseprorvpnrg14_firewall
                          routeTables_routetable_name                  String                     aseprorvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     aseprorvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     aseprorvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     aseprorvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     aseprorvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     aseprorvpnrg14_connection
                          location                                     String                     East US
                          vnetIPv4AddressSpace                         String                     172.24.0.0/16
                          defaultSubnetIPv4AddressSpace                String                     172.24.0.0/24
                          firewallSubnetIPv4AddressSpace               String                     172.24.1.0/24
                          gatewaySubnetIPv4AddressSpace                String                     172.24.2.0/24
                          gatewaySubnetIPv4bgpPeeringAddress           String                     172.24.2.254
                          customerNetworkAddressSpace                  String                     10.0.0.0/18
                          customerPublicNetworkAddressSpace            String                     207.68.128.0/24
                          dbeIOTNetworkAddressSpace                    String                     10.139.218.0/24
                          azureVPNsharedKey                            String                     1234567890
                          dbE-Gateway-ipaddress                        String                     207.68.128.113

Outputs                 :
                          Name                     Type                       Value
                          =======================  =========================  ==========
                          virtualNetwork           Object                     {
                            "provisioningState": "Succeeded",
                            "resourceGuid": "dcf673d3-5c73-4764-b077-77125eda1303",
                            "addressSpace": {
                              "addressPrefixes": [
                                "172.24.0.0/16"
                              ]
================= CUT ============================= CUT ===========================
```


```
Get-AzResourceGroupDeploymentOperation -ResourceGroupName $ResourceGroupName -DeploymentName $AzureDeploymentName
```

## <a name="next-steps"></a>Pasos siguientes

[Configuración de la VPN a través de la interfaz de usuario local en el dispositivo de Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn).
