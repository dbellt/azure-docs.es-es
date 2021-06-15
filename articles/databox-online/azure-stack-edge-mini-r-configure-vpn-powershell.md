---
title: Configuración de VPN en el dispositivo Azure Stack Edge Mini R mediante Azure PowerShell
description: Describe cómo configurar VPN en el dispositivo Azure Stack Edge Mini R mediante un script de Azure PowerShell para crear recursos de Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 11/17/2020
ms.author: alkohli
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 487d30c350fc6871122c66bbb3faa6b656ea96f9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698635"
---
# <a name="configure-vpn-on-your-azure-stack-edge-mini-r-device-via-azure-powershell"></a>Configuración de VPN en el dispositivo Azure Stack Edge Mini R a través de Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

La opción de VPN proporciona una segunda capa de cifrado para los datos en movimiento a través de *TLS* desde el dispositivo Azure Stack Edge Mini R o Azure Stack Edge Pro R a Azure. Puede configurar VPN en el dispositivo Azure Stack Edge Mini R a través de Azure Portal o a través de Azure PowerShell. 

En este artículo se describen los pasos necesarios para configurar una VPN de punto a sitio (P2S) en el dispositivo Azure Stack Edge Mini R mediante un script de Azure PowerShell para crear la configuración en la nube. La configuración en el dispositivo Azure Stack Edge se realiza a través de la interfaz de usuario local.

## <a name="about-vpn-setup"></a>Acerca de la configuración de VPN

Una conexión de VPN Gateway P2S permite crear una conexión segura a la red virtual desde un equipo cliente individual o el dispositivo Azure Stack Edge Mini R. La conexión P2S se inicia desde el equipo cliente o el dispositivo. En este caso, la conexión P2S usa VPN IKEv2, una solución de VPN con protocolo de seguridad de Internet basada en estándares.

El flujo de trabajo típico incluye los siguientes pasos:

1. Configurar los requisitos previos.
2. Configurar los recursos necesarios en Azure.
    1. Crear y configurar una red virtual y las subredes necesarias. 
    2. Crear y configurar una instancia de Azure VPN Gateway (puerta de enlace de red virtual).
    3. Configurar Azure Firewall y agregar reglas de red y aplicación.
    4. Crear tablas de enrutamiento de Azure y agregar rutas.
    5. Habilitar la conexión de punto a sitio en una instancia de VPN Gateway.
        1. Incorpore el grupo de direcciones del cliente.
        2. Configurar el tipo de túnel.
        3. Configurar el tipo de autenticación.
        4. Crear un certificado.
        5. Cargar el certificado.
    6. Descargar la libreta de teléfonos.
3. Configurar VPN en la interfaz de usuario web local del dispositivo. 
    1. Proporcionar la libreta de teléfonos.
    2. Proporcionar el archivo (json) de etiquetas de servicio.


En las secciones siguientes se proporcionan los pasos detallados.

## <a name="configure-prerequisites"></a>Configuración de los requisitos previos

- Debe tener acceso a un dispositivo Azure Stack Edge Mini R que se instale según las instrucciones incluidas en el artículo sobre cómo [instalar el dispositivo Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md). Este dispositivo establecerá una conexión P2S con Azure. 

- Debe tener acceso a una suscripción de Azure válida habilitada para el servicio Azure Stack Edge en Azure. Use esta suscripción para crear un recurso correspondiente en Azure a fin de administrar el dispositivo Azure Stack Edge Mini R.  

- Tiene acceso a un cliente Windows que usará para tener acceso al dispositivo Azure Stack Edge Mini R. Usará este cliente para crear mediante programación la configuración en la nube.

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
        Proporcione el nombre de la suscripción de Azure que usa con el dispositivo Azure Stack Mini R para configurar la VPN.

    3. [Descargue el script](https://aka.ms/ase-vpn-deployment) necesario para crear la configuración en la nube. Este script le permitirá hacer lo siguiente:
        
        - Crear una instancia de Azure Virtual Network y las subredes siguientes: *GatewaySubnet* y *AzureFirewallSubnet*.
        - Crear y configurar una instancia de Azure VPN Gateway.
        - Crear y configurar una puerta de enlace de red local de Azure.
        - Crear y configurar una conexión VPN de Azure entre la instancia de Azure VPN Gateway y la puerta de enlace de red local.
        - Crear una instancia de Azure Firewall y agregar reglas de red y aplicación.
        - Crear una tabla de enrutamiento de Azure y agregarle rutas.

    4. Crear el grupo de recursos en Azure Portal en el que desea que desea que se creen los recursos de Azure. Ir a la lista de servicios en Azure Portal, seleccionar **Grupo de recursos** y, a continuación, seleccionar **+ Agregar**. Proporcionar la información de suscripción y el nombre del grupo de recursos y, a continuación, seleccionar **Crear**. Si va a este grupo de recursos, no debería tener ningún recurso en este momento.

        ![Grupo de recursos de Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-resource-group-1.png)
    
    5. Tendrá que tener un certificado codificado en Base 64 en formato `.cer` para el dispositivo Azure Stack Edge Mini R. Este certificado debe cargarse en el dispositivo Azure Stack Edge como `pfx` con una clave privada. Este certificado también debe instalarse en la raíz de confianza del almacén en el cliente que intente establecer la conexión P2S.

## <a name="use-the-script"></a>Uso del script

En primer lugar, modifique el archivo `parameters-p2s.json` para escribir los parámetros. A continuación, ejecute el script con el archivo JSON modificado.

En las siguientes secciones se explican cada uno de estos pasos.

### <a name="download-service-tags-file"></a>Descarga del archivo de etiquetas de servicio

Es posible que ya tenga un archivo `ServiceTags.json` en la carpeta donde descargó el script. Si no es así, puede descargar el archivo de etiquetas de servicio.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Modificación del archivo de parámetros

El primer paso sería modificar el archivo `parameters-p2s.json` y guardar los cambios. 

Para los recursos de Azure que cree, proporcionará los siguientes nombres:

|Nombre de parámetro  |Descripción  |
|---------|---------|
|virtualNetworks_vnet_name    | Nombre de Azure Virtual Network        |
|azureFirewalls_firewall_name     | Nombre de Azure Firewall        |
|routeTables_routetable_name     | Nombre de tabla de rutas de Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Nombre de dirección IP pública de la puerta de enlace de red virtual       |
|virtualNetworkGateways_VNGW_name    | Nombre de Azure VPN Gateway (puerta de enlace de red virtual)        |
|publicIPAddresses_firewall_public_ip_name     | Nombre de dirección IP pública de su instancia de Azure Firewall         |
|ubicación     |Esta es la región en la que desea crear la red virtual. Seleccione la misma región que la asociada al dispositivo.         |
|RouteTables_routetable_onprem_name| Este es el nombre de la tabla de rutas adicional para ayudar al firewall a volver a enrutar los paquetes al dispositivo Azure Stack Edge. El script crea dos rutas adicionales y asocia *predeterminada* y *FirewallSubnet* a esta tabla de rutas.|

Proporcione las siguientes direcciones IP y espacios de direcciones para los recursos de Azure creados, entre los que se incluyen la red virtual y las subredes asociadas (*predeterminada*, de *firewall* y *GatewaySubnet*).

|Nombre de parámetro  |Descripción  |
|---------|---------|
|VnetIPv4AddressSpace    | Este es el espacio de direcciones asociado a la red virtual. Proporcione el intervalo IP de red virtual como intervalo IP privado (https://en.wikipedia.org/wiki/Private_network#Private_IPv4_addresses).     |
|DefaultSubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `Default` de la red virtual.         |
|FirewallSubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `Firewall` de la red virtual.          |
|GatewaySubnetIPv4AddressSpace    |Este es el espacio de direcciones asociado a la subred `GatewaySubnet` de la red virtual.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Esta es la dirección IP que se reserva para la comunicación BGP y se basa en el espacio de direcciones asociado a la subred `GatewaySubnet` de la red virtual.          |
|ClientAddressPool    | Esta dirección IP se usa para el grupo de direcciones en la configuración de P2S en Azure Portal.         |
|PublicCertData     | VPN Gateway usa los datos de certificado público para autenticar a los clientes de P2S que se conectan a ella. Para obtener los datos del certificado, instale el certificado raíz. Asegúrese de que el certificado está codificado en Base 64 con una extensión .cer. Abra este certificado y copie el texto en el certificado entre ==BEGIN CERTIFICATE== y ==END CERTIFICATE== en una línea continua.     |



### <a name="run-the-script"></a>Ejecute el script.

Siga estos pasos para usar el archivo `parameters-p2s.json` modificado y ejecutar el script para crear recursos de Azure.

1. Ejecute PowerShell. Cambie al directorio donde se encuentra el script.

3. Ejecute el script.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    > [!NOTE]
    > En esta versión, el script funciona solo en la ubicación Este de EE. UU.

    Tendrá que escribir la siguiente información al ejecutar el script:

    
    |Parámetro  |Descripción  |
    |---------|---------|
    |Location     |Esta es la región en la que se deben crear los recursos de Azure.         |
    |AzureAppRuleFilePath     | Esta es la ruta de acceso del archivo para `appRule.json`.       |
    |AzureIPRangesFilePath     |Este es el archivo JSON de la etiqueta de servicio que descargó en el paso anterior.         |
    |ResourceGroupName     | Este es el nombre del grupo de recursos en el que se crean todos los recursos de Azure.        |
    |AzureDeploymentName    |Este es el nombre de la implementación de Azure.         |
    |NetworkRuleCollectionName            | Este es el nombre de la colección de todas las reglas de red que se crean y agregan a su instancia de Azure Firewall.             |
    |Priority            | Esta es la prioridad asignada a todas las reglas de red y de aplicación que se crean.              |
    |AppRuleCollectionName            |Este es el nombre de la colección de todas las reglas de aplicación que se crean y agregan a su instancia de Azure Firewall.                |


    A continuación se muestra una salida de ejemplo.
    
    ```powershell
    PS C:\Offline docs\AzureVpnConfigurationScripts> .\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath ".\ServiceTags_Public_20200203.json" -ResourceGroupName "mytmarg3" -AzureDeploymentName "tmap2stestdeploy1" -NetworkRuleCollectionName "testnrc1" -Priority 115 -AppRuleCollectionName "testarc2"
        validating vpn deployment parameters
        Starting vpn deployment
        C:\Offline docs\AzureVpnConfigurationScripts\parameters-p2s.json
        C:\Offline docs\AzureVpnConfigurationScripts\template-p2s.json
        vpn deployment: tmap2stestdeploy1 started and status: Running
        Waiting for vpn deployment completion....
        ==== CUT ==================== CUT ==============
        Adding route 191.236.0.0/18 for AzureCloud.eastus
        Adding route 191.237.0.0/17 for AzureCloud.eastus
        Adding route 191.238.0.0/18 for AzureCloud.eastus
        Total Routes:294, Existing Routes: 74, New Routes Added: 220
        Additional routes getting added
    ```    

    > [!IMPORTANT]
    > - El script tarda 90 minutos aproximadamente en ejecutarse. Asegúrese de iniciar sesión en la red justo antes de que se inicie el script.
    > - Si, por alguna razón, se produce un error en una sesión con el script, asegúrese de eliminar el grupo de recursos para eliminar todos los recursos creados en él.
  
    
    Una vez completado el script, se genera un registro de implementación en la misma carpeta donde reside el script.


## <a name="verify-the-azure-resources"></a>Comprobación de los recursos de Azure

Una vez que haya ejecutado correctamente el script, compruebe que todos los recursos se crearon en Azure. Vaya al grupo de recursos que ha creado. Aparecerán los siguientes recursos:

![Recursos de Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/script-resources.png)


<!--## Enable point-to-site in VPN gateway

1. In the Azure portal, go to the resource group and then select the virtual network gateway that you created in the earlier step.

    ![Azure virtual network gateway](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Go to **Settings > Point-to-site configuration**. Select **Configure now**.

    ![Enable P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s.png)


3.  On the **Point-to-site configuration** blade:

    1. You'll add the client address pool. This pool is a range of private IP addresses that you specify. The clients that connect over a P2S VPN dynamically receive an IP address from this range. Use a private IP address range that does not overlap with the on-premises location that you connect from, or the VNet that you want to connect to. 
    2. You can select the tunnel type. For the VPN tunnel, you will use the IKEv2 protocol. 
    3. You will now define the type of authentication. Before Azure accepts a P2S VPN connection, the user has to be authenticated first. In this case, you authenticate using the native Azure certificate authentication. Set **Authentication type** to **Azure certificate**.
    4. You will now create and upload the certificates to Azure and your client. You will need to install a root certificate on your VPN gateway. The VPN gateway validates the client certificates when the P2S connection is being established by the client. The root certificate is required for the validation and must be uploaded to Azure.
    
        The clients trying to establish the P2S connection will have a client certificate that authenticates the connecting user.  To create these certificates, follow the steps in [Generate and export certificates for Point-to-Site using PowerShell](../vpn-gateway/vpn-gateway-certificates-point-to-site.md). 

        To install the root certificate, make sure the certificate is Base-64 encoded with a .cer extension. Open this certificate and copy the text in the certificate between ==BEGIN CERTIFICATE== and ==END CERTIFICATE== in one continuous line in the public certificate data under Root certificates.

        To upload the root certificates, follow the detailed steps in [Upload the root certificate public certificate data](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
    
    5. Save the configuration.

        ![Save P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s-config.png)-->

## <a name="download-phone-book-for-vpn-profile"></a>Descarga de la libreta de teléfonos para el perfil de VPN

En este paso, descargará el perfil de VPN para el dispositivo.

1. En Azure Portal, vaya al grupo de recursos y, a continuación, seleccione la puerta de enlace de red virtual que creó en el paso anterior.

    ![Puerta de enlace de red virtual de Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Vaya a **Configuración > Configuración de punto a sitio**. Seleccione **Descargar cliente VPN**.

    ![Habilitación de la configuración de P2S 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/download-vpn-client.png)

2. Guarde el perfil comprimido y extráigalo en el cliente Windows.

    ![Habilitación de la configuración de P2S 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/save-extract-profile.png)

3. Vaya a la carpeta *WindowsAmd64* y, a continuación, extraiga el archivo `.exe`: *VpnClientSetupAmd64.exe*.

    ![Habilitación de la configuración de P2S 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/extract-exe.png)

3. Cree una ruta de acceso temporal. Por ejemplo:

    `C:\NewTemp\vnet\tmp`

4. Ejecute PowerShell y vaya al directorio donde se encuentra el archivo `.exe`. Para ejecutar el archivo `.exe`, escriba:

    `.\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"`

5. La ruta de acceso temporal tendrá nuevos archivos. Este es una salida de ejemplo:

    ```powershell
    
    PS C:\windows\system32> cd "C:\Users\Ase\Downloads\vngw5\WindowsAmd64"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> .\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> cd "C:\NewTemp\vnet"
    PS C:\NewTemp\vnet> ls .\tmp
    
        Directory: C:\NewTemp\vnet\tmp
    
    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    -a----         2/6/2020   6:18 PM            947 8c670077-470b-421a-8dd8-8cedb4f2f08a.cer
    -a----         2/6/2020   6:18 PM            155 8c670077-470b-421a-8dd8-8cedb4f2f08a.cmp
    -a----         2/6/2020   6:18 PM           3564 8c670077-470b-421a-8dd8-8cedb4f2f08a.cms
    -a----         2/6/2020   6:18 PM          11535 8c670077-470b-421a-8dd8-8cedb4f2f08a.inf
    -a----         2/6/2020   6:18 PM           2285 8c670077-470b-421a-8dd8-8cedb4f2f08a.pbk
    -a----         2/6/2020   6:18 PM           5430 azurebox16.ico
    -a----         2/6/2020   6:18 PM           4286 azurebox32.ico
    -a----         2/6/2020   6:18 PM         138934 azurevpnbanner.bmp
    -a----         2/6/2020   6:18 PM          46064 cmroute.dll
    -a----         2/6/2020   6:18 PM            196 routes.txt
    
    PS C:\NewTemp\vnet>
    ```

6. El archivo *.pbk* es la libreta de teléfonos del perfil de VPN. Se usará en la interfaz de usuario local.


## <a name="vpn-configuration-on-the-device"></a>Configuración de VPN en el dispositivo

Siga estos pasos en la interfaz de usuario local del dispositivo Azure Stack Edge.

1. En la interfaz de usuario local, vaya a la página **VPN**. En el estado de la VPN, seleccione **Configurar**.

    ![Configuración de VPN 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-1.png)

2. En la hoja **Configure VPN** (Configurar VPN):
    
    1. En el campo de carga del archivo de libreta de teléfonos, seleccione el archivo .pbk que creó en el paso anterior.
    2. En el campo de carga del archivo de configuración de la lista de direcciones IP públicas, proporcione el archivo JSON de intervalo IP de centro de datos de Azure como entrada. Descargó este archivo en un paso anterior desde: [https://www.microsoft.com/download/details.aspx?id=56519](https://www.microsoft.com/download/details.aspx?id=56519).
    3. Seleccione **eastus** como región y seleccione **Aplicar**.

    ![Configuración de VPN 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-2.png)

3. En la sección **IP address ranges to be accessed using VPN only** (Intervalos de direcciones IP a los que acceder solo mediante VPN), escriba el intervalo IPv4 Vnet que había elegido para Azure Virtual Network.

    ![Configuración de VPN 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-3.png)

## <a name="verify-client-connection"></a>Comprobación de la conexión de cliente

1. En Azure Portal, vaya a VPN Gateway.
2. Vaya a **Configuración > Configuración de punto a sitio**. En **Direcciones IP asignadas**, debe aparecer la dirección IP del dispositivo Azure Stack Edge.

## <a name="validate-data-transfer-through-vpn"></a>Validación de la transferencia de datos a través de VPN

Para confirmar que la VPN funciona, copie los datos en un recurso compartido SMB. Siga los pasos descritos en [Agregar un recurso compartido](azure-stack-edge-gpu-manage-shares.md#add-a-share) en el dispositivo Azure Stack Edge. 

1. Copie un archivo, por ejemplo \data\pictures\waterfall.jpg en el recurso compartido SMB que montó en el sistema cliente. 
2. Para validar que los datos pasan por VPN mientras se copian:

    1. Vaya a VPN Gateway en Azure Portal. 

    2. Vaya a **Supervisión > Métricas**.

    3. En el panel de la derecha, elija el **Ámbito** como su instancia de VPN Gateway, la **Métrica** como el ancho de banda P2S de puerta de enlace y la **Agregación** como Promedio.

    4. A medida que se copian los datos, verá un aumento en la utilización del ancho de banda y, cuando se complete la copia de datos, dicha utilización desaparecerá.

        ![Métricas de VPN de Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/vpn-metrics-1.png)

3. Compruebe que este archivo se muestra en la cuenta de almacenamiento de la nube.
 
## <a name="debug-issues"></a>Depuración de problemas

Para depurar cualquier problema, use los siguientes comandos:

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

A continuación se muestra la salida de ejemplo:


```azurepowershell
PS C:\Projects\TZL\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "tznvpnrg14_deployment" -ResourceGroupName "tznvpnrg14"


DeploymentName          : tznvpnrg14_deployment
ResourceGroupName       : tznvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 1/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     tznvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     tznvpnrg14_firewall
                          routeTables_routetable_name                  String                     tznvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     tznvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     tznvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     tznvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     tznvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     tznvpnrg14_connection
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

[Configure la VPN a través de la interfaz de usuario local en el dispositivo Azure Stack Edge](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn).
