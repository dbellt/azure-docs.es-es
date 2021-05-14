---
title: 'Conexión a una red virtual mediante una VPN P2S y autenticación de certificados: portal'
titleSuffix: Azure VPN Gateway
description: Aprenda cómo conectar los clientes de Windows, macOS y Linux de forma segura a una red virtual mediante conexiones VPN Gateway de punto a sitio y certificados autofirmados o emitidos por una entidad de certificación.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/28/2021
ms.author: cherylmc
ms.openlocfilehash: cb296d12d03bf05ffe628bd435ae15cfbb7b79e5
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204952"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-azure-portal"></a>Configure una conexión VPN de punto a sitio a una red virtual mediante la autenticación nativa de los certificados de Azure: Portal de Azure

Este artículo le ayudará con la conexión segura de clientes que ejecutan Windows, Linux o macOS a una red virtual de Azure. Las conexiones VPN de punto a sitio son útiles cuando desea conectarse a la red virtual desde una ubicación remota, como desde casa o desde una conferencia. También puede usar P2S en lugar de una VPN de sitio a sitio cuando son pocos los clientes que necesitan conectarse a una red virtual. Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público. P2S crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros) o IKEv2. Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/point-to-site-diagram.png" alt-text="Conexión de un equipo a una red virtual de Azure: diagrama de conexión de punto a sitio":::

Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md). Para crear esta configuración mediante Azure PowerShell, consulte [Configuración de una conexión VPN de punto a sitio con Azure PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md).

[!INCLUDE [P2S basic architecture](../../includes/vpn-gateway-p2s-architecture.md)]

## <a name="prerequisites"></a>Requisitos previos

Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

### <a name="example-values"></a><a name="example"></a>Valores del ejemplo

Puede usar los siguientes valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo:

* **Nombre de la red virtual:** VNet1
* **Espacio de direcciones**: 10.1.0.0/16<br>En este ejemplo, se utiliza solo un espacio de direcciones. Puede tener más de un espacio de direcciones para la red virtual.
* **Nombre de subred:** FrontEnd
* **Intervalo de direcciones de subred:** 10.1.0.0/24
* **Subscription** (Suscripción): si tiene más de una suscripción, compruebe que usa la correcta.
* **Grupos de recursos:** TestRG1
* **Ubicación:** Este de EE. UU.
* **GatewaySubnet:** 10.1.255.0/27<br>
* **Nombre de la puerta de enlace de red virtual:** VNet1GW
* **Tipo de puerta de enlace:** VPN
* **Tipo de VPN:** basada en rutas
* **Nombre de dirección IP pública:** VNet1GWpip
* **Tipo de conexión**: De punto a sitio
* **Grupo de direcciones de clientes:** 172.16.201.0/24<br>Los clientes de VPN que se conectan a la red virtual mediante esta conexión de punto a sitio reciben una dirección IP del grupo de clientes.

## <a name="virtual-network"></a><a name="createvnet"></a>Virtual Network

En esta sección, creará una red virtual.

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Basic Point-to-Site VNet](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="virtual-network-gateway"></a><a name="creategw"></a>Puerta de enlace de red virtual

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

>[!NOTE]
>La SKU de puerta de enlace de nivel Básico no admite la autenticación de IKEv2 o RADIUS. Si planea que clientes Mac se conecten a la red virtual, no use la SKU de nivel Básico.
>

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="generate-certificates"></a><a name="generatecert"></a>Generación de certificados

Azure usa certificados para autenticar a los clientes para conectarse a una red virtual a través de una conexión VPN de punto a sitio. Una vez que obtenga el certificado raíz, [cargue](#uploadfile) la información de clave pública en Azure. En ese momento, Azure considera que el certificado raíz es "de confianza" para conectarse de P2S a la red virtual. También genere certificados de cliente desde el certificado raíz de confianza y, a continuación, vuelva a instalarlos en cada equipo cliente. El certificado de cliente se utiliza para autenticar al cliente cuando se inicia una conexión con la red virtual.

### <a name="generate-a-root-certificate"></a><a name="getcer"></a>Generación de un certificado raíz

[!INCLUDE [root-certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="generate-client-certificates"></a><a name="generateclientcert"></a>Generación de certificados de cliente

[!INCLUDE [generate-client-cert](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="client-address-pool"></a><a name="addresspool"></a>Grupo de direcciones de clientes

El grupo de direcciones de cliente es un intervalo de direcciones IP privadas que usted especifica. Los clientes que se conectan de forma dinámica a través de una VPN de punto a sitio reciben una dirección IP de este intervalo. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que desea conectarse. Si configura varios protocolos y SSTP es uno de ellos, el grupo de direcciones configurado se divide equitativamente entre los protocolos configurados.

1. Una vez creada la puerta de enlace de red virtual, navegue hasta la sección **Valores** de la página de la puerta de enlace de red virtual. En **Configuración**, seleccione **Configuración de punto a sitio**. Seleccione **Configure now** (Configurar ahora) para abrir la página de configuración.

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png" alt-text="Configuración de punto a sitio" lightbox="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png":::
1. En la página **Configuración de punto a sitio**, en el cuadro **Grupo de direcciones**, agregue el intervalo de direcciones IP privadas que quiere usar. Los clientes VPN reciben de forma dinámica una dirección IP del intervalo que especifique. La máscara de subred mínima es de 29 bits para la configuración activa/pasiva, y de 28 bits para la configuración activa/activa.
1. Continúe con la sección siguiente para configurar los tipos de autenticación y de túnel.

## <a name="authentication-and-tunnel-types"></a><a name="type"></a>Tipos de autenticación y de túnel

En esta sección, configurará el tipo de autenticación y el tipo de túnel. En la página **Configuración de punto a sitio**, si no ve **Tipo de túnel** o **Tipo de autenticación**, significa que la puerta de enlace usa la SKU básica. La SKU básica no admite la autenticación de IKEv2 o RADIUS. Si quiere usar esta configuración, debe eliminar y volver a crear la puerta de enlace con una SKU de puerta de enlace diferente.

### <a name="tunnel-type"></a><a name="tunneltype"></a>Tipo de túnel

En la página **Configuración de punto a sitio**, seleccione el tipo de túnel. Las opciones de túnel son OpenVPN, SSTP y IKEv2.

* El cliente Strongswan de Linux y Android, y el cliente VPN IKEv2 nativo de iOS y OSX solo utilizarán el túnel IKEv2 para conectarse.
* Los clientes Windows prueban primero el túnel IKEv2 y, si no se conecta, recurren a SSTP.
* Puede usar el cliente OpenVPN para conectarse con el tipo de túnel OpenVPN.

### <a name="authentication-type"></a><a name="authenticationtype"></a>Tipo de autenticación

En **Tipo de autenticación**, seleccione **Certificado de Azure**.


## <a name="root-certificate-data"></a><a name="uploadfile"></a>Datos del certificado raíz

En esta sección, cargará los datos del certificado raíz público en Azure. Una vez que se han cargado los datos de certificado público, Azure puede usarlos para autenticar a los clientes que tienen instalado un certificado de cliente generado a partir del certificado raíz de confianza.

1. Los certificados se agregan en la página **Configuración de punto a sitio** de la sección **Certificado raíz**.
1. Asegúrese de exportar el certificado raíz como archivo X.509 codificado base 64 (.cer). Debe exportar el certificado en este formato para poder abrirlo con un editor de texto.
1. Abra el certificado con un editor de texto como Bloc de notas. Al copiar los datos del certificado, asegúrese de copiar el texto como una línea continua sin retornos de carro ni avances de línea. Es posible que para ver los retornos de carro y los avances de línea deba cambiar la vista del editor de texto de forma que se muestren los símbolos y todos los caracteres. Copie solo la siguiente sección como una línea continua:

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/notepadroot.png" alt-text="Datos del certificado" border="false":::
1. Pegue los datos del certificado en la sección **Public Certificate Data** (Datos de certificado público). En **Nombre** asigne un nombre al certificado y, luego, seleccione **Guardar**. Puede agregar hasta 20 certificados raíz de confianza.

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/uploaded.png" alt-text="Pegado de los datos del certificado" border="false":::
1. Seleccione **Guardar** en la parte superior de la página para guardar todos los valores de configuración.

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/save.png" alt-text="Guardar configuración" border="false":::

## <a name="client-certificate"></a><a name="installclientcert"></a>Certificado de cliente

Si desea crear una conexión P2S desde un equipo cliente distinto del que usó para generar los certificados de cliente, debe instalar un certificado de cliente. Al instalar un certificado de cliente, necesita la contraseña que se creó cuando se exportó el certificado de cliente.

Asegúrese de que se exportó el certificado de cliente como un .pfx junto con la cadena de certificados completa (que es el valor predeterminado). En caso contrario, la información del certificado raíz no está presente en el equipo cliente y el cliente no podrá realizar la autenticación correctamente.

Para obtener los pasos de instalación, consulte [Instalación de un certificado de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

## <a name="vpn-client-configuration-package"></a><a name="clientconfig"></a>Paquete de configuración de cliente VPN

Los clientes VPN deben configurarse con las opciones de configuración de cliente. El paquete de configuración de cliente VPN contiene archivos con los valores para configurar clientes VPN con el fin de conectarse a una red virtual a través de una conexión P2S.

Para conocer los pasos para generar e instalar archivos de configuración de cliente VPN, consulte [Creación e instalación de archivos de configuración de cliente VPN para configuraciones P2S de autenticación nativa con certificados de Azure](point-to-site-vpn-client-configuration-azure-cert.md).

## <a name="connect-to-azure"></a><a name="connect"></a>Conexión con Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>Para conectarse desde un cliente VPN en Windows

[!INCLUDE [Connect from a Windows client](../../includes/vpn-gateway-p2s-connect-windows-client.md)]

[!INCLUDE [verifies client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="to-connect-from-a-mac-vpn-client"></a>Para conectarse desde un cliente VPN en Mac

En el cuadro de diálogo Red, localice el perfil de cliente que quiere usar, especifique la configuración de [VpnSettings.xml](point-to-site-vpn-client-configuration-azure-cert.md#installmac) y, después, haga clic en **Conectar**.

Para obtener instrucciones detalladas al respecto, consulte [Instalación: Mac (OS X)](./point-to-site-vpn-client-configuration-azure-cert.md#installmac). Si tiene problemas para conectarse, compruebe que la puerta de enlace de red virtual no está usando una SKU de nivel Básico. La SKU de nivel Básico no es compatible con los clientes Mac.

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png" alt-text="Conexión de cliente de VPN para Mac" border="false":::

## <a name="to-verify-your-connection"></a><a name="verify"></a>Para comprobar la conexión

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
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

Estas instrucciones se aplican a los clientes Windows.

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Compruebe que el paquete de configuración de cliente de VPN se generó después de que se especificaran las direcciones IP del servidor DNS para la red virtual. Si actualizó las direcciones IP de servidor DNS, genere un nuevo paquete de configuración de cliente de VPN e instálelo.

* Use "ipconfig" para comprobar la dirección IPv4 asignada al adaptador de Ethernet en el equipo desde el que está intentando conectarse. Si la dirección IP está dentro del intervalo de direcciones de la red virtual a la que se va a conectar o dentro del intervalo de direcciones de su VPNClientAddressPool, esto se conoce como un espacio de direcciones superpuesto. Cuando el espacio de direcciones se superpone de esta manera, el tráfico de red no llega a Azure, sino que se mantiene en la red local.

## <a name="to-add-or-remove-trusted-root-certificates"></a><a name="add"></a>Para agregar o quitar certificados raíz de confianza

Puede agregar y quitar certificados raíz de confianza de Azure. Al quitar un certificado raíz, los clientes que tienen un certificado generado a partir de dicha raíz no podrán autenticarse y, por tanto, no podrán conectarse. Si desea que un cliente se autentique y se conecte, es preciso que instale un nuevo certificado de cliente generado a partir de un certificado raíz que sea de confianza (se cargue) en Azure.

### <a name="to-add-a-trusted-root-certificate"></a>Para agregar un certificado raíz de confianza

Puede agregar hasta 20 archivos .cer de certificado raíz de confianza a Azure. Para ver instrucciones, consulte la sección [Carga del archivo .cer del certificado raíz](#uploadfile) de este artículo.

### <a name="to-remove-a-trusted-root-certificate"></a>Eliminación de un certificado raíz de confianza

1. Para quitar un certificado raíz de confianza, vaya a la página **Configuración de punto a sitio** para la puerta de enlace de red virtual.
1. En la sección **Certificado raíz** de la página, busque el certificado que desea quitar.
1. Seleccione los puntos suspensivos junto al certificado y, luego, "Quitar".

## <a name="to-revoke-a-client-certificate"></a><a name="revokeclient"></a>Para revocar un certificado de cliente

Puede revocar certificados de cliente. La lista de revocación de certificados permite denegar de forma selectiva la conectividad de punto a sitio basada en certificados de cliente individuales. Esto difiere de la forma en que se quita un certificado raíz de confianza. Si quita de Azure un archivo .cer de certificado raíz de confianza, se revoca el acceso para todos los certificados de cliente generados y firmados con el certificado raíz revocado. Al revocarse un certificado de cliente, en lugar del certificado raíz, se permite que el resto de los certificados que se generaron a partir del certificado raíz sigan usándose para la autenticación.

Lo más habitual es usar el certificado raíz para administrar el acceso a nivel de equipo u organización, mientras que los certificados de cliente revocados se usan para un control de acceso específico para usuarios individuales.

### <a name="revoke-a-client-certificate"></a>Revocar un certificado de cliente

Puede revocar un certificado de cliente si agrega la huella digital a la lista de revocación.

1. Recupere la huella digital del certificado de cliente. Para más información, consulte [Cómo recuperar la huella digital de un certificado](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).
1. Copie la información en un editor de texto y quite todos los espacios de forma que sea una sola cadena continua.
1. Vaya a la página **Configuración de punto a sitio** de la puerta de enlace de red virtual. Se trata de la misma hoja que utilizó para [cargar un certificado raíz de confianza](#uploadfile).
1. En la sección **Certificados revocados**, especifique un nombre descriptivo para el certificado (no es necesario que sea el CN del certificado).
1. Copie y pegue la cadena de huella digital en el campo **Huella digital**.
1. Se valida la huella digital y se agrega automáticamente a la lista de revocación. Aparece un mensaje en la pantalla que indica que se está actualizando la lista. 
1. Una vez finalizada la actualización, el certificado no se puede usar para conectarse. Los clientes que intenten conectarse con este certificado reciben un mensaje que indica que el certificado ya no es válido.

## <a name="point-to-site-faq"></a><a name="faq"></a>Preguntas más frecuentes sobre la conexión de punto a sitio

Esta sección contiene información sobre las preguntas frecuentes relacionadas con las configuraciones de punto a sitio. También puede ver las [preguntas frecuentes sobre VPN Gateway](vpn-gateway-vpn-faq.md) para información adicional sobre VPN Gateway.

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](../index.yml) para más información. Para más información acerca de las redes y las máquinas virtuales, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/network-overview.md).

Para obtener información sobre solución de problemas de P2S, vea [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
