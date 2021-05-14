---
title: Uso de la dirección IP de IoT Hub | Microsoft Docs
description: Aprenda a consultar la dirección IP de IoT Hub y sus propiedades. La dirección IP de IoT Hub puede cambiar durante determinados escenarios, como la recuperación ante desastres o la conmutación por error regional.
author: philmea
ms.author: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/21/2021
ms.openlocfilehash: 7d807a15d358bd621baedbff253f0c731e43ed26
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107874178"
---
# <a name="iot-hub-ip-addresses"></a>Direcciones IP de IoT Hub

Los prefijos de dirección IP de los puntos de conexión de IoT Hub se publican periódicamente con la [etiqueta de servicio](../virtual-network/service-tags-overview.md) _AzureIoTHub_.

> [!NOTE]
> En el caso de los dispositivos que se implementan en redes locales, Azure IoT Hub admite la integración de la conectividad de red virtual con puntos de conexión privados. Para más información, consulte [Compatibilidad de IoT Hub con redes virtuales](./virtual-network-support.md).


Puede usar estos prefijos de dirección IP para controlar la conectividad entre IoT Hub y los dispositivos o recursos de red para implementar diversos objetivos de aislamiento de red:

| Objetivo | Escenarios aplicables | Enfoque |
|------|-----------|----------|
| Comprobación de que los dispositivos y servicios solo se comunican con puntos de conexión de IoT Hub | Mensajería [del dispositivo a la nube](./iot-hub-devguide-messaging.md) y [de la nube al dispositivo](./iot-hub-devguide-messages-c2d.md), [métodos directos](./iot-hub-devguide-direct-methods.md), [dispositivos y módulos gemelos](./iot-hub-devguide-device-twins.md) y [flujos de dispositivo](./iot-hub-device-streams-overview.md) | Use las etiquetas de servicio _AzureIoTHub_ y _EventHub_ para detectar IoT Hub y prefijos de direcciones IP del centro de eventos y configurar reglas de permiso en la configuración del firewall de los dispositivos y los servicios para esos prefijos de direcciones IP en consecuencia; no acepte tráfico a otras direcciones IP de destino con las que no quiere que se comuniquen los dispositivos o servicios. |
| Comprobación de que el punto de conexión de dispositivo IoT Hub solo recibe conexiones de los dispositivos y los recursos de red | Mensajería [del dispositivo a la nube](./iot-hub-devguide-messaging.md) y [de la nube al dispositivo](./iot-hub-devguide-messages-c2d.md), [métodos directos](./iot-hub-devguide-direct-methods.md), [dispositivos y módulos gemelos](./iot-hub-devguide-device-twins.md) y [flujos de dispositivo](./iot-hub-device-streams-overview.md) | Use la [característica de filtro IP](iot-hub-ip-filtering.md) de IoT Hub para permitir conexiones desde sus dispositivos y direcciones IP de recursos de red (consulte la sección de [limitaciones](#limitations-and-workarounds)). | 
| Comprobación de que los recursos de punto de conexión personalizados de las rutas (cuentas de almacenamiento, Service Bus y centros de eventos) solo son accesibles desde los recursos de red | [Enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md) | Siga las instrucciones del recurso para restringir la conectividad (por ejemplo, a través de [reglas de firewall](../storage/common/storage-network-security.md), [vínculos privados](../private-link/private-endpoint-overview.md)o [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md)); use etiquetas de servicio _AzureIoTHub_ para detectar prefijos de dirección IP de IoT Hub y agregar reglas de permiso para esos prefijos IP en la configuración de firewall del recurso (consulte la sección de [limitaciones](#limitations-and-workarounds)). |



## <a name="best-practices"></a>Procedimientos recomendados

* Al agregar reglas de permiso en la configuración del firewall de los dispositivos, es mejor proporcionar [puertos específicos utilizados por los protocolos aplicables](./iot-hub-devguide-protocols.md#port-numbers).

* Los prefijos de dirección IP de IoT Hub están sujetos a cambios. Estos cambios se publican periódicamente mediante etiquetas de servicio antes de entrar en vigor. Por lo tanto, es importante que desarrolle procesos para recuperar y usar con regularidad las etiquetas de servicio más recientes. Este proceso se puede automatizar mediante la [API de detección de etiquetas de servicio](../virtual-network/service-tags-overview.md#service-tags-on-premises). Tenga en cuenta que la API de detección de etiquetas de servicio todavía está en versión preliminar y, en algunos casos, es posible que no genere la lista completa de etiquetas y direcciones IP. Hasta que la API de detección esté disponible con carácter general, considere la posibilidad de usar las [etiquetas de servicio en formato JSON descargable](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). 

* Use la etiqueta *AzureIoTHub.[nombre de la región]* para identificar prefijos de dirección IP que usan los puntos de conexión de IoT Hub en una región determinada. Para tener en cuenta la recuperación ante desastres de un centro de datos o la [conmutación por error regional](iot-hub-ha-dr.md), asegúrese de que también esté habilitada la conectividad a los prefijos de dirección IP de la región del par de replicación geográfica de su instancia de IoT Hub.

* La configuración de reglas de firewall en IoT Hub puede bloquear la conectividad necesaria para ejecutar la CLI de Azure y comandos de PowerShell en IoT Hub. Para evitar esto, puede agregar reglas ALLOW a los prefijos de dirección IP de los clientes para volver a habilitar la CLI o para que los clientes de PowerShell se comuniquen con su instancia de IoT Hub.  


## <a name="limitations-and-workarounds"></a>Limitaciones y soluciones alternativas

* La característica de filtro IP de IoT Hub tiene un límite de 100 reglas. Este límite se puede aumentar mediante solicitudes a través del servicio de soporte al cliente de Azure. 

* Las [reglas de filtrado de IP](iot-hub-ip-filtering.md) configuradas solo se aplican en los puntos de conexión de dirección IP de IoT Hub y no en el punto de conexión del centro de eventos integrado en IoT Hub. Si también necesita que se aplique el filtrado de IP en el centro de eventos en el que se almacenan los mensajes, puede hacerlo con su propio recurso de centro de eventos, donde puede configurar las reglas de filtrado de IP que quiera directamente. Para ello, ha de aprovisionar su propio recurso del centro de eventos y configurar [enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md) para enviar los mensajes a ese recurso en lugar de al centro de eventos integrado en IoT Hub. Por último, como se describe en la tabla anterior, para habilitar la funcionalidad de enrutamiento de mensajes también debe permitir la conectividad desde los prefijos de dirección IP de IoT Hub al recurso del centro de eventos aprovisionado.

* Al enrutar a una cuenta de almacenamiento, solo es posible permitir el tráfico desde los prefijos de dirección IP de IoT Hub cuando la cuenta de almacenamiento se encuentra en una región distinta que la de IoT Hub.

## <a name="support-for-ipv6"></a>Compatibilidad con IPv6 

Actualmente, IPv6 no se admite en IoT Hub.
