---
title: Solución de problemas de una conexión VPN de sitio a sitio de Azure que no puede realizarse
titleSuffix: Azure VPN Gateway
description: Obtenga información acerca de cómo solucionar problemas de una conexión VPN de sitio a sitio que deja de funcionar repentinamente y no se puede volver a conectar.
services: vpn-gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/22/2021
ms.author: genli
ms.openlocfilehash: e37e1126f71585467b4fa25877289d2bbcf8113a
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204754"
---
# <a name="troubleshooting-an-azure-site-to-site-vpn-connection-cannot-connect-and-stops-working"></a>Solución de problemas: la conexión VPN de sitio a sitio de Azure no puede conectarse y deja de funcionar

Después de configurar una conexión VPN de sitio a sitio entre una red local y una red virtual de Azure, la conexión VPN de repente deja de funcionar y no se puede volver a conectar. Este artículo proporciona pasos de solución de problemas para ayudarle a resolver este problema. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

Para resolver el problema, primero intente [restablecer la puerta de enlace de la VPN de Azure](./reset-gateway.md) y restablecer el túnel desde el dispositivo VPN local. Si el problema continua, siga estos pasos para identificar la causa del problema.

### <a name="prerequisite-step"></a>Paso de requisito previo

Comprobación del tipo de la puerta de enlace de VPN de Azure.

1. Vaya a [Azure Portal](https://portal.azure.com).

2. Compruebe la página **Información general** de la puerta de enlace de la VPN para obtener la información del tipo.
    
    ![Información general de la puerta de enlace](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>Paso 1. Compruebe si el dispositivo VPN local está validado

1. Compruebe si está usando una [versión del sistema operativo y dispositivo VPN validada](vpn-gateway-about-vpn-devices.md#devicetable). Si el dispositivo no es un dispositivo VPN validado, tendrá que ponerse en contacto con el fabricante del dispositivo para ver si hay algún problema de compatibilidad.

2. Asegúrese de que el dispositivo VPN esté configurado correctamente. Para más información, consulte [Ejemplos de edición de configuración de dispositivos](vpn-gateway-about-vpn-devices.md#editing).

### <a name="step-2-verify-the-shared-key"></a>Paso 2. Compruebe la clave compartida

Compare la clave compartida del dispositivo VPN local y la de la VPN de la instancia de Azure Virtual Network para asegurarse de que las claves coinciden. 

Para ver la clave compartida de la conexión VPN de Azure, use uno de los siguientes métodos:

**Azure Portal**

1. Vaya a la conexión de sitio a sitio de la puerta de enlace VPN que ha creado.

2. En la sección **Configuración**, haga clic en **Clave compartida**.
    
    ![Clave compartida](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/sharedkey.png)

**Azure PowerShell**

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para el modelo de implementación de Azure Resource Manager:

```azurepowershell
Get-AzVirtualNetworkGatewayConnectionSharedKey -Name <Connection name> -ResourceGroupName <Resource group name>
```

Para el modelo de implementación clásico:

```azurepowershell
Get-AzureVNetGatewayKey -VNetName -LocalNetworkSiteName
```

### <a name="step-3-verify-the-vpn-peer-ips"></a>Paso 3. Compruebe las direcciones IP del par de la VPN

-   La definición de IP en el objeto **Puerta de enlace de red local** en Azure debe coincidir con el IP de dispositivo local.
-   La definición de la dirección IP de la puerta de enlace de Azure establecida en el dispositivo local debe coincidir con la dirección IP de la puerta de enlace de Azure.

### <a name="step-4-check-udr-and-nsgs-on-the-gateway-subnet"></a>Paso 4. Compruebe las UDR y los NSG de la subred de la puerta de enlace

Busque y quite Enrutamiento definido por el usuario (UDR) o Grupos de seguridad de red (NSG) en la subred de puerta de enlace y, a continuación, pruebe el resultado. Si se resuelve el problema, valide la configuración de NSG o UDR aplicada.

### <a name="step-5-check-the-on-premises-vpn-device-external-interface-address"></a>Paso 5. Compruebe la dirección de la interfaz externa del dispositivo VPN local

Si la dirección IP accesible desde Internet del dispositivo VPN está incluida en la definición de **Red local** en Azure, es posible que experimente desconexiones esporádicas.

### <a name="step-6-verify-that-the-subnets-match-exactly-azure-policy-based-gateways"></a>Paso 6. Compruebe que las subredes coinciden exactamente (puertas de enlace basadas en directivas de Azure)

-   Compruebe que los espacios de direcciones de red virtual coinciden exactamente entre la red virtual de Azure y las definiciones locales.
-   Compruebe que las subredes coinciden exactamente entre la **puerta de enlace de la red local** y las definiciones locales para la red local.

### <a name="step-7-verify-the-azure-gateway-health-probe"></a>Paso 7. Compruebe el sondeo de estado de la puerta de enlace de Azure

1. Abra el sondeo de estado en la dirección URL siguiente:

    `https://<YourVirtualNetworkGatewayIP>:8081/healthprobe`

2. Haga clic en la advertencia de certificado.
3. Si recibe una respuesta, se considera que la puerta de enlace de la VPN es correcta. Si no recibe una respuesta, puede que la puerta de enlace no sea correcta o que haya un NSG en la subred de puerta de enlace que provoque el problema. El siguiente texto es una respuesta de ejemplo:

    ```xml
    <?xml version="1.0"?>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">Primary Instance: GatewayTenantWorker_IN_1 GatewayTenantVersion: 14.7.24.6</string>
    ```

### <a name="step-8-check-whether-the-on-premises-vpn-device-has-the-perfect-forward-secrecy-feature-enabled"></a>Paso 8. Compruebe si el dispositivo VPN local cuenta con la característica confidencialidad directa perfecta habilitada

La característica confidencialidad directa perfecta puede provocar problemas de desconexión. Si el dispositivo VPN cuenta con la característica confidencialidad directa perfecta habilitada, deshabilítela. A continuación, actualice la directiva IPsec de puerta de enlace de VPN.

## <a name="next-steps"></a>Pasos siguientes

-   [Configuración de una conexión de sitio a sitio en una red virtual](./tutorial-site-to-site-portal.md)
-   [Configuración de una directiva IPsec/IKE para conexiones VPN de sitio a sitio](vpn-gateway-ipsecikepolicy-rm-powershell.md)