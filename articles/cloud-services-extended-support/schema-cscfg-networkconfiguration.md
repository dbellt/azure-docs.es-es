---
title: Esquema de NetworkConfiguration de Azure Cloud Services (soporte extendido) | Microsoft Docs
description: Información relacionada con el esquema de configuración de red de Cloud Services (soporte extendido)
ms.topic: article
ms.service: cloud-services-extended-support
ms.date: 10/14/2020
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: ed2d48288bf97fe3ebaa1e8ffc1336d8a82d940e
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107719032"
---
# <a name="azure-cloud-services-extended-support-config-networkconfiguration-schema"></a>Esquema de NetworkConfiguration de Azure Cloud Services (soporte extendido)

El elemento `NetworkConfiguration` del archivo de configuración de servicio especifica valores de DNS y red virtual. Estas configuraciones son opcionales para Cloud Services.

Puede usar los siguientes recursos para obtener más información acerca de las redes virtuales y los esquemas asociados:

- [Esquema de configuración de Cloud Services (soporte extendido)](schema-cscfg-file.md).
- [Esquema de definición de Cloud Services (soporte extendido)](schema-csdef-file.md)
- [Creación de una red virtual](../virtual-network/manage-virtual-network.md).

## <a name="networkconfiguration-element"></a>Elemento NetworkConfiguration
En el ejemplo siguiente se muestra el elemento `NetworkConfiguration` y sus elementos secundarios.

```xml
<ServiceConfiguration>
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="aclName1">
        <Rule order="<rule-order>" action="<rule-action>" remoteSubnet="<subnet-address>" description="rule-description"/>
      </AccessControl>
    </AccessControls>
    <EndpointAcls>
      <EndpointAcl role="<role-name>" endpoint="<endpoint-name>" accessControl="<acl-name>"/>
    </EndpointAcls>
    <Dns>
      <DnsServers>
        <DnsServer name="<server-name>" IPAddress="<server-address>" />
      </DnsServers>
    </Dns>
    <VirtualNetworkSite name="Group <RG-VNet> <VNet-name>"/>
    <AddressAssignments>
      <InstanceAddress roleName="<role-name>">
        <Subnets>
          <Subnet name="<subnet-name>"/>
        </Subnets>
      </InstanceAddress>
      <ReservedIPs>
        <ReservedIP name="<reserved-ip-name>"/>
      </ReservedIPs>
    </AddressAssignments>
  </NetworkConfiguration>
</ServiceConfiguration>
```

En la siguiente tabla se describen los elementos secundarios del elemento `NetworkConfiguration`.

| Elemento       | Descripción |
| ------------- | ----------- |
| AccessControl | Opcional. Especifica las reglas de acceso a los puntos de conexión de un servicio en la nube. El nombre de control de acceso se define mediante una cadena en el atributo `name`. El elemento `AccessControl` contiene uno o varios elementos `Rule`. Se puede definir más de un elemento `AccessControl`.|
| Regla | Opcional. Indica la acción que debe realizarse para un intervalo de direcciones IP de subred especificado. El orden de la regla se define mediante un valor de cadena en el atributo `order`. Cuanto menor sea el número de regla, mayor será la prioridad. Por ejemplo, se pueden especificar reglas con números de orden de 100, 200 y 300. La regla con el número de orden de 100 tiene prioridad sobre la regla que tiene un orden de 200.<br /><br /> La acción de la regla se define mediante una cadena en el atributo `action`. Los valores posibles son:<br /><br /> -   `permit`: especifica que solo los paquetes del intervalo de subred especificado pueden comunicarse con el punto de conexión.<br />-   `deny`: especifica que se deniega el acceso a los puntos de conexión del intervalo de subred especificado.<br /><br /> El intervalo de subred de direcciones IP que resultan afectados por la regla se definen mediante una cadena en el atributo `remoteSubnet`. La descripción de la regla se define mediante una cadena en el atributo `description`.|
| EndpointAcl | Opcional. Especifica la asignación de reglas de control de acceso a un punto de conexión. El nombre del rol que contiene el punto de conexión se define mediante una cadena en el atributo `role`. El nombre del punto de conexión se define mediante una cadena en el atributo `endpoint`. El nombre del conjunto de reglas de `AccessControl` que se deben aplicar al punto de conexión se definen en una cadena en el atributo `accessControl`. Se puede definir más de un elemento `EndpointAcl`.|
| DnsServer | Opcional. Especifica la configuración de un servidor DNS. Puede especificar la configuración de servidores DNS sin una red virtual. El nombre del servidor DNS se define mediante una cadena en el atributo `name`. La dirección IP del servidor DNS se define mediante una cadena en el atributo `IPAddress`. La dirección IP debe ser una dirección IPv4 válida.|
| VirtualNetworkSite | Opcional. Especifica el nombre del sitio de red virtual en el que quiere implementar el servicio en la nube. Este valor no crea un sitio de red virtual. Hace referencia a un sitio que se ha definido anteriormente en el archivo de red para la red virtual. Un servicio en la nube solo puede ser miembro de una red virtual. Si no especifica este valor, el servicio en la nube no se implementará en una red virtual. El nombre del sitio de red virtual se define mediante una cadena en el atributo `name`.|
| InstanceAddress | Opcional. Especifica la asociación de un rol a una subred o un conjunto de subredes en la red virtual. Al asociar un nombre de rol a una dirección de instancia, puede especificar las subredes a las que quiere asociar este rol. El elemento `InstanceAddress` contiene un elemento Subnets. El nombre del rol que está asociado con la subred o las subredes se define mediante una cadena en el atributo `roleName`.|
| Subnet | Opcional. Especifica la subred que corresponde al nombre de subred en el archivo de configuración de red. El nombre de la subred se define mediante una cadena en el atributo `name`.|
| ReservedIP | Opcional. Especifica la dirección IP reservada que se debe asociar con la implementación. El método de asignación para una dirección IP reservada debe especificarse como `Static` para las implementaciones de plantilla y PowerShell. Cada implementación de un servicio en la nube se puede asociar solo a una dirección IP reservada. El nombre de la dirección IP reservada se define mediante una cadena en el atributo `name`.|

## <a name="see-also"></a>Vea también
[Esquema de configuración de Cloud Services (soporte extendido)](schema-cscfg-file.md).
