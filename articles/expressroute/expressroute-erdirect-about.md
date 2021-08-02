---
title: Acerca de Azure ExpressRoute Direct
description: Obtenga información sobre las características clave de Azure ExpressRoute Direct y la información necesaria para incorporarse a ExpressRoute Direct, como las SKU disponibles y los requisitos técnicos.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: duau
ms.openlocfilehash: fd890c8e8d72609d109fa466e260dd7315fd553b
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111441229"
---
# <a name="about-expressroute-direct"></a>Acerca de ExpressRoute Direct

ExpressRoute Direct le ofrece la capacidad para conectarse directamente a la red global de Microsoft en ubicaciones de emparejamiento distribuidas estratégicamente por todo el mundo. ExpressRoute Direct proporciona conectividad dual de 100 Gbps o 10 Gbps, que es compatible con la conectividad activa/activa a escala. Puede trabajar con cualquier proveedor de servicios para ER Direct.

Algunas de las características clave que ofrece ExpressRoute Direct incluyen:

* Ingesta de datos masivos en servicios como Storage y Cosmos DB
* Aislamiento físico para sectores regulados y que necesitan una conectividad dedicada y aislada, como: banca, gubernamentales y venta al por menor
* Control granular del circuito de distribución en función de la unidad de negocio

## <a name="onboard-to-expressroute-direct"></a>Incorporación a ExpressRoute Direct

Para poder usar ExpressRoute Direct, primero hay que inscribir la suscripción. Para realizar la inscripción, ejecute los siguientes comandos con Azure PowerShell:

1.  Inicie sesión en Azure y seleccione la suscripción a la que quiera inscribirse.

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

1. Registre la suscripción para la versión preliminar pública con el siguiente comando:
1. 
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowExpressRoutePorts -ProviderNamespace Microsoft.Network
    ```

Una vez inscrito, compruebe que el proveedor de recursos **Microsoft.Network** está registrado en su suscripción. Al registrar un proveedor de recursos se configura la suscripción para que funcione con este.

1. Acceda a la configuración de la suscripción como se describe en [Proveedores y tipos de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).

1. En la suscripción, en **Proveedores de recursos**, compruebe que el proveedor **Microsoft.Network** muestra un estado **Registrado**. Si el proveedor de recursos Microsoft.Network no aparece incluido en la lista de proveedores registrados, agréguelo.

Si empieza a usar ExpressRoute Direct y observa que no hay ningún puerto disponible en la ubicación de emparejamiento seleccionada, envíe un correo electrónico a ExpressRouteDirect@microsoft.com para solicitar más inventario.

## <a name="expressroute-using-a-service-provider-and-expressroute-direct"></a>ExpressRoute con un proveedor de servicios y ExpressRoute Direct

| **ExpressRoute con un proveedor de servicios** | **ExpressRoute Direct** | 
| --- | --- |
| Usa proveedores de servicios para permitir la incorporación y la conectividad de forma rápida en la infraestructura existente | Requiere una infraestructura de 100 Gbps/10 Gbps y administración completa de todas las capas
| Se integra con cientos de proveedores, incluidos Ethernet y MPLS | Capacidad dedicada o de Direct para los sectores regulados y la ingesta de datos masivos |
| SKU de circuitos de 50 Mbps a 10 Gbps | El cliente puede seleccionar una combinación de las SKU de circuito siguientes en instancias de ExpressRoute Direct de 100 Gbps: <ul><li>5 Gbps</li><li>10 Gbps</li><li>40 Gbps</li><li>100 Gbps</li></ul> El cliente puede seleccionar una combinación de las SKU de circuito siguientes en instancias de ExpressRoute Direct de 10 Gbps:<ul><li>1 Gbps</li><li>2 Gbps</li><li>5 Gbps</li><li>10 Gbps</li></ul>
| Optimizado para un solo inquilino | Optimizado para un solo inquilino con varias unidades de negocio y varios entornos de trabajo

## <a name="expressroute-direct-circuits"></a>Circuitos de ExpressRoute Direct

Microsoft Azure ExpressRoute permite ampliar las redes locales en la nube de Microsoft a través de una conexión privada que un proveedor de conectividad ha hecho más sencilla. Con ExpressRoute, puede establecer conexiones con Servicios en la nube de Microsoft, como Microsoft Azure y Microsoft 365.

Cada ubicación de emparejamiento tiene acceso a la red global de Microsoft y puede acceder a cualquier región de una zona geopolítica de manera predeterminada. También puede acceder a todas las regiones globales con un circuito prémium.  

En la mayoría de los escenarios, la funcionalidad es equivalente a circuitos que usan un proveedor de servicios de ExpressRoute para su funcionamiento. Para admitir más granularidad y nuevas capacidades que se ofrecen con ExpressRoute Direct, hay ciertas capacidades claves que existen en los circuitos de ExpressRoute Direct.

## <a name="circuit-skus"></a>SKU de circuitos

ExpressRoute Direct admite escenarios de ingesta de datos masivos en Azure Storage y otros servicios de macrodatos. Ahora, los circuitos ExpressRoute de ExpressRoute Direct 100 Gbps también admiten SKU de circuito de **40 Gbps** y **100 Gbps**. Los pares de puertos físicos solamente son de **100 o 10 Gbps** y pueden tener varios circuitos virtuales. Tamaños del circuito:

| **ExpressRoute Direct de 100 Gbps** | **ExpressRoute Direct de 10 Gbps** | 
| --- | --- |
| **Ancho de banda suscrito**: 200 Gbps | **Ancho de banda suscrito**: 20 Gbps |
| <ul><li>5 Gbps</li><li>10 Gbps</li><li>40 Gbps</li><li>100 Gbps</li></ul> | <ul><li>1 Gbps</li><li>2 Gbps</li><li>5 Gbps</li><li>10 Gbps</li></ul>

## <a name="technical-requirements"></a>Requisitos técnicos

* Interfaces del enrutador de perímetro empresarial de Microsoft (MSEE):
    * Puertos dobles Ethernet de 10 o 100 gigabits solo en pares de enrutadores.
    * Conectividad de fibra LR de modo único.
    * IPv4 e IPv6.
    * MTU de IP de 1500 bytes.

* Conectividad de nivel 2 o nivel 3 del conmutador o enrutador:
    * Debe admitir una etiqueta 1 802.1Q (Dot1Q) o una encapsulación de etiquetas de dos etiquetas 802.1Q (QinQ).
    * Ethertype = 0x8100.
    * Debe agregar la etiqueta de VLAN externa (STAG) en función del identificador de VLAN especificado por Microsoft: *aplicable solo en QinQ*.
    * Debe admitir varias sesiones de BGP (VLAN) por puerto y dispositivo.
    * Conectividad IPv4 e IPv6. *En IPv6 no se creará ninguna subinterfaz adicional. La dirección IPv6 se agregará a la subinterfaz existente.* 
    * Opcional: Compatibilidad con la [detección de reenvío bidireccional (BFD)](./expressroute-bfd.md), que se configura de manera predeterminada en todos los emparejamientos privados en circuitos de ExpressRoute.

## <a name="vlan-tagging"></a>Etiquetado de VLAN

ExpressRoute Direct admite los etiquetados QinQ y Dot1Q de VLAN.

* El **etiquetado QinQ de VLAN** se permite para los dominios de enrutamiento aislados por circuito ExpressRoute. Durante la creación del circuito, Azure asigna de forma dinámica una etiqueta S que no se puede cambiar. Cada emparejamiento del circuito (privado y de Microsoft) usará una etiqueta C única como VLAN. No es necesario que la etiqueta C sea única en los circuitos de los puertos de ExpressRoute Direct.

* El **etiquetado Dot1Q de VLAN** se permite para una única VLAN etiquetada por puerto de ExpressRoute Direct. Una etiqueta C que se use en un emparejamiento debe ser única en todos los circuitos y emparejamientos del par de puertos de ExpressRoute Direct.

## <a name="workflow"></a>Flujo de trabajo

[![workflow](./media/expressroute-erdirect-about/workflow1.png)](./media/expressroute-erdirect-about/workflow1.png#lightbox)

## <a name="sla"></a>Contrato de nivel de servicio

ExpressRoute Direct proporciona el mismo acuerdo de nivel de servicio de nivel empresarial con conexiones redundantes activa-activa a la red global de Microsoft. La infraestructura de ExpressRoute es redundante, así como la conectividad a la red global de Microsoft es redundante y diversa y se escala correctamente con los requisitos del cliente. 

## <a name="next-steps"></a>Pasos siguientes

[Configuración de ExpressRoute Direct](expressroute-howto-erdirect.md)
