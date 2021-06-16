---
title: Configuración de una VPN de punto a sitio (P2S) en Windows para su uso con Azure Files | Microsoft Docs
description: Cómo configurar una VPN de punto a sitio (P2S) en Windows para su uso con Azure Files
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 979d1c9148e079728b240f73b467e6d9aee13067
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110663487"
---
# <a name="configure-a-point-to-site-p2s-vpn-on-windows-for-use-with-azure-files"></a>Configuración de una VPN de punto a sitio (P2S) en Windows para su uso con Azure Files
Puede usar una conexión VPN de punto a sitio (P2S) para montar los recursos compartidos de archivos de Azure a través de SMB desde fuera de Azure, sin necesidad de abrir el puerto 445. Una conexión VPN de punto a sitio es una conexión VPN entre Azure y un cliente individual. Para usar una conexión VPN de punto a sitio con Azure Files, se debe configurar una conexión VPN de punto a sitio para cada cliente que quiera conectarse. Si tiene muchos clientes que necesitan conectarse a sus recursos compartidos de archivos de Azure desde la red local, puede usar una conexión VPN de sitio a sitio (S2S) en lugar de una conexión de punto a sitio para cada cliente. Para obtener más información, consulte [Configuración de una VPN de sitio a sitio para su uso con Azure Files](storage-files-configure-s2s-vpn.md).

Se recomienda que consulte [Consideraciones sobre redes para el acceso directo a los recursos compartidos de archivos de Azure Files](storage-files-networking-overview.md) antes de continuar con este artículo para obtener una descripción completa de las opciones de red disponibles para Azure Files.

En este artículo se detallan los pasos para configurar una VPN de punto a sitio en Windows (cliente de Windows y Windows Server) para montar recursos compartidos de archivos de Azure directamente en el entorno local. Si quiere enrutar el tráfico de Azure File Sync a través de una VPN, consulte [Configuración del proxy y el firewall de Azure File Sync](../file-sync/file-sync-firewall-and-proxy.md).

## <a name="prerequisites"></a>Requisitos previos
- La versión más reciente del módulo de Azure PowerShell. Para obtener más información sobre cómo instalar Azure PowerShell, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps) y seleccione el sistema operativo. Si prefiere usar la CLI de Azure en Windows, puede hacerlo. Sin embargo, las instrucciones siguientes son para Azure PowerShell.

- Un recurso compartido de archivos de Azure que le gustaría montar en el entorno local. Los recursos compartidos de archivos de Azure se implementan en cuentas de almacenamiento, que son construcciones que representan un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas. Puede encontrar más información sobre cómo implementar cuentas de almacenamiento y recursos compartidos de archivos de Azure en [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).

- Una red virtual con un punto de conexión privado para la cuenta de almacenamiento que contiene el recurso compartido de archivos de Azure que quiere montar localmente. Para más información sobre cómo crear un punto de conexión privado, consulte [Configuración de puntos de conexión de red de Azure Files](storage-files-networking-endpoints.md?tabs=azure-powershell). 

## <a name="collect-environment-information"></a>Recopilación de información del entorno
Para configurar la VPN de punto a sitio, primero es necesario recopilar información sobre el entorno para su uso a lo largo de la guía. Consulte la sección de [requisitos previos](#prerequisites) si aún no ha creado una cuenta de almacenamiento, una red virtual o puntos de conexión privados.

No olvide reemplazar `<resource-group>`, `<vnet-name>`, `<subnet-name>` y `<storage-account-name>` por los valores correctos para su entorno.

```PowerShell
$resourceGroupName = "<resource-group-name>" 
$virtualNetworkName = "<vnet-name>"
$subnetName = "<subnet-name>"
$storageAccountName = "<storage-account-name>"

$virtualNetwork = Get-AzVirtualNetwork `
    -ResourceGroupName $resourceGroupName `
    -Name $virtualNetworkName

$subnetId = $virtualNetwork | `
    Select-Object -ExpandProperty Subnets | `
    Where-Object { $_.Name -eq "StorageAccountSubnet" } | `
    Select-Object -ExpandProperty Id

$storageAccount = Get-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName

$privateEndpoint = Get-AzPrivateEndpoint | `
    Where-Object {
        $subnets = $_ | `
            Select-Object -ExpandProperty Subnet | `
            Where-Object { $_.Id -eq $subnetId }

        $connections = $_ | `
            Select-Object -ExpandProperty PrivateLinkServiceConnections | `
            Where-Object { $_.PrivateLinkServiceId -eq $storageAccount.Id }
        
        $null -ne $subnets -and $null -ne $connections
    } | `
    Select-Object -First 1
```

## <a name="create-root-certificate-for-vpn-authentication"></a>Creación de certificados raíz para la autenticación de VPN
Para que las conexiones VPN de las máquinas Windows locales se autentiquen para acceder a la red virtual, debe crear dos certificados: un certificado raíz, que se proporcionará a la puerta de enlace de máquina virtual, y un certificado de cliente, que se firmará con el certificado raíz. El siguiente PowerShell crea el certificado raíz; el certificado de cliente se creará después de que se cree la puerta de enlace de red virtual de Azure con información de la puerta de enlace. 

```PowerShell
$rootcertname = "CN=P2SRootCert"
$certLocation = "Cert:\CurrentUser\My"
$vpnTemp = "C:\vpn-temp\"
$exportedencodedrootcertpath = $vpnTemp + "P2SRootCertencoded.cer"
$exportedrootcertpath = $vpnTemp + "P2SRootCert.cer"

if (-Not (Test-Path $vpnTemp)) {
    New-Item -ItemType Directory -Force -Path $vpnTemp | Out-Null
}

if ($PSVersionTable.PSVersion -ge [System.Version]::new(6, 0)) {
    Install-Module WindowsCompatibility
    Import-WinModule PKI
}

$rootcert = New-SelfSignedCertificate `
    -Type Custom `
    -KeySpec Signature `
    -Subject $rootcertname `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation $certLocation `
    -KeyUsageProperty Sign `
    -KeyUsage CertSign

Export-Certificate `
    -Cert $rootcert `
    -FilePath $exportedencodedrootcertpath `
    -NoClobber | Out-Null

certutil -encode $exportedencodedrootcertpath $exportedrootcertpath | Out-Null

$rawRootCertificate = Get-Content -Path $exportedrootcertpath

[System.String]$rootCertificate = ""
foreach($line in $rawRootCertificate) { 
    if ($line -notlike "*Certificate*") { 
        $rootCertificate += $line 
    } 
}
```

## <a name="deploy-virtual-network-gateway"></a>Implementación de puerta de enlace de red virtual
La puerta de enlace de red virtual de Azure es el servicio al que se conectarán las máquinas Windows locales. La implementación de este servicio requiere dos componentes básicos: una dirección IP pública que identificará la puerta de enlace para los clientes dondequiera que se encuentren en el mundo y un certificado raíz creado anteriormente que se usará para autenticar a los clientes.

No olvide reemplazar `<desired-vpn-name-here>` por el nombre que quiera para estos recursos.

> [!Note]  
> La implementación de una puerta de enlace de red virtual de Azure puede tardar hasta 45 minutos. Aunque este recurso va a implementarse, el script de PowerShell se bloqueará para que se complete la implementación. Se espera que esto sea así.

```PowerShell
$vpnName = "<desired-vpn-name-here>" 
$publicIpAddressName = "$vpnName-PublicIP"

$publicIPAddress = New-AzPublicIpAddress `
    -ResourceGroupName $resourceGroupName `
    -Name $publicIpAddressName `
    -Location $region `
    -Sku Basic `
    -AllocationMethod Dynamic

$gatewayIpConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "vnetGatewayConfig" `
    -SubnetId $gatewaySubnet.Id `
    -PublicIpAddressId $publicIPAddress.Id

$azRootCertificate = New-AzVpnClientRootCertificate `
    -Name "P2SRootCert" `
    -PublicCertData $rootCertificate

$vpn = New-AzVirtualNetworkGateway `
    -ResourceGroupName $resourceGroupName `
    -Name $vpnName `
    -Location $region `
    -GatewaySku VpnGw1 `
    -GatewayType Vpn `
    -VpnType RouteBased `
    -IpConfigurations $gatewayIpConfig `
    -VpnClientAddressPool "172.16.201.0/24" `
    -VpnClientProtocol IkeV2 `
    -VpnClientRootCertificates $azRootCertificate
```

## <a name="create-client-certificate"></a>Creación del certificado de cliente
El certificado de cliente se crea con el URI de la puerta de enlace de red virtual. Este certificado está firmado con el certificado raíz que creó anteriormente.

```PowerShell
$clientcertpassword = "1234"

$vpnClientConfiguration = New-AzVpnClientConfiguration `
    -ResourceGroupName $resourceGroupName `
    -Name $vpnName `
    -AuthenticationMethod EAPTLS

Invoke-WebRequest `
    -Uri $vpnClientConfiguration.VpnProfileSASUrl `
    -OutFile "$vpnTemp\vpnclientconfiguration.zip"

Expand-Archive `
    -Path "$vpnTemp\vpnclientconfiguration.zip" `
    -DestinationPath "$vpnTemp\vpnclientconfiguration"

$vpnGeneric = "$vpnTemp\vpnclientconfiguration\Generic"
$vpnProfile = ([xml](Get-Content -Path "$vpnGeneric\VpnSettings.xml")).VpnProfile

$exportedclientcertpath = $vpnTemp + "P2SClientCert.pfx"
$clientcertname = "CN=" + $vpnProfile.VpnServer

$clientcert = New-SelfSignedCertificate `
    -Type Custom `
    -DnsName $vpnProfile.VpnServer `
    -KeySpec Signature `
    -Subject $clientcertname `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation $certLocation `
    -Signer $rootcert `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")

$mypwd = ConvertTo-SecureString -String $clientcertpassword -Force -AsPlainText

Export-PfxCertificate `
    -FilePath $exportedclientcertpath `
    -Password $mypwd `
    -Cert $clientcert | Out-Null
```

## <a name="configure-the-vpn-client"></a>Configuración del cliente VPN
La puerta de enlace de red virtual de Azure creará un paquete descargable con los archivos de configuración necesarios para inicializar la conexión VPN en la máquina Windows local. La conexión VPN se configurará con la característica [Always On para VPN](/windows-server/remote/remote-access/vpn/always-on-vpn/) de Windows 10 y versiones posteriores a Windows Server 2016. Este paquete también contiene paquetes ejecutables que configurarán el cliente VPN de Windows heredado, si así lo desea. En esta guía se usa Always On para VPN en lugar del cliente VPN heredado de Windows, ya que el cliente VPN de Always On permite a los usuarios finales conectarse o desconectarse de la VPN de Azure sin tener permisos de administrador en el equipo. 

El siguiente script instalará el certificado de cliente necesario para la autenticación en la puerta de enlace de red virtual; después, descargará e instalará el paquete de VPN. No olvide reemplazar `<computer1>` y `<computer2>` por los equipos deseados. Puede ejecutar este script en tantas máquinas como desee. Para ello, agregue más sesiones de PowerShell a la matriz `$sessions`. La cuenta que use debe ser un administrador en cada una de las máquinas. Si una de las máquinas es el equipo local desde el que se ejecuta el script, debe ejecutar el script desde una sesión de PowerShell con privilegios elevados. 

```PowerShell
$sessions = [System.Management.Automation.Runspaces.PSSession[]]@()
$sessions += New-PSSession -ComputerName "<computer1>"
$sessions += New-PSSession -ComputerName "<computer2>"

foreach ($session in $sessions) {
    Invoke-Command -Session $session -ArgumentList $vpnTemp -ScriptBlock { 
        $vpnTemp = $args[0]
        if (-Not (Test-Path $vpnTemp)) {
            New-Item `
                -ItemType Directory `
                -Force `
                -Path "C:\vpn-temp" | Out-Null
        }
    }

    Copy-Item `
        -Path $exportedclientcertpath, $exportedrootcertpath, "$vpnTemp\vpnclientconfiguration.zip" `
        -Destination $vpnTemp `
        -ToSession $session

    Invoke-Command `
        -Session $session `
        -ArgumentList `
            $mypwd, `
            $vpnTemp, `
            $virtualNetworkName `
        -ScriptBlock { 
            $mypwd = $args[0] 
            $vpnTemp = $args[1]
            $virtualNetworkName = $args[2]

            Import-PfxCertificate `
                -Exportable `
                -Password $mypwd `
                -CertStoreLocation "Cert:\LocalMachine\My" `
                -FilePath "$vpnTemp\P2SClientCert.pfx" | Out-Null

            Import-Certificate `
                -FilePath "$vpnTemp\P2SRootCert.cer" `
                -CertStoreLocation "Cert:\LocalMachine\Root" | Out-Null

            Expand-Archive `
                -Path "$vpnTemp\vpnclientconfiguration.zip" `
                -DestinationPath "$vpnTemp\vpnclientconfiguration"
            $vpnGeneric = "$vpnTemp\vpnclientconfiguration\Generic"

            $vpnProfile = ([xml](Get-Content -Path "$vpnGeneric\VpnSettings.xml")).VpnProfile

            Add-VpnConnection `
                -Name $virtualNetworkName `
                -ServerAddress $vpnProfile.VpnServer `
                -TunnelType Ikev2 `
                -EncryptionLevel Required `
                -AuthenticationMethod MachineCertificate `
                -SplitTunneling `
                -AllUserConnection

            Add-VpnConnectionRoute `
                -Name $virtualNetworkName `
                -DestinationPrefix $vpnProfile.Routes `
                -AllUserConnection

            Add-VpnConnectionRoute `
                -Name $virtualNetworkName `
                -DestinationPrefix $vpnProfile.VpnClientAddressPool `
                -AllUserConnection

            rasdial $virtualNetworkName
        }
}

Remove-Item -Path $vpnTemp -Recurse
```

## <a name="mount-azure-file-share"></a>Montaje de un recurso compartido de archivos de Azure
Ahora que ha configurado la VPN de punto a sitio, puede usarla para montar el recurso compartido de archivos de Azure en los equipos que configure a través de PowerShell. En el ejemplo siguiente se montará el recurso compartido, se mostrará el directorio raíz del recurso compartido para demostrar que el recurso compartido se ha montado realmente y después se desmontará el recurso compartido. Desafortunadamente, no es posible montar el recurso compartido de forma persistente mediante la comunicación remota de PowerShell. Para montar de forma persistente, consulte [Uso de un recurso compartido de archivos de Azure con Windows](storage-how-to-use-files-windows.md). 

```PowerShell
$myShareToMount = "<file-share>"

$storageAccountKeys = Get-AzStorageAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName
$storageAccountKey = ConvertTo-SecureString `
    -String $storageAccountKeys[0].Value `
    -AsPlainText `
    -Force

$nic = Get-AzNetworkInterface -ResourceId $privateEndpoint.NetworkInterfaces[0].Id
$storageAccountPrivateIP = $nic.IpConfigurations[0].PrivateIpAddress

Invoke-Command `
    -Session $sessions `
    -ArgumentList  `
        $storageAccountName, `
        $storageAccountKey, `
        $storageAccountPrivateIP, `
        $myShareToMount `
    -ScriptBlock {
        $storageAccountName = $args[0]
        $storageAccountKey = $args[1]
        $storageAccountPrivateIP = $args[2]
        $myShareToMount = $args[3]

        $credential = [System.Management.Automation.PSCredential]::new(
            "AZURE\$storageAccountName", 
            $storageAccountKey)

        New-PSDrive `
            -Name Z `
            -PSProvider FileSystem `
            -Root "\\$storageAccountPrivateIP\$myShareToMount" `
            -Credential $credential `
            -Persist | Out-Null
        Get-ChildItem -Path Z:\
        Remove-PSDrive -Name Z
    }
```

## <a name="see-also"></a>Consulte también
- [Consideraciones de red para el acceso directo a los recursos compartidos de archivos de Azure](storage-files-networking-overview.md)
- [Configuración de una VPN de punto a sitio (P2S) en Linux para su uso con Azure Files](storage-files-configure-p2s-vpn-linux.md)
- [Configuración de una VPN de sitio a sitio (S2S) para su uso con Azure Files](storage-files-configure-s2s-vpn.md)
