---
title: Configuración de una VPN de punto a sitio (P2S) en Linux para su uso con Azure Files | Microsoft Docs
description: Cómo configurar una VPN de punto a sitio (P2S) en Linux para su uso con Azure Files
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 9608e3bdaab033d58796a3841e8cd92d7a8a81ef
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777984"
---
# <a name="configure-a-point-to-site-p2s-vpn-on-linux-for-use-with-azure-files"></a>Configuración de una VPN de punto a sitio (P2S) en Linux para su uso con Azure Files
Puede usar una conexión VPN de punto a sitio (P2S) para montar los recursos compartidos de archivos de Azure a través de SMB desde fuera de Azure, sin necesidad de abrir el puerto 445. Una conexión VPN de punto a sitio es una conexión VPN entre Azure y un cliente individual. Para usar una conexión VPN de punto a sitio con Azure Files, se debe configurar una conexión VPN de punto a sitio para cada cliente que quiera conectarse. Si tiene muchos clientes que necesitan conectarse a sus recursos compartidos de archivos de Azure desde la red local, puede usar una conexión VPN de sitio a sitio (S2S) en lugar de una conexión de punto a sitio para cada cliente. Para obtener más información, consulte [Configuración de una VPN de sitio a sitio para su uso con Azure Files](storage-files-configure-s2s-vpn.md).

Se recomienda que lea [Introducción a las redes de Azure Files](storage-files-networking-overview.md) antes de continuar con este artículo para obtener una descripción completa de las opciones de red disponibles para Azure Files.

En este artículo se detallan los pasos para configurar una VPN de punto a sitio en Linux para montar recursos compartidos de archivos de Azure directamente en el entorno local. Si quiere enrutar el tráfico de Azure File Sync a través de una VPN, consulte [Configuración del proxy y el firewall de Azure File Sync](../file-sync/file-sync-firewall-and-proxy.md).

## <a name="prerequisites"></a>Requisitos previos
- La versión más reciente de la CLI de Azure. Para obtener más información sobre cómo instalar la CLI de Azure, consulte [Instalación de la CLI de Azure PowerShell](/cli/azure/install-azure-cli) y seleccione el sistema operativo. Si prefiere usar el módulo de Azure PowerShell en Linux, puede hacerlo. Sin embargo, las instrucciones siguientes son para la CLI de Azure.

- Un recurso compartido de archivos de Azure que le gustaría montar en el entorno local. Los recursos compartidos de archivos de Azure se implementan en cuentas de almacenamiento, que son construcciones que representan un grupo compartido de almacenamiento en el que puede implementar varios recursos compartidos de archivos u otros recursos de almacenamiento, como contenedores de blobs o colas. Puede encontrar más información sobre cómo implementar cuentas de almacenamiento y recursos compartidos de archivos de Azure en [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).

- Un punto de conexión privado para la cuenta de almacenamiento que contiene el recurso compartido de archivos de Azure que quiere montar localmente. Para más información sobre cómo crear un punto de conexión privado, consulte [Configuración de puntos de conexión de red de Azure Files](storage-files-networking-endpoints.md?tabs=azure-cli). 

## <a name="install-required-software"></a>Instalación del software necesario
La puerta de enlace de red virtual de Azure puede ofrecer conexiones VPN mediante varios protocolos VPN, entre los que se incluyen IPsec y OpenVPN. En esta guía se muestra cómo usar IPsec y se usa el paquete strongSwan para proporcionar compatibilidad con Linux. 

> Comprobada con Ubuntu 18.10.

```bash
sudo apt install strongswan strongswan-pki libstrongswan-extra-plugins curl libxml2-utils cifs-utils

installDir="/etc/"
```

### <a name="deploy-a-virtual-network"></a>Implementar una red virtual 
Para acceder al recurso compartido de archivos de Azure y a otros recursos de Azure desde el entorno local a través de una VPN de punto a sitio, debe crear una red virtual o VNet. La conexión VPN de punto a sitio que creará automáticamente es un puente entre la máquina Linux local y esta red virtual de Azure.

El siguiente script creará una red virtual de Azure con tres subredes: una para el punto de conexión de servicio de la cuenta de almacenamiento, otra para el punto de conexión privado de la cuenta de almacenamiento (que es necesaria para acceder a la cuenta de almacenamiento local sin crear un enrutamiento personalizado para la dirección IP pública de la cuenta de almacenamiento que puede cambiar) y otra para la puerta de enlace de red virtual que proporciona el servicio VPN. 

No olvide reemplazar `<region>`, `<resource-group>` y `<desired-vnet-name>` por los valores correctos para su entorno.

```bash
region="<region>"
resourceGroupName="<resource-group>"
virtualNetworkName="<desired-vnet-name>"

virtualNetwork=$(az network vnet create \
    --resource-group $resourceGroupName \
    --name $virtualNetworkName \
    --location $region \
    --address-prefixes "192.168.0.0/16" \
    --query "newVNet.id" | tr -d '"')

serviceEndpointSubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "ServiceEndpointSubnet" \
    --address-prefixes "192.168.0.0/24" \
    --service-endpoints "Microsoft.Storage" \
    --query "id" | tr -d '"')

privateEndpointSubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "PrivateEndpointSubnet" \
    --address-prefixes "192.168.1.0/24" \
    --query "id" | tr -d '"')

gatewaySubnet=$(az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $virtualNetworkName \
    --name "GatewaySubnet" \
    --address-prefixes "192.168.2.0/24" \
    --query "id" | tr -d '"')
```

## <a name="create-certificates-for-vpn-authentication"></a>Creación de certificados para la autenticación de VPN
Para que las conexiones VPN de las máquinas Linux locales se autentiquen para acceder a la red virtual, debe crear dos certificados: un certificado raíz, que se proporcionará a la puerta de enlace de máquina virtual, y un certificado de cliente, que se firmará con el certificado raíz. El siguiente script crea los certificados necesarios.

```bash
rootCertName="P2SRootCert"
username="client"
password="1234"

mkdir temp
cd temp

sudo ipsec pki --gen --outform pem > rootKey.pem
sudo ipsec pki --self --in rootKey.pem --dn "CN=$rootCertName" --ca --outform pem > rootCert.pem

rootCertificate=$(openssl x509 -in rootCert.pem -outform der | base64 -w0 ; echo)

sudo ipsec pki --gen --size 4096 --outform pem > "clientKey.pem"
sudo ipsec pki --pub --in "clientKey.pem" | \
    sudo ipsec pki \
        --issue \
        --cacert rootCert.pem \
        --cakey rootKey.pem \
        --dn "CN=$username" \
        --san $username \
        --flag clientAuth \
        --outform pem > "clientCert.pem"

openssl pkcs12 -in "clientCert.pem" -inkey "clientKey.pem" -certfile rootCert.pem -export -out "client.p12" -password "pass:$password"
```

## <a name="deploy-virtual-network-gateway"></a>Implementación de puerta de enlace de red virtual
La puerta de enlace de red virtual de Azure es el servicio al que se conectarán las máquinas Linux locales. La implementación de este servicio requiere dos componentes básicos: una dirección IP pública que identificará la puerta de enlace para los clientes dondequiera que se encuentren en el mundo y un certificado raíz creado anteriormente que se usará para autenticar a los clientes.

No olvide reemplazar `<desired-vpn-name-here>` por el nombre que quiera para estos recursos.

> [!Note]  
> La implementación de una puerta de enlace de red virtual de Azure puede tardar hasta 45 minutos. Aunque este recurso va a implementarse, el script de Bash se bloqueará para que se complete la implementación.
>
> Las conexiones P2S IKEv2/OpenVPN no son compatibles con la SKU **básica**. Este script usa la SKU **VpnGw1** para la puerta de enlace de red virtual, según corresponda.

```bash
vpnName="<desired-vpn-name-here>"
publicIpAddressName="$vpnName-PublicIP"

publicIpAddress=$(az network public-ip create \
    --resource-group $resourceGroupName \
    --name $publicIpAddressName \
    --location $region \
    --sku "Basic" \
    --allocation-method "Dynamic" \
    --query "publicIp.id" | tr -d '"')

az network vnet-gateway create \
    --resource-group $resourceGroupName \
    --name $vpnName \
    --vnet $virtualNetworkName \
    --public-ip-addresses $publicIpAddress \
    --location $region \
    --sku "VpnGw1" \
    --gateway-typ "Vpn" \
    --vpn-type "RouteBased" \
    --address-prefixes "172.16.201.0/24" \
    --client-protocol "IkeV2" > /dev/null

az network vnet-gateway root-cert create \
    --resource-group $resourceGroupName \
    --gateway-name $vpnName \
    --name $rootCertName \
    --public-cert-data $rootCertificate \
    --output none
```

## <a name="configure-the-vpn-client"></a>Configuración del cliente VPN
La puerta de enlace de red virtual de Azure creará un paquete descargable con los archivos de configuración necesarios para inicializar la conexión VPN en la máquina Linux local. El siguiente script colocará los certificados creados en el lugar correcto y configurará el archivo `ipsec.conf` con los valores correctos del archivo de configuración en el paquete descargable.

```bash
vpnClient=$(az network vnet-gateway vpn-client generate \
    --resource-group $resourceGroupName \
    --name $vpnName \
    --authentication-method EAPTLS | tr -d '"')

curl $vpnClient --output vpnClient.zip
unzip vpnClient.zip

vpnServer=$(xmllint --xpath "string(/VpnProfile/VpnServer)" Generic/VpnSettings.xml)
vpnType=$(xmllint --xpath "string(/VpnProfile/VpnType)" Generic/VpnSettings.xml | tr '[:upper:]' '[:lower:]')
routes=$(xmllint --xpath "string(/VpnProfile/Routes)" Generic/VpnSettings.xml)

sudo cp "${installDir}ipsec.conf" "${installDir}ipsec.conf.backup"
sudo cp "Generic/VpnServerRoot.cer" "${installDir}ipsec.d/cacerts"
sudo cp "${username}.p12" "${installDir}ipsec.d/private" 

echo -e "\nconn $virtualNetworkName" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tkeyexchange=$vpnType" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\ttype=tunnel" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftfirewall=yes" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleft=%any" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftauth=eap-tls" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftid=%client" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tright=$vpnServer" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\trightid=%$vpnServer" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\trightsubnet=$routes" | sudo tee -a "${installDir}ipsec.conf" > /dev/null
echo -e "\tleftsourceip=%config" | sudo tee -a "${installDir}ipsec.conf" > /dev/null 
echo -e "\tauto=add" | sudo tee -a "${installDir}ipsec.conf" > /dev/null

echo ": P12 client.p12 '$password'" | sudo tee -a "${installDir}ipsec.secrets" > /dev/null

sudo ipsec restart
sudo ipsec up $virtualNetworkName 
```

## <a name="mount-azure-file-share"></a>Montaje de un recurso compartido de archivos de Azure
Ahora que ha configurado la VPN de punto a sitio, puede montar el recurso compartido de archivos de Azure. En el ejemplo siguiente se montará el recurso compartido de forma no persistente. Para montar de forma persistente, consulte [Uso de un recurso compartido de archivos de Azure con Linux](storage-how-to-use-files-linux.md). 

```bash
fileShareName="myshare"

mntPath="/mnt/$storageAccountName/$fileShareName"
sudo mkdir -p $mntPath

storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')

smbPath="//$storageAccountPrivateIP/$fileShareName"
sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
```

## <a name="see-also"></a>Consulte también
- [Introducción a las redes de Azure Files](storage-files-networking-overview.md)
- [Configuración de una VPN de punto a sitio (P2S) en Windows para su uso con Azure Files](storage-files-configure-p2s-vpn-windows.md)
- [Configuración de una VPN de sitio a sitio (S2S) para su uso con Azure Files](storage-files-configure-s2s-vpn.md)