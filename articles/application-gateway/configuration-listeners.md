---
title: Configuración del cliente de escucha de Azure Application Gateway
description: En este artículo se describe cómo configurar los clientes de escucha de Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0a88acaf29187c648b24f74877a260a8772f6008
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319690"
---
# <a name="application-gateway-listener-configuration"></a>Configuración del cliente de escucha de Application Gateway

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Un cliente de escucha es una entidad lógica que comprueba las solicitudes de conexión entrantes mediante el puerto, protocolo, host y dirección IP. Al configurar el cliente de escucha, debe especificar valores que coincidan con los valores correspondientes de la solicitud entrante de la puerta de enlace.

Cuando crea una puerta de enlace de aplicaciones mediante Azure Portal, también puede crear un cliente de escucha predeterminado eligiendo el protocolo y el puerto de este cliente. Puede elegir si habilita compatibilidad con HTTP2 en el cliente de escucha o no. Después de crear la puerta de enlace de aplicaciones, puede editar la configuración de ese cliente de escucha predeterminado (*appGatewayHttpListener*) o crear otros clientes nuevos.

## <a name="listener-type"></a>Tipo de cliente de escucha

Cuando crea un nuevo cliente de escucha, puede elegir entre [*básico* y *multisitio*](./application-gateway-components.md#types-of-listeners).

- Si desea que se acepten todas las solicitudes (para cualquier dominio) y se reenvíen a los grupos de servidores back-end, elija básico. Aprenda a [crear una puerta de enlace de aplicaciones con un cliente de escucha básico](./quick-create-portal.md).

- Si quiere reenviar las solicitudes a diferentes grupos de servidores back-end en función del encabezado *host* o del nombre de host, elija un cliente de escucha multisitio; en este caso, también debe especificar un nombre de host que coincida con la solicitud entrante. Esto se debe a que Application Gateway se basa en los encabezados de host HTTP 1.1 para hospedar más de un sitio web en la misma dirección IP pública y en el mismo puerto. Para obtener más información, consulte [Hospedaje de varios sitios mediante Application Gateway](multiple-site-overview.md).

### <a name="order-of-processing-listeners"></a>Orden de procesamiento de los clientes de escucha

En el caso de la SKU v1, la correspondencia de las solicitudes se establece según el orden de las reglas y el tipo de cliente de escucha. Si una regla con un cliente de escucha básico está en primera posición según el orden, se procesa en primer lugar y acepta cualquier solicitud para esa combinación de puerto y dirección IP. Para evitar este comportamiento, configure las reglas con clientes de escucha multisitio en primer lugar e inserte la regla con el cliente de escucha básico en el último lugar de la lista.

Para la SKU v2, los clientes de escucha multisitio se procesan antes que los básicos.

## <a name="front-end-ip-address"></a>Dirección IP de front-end

Elija la dirección IP de front-end que va a asociar a este cliente de escucha. El cliente de escucha escuchará las solicitudes entrantes en esta dirección IP.

## <a name="front-end-port"></a>Puerto de front-end

Elija el puerto de front-end. Seleccione un puerto existente o cree uno nuevo. Elija cualquier valor del [intervalo de puertos permitidos](./application-gateway-components.md#ports). Puede usar no solo los puertos conocidos, como el 80 y el 443, sino cualquier puerto permitido personalizado que sea adecuado. Se puede usar un puerto para los clientes de escucha de acceso público y los de acceso privado.

## <a name="protocol"></a>Protocolo

Elija HTTP o HTTPS:

- Si elige HTTP, se descifrará el tráfico entre el cliente y la puerta de enlace de aplicaciones.

- Seleccione HTTPS si desea [terminación TLS](features.md#secure-sockets-layer-ssltls-termination) o [cifrado TLS de un extremo a otro](./ssl-overview.md). Si lo hace, se cifrará el tráfico entre el cliente y la puerta de enlace de aplicaciones. Y la conexión TLS termina en la puerta de enlace de aplicaciones. Si quiere cifrado TLS de un extremo a otro, debe elegir HTTPS y configurar el valor de **HTTP de back-end**. Esto garantiza que el tráfico se volverá a cifrar cuando vaya desde la puerta de enlace de aplicaciones al back-end.


Para configurar la terminación TLS y el cifrado TLS de un extremo a otro, debe agregar un certificado al cliente de escucha para permitir a la puerta de enlace de aplicaciones derivar una clave simétrica. Esto viene determinado por la especificación del protocolo TLS. La clave simétrica se usa para cifrar y descifrar el tráfico que se envía a la puerta de enlace. El certificado de la puerta de enlace debe estar en formato de Intercambio de información personal (PFX). Este formato le permite exportar la clave privada que usa la puerta de enlace para cifrar y descifrar el tráfico.

## <a name="supported-certificates"></a>Certificados admitidos

Consulte [Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway](ssl-overview.md#certificates-supported-for-tls-termination).

## <a name="additional-protocol-support"></a>Compatibilidad con protocolos adicionales

### <a name="http2-support"></a>Compatibilidad con HTTP2

La compatibilidad con el protocolo HTTP/2 está disponible únicamente para los clientes que se conectan a los clientes de escucha de la puerta de aplicaciones. La comunicación con granjas de servidores back-end es a través de HTTP/1.1. De forma predeterminada, HTTP/2 está deshabilitado. El siguiente fragmento de código de Azure PowerShell muestra cómo habilitarlo:

```azurepowershell
$gw = Get-AzApplicationGateway -Name test -ResourceGroupName hm

$gw.EnableHttp2 = $true

Set-AzApplicationGateway -ApplicationGateway $gw
```

### <a name="websocket-support"></a>Compatibilidad con WebSocket

La compatibilidad con WebSocket está habilitada de forma predeterminada. No hay ninguna opción de configuración que permita al usuario habilitarla o deshabilitarla. Puede usar WebSockets con clientes de escucha HTTP y HTTPS.

## <a name="custom-error-pages"></a>Páginas de error personalizadas

Puede definir el error personalizado en el nivel global o en el nivel del cliente de escucha. Actualmente no se admite la creación de páginas de error personalizadas en el nivel global desde Azure Portal. Puede configurar una página de error personalizada para un error 403 de firewall de aplicaciones web o una página de mantenimiento 502 en el nivel del cliente de escucha. También debe especificar una dirección URL para el blob accesible públicamente para el código de estado de error especificado. Para más información, consulte [Create Application Gateway custom error pages](./custom-error.md) (Creación de páginas de error personalizadas de Application Gateway).

![Códigos de error de Application Gateway](/azure/application-gateway/media/custom-error/ag-error-codes.png)

Para configurar una página de error personalizada global, consulte [Configuración de Azure PowerShell](./custom-error.md#azure-powershell-configuration).

## <a name="tls-policy"></a>Directiva TLS

Puede centralizar la administración de certificados TLS/SSL y reducir la sobrecarga de cifrado y descifrado de una granja de servidores de back-end. El control centralizado de TLS también le permite especificar una directiva TLS central adaptada a sus requisitos de seguridad. Puede elegir la directiva TLS *predeterminada*, *predefinida* o *personalizada*.

Puede configurar la directiva TLS para controlar las versiones del protocolo TLS. Puede configurar una puerta de enlace de aplicaciones para que use una versión de protocolo mínima para los protocolos de enlace TLS desde TLS 1.0, TLS 1.1 y TLS 1.2. De forma predeterminada, SSL 2.0 y 3.0 están deshabilitadas y no se pueden configurar. Para más información, consulte [Introducción a la directiva TLS de Application Gateway](./application-gateway-ssl-policy-overview.md).

Después de crear un cliente de escucha, puede asociarlo a una regla de enrutamiento de solicitud. Esta regla determina cómo se enrutan las solicitudes que se reciben en el cliente de escucha hacia el back-end.

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información sobre las reglas de enrutamiento de solicitudes](configuration-request-routing-rules.md).
