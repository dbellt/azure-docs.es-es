---
title: Compatibilidad de Azure IoT Hub con redes virtuales
description: Cómo usar el patrón de conectividad de redes virtuales con IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 12/18/2020
ms.author: jlian
ms.openlocfilehash: 726e482f64f7d9c1513f5ce362c232e225b9ee27
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109712863"
---
# <a name="iot-hub-support-for-virtual-networks-with-private-link-and-managed-identity"></a>Compatibilidad de IoT Hub con redes virtuales mediante Private Link e identidad administrada

De forma predeterminada, los nombres de host de IoT Hub se asignan a un punto de conexión público con una dirección IP enrutable públicamente a través de Internet. Diferentes clientes comparten este punto de conexión público de IoT Hub, que es accesible para todos los dispositivos de IoT a través de redes de área extensa y de redes locales.

![Punto de conexión público de IoT Hub](./media/virtual-network-support/public-endpoint.png)

De forma similar, las características de IoT Hub, entre las que se incluyen el [enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md), [carga de archivos](./iot-hub-devguide-file-upload.md) e [importación/exportación en bloque de dispositivos](./iot-hub-bulk-identity-mgmt.md), requieren conectividad de IoT Hub a un recurso de Azure propiedad del cliente a través de su punto de conexión público. Estas rutas de conectividad conforman colectivamente el tráfico de salida de IoT Hub a los recursos del cliente.

Es posible que desee restringir la conectividad a los recursos de Azure (incluido IoT Hub) para que solo se realice a través de una red virtual de su propiedad y bajo su control. Estas razones incluyen las siguientes:

* Introducción del aislamiento de red para IoT Hub evitando la exposición de la conectividad a la red pública de Internet.

* Habilitación de una experiencia de conectividad privada desde los recursos de la red local, lo que garantiza que sus datos y tráfico se transmiten directamente a la red troncal de Azure.

* Prevención de ataques de exfiltración de redes locales confidenciales. 

* Seguimiento de los patrones de conectividad generales de Azure establecidos mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md).

En este artículo se describe cómo lograr estos objetivos mediante el uso de [Azure Private Link](../private-link/private-link-overview.md) para la conectividad de entrada a IoT Hub y el uso de la excepción de servicios de Microsoft de confianza para la conectividad de salida de IoT Hub a otros recursos de Azure.

## <a name="ingress-connectivity-to-iot-hub-using-azure-private-link"></a>Conectividad de entrada a IoT Hub mediante Azure Private Link

Un punto de conexión privado es una dirección IP privada asignada en una red virtual propiedad del cliente a través de la cual se puede acceder a un recurso de Azure. Con Azure Private Link, puede configurar un punto de conexión privado para su instancia de IoT Hub, para permitir así que los servicios que operan en la red virtual accedan a IoT Hub sin necesidad de enviar el tráfico al punto de conexión público de IoT Hub. Del mismo modo, los dispositivos locales pueden usar la [red privada virtual (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) o el emparejamiento de [ExpressRoute](https://azure.microsoft.com/services/expressroute/) para obtener conectividad con la red virtual y la instancia de IoT Hub (a través de su punto de conexión privado). Por tanto, puede restringir o bloquear por completo la conectividad en los puntos de conexión públicos de la instancia de IoT Hub utilizando un [filtro de IP de IoT Hub](./iot-hub-ip-filtering.md) o [activando y desactivando el acceso a la red pública](iot-hub-public-network-access.md). Este enfoque mantiene la conectividad con la instancia de IoT Hub usando el punto de conexión privado para los dispositivos. El objetivo principal de esta configuración es para los dispositivos de una red local. No se recomienda esta configuración para los dispositivos implementados en una red de área extensa.

![Salida de red virtual de IoT Hub](./media/virtual-network-support/virtual-network-ingress.png)

Antes de continuar, asegúrese de que se cumplen los requisitos previos siguientes:

* Ha [creado una red virtual de Azure](../virtual-network/quick-create-portal.md) con una subred en la que se creará el punto de conexión privado.

* En el caso de los dispositivos que operan en redes locales, configure la [red privada virtual (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) o el emparejamiento privado de [ExpressRoute](https://azure.microsoft.com/services/expressroute/) en su red virtual de Azure.

### <a name="set-up-a-private-endpoint-for-iot-hub-ingress"></a>Configuración de un punto de conexión privado para la entrada de IoT Hub

El punto de conexión privado funciona para API de dispositivo de IoT Hub (como los mensajes de dispositivo a nube), así como para API de servicio (como la creación y la actualización de dispositivos).

1. En Azure Portal, seleccione **Redes**, **Conexiones de punto de conexión privado** y haga clic en **+ Punto de conexión privado**.

    :::image type="content" source="media/virtual-network-support/private-link.png" alt-text="Captura de pantalla que muestra dónde agregar un punto de conexión privado para IoT Hub":::

1. Indique la suscripción, el grupo de recursos, el nombre y la región donde desea crear el nuevo punto de conexión privado. Lo más adecuado es que el punto de conexión privado se cree en la misma región que el centro de conectividad.

1. Haga clic en **Siguiente: Recurso** y proporcione la suscripción para el recurso de IoT Hub; y seleccione **"Microsoft.Devices/IotHubs"** como tipo de recurso, su nombre de IoT Hub como **recurso** e **iotHub** como subrecurso de destino.

1. Haga clic en **Siguiente: Configuración** y proporcione la red virtual y la subred donde se creará el punto de conexión privado. Si lo desea, seleccione la opción para la integración con la zona DNS privada de Azure.

1. Haga clic en **Siguiente: Etiquetas** y, de manera opcional, proporcione las etiquetas del recurso.

1. Haga clic en **Revisar y crear** para crear el recurso de Private Link.

### <a name="built-in-event-hub-compatible-endpoint"></a>Punto de conexión integrado compatible con el centro de eventos 

El [punto de conexión integrado compatible con el centro de eventos](iot-hub-devguide-messages-read-builtin.md) también permite acceder con un punto de conexión privado. Cuando configure el vínculo privado, debería ver otra conexión de punto de conexión privado para el punto de conexión integrado. Es el que tiene `servicebus.windows.net` en FQDN.

:::image type="content" source="media/virtual-network-support/private-built-in-endpoint.png" alt-text="Imagen de dos puntos de conexión privados, cada uno con un vínculo privado a IoT Hub":::

De manera opcional, el [filtro IP](iot-hub-ip-filtering.md) de IoT Hub puede controlar el acceso público al punto de conexión integrado. 

Para bloquear completamente el acceso de la red pública a su centro de IoT, [desactive el acceso a la red pública](iot-hub-public-network-access.md) o use el filtro de IP para bloquear todas las direcciones IP y seleccione la opción para aplicar reglas al punto de conexión integrado.

### <a name="pricing-for-private-link"></a>Precios de Private Link

Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="egress-connectivity-from-iot-hub-to-other-azure-resources"></a>Conectividad de salida de IoT Hub a otros recursos de Azure

IoT Hub puede conectar con los recursos de Azure Blob Storage, Event Hubs, Service Bus para el [enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md), la [carga de archivos](./iot-hub-devguide-file-upload.md) y la [importación/exportación en bloque de dispositivos](./iot-hub-bulk-identity-mgmt.md) a través del punto de conexión público de los recursos. Al enlazar el recurso a una red virtual, se bloquea la conectividad al recurso de forma predeterminada. Como resultado, esta configuración impide que las instancias de IoT Hub trabajen en el envío de datos a los recursos. Para corregir este problema, habilite la conectividad desde el recurso de IoT Hub a su recurso de cuenta de almacenamiento, Event Hubs o Service Bus a través de la opción de **servicio de confianza de Microsoft**. 

Para que otros servicios puedan encontrar la instancia de IoT Hub como un servicio de confianza de Microsoft, la central debe usar la identidad administrada por el sistema. Una vez aprovisionada una identidad administrada, debe conceder el permiso RBAC de Azure a la identidad administrada de la central para acceder al punto de conexión personalizado. Siga las instrucciones del artículo [Compatibilidad con identidades administradas en IoT Hub](./iot-hub-managed-identity.md) para aprovisionar una identidad administrada con el permiso RBAC de Azure y agregue el punto de conexión personalizado a IoT Hub. Asegúrese de activar la excepción principal de Microsoft de confianza para permitir el acceso de IoT Hub al punto de conexión personalizado si tiene activadas las configuraciones del firewall.

### <a name="pricing-for-trusted-microsoft-service-option"></a>Precios de la opción de servicio de Microsoft de confianza
La característica de excepción de servicios de confianza de Microsoft es gratuita. Los cargos por las cuentas de almacenamiento aprovisionadas, las instancias de Event Hubs o los recursos de Service Bus se aplican por separado.
## <a name="next-steps"></a>Pasos siguientes

Use los vínculos siguientes para obtener más información sobre las características de IoT Hub:

* [Enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md)
* [Carga de archivos](./iot-hub-devguide-file-upload.md)
* [Importación/exportación masiva de dispositivos](./iot-hub-bulk-identity-mgmt.md)
