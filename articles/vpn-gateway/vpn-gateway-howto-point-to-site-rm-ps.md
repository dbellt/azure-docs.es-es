---
title: 'Conexión a una red virtual desde un equipo: autenticación de certificados de Azure nativo y VPN de punto a sitio: PowerShell'
description: Obtenga información sobre cómo conectar de manera segura los clientes de Windows y macOS a la red virtual de Azure mediante P2S y certificados autofirmados o emitidos por una entidad de certificación.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/29/2020
ms.author: cherylmc
ms.openlocfilehash: cf1cd8eb2d9723d435f277b9c029b15c843b9afd
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108226397"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-powershell"></a>Configure una conexión VPN de punto a sitio a una red virtual mediante la autenticación nativa de los certificados de Azure: PowerShell

Este artículo le ayudará con la conexión segura de clientes que ejecutan Windows, Linux o macOS a una red virtual de Azure. Las conexiones VPN de punto a sitio son útiles cuando desea conectarse a la red virtual desde una ubicación remota, como desde casa o desde una conferencia. También puede usar P2S en lugar de una VPN de sitio a sitio cuando son pocos los clientes que necesitan conectarse a una red virtual. Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público. P2S crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros) o IKEv2.

:::image type="content" source="./media/vpn-gateway-how-to-point-to-site-rm-ps/point-to-site-diagram.png" alt-text="Conexión de un equipo a una red virtual de Azure: diagrama de conexión de punto a sitio":::

Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md). Para crear esta configuración mediante Azure Portal, consulte [Configuración de una conexión VPN de punto a sitio con Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

[!INCLUDE [P2S basic architecture](../../includes/vpn-gateway-p2s-architecture.md)]

## <a name="prerequisites"></a>Requisitos previos

Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

### <a name="azure-powershell"></a>Azure PowerShell

>[!IMPORTANT]
> Para muchos de los pasos descritos en este artículo puede usar Azure Cloud Shell. Sin embargo, no puede usar Cloud Shell para generar certificados. Además, para cargar la clave pública del certificado raíz, debe usar Azure PowerShell localmente o Azure Portal.
>

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="1-sign-in"></a><a name="signin"></a>1. Iniciar sesión

[!INCLUDE [sign in](../../includes/vpn-gateway-cloud-shell-ps-login.md)]

## <a name="2-declare-variables"></a><a name="declare"></a>2. Declaración de variables

Usamos variables en este artículo para que pueda cambiar fácilmente los valores que se aplican a su propio entorno sin tener que cambiar los ejemplos. Declare las variables que desea utilizar. Puede usar el ejemplo siguiente sustituyendo los valores por los suyos propios cuando sea necesario. Si cierra la sesión de PowerShell/Cloud Shell en cualquier momento durante el ejercicio, copie y pegue los valores de nuevo para volver a declarar las variables.

```azurepowershell-interactive
$VNetName  = "VNet1"
$FESubName = "FrontEnd"
$GWSubName = "GatewaySubnet"
$VNetPrefix = "10.1.0.0/16"
$FESubPrefix = "10.1.0.0/24"
$GWSubPrefix = "10.1.255.0/27"
$VPNClientAddressPool = "172.16.201.0/24"
$RG = "TestRG1"
$Location = "EastUS"
$GWName = "VNet1GW"
$GWIPName = "VNet1GWpip"
$GWIPconfName = "gwipconf"
$DNS = "10.2.1.4"
```

## <a name="3-configure-a-vnet"></a><a name="ConfigureVNet"></a>3. Configuración de una red virtual

1. Cree un grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG -Location $Location
   ```

1. Cree las configuraciones de subred para la red virtual con los nombres *FrontEnd* y *GatewaySubnet*. Estos prefijos deben formar parte del espacio de direcciones de la red virtual que declaró anteriormente.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
   ```

1. Creación de la red virtual.

   En este ejemplo, el parámetro del servidor -DnsServer es opcional. La especificación de un valor no crea un servidor DNS nuevo. La dirección IP del servidor DNS que especifique debe ser un servidor DNS que pueda resolver los nombres de los recursos a los que se conecta desde la red virtual. En este ejemplo, se usa una dirección IP privada, pero es probable que no sea la dirección IP del servidor DNS. Asegúrese de utilizar sus propios valores. El valor especificado lo utilizan los recursos que se implementan en la red virtual, no la conexión de punto a sitio ni el cliente VPN.

   ```azurepowershell-interactive
       New-AzVirtualNetwork `
      -ResourceGroupName $RG `
      -Location $Location `
      -Name $VNetName `
      -AddressPrefix $VNetPrefix `
      -Subnet $fesub, $gwsub `
      -DnsServer $DNS
   ```

1. Especifique las variables de la red virtual que creó.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   ```

1. Una puerta de enlace VPN debe tener una dirección IP pública. Primero se solicita el recurso de la dirección IP y, después, se hace referencia a él al crear la puerta de enlace de red virtual. La dirección IP se asigna dinámicamente al recurso cuando se crea la puerta de enlace VPN. Actualmente, VPN Gateway solo admite la asignación de direcciones IP públicas *dinámicas*. No se puede solicitar una asignación de direcciones IP públicas estáticas. Sin embargo, esto no significa que la dirección IP cambie después de que se haya asignado a la puerta de enlace VPN. La única vez que la dirección IP pública cambia es cuando la puerta de enlace se elimina y se vuelve a crear. No cambia cuando se cambia el tamaño, se restablece o se realizan actualizaciones u otras operaciones de mantenimiento interno de una puerta de enlace VPN.

   Solicite una dirección IP pública asignada de forma dinámica.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="4-create-the-vpn-gateway"></a><a name="creategateway"></a>4. Creación de la puerta de enlace VPN

En este paso, configurará y creará la puerta de enlace de red virtual para la red virtual.

* El valor de -GatewayType debe ser **Vpn** y el valor de -VpnType debe ser **RouteBased**.
* -VpnClientProtocol se utiliza para especificar los tipos de túneles que desea habilitar. Las opciones de túnel son **OpenVPN, SSTP** y **IKEv2**. Puede habilitar una de ellas o cualquier combinación admitida. Si desea habilitar varios tipos, especifique los nombres separados por una coma. OpenVPN y SSTP no se pueden habilitar juntas. El cliente strongSwan de Linux y Android y el cliente VPN IKEv2 nativo de iOS y OSX solo utilizarán el túnel IKEv2 para conectarse. Los clientes Windows prueban primero el túnel IKEv2 y, si no se conecta, recurren a SSTP. Puede usar el cliente OpenVPN para conectarse al tipo de túnel OpenVPN.
* La SKU "básica" de la puerta de enlace de red virtual no admite la autenticación de IKEv2, OpenVPN o RADIUS. Si planea que clientes Mac se conecten a su red virtual, no use la SKU de nivel Básico.
* Una puerta de enlace de VPN puede tardar hasta 45 minutos en completarse, según la [SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md) que seleccione. Este ejemplo utiliza IKEv2.

1. Configure y cree la puerta de enlace de red virtual para la red virtual. La puerta de enlace tarda aproximadamente 45 minutos en crearse.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
   -Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
   -VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
   ```

1. Una vez creada la puerta de enlace, puede verla mediante el ejemplo siguiente. Si cerró PowerShell o agotó el tiempo de espera mientras se creaba la puerta de enlace, puede [declarar las variables](#declare) de nuevo.

   ```azurepowershell-interactive
   Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroup $RG
   ```

## <a name="5-add-the-vpn-client-address-pool"></a><a name="addresspool"></a>5. Adición del grupo de direcciones de clientes de VPN

Cuando acabe de crear la puerta de enlace VPN, puede agregar el grupo de direcciones de cliente VPN. El grupo de direcciones del cliente de VPN es el intervalo del que los clientes de VPN reciben una dirección IP al conectarse. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que desea conectarse.

En este ejemplo, el grupo de direcciones del cliente de VPN se declara como [variable](#declare) en el paso anterior.

```azurepowershell-interactive
$Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```

## <a name="6-generate-certificates"></a><a name="Certificates"></a>6. Generación de certificados

>[!IMPORTANT]
> No se pueden generar certificados con Azure Cloud Shell. Debe usar uno de los métodos descritos en esta sección. Si desea usar PowerShell, debe instalarlo de forma local.
>

Azure usa los certificados para autenticar a los clientes de VPN para las VPN de punto a sitio. Cargue la información de clave pública del certificado raíz en Azure. La clave pública se considerará "de confianza". Deben generarse certificados de cliente a partir del certificado raíz de confianza e instalarse en cada equipo cliente en el almacén de certificados Certificados - Usuario actual/Personal. El certificado se utiliza para autenticar al cliente cuando inicia una conexión con la red virtual. 

Si usa certificados autofirmados, se deben crear con parámetros específicos. Puede crear un certificado autofirmado con las instrucciones para [PowerShell y Windows 10](vpn-gateway-certificates-point-to-site.md), o bien, si no tiene Windows 10, puede usar [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Es importante que siga los pasos de las instrucciones al generar certificados raíz autofirmados y certificados de cliente. En caso contrario, los certificados que genere no serán compatibles con las conexiones P2S y recibirá un error de conexión.

### <a name="root-certificate"></a><a name="cer"></a>Certificado raíz

1. [!INCLUDE [Root certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

1. Después de crear el certificado raíz, [exporte](vpn-gateway-certificates-point-to-site.md#cer) los datos (no la clave privada) del certificado público como un archivo .cer X.509 con codificación en Base64.

### <a name="client-certificate"></a><a name="generate"></a>Certificado de cliente

1. [!INCLUDE [Generate a client certificate](../../includes/vpn-gateway-p2s-clientcert-include.md)]

1. Después de crear el certificado de cliente, [expórtelo](vpn-gateway-certificates-point-to-site.md#clientexport). El certificado de cliente se distribuirá a los equipos cliente que se conectarán.

## <a name="7-upload-the-root-certificate-public-key-information"></a><a name="upload"></a>7. Cargue la información de clave pública del certificado raíz

Compruebe que ha terminado de crear la puerta de enlace VPN. Una vez terminado, puede cargar el archivo .cer (que contiene la información de clave pública) para un certificado raíz de confianza en Azure. Una vez que se ha cargado un archivo .cer, Azure puede usarlo para autenticar a los clientes que tienen instalado un certificado de cliente generado a partir del certificado raíz de confianza. Más adelante, puede cargar más archivos de certificado raíz de confianza, hasta un total de 20, si es necesario.

>[!NOTE]
> No se puede cargar el archivo .cer mediante Azure Cloud Shell. Puede usar PowerShell localmente en el equipo o seguir los [pasos de Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
>

1. Declare la variable del nombre del certificado, pero reemplace el valor por el suyo propio.

   ```azurepowershell
   $P2SRootCertName = "P2SRootCert.cer"
   ```

1. Reemplace la ruta de acceso del archivo por la del suyo y ejecute los cmdlets.

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Cargue la información de clave pública en Azure. Una vez que se carga la información del certificado, Azure considera que se trata de un certificado raíz de confianza. Al realizar la carga, asegúrese de que está ejecutando PowerShell localmente en el equipo o, en su lugar, puede seguir los [pasos de Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile). No se puede realizar la carga mediante Azure Cloud Shell.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64
   ```

## <a name="8-install-an-exported-client-certificate"></a><a name="clientcertificate"></a>8. Instalación de un certificado de cliente exportado

Los siguientes pasos le ayudarán a completar la instalación en un cliente Windows. Para obtener más información, consulte [Instalación de un certificado de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

Asegúrese de que se exportó el certificado de cliente como un .pfx junto con la cadena de certificados completa (que es el valor predeterminado). En caso contrario, la información del certificado raíz no está presente en el equipo cliente y el cliente no podrá realizar la autenticación correctamente.

## <a name="9-configure-the-vpn-client"></a><a name="clientconfig"></a>9. Configuración del cliente VPN

En esta sección, configurará el cliente nativo para que el equipo se conecte a la puerta de enlace de red virtual. Por ejemplo, al ir a la configuración de VPN en el equipo Windows, puede agregar conexiones VPN. Una conexión de punto a sitio requiere opciones de configuración específicas. Estos pasos le ayudarán a crear un paquete con la configuración específica que el cliente VPN nativo necesita para conectarse a la red virtual a través de una conexión de punto a sitio.

Puede usar los siguientes ejemplos rápidos para generar e instalar el paquete de configuración del cliente. Para obtener más información sobre el contenido del paquete y otras instrucciones sobre cómo generar e instalar archivos de configuración de cliente de VPN, consulte [Creación e instalación de archivos de configuración de cliente VPN](point-to-site-vpn-client-configuration-azure-cert.md).

Si necesita volver a declarar las variables, puede encontrarlas [aquí](#declare).

### <a name="to-generate-configuration-files"></a>Para generar archivos de configuración

```azurepowershell-interactive
$profile=New-AzVpnClientConfiguration -ResourceGroupName $RG -Name $GWName -AuthenticationMethod "EapTls"

$profile.VPNProfileSASUrl
```

### <a name="to-install-the-client-configuration-package"></a>Para instalar el paquete de configuración del cliente

[!INCLUDE [Windows instructions](../../includes/vpn-gateway-p2s-client-configuration-windows.md)]

## <a name="10-connect-to-azure"></a><a name="connect"></a>10. Conexión con Azure

### <a name="windows-vpn-client"></a>Cliente de VPN de Windows

[!INCLUDE [Connect from Windows client](../../includes/vpn-gateway-p2s-connect-windows-client.md)]

[!INCLUDE [Client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="mac-vpn-client"></a>Cliente de VPN de Mac

En el cuadro de diálogo Red, localice el perfil de cliente que desea utilizar y, después, haga clic en **Conectar**.
Para obtener instrucciones detalladas al respecto, consulte [Instalación: Mac (OS X)](./point-to-site-vpn-client-configuration-azure-cert.md#installmac). Si tiene problemas para conectarse, compruebe que la puerta de enlace de red virtual no está usando una SKU de nivel Básico. La SKU de nivel Básico no es compatible con los clientes Mac.

  ![Conexión de Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-a-connection"></a><a name="verify"></a>Para comprobar una conexión

Estas instrucciones se aplican a los clientes Windows.

1. Para comprobar que la conexión VPN está activa, abra un símbolo del sistema con privilegios elevados y ejecute *ipconfig/all*.
2. Vea los resultados. Observe que la dirección IP que recibió es una de las direcciones dentro del grupo de direcciones de cliente de VPN punto a sitio que especificó en la configuración. Los resultados son similares a los del ejemplo siguiente:

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.13(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

Estas instrucciones se aplican a los clientes Windows.

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Compruebe que el paquete de configuración de cliente de VPN se generó después de que se especificaran las direcciones IP del servidor DNS para la red virtual. Si actualizó las direcciones IP de servidor DNS, genere un nuevo paquete de configuración de cliente de VPN e instálelo.

* Use "ipconfig" para comprobar la dirección IPv4 asignada al adaptador de Ethernet en el equipo desde el que está intentando conectarse. Si la dirección IP está dentro del intervalo de direcciones de la red virtual a la que se va a conectar o dentro del intervalo de direcciones de su VPNClientAddressPool, esto se conoce como un espacio de direcciones superpuesto. Cuando el espacio de direcciones se superpone de esta manera, el tráfico de red no llega a Azure, sino que se mantiene en la red local.

## <a name="to-add-or-remove-a-root-certificate"></a><a name="addremovecert"></a>Adición o eliminación de un certificado raíz

Puede agregar y quitar certificados raíz de confianza de Azure. Al quitar un certificado raíz, los clientes que cuentan con un certificado generado a partir del certificado raíz no pueden autenticarse y no podrán conectarse. Si desea que un cliente se autentique y se conecte, es preciso que instale un nuevo certificado de cliente generado a partir de un certificado raíz que sea de confianza (se cargue) en Azure. Estos pasos requieren los cmdlets de Azure PowerShell instalados localmente en el equipo (no Azure Cloud Shell). También puede usar Azure Portal para agregar certificados raíz.

**Para agregar:**

Puede agregar hasta 20 archivos .cer de certificado raíz a Azure. Los pasos siguientes lo ayudan a agregar un certificado raíz. 

1. Preparar el archivo .cer que desea cargar:

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert3.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64_3 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Cargar el archivo. Solo puede cargar un archivo a la vez.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64_3
   ```

1. Para comprobar que se ha cargado el archivo de certificado:

   ```azurepowershell
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

**Para quitar:**

1. Declare las variables. Modifique las variables del ejemplo para que coincidan con el certificado que desea quitar.

   ```azurepowershell-interactive
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   $P2SRootCertName2 = "ARMP2SRootCert2.cer"
   $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
   ```

1. Quite el certificado.

   ```azurepowershell-interactive
   Remove-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
   ```

1. Use el siguiente ejemplo para comprobar que el certificado se quitó correctamente.

   ```azurepowershell-interactive
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

## <a name="to-revoke-or-reinstate-a-client-certificate"></a><a name="revoke"></a>Para revocar o restablecer un certificado de cliente

Puede revocar certificados de cliente. La lista de revocación de certificados permite denegar de forma selectiva la conectividad de punto a sitio basada en certificados de cliente individuales. Esto difiere de la forma en que se quita un certificado raíz de confianza. Si quita de Azure un archivo .cer de certificado raíz de confianza, se revoca el acceso para todos los certificados de cliente generados y firmados con el certificado raíz revocado. Al revocarse un certificado de cliente, en lugar del certificado raíz, se permite que el resto de los certificados que se generaron a partir del certificado raíz sigan usándose para la autenticación.

Lo más habitual es usar el certificado raíz para administrar el acceso a nivel de equipo u organización, mientras que los certificados de cliente revocados se usan para un control de acceso específico para usuarios individuales.

**Para revocar:**

1. Recupere la huella digital del certificado de cliente. Para más información, consulte [Cómo recuperar la huella digital de un certificado](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).

1. Copie la información en un editor de texto y quite todos los espacios de forma que sea una sola cadena continua. Esta cadena se declara como variable en el paso siguiente.

1. Declare las variables. Asegúrese de declarar la huella digital que recuperó en el paso anterior.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Agregue la huella digital a la lista de certificados revocados. Cuando haya agregado la huella digital, aparece un aviso que le indica que la acción se ha completado "correctamente".

   ```azurepowershell-interactive
   Add-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG `
   -Thumbprint $RevokedThumbprint1
   ```

1. Compruebe que la huella digital se agregó a la lista de revocación de certificados.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

1. Una vez agregada la huella digital, el certificado ya no podrá utilizarse para conectarse. Los clientes que intenten conectarse con este certificado reciben un mensaje que indica que el certificado ya no es válido.

**Para restablecer:**

Puede restablecer un certificado de cliente quitando la huella digital de la lista de certificados de cliente revocados.

1. Declare las variables. Asegúrese de que declara la huella digital correcta para el certificado que desea restablecer.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Quite la huella digital del certificado de la lista de revocación de certificados.

   ```azurepowershell-interactive
   Remove-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
   ```

1. Compruebe si la huella digital se quita de la lista revocada.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

## <a name="point-to-site-faq"></a><a name="faq"></a>Preguntas más frecuentes sobre la conexión de punto a sitio

Para obtener información adicional de punto a sitio, consulte las [preguntas frecuentes de punto a sitio de VPN Gateway](vpn-gateway-vpn-faq.md#P2S).

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](../index.yml) para más información. Para más información acerca de las redes y las máquinas virtuales, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/network-overview.md).

Para información de solución de problemas de P2S, consulte el artículo de [solución de problemas de conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).