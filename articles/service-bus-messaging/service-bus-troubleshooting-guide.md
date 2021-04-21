---
title: Guía para la solución de problemas de Azure Service Bus | Microsoft Docs
description: Conozca las sugerencias y recomendaciones para la solución de algunos problemas que pueden aparecer al usar Azure Service Bus.
ms.topic: article
ms.date: 03/03/2021
ms.openlocfilehash: 27249d7e016ea8aee0552bbbf1687647760d4b6f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786574"
---
# <a name="troubleshooting-guide-for-azure-service-bus"></a>Guía para la solución de problemas de Azure Service Bus
En este artículo se proporcionan sugerencias y recomendaciones para la solución de algunos problemas que pueden aparecer al usar Azure Service Bus. 

## <a name="connectivity-certificate-or-timeout-issues"></a>Problemas de conectividad, certificados o tiempo de espera
Los pasos siguientes pueden ayudarle a solucionar problemas de conectividad, certificados y tiempo de espera para todos los servicios de *.servicebus.windows.net. 

- Desplácese a `https://<yournamespace>.servicebus.windows.net/` o use [wget](https://www.gnu.org/software/wget/) ir allí. Le permite comprobar si tiene problemas con la cadena de certificados, el filtrado IP o la red virtual, lo que suele ser común al usar el SDK de Java.

    Un ejemplo de mensaje correcto:
    
    ```xml
    <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
    ```
    
    Un ejemplo de mensaje de error:

    ```xml
    <Error>
        <Code>400</Code>
        <Detail>
            Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
        </Detail>
    </Error>
    ```
- Ejecute el siguiente comando para comprobar si hay algún puerto bloqueado en el firewall. Los puertos empleados son 443 (HTTPS), 5671 (AMQP) y 9354 (mensajería .NET o SBMP). En función de la biblioteca que use, también se usan otros puertos. Este es el comando de ejemplo que comprueba si el puerto 5671 está bloqueado. 

    ```powershell
    tnc <yournamespacename>.servicebus.windows.net -port 5671
    ```

    En Linux:

    ```shell
    telnet <yournamespacename>.servicebus.windows.net 5671
    ```
- Si hay problemas de conectividad intermitentes, ejecute el siguiente comando para comprobar si hay paquetes descartados. Este comando intentará establecer 25 conexiones TCP diferentes cada segundo con el servicio. A continuación, puede comprobar cuántas de ellas se han realizado correctamente y cuántas han fallado y, además, ver la latencia de conexión TCP. Puede descargar la herramienta `psping` desde [aquí](/sysinternals/downloads/psping).

    ```shell
    .\psping.exe -n 25 -i 1 -q <yournamespace>.servicebus.windows.net:5671 -nobanner     
    ```
    Puede usar comandos equivalentes si utiliza otras herramientas como `tnc`, `ping`, etc. 
- Realice un seguimiento de red si los pasos anteriores no ayudan y analícelo con herramientas como [Wireshark](https://www.wireshark.org/). Si lo necesita, póngase en contacto con el [soporte técnico de Microsoft](https://support.microsoft.com/). 
- Para buscar las direcciones IP correctas que se van a agregar a la lista de conexiones permitidas, consulte [¿Qué direcciones IP debo agregar a la lista de permitidas?](service-bus-faq.yml#what-ip-addresses-do-i-need-to-add-to-allow-list-) 


## <a name="issues-that-may-occur-with-service-upgradesrestarts"></a>Problemas que se pueden producir con las actualizaciones o reinicios de servicios

### <a name="symptoms"></a>Síntomas
- Puede que las solicitudes se limiten momentáneamente.
- Puede haber una caída en la llegada de mensajes o solicitudes entrantes.
- El archivo de registro puede contener mensajes de error.
- Puede que las aplicaciones se desconecten del servicio durante unos segundos.

### <a name="cause"></a>Causa
Las actualizaciones y los reinicios de servicios back-end pueden provocar estos problemas en las aplicaciones:

### <a name="resolution"></a>Resolución
Si el código de aplicación usa el SDK, la directiva de reintentos ya está integrada y activa. La aplicación se volverá a conectar sin un impacto significativo en la aplicación o el flujo de trabajo.

## <a name="unauthorized-access-send-claims-are-required"></a>Acceso no autorizado: las notificaciones de envío son necesarias

### <a name="symptoms"></a>Síntomas 
Es posible que vea este error al intentar obtener acceso a un tema de Service Bus desde Visual Studio en un equipo local mediante una identidad administrada asignada por el usuario con permisos de envío.

```bash
Service Bus Error: Unauthorized access. 'Send' claim\(s\) are required to perform this operation.
```

### <a name="cause"></a>Causa
La identidad no tiene permisos de acceso al tema de Service Bus. 

### <a name="resolution"></a>Resolución
Para resolverlo, instale la biblioteca [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication/).  Par obtener más información, consulte [Autenticación de desarrollo local](/dotnet/api/overview/azure/service-to-service-authentication#local-development-authentication). 

Para obtener información sobre cómo asignar permisos a los roles, consulte [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Azure Service Bus](service-bus-managed-service-identity.md).

## <a name="service-bus-exception-put-token-failed"></a>Excepción de Service Bus: error al colocar el token

### <a name="symptoms"></a>Síntomas
Al intentar enviar más de 1000 mensajes con la misma conexión de Service Bus, recibirá el mensaje de error siguiente: 

`Microsoft.Azure.ServiceBus.ServiceBusException: Put token failed. status-code: 403, status-description: The maximum number of '1000' tokens per connection has been reached.` 

### <a name="cause"></a>Causa
Existe un límite en el número de tokens que se usan para enviar y recibir mensajes mediante una única conexión a un espacio de nombres de Service Bus. Este límite es 1000. 

### <a name="resolution"></a>Resolución
Abra una nueva conexión con el espacio de nombres de Service Bus para enviar más mensajes.

## <a name="adding-virtual-network-rule-using-powershell-fails"></a>No se puede agregar una regla de red virtual mediante PowerShell

### <a name="symptoms"></a>Síntomas
Ha configurado dos subredes a partir de una única red virtual en una regla de red virtual. Al intentar quitar una subred con el cmdlet [Remove-AzServiceBusVirtualNetworkRule](/powershell/module/az.servicebus/remove-azservicebusvirtualnetworkrule), dicha subred no se quita de la regla de red virtual. 

```azurepowershell-interactive
Remove-AzServiceBusVirtualNetworkRule -ResourceGroupName $resourceGroupName -Namespace $serviceBusName -SubnetId $subnetId
```

### <a name="cause"></a>Causa
Es posible que el identificador de Azure Resource Manager que especificó para la subred no sea válido. Esto puede ocurrir cuando la red virtual se encuentra en un grupo de recursos diferente del que tiene el espacio de nombres de Service Bus. Si no especifica explícitamente el grupo de recursos de la red virtual, el comando de la CLI crea el identificador de Azure Resource Manager con el grupo de recursos del espacio de nombres de Service Bus. Por lo tanto, no puede quitar la subred de la regla de red. 

### <a name="resolution"></a>Solución
Especifique el identificador de Azure Resource Manager completo de la subred que incluye el nombre del grupo de recursos que contiene la red virtual. Por ejemplo:

```azurepowershell-interactive
Remove-AzServiceBusVirtualNetworkRule -ResourceGroupName myRG -Namespace myNamespace -SubnetId "/subscriptions/SubscriptionId/resourcegroups/ResourceGroup/myOtherRG/providers/Microsoft.Network/virtualNetworks/myVNet/subnets/mySubnet"
```

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 

- [Excepciones de Azure Resource Manager](service-bus-resource-manager-exceptions.md) Se enumeran las excepciones generadas al interactuar con Azure Service Bus mediante Azure Resource Manager (a través de plantillas o de llamadas directas).
- [Excepciones de mensajería](service-bus-messaging-exceptions.md) Proporciona una lista de excepciones generadas por .NET Framework para Azure Service Bus.