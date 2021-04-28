---
title: Requisitos del sistema de Microsoft Azure Data Box Gateway | Microsoft Docs
description: Obtenga más información sobre los requisitos de software y red de Azure Data Box Gateway
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 04/26/2021
ms.author: alkohli
ms.openlocfilehash: 19b30681632c63657accf130888ee1251366fee6
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108072715"
---
# <a name="azure-data-box-gateway-system-requirements"></a>Requisitos del sistema de Azure Data Box Gateway

En este artículo se describen los requisitos del sistema importantes de la solución Microsoft Azure Data Box Gateway y de los clientes que se conectan a Data Box Gateway. Es aconsejable que revise cuidadosamente la siguiente información antes de implementar Data Box Gateway y que luego la consulte según sea necesario durante la implementación y el funcionamiento posterior. 

Los requisitos del sistema del dispositivo virtual de Data Box Gateway incluyen:

- **Requisitos de software para hosts**: describe las plataformas compatibles, los exploradores de la interfaz de usuario de configuración local, los clientes SMB y los requisitos adicionales de los hosts que se conectan al dispositivo.
- **Requisitos de red para el dispositivo**: proporciona información acerca de los requisitos de red para el funcionamiento del dispositivo virtual.


## <a name="specifications-for-the-virtual-device"></a>Especificaciones para el dispositivo virtual

El sistema host subyacente para Data Box Gateway puede dedicar los recursos siguientes para aprovisionar el dispositivo virtual:

| Especificaciones                                          | Descripción              |
|---------------------------------------------------------|--------------------------|
| Procesadores virtuales (núcleos)   | Cuatro como mínimo |
| Memoria  | 8 GB como mínimo Se recomienda encarecidamente disponer de al menos 16 GB. |
| Disponibilidad|Nodo único|
| Discos| Disco del sistema operativo: 250 GB <br> Disco de datos: 2 TB como mínimo, con aprovisionamiento fino y debe estar basado en SSD|
| Interfaces de red|Una o varias interfaces de red virtual|


## <a name="supported-os-for-clients-connected-to-device"></a>Sistema operativo compatible con los clientes conectados al dispositivo

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>Protocolos compatibles con los clientes que acceden al dispositivo

[!INCLUDE [Supported protocols for clients accessing device](../../includes/data-box-edge-gateway-supported-client-protocols.md)]

## <a name="supported-virtualization-platforms-for-device"></a>Plataformas de virtualización compatibles con el dispositivo

| **Sistema operativo o plataforma**  |**Versiones**   |**Notas**  |
|---------|---------|---------|
|Hyper-V  |  2012 R2 <br> 2016 <br> 2019 |         |
|VMware ESXi     | 6.0 <br> 6.5 <br> 6.7       |No se admiten las herramientas de VMware.         |


## <a name="supported-storage-accounts"></a>Cuentas de almacenamiento admitidas

[!INCLUDE [Supported storage accounts](../../includes/data-box-edge-gateway-supported-storage-accounts.md)]


## <a name="supported-storage-types"></a>Tipos de almacenamiento compatibles

[!INCLUDE [Supported storage types](../../includes/data-box-edge-gateway-supported-storage-types.md)]

## <a name="supported-browsers-for-local-web-ui"></a>Exploradores compatibles con la interfaz de usuario web local

[!INCLUDE [Supported browsers for local web UI](../../includes/data-box-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>Requisitos de los puertos de redes

La siguiente tabla enumera los puertos que deben abrirse en el firewall para permitir el tráfico de administración, de la nube o de SMB. En esta tabla, *dentro* o *entrante* hace referencia a la dirección desde la que el cliente entrante solicita acceso al dispositivo. *Fuera* o *saliente* hace referencia a la dirección en la que el dispositivo de Data Box Gateway envía datos externamente después de la implementación: por ejemplo, saliente a Internet.

[!INCLUDE [Port configuration for device](../../includes/data-box-edge-gateway-port-config.md)]

## <a name="url-patterns-for-firewall-rules"></a>Patrones de URL para reglas de firewall

Con frecuencia, los administradores de red pueden configurar reglas avanzadas de firewall de acuerdo con los patrones de URL para filtrar el tráfico saliente y entrante. El dispositivo de Data Box Gateway y el servicio Data Box Gateway dependen de otras aplicaciones de Microsoft, como Azure Service Bus, Azure Active Directory Access Control, las cuentas de almacenamiento y los servidores de Microsoft Update. Es posible usar los patrones de URL asociados a estas aplicaciones para configurar las reglas de firewall. Es importante entender que los patrones de URL asociados a estas aplicaciones pueden cambiar. Esta realidad, a su vez, requiere que el administrador de red supervise y actualice las reglas de firewall de Data Box Gateway de forma pertinente y oportuna.

Se recomienda que establezca las reglas de firewall para el tráfico saliente, basándose en las direcciones IP fijas de Data Box Gateway, de forma generosa en la mayoría de los casos. Sin embargo, puede utilizar la información siguiente con el objetivo de establecer las reglas avanzadas de firewall que se necesitan para crear entornos seguros.

> [!NOTE]
> - Las direcciones IP del dispositivo (origen) siempre se deben establecer en todas las interfaces de red habilitadas para la nube.
> - Las IP de destino, por su parte, se deben establecer en los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653).

[!INCLUDE [URL patterns for firewall](../../includes/data-box-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-azure-government"></a>Patrones de dirección URL para Azure Government

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/data-box-edge-gateway-gov-url-patterns-firewall.md)]

## <a name="internet-bandwidth"></a>Ancho de banda de Internet

[!INCLUDE [Internet bandwidth](../../includes/data-box-edge-gateway-internet-bandwidth.md)]

## <a name="next-step"></a>Paso siguiente

* [Implementación de Azure Data Box Gateway](data-box-gateway-deploy-prep.md)

