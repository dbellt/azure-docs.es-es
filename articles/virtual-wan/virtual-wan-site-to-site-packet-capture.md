---
title: 'Tutorial: Captura de paquetes en conexiones sitio a sitio de Azure Virtual WAN'
description: En este tutorial, aprenderá a capturar paquetes en puertas de enlace de VPN de sitio a sitio de Virtual WAN.
services: virtual-wan
author: wellee
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 04/13/2021
ms.author: wellee
Customer intent: As someone with a networking background using Virtual WAN, I want to perform a packet capture on my Site-to-site VPN Gateway.
ms.openlocfilehash: dbe7e06484797063ed4122ee3fdde625dc66c41f
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107879022"
---
# <a name="perform-packet-capture-on-the-azure-virtual-wan-site-to-site-vpn-gateway"></a>Captura de paquetes en puertas de enlace de VPN de sitio a sitio de Azure Virtual WAN 

Los problemas relacionados con la conectividad y el rendimiento suelen ser complejos. Puede tardar mucho tiempo y esfuerzo en suavizar la causa del problema. La captura de paquetes puede ayudarle a limitar el ámbito de un problema a algunas partes de la red. Puede ayudarle a determinar si el problema está en el lado del cliente de la red, en el lado de Azure de la red o en algún punto entre ambos. Después de limitar el problema, la depuración y las medidas correctivas son más eficaces.

En el siguiente artículo se describe cómo iniciar y detener una captura de paquetes en puertas de enlace de VPN de sitio a sitio de Azure Virtual WAN. Actualmente, esta funcionalidad solo está disponible a través de PowerShell.

## <a name="prerequisites"></a>Requisitos previos

Para seguir los pasos de este artículo debe tener la configuración siguiente ya definida en el entorno:

* Una red de Virtual WAN, un centro de conectividad virtual y una puerta de enlace de VPN de sitio a sitio implementados en el centro de conectividad virtual. También puede tener conexiones que conecten sitios VPN a su puerta de enlace de VPN de sitio a sitio.


### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="set-up-the-environment"></a>Configuración del entorno

Asegúrese de que se encuentra en el contexto de suscripción correcto mediante la ejecución del siguiente script en PowerShell. Esto garantiza que haya iniciado sesión en un usuario que tenga permisos para realizar la captura de paquetes en la puerta de enlace de VPN de sitio a sitio.

   ```azurepowershell-interactive
    $subid = “<insert Virtual WAN subscription ID here>”
    Set-AzContext -SubscriptionId $subid
   ```

## <a name="create-a-storage-account"></a><a name="createstorage"></a>Creación de una cuenta de almacenamiento

Debe crear una cuenta de almacenamiento en su suscripción de Azure para almacenar los resultados de la captura de paquetes. Consulte este [documento](.././storage/common/storage-account-create.md) para obtener instrucciones sobre cómo crear una cuenta de almacenamiento.

Después de crear la cuenta, ejecute los siguientes comandos para generar una dirección URL de firma de acceso compartido (SAS). Los resultados de la captura de paquetes se almacenarán a través de esta dirección URL.
   ```azurepowershell-interactive
  $rgname = “<resource group name containing storage account>” 
$storeName = “<name of storage account> “
$containerName = “<name of container you want to store packet capture in>
$key = Get-AzStorageAccountKey -ResourceGroupName $rgname -Name $storeNAme
$context = New-AzStorageContext -StorageAccountName  $storeName -StorageAccountKey $key[0].value
New-AzStorageContainer -Name $containerName -Context $context
$container = Get-AzStorageContainer -Name $containerName  -Context $context
$now = get-date
$sasurl = New-AzureStorageContainerSASToken -Name $containerName -Context $context -Permission "rwd" -StartTime $now.AddHours(-1) -ExpiryTime $now.AddDays(1) -FullUri
   ```

## <a name="start-the-packet-capture"></a>Inicio de la captura de paquetes

Para iniciar la captura de paquetes, tiene dos opciones: capturar paquetes en una sola conexión VPN o en toda la puerta de enlace de VPN de sitio a sitio. Tenga en cuenta que si decide realizar capturas en conexiones VPN, solo puede realizar capturas en seis conexiones a la vez.

### <a name="packet-capture-on-a-site-to-site-vpn-gateway-all-connections"></a>Captura de paquetes en una puerta de enlace de VPN de sitio a sitio (todas las conexiones)

Ejecute los comandos siguientes. Para encontrar el nombre de la puerta de enlace de VPN de sitio a sitio, vaya al centro de conectividad virtual y haga clic en VPN (de sitio a sitio) en Conectividad.

:::image type="content" source="./media/virtual-wan-pcap-screenshots/vpn-gateway-name.png" alt-text="Imagen del nombre de la puerta de enlace de Virtual WAN." lightbox="./media/virtual-wan-pcap-screenshots/vpn-gateway-name.png":::

   ```azurepowershell-interactive
Start-AzVpnGatewayPacketCapture -ResourceGroupName $rg -Name "<name of the Gateway>" -Sasurl $sasurl
   ```

### <a name="packet-capture-on-specific-site-to-site-vpn-connections"></a>Captura de paquetes en conexiones VPN de sitio a sitio específicas

>[!NOTE]
> Tenga en cuenta que solo puede ejecutar una captura de paquetes en cinco conexiones VPN de manera simultánea.


Ejecute los comandos siguientes. Para encontrar el nombre de las conexiones VPN de sitio a sitio, vaya al centro virtual y haga clic en VPN (de sitio a sitio) en Conectividad. A continuación, vaya al sitio VPN en el que desea realizar la captura de paquetes y haga clic en los tres puntos de la derecha. Haga clic en **Editar conexión VPN** en el menú que aparece.

:::image type="content" source="./media/virtual-wan-pcap-screenshots/sample-connection.png" alt-text="Imagen de cómo buscar nombres de conexión VPN." lightbox="./media/virtual-wan-pcap-screenshots/sample-connection.png":::

Para encontrar el nombre de los vínculos conectados a un sitio VPN específico, haga clic en el sitio VPN de la sección anterior y consulte la tabla **Vínculos**. 

:::image type="content" source="./media/virtual-wan-pcap-screenshots/link-name-sample.png" alt-text="Imagen de cómo buscar el nombre del vínculo VPN." lightbox="./media/virtual-wan-pcap-screenshots/link-name-sample.png":::

   ```azurepowershell-interactive
Start-AzVpnConnectionPacketCapture -ResourceGroupName $rg -Name "<name of the VPN connection>" -ParentResourceName “<name of the Gateway>” -LinkConnection “<comma separated list of links eg. "link1,link2">” -Sasurl $sasurl 
   ```

## <a name="optional-specifying-filters"></a>Opcional: especificación de filtros

Existen algunas herramientas de capturas de paquetes disponibles comúnmente. La obtención de capturas de paquetes apropiadas con estas herramientas puede resultar complicada, especialmente cuando se trabaja en escenarios con un gran volumen de tráfico. Para simplificar las capturas de paquetes, puede especificar filtros para centrarse en comportamientos específicos. A continuación, verá los parámetros disponibles:

>[!NOTE]
> Para TracingFlags y TCPFlags, puede especificar varios protocolos sumando los valores numéricos de los protocolos que quiere capturar (igual que un operador OR lógico). Por ejemplo, si solo quisiera capturar paquetes ESP y OPVN, especificaría un valor TracingFlag de 8+1 = 9.  

| Parámetro | Descripción | Valores predeterminados | Valores disponibles|
|--- |--- | --- | ---|
| TracingFlags | Entero que determina qué tipos de paquetes se capturan | 11 (ESP, IKE, OVPN) | ESP = 1 IKE = 2 OPVN = 8 |
| TCPFlags | Entero que determina qué tipos de paquetes TCP se capturan | 0 (ninguno) | FIN = 1, SYN = 2, RST = 4, PSH = 8, ACK = 16, URG = 32, ECE = 64, CWR = 128| 
| MaxPacketBufferSize|Tamaño máximo de un paquete capturado en bytes. Los paquetes se truncan si son mayores que el valor proporcionado. |120|Any|
| MaxFileSize |Tamaño de archivo de captura máximo en MB. Las capturas se almacenan en un búfer circular, por lo que el desbordamiento se gestiona de manera FIFO (los paquetes más antiguos se quitan primero)|100|Any|
|SourceSubnets|Se capturan los paquetes de los intervalos de CIDR especificados. Se especifica como una matriz.|[] (todas las direcciones IPv4)|Matriz de subredes IPV4 separadas por comas|
| DestinationSubnets |Se capturan los paquetes destinados a los intervalos de CIDR especificados. Se especifica como una matriz. |[] (todas las direcciones IPv4)|Matriz de subredes IPV4 separadas por comas|
|SourcePort |Se capturan los paquetes con origen en los intervalos especificados. Se especifica como una matriz.|[] (todos los puertos)|Matriz de puertos separados por comas|
|DestinationPort |Se capturan los paquetes con destino en los intervalos especificados. Se especifica como una matriz.|[] (todos los puertos)|Matriz de puertos separados por comas|
| CaptureSingleDirectionTrafficOnly |Si el resultado es true, solo aparecerá una dirección de un flujo bidireccional en la captura de paquetes. Esto capturará todas las combinaciones posibles de IP y puertos.|True|True, False|
|Protocolo|Matriz de enteros que corresponden a protocolos IANA. |[] (todos los protocolos)| Todos los protocolos que se encuentran [aquí](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml) |

A continuación, se muestra un ejemplo de captura de paquetes mediante una cadena de filtro. Puede cambiar los parámetros para adaptarlos a sus necesidades en función de la tabla anterior.  

   ```azurepowershell-interactive
$filter="{`"TracingFlags`":11,`"MaxPacketBufferSize`":120,`"MaxFileSize`":500,`"Filters`":[{`"SourceSubnets`":[`"10.19.0.4/32`",`"10.20.0.4/32`"],`"DestinationSubnets`":[`"10.20.0.4/32`",`"10.19.0.4/32`"],`"TcpFlags`":9,`"CaptureSingleDirectionTrafficOnly`":true}]}"
Start-AzVpnConnectionPacketCapture -ResourceGroupName $rg -Name "<name of the VPN connection>" -ParentResourceName “<name of the Gateway>” -LinkConnection “<comma separated list of links>” -Sasurl $sasurl -FilterData $filter

Start-AzVpnGatewayPacketCapture -ResourceGroupName $rg -Name "<name of the Gateway>" -Sasurl $sasurl -FilterData $filter
   ```

## <a name="stopping-the-packet-capture"></a>Detención de la captura de paquetes
Se recomienda dejar que la captura de paquetes se ejecute durante al menos 600 segundos. Debido a los problemas de sincronización entre varios componentes de la ruta de acceso, es posible que las capturas de paquetes más cortas no proporcionen datos completos. Cuando sea el momento de detener la captura de paquetes, ejecute el siguiente comando.

Los parámetros son parecidos a los de la sección "Iniciar una captura de paquetes". La dirección URL de SAS se generó en la sección [Creación de una cuenta de almacenamiento](#createstorage).

### <a name="gateway-level-packet-capture"></a>Captura de paquetes de nivel de puerta de enlace

   ```azurepowershell-interactive
Stop-AzVpnGatewayPacketCapture -ResourceGroupName $rg -Name <GatewayName> -SasUrl $sas
   ```

### <a name="connection-level-packet-captures"></a>Capturas de paquetes de nivel de conexión

   ```azurepowershell-interactive
Stop-AzVpnConnectionPacketCapture -ResourceGroupName $rg -Name <name of the VPN connection> -ParentResourceName "<name of VPN Gateway>" -LinkConnectionName <comma separated list of links e.g. "link1,link2">-SasUrl $sas
   ```

## <a name="viewing-your-packet-capture"></a>Visualización de la captura de paquetes

En Azure Portal, vaya a la cuenta de almacenamiento creada en "Creación de una cuenta de almacenamiento", haga clic en el contenedor y descargue el archivo. Los archivos de datos de captura de paquetes se generan en formato PCAP. Use Wireshark u otra aplicación que haya disponible para abrir archivos PCAP.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Virtual WAN en:

> [!div class="nextstepaction"]
> * [Preguntas más frecuentes sobre Virtual WAN](virtual-wan-faq.md)