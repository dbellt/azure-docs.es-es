---
title: Reglas de salida en Azure Load Balancer
description: En este artículo se explica cómo configurar reglas de salida para controlar el tráfico de salida de Internet con Azure Load Balancer.
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 10/13/2020
ms.author: allensu
ms.openlocfilehash: 339bbd7edf48737113de360812165dc8148c5b93
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375872"
---
# <a name="outbound-rules-azure-load-balancer"></a><a name="outboundrules"></a>Reglas de salida en Azure Load Balancer

Las reglas de salida permiten definir de forma explícita SNAT (traducción de direcciones de red de origen) para un equilibrador de carga público estándar. Esta configuración permite usar las direcciones IP públicas del equilibrador de carga para proporcionar conectividad de Internet de salida a las instancias de back-end.

Esta configuración habilita:

* El enmascaramiento de IP
* La simplificación de las listas de permitidos
* La reducción del número de recursos de IP pública para la implementación

Con las reglas de salida, tiene un control declarativo completo sobre la conectividad saliente de Internet. Las reglas de salida le permiten escalar y ajustar esta capacidad a sus necesidades específicas. 

Solo se seguirán las reglas de salida si la VM de back-end no tiene una dirección IP pública de nivel de instancia (ILPIP).

![Reglas de salida de Load Balancer](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

Con las reglas de salida, puede definir explícitamente un comportamiento **SNAT**.

Las reglas de salida permiten controlar:

* **Qué máquinas virtuales se deben traducir a qué direcciones IP públicas.**
     * Dos reglas en las que el grupo de back-end 1 usa la dirección IP azul 1 y 2, el grupo de back-end 2 usa el prefijo de IP amarillo.
* **Cómo se asignan los puertos de SNAT de salida.**
     * Si el grupo de back-end 2 es el único grupo que realiza conexiones salientes, concede todos los puertos SNAT al grupo de back-end 2 y ninguno al grupo de back-end 1.
* **A qué protocolos se proporciona traducción de salida.**
     * Si el grupo de back-end 2 necesita puertos UDP para salida, y el grupo de back-end 1 necesita TCP, asigne puertos TCP a 1 y puertos UDP a 2.
* **Qué duración se debe usar para el tiempo de espera de inactividad de conexión (de 4 a 120 minutos).**
     * Si hay conexiones de larga ejecución con Keepalives, reserve puertos inactivos para las conexiones de larga ejecución durante un máximo de 120 minutos. Suponga que las conexiones obsoletas se han abandonado y libere los puertos en 4 minutos para las conexiones nuevas. 
* **Si quiere enviar un restablecimiento de TCP en caso de tiempo de espera de inactividad.**
     * Al agotarse el tiempo de espera de las conexiones inactivas, ¿se envía un RST de TCP al cliente y al servidor para que sepan que se ha abandonado el flujo?

## <a name="outbound-rule-definition"></a>Definición de la regla de salida

Las reglas de salida siguen la misma sintaxis conocida que las reglas de NAT de entrada y de equilibrio de carga: **front-end** + **parámetros** + **grupo de back-end**. 

Una regla de salida configura una NAT de salida para _todas las máquinas virtuales identificadas por el grupo de back-end_ que se deben traducir para el _front-end_.  

Los _parámetros_ proporcionan un control más preciso sobre el algoritmo de NAT de salida.

## <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a> NAT de salida de escala con varias direcciones IP

Cada dirección IP adicional que proporciona un front-end ofrece 64 000 puertos efímeros adicionales que el equilibrador de carga puede usar como puertos SNAT. 

Use varias direcciones IP para planear escenarios a gran escala. Use reglas de salida para mitigar el [agotamiento de SNAT](troubleshoot-outbound-connection.md#snatexhaust). 

También puede usar un [prefijo de IP pública](./load-balancer-outbound-connections.md#outboundrules) directamente con una regla de salida. 

Un prefijo de dirección IP pública aumenta el escalado de la implementación. El prefijo se puede agregar a la lista de permitidos de flujos procedentes de los recursos de Azure. Puede definir una configuración de IP de front-end en el equilibrador de carga para hacer referencia a un prefijo de dirección IP pública.  

El equilibrador de carga tiene control sobre el prefijo de IP pública. La regla de salida usará automáticamente todas las direcciones IP públicas que contiene el prefijo de IP pública para las conexiones de salida. 

Cada una de las direcciones IP dentro del prefijo de IP pública proporciona 64 000 puertos efímeros adicionales por dirección IP que el equilibrador de carga puede usar como puertos SNAT.

## <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a> Tiempo de espera de inactividad de flujo de salida y restablecimiento de TCP

Las reglas de salida proporcionan un parámetro de configuración para controlar el tiempo de espera de inactividad del flujo de salida y que se corresponda con las necesidades de su aplicación. El tiempo de espera de inactividad de salida predeterminado es de 4 minutos. Para obtener más información, consulte la [configuración de tiempos de espera de inactividad](load-balancer-tcp-idle-timeout.md). 

El comportamiento predeterminado del equilibrador de carga es anular el flujo en silencio cuando se alcanza el tiempo de espera de inactividad de salida. El parámetro `enableTCPReset` habilita un control y comportamiento predecibles de la aplicación. El parámetro indica si se debe enviar el restablecimiento de TCP (TCP RST) bidireccional en el tiempo de espera del tiempo de espera de inactividad de salida. 

Revise [Restablecimiento de TCP al agotarse el tiempo de espera de inactividad](./load-balancer-tcp-reset.md) para obtener detalles como la disponibilidad por regiones.

## <a name="securing-and-controlling-outbound-connectivity-explicitly"></a><a name="preventoutbound"></a>Protección y control de la conectividad de salida de manera explícita

Las reglas de equilibrio de carga proporcionan programación automática de NAT de salida. Algunos escenarios requieren que deshabilite la programación automática de NAT de salida por parte de la regla de equilibrio de carga. Deshabilitar a través de la regla le permite controlar o perfeccionar el comportamiento.  

Puede usar este parámetro de dos maneras:

1. Prevención de la dirección IP de entrada para SNAT de salida. Deshabilite la SNAT de salida en la regla de equilibrio de carga.
  
2. Ajuste los parámetros de **SNAT** de salida de una dirección IP que se usa para el tráfico de entrada y salida al mismo tiempo. La NAT de salida automática debe deshabilitarse para permitir que una regla de salida tome el control. Para cambiar la asignación de puertos SNAT de una dirección que también se usa para la entrada, el parámetro `disableOutboundSnat` se debe establecer en true. 

Se producirá un error en la operación de configuración de una regla de salida si intenta redefinir una dirección IP que se usa para la entrada.  Deshabilite primero la NAT de salida de la regla de equilibrio de carga.

>[!IMPORTANT]
> La máquina virtual no tendrá conectividad de salida si este parámetro se establece en true y no tiene una regla de salida para definir la conectividad de salida.  Algunas operaciones de la VM o la aplicación pueden depender de la conectividad de salida disponible. Asegúrese de comprender las dependencias del escenario y de haber considerado el impacto de hacer este cambio.

En ocasiones, no es aconsejable que una VM cree un flujo de salida. Puede que exista un requisito para administrar qué destinos reciben flujos de salida o qué destinos comienzan los flujos de entrada. Use los [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md) para administrar los destinos a los que se conecta la VM. Use los NSG para administrar qué destinos públicos inician los flujos de entrada.

Cuando aplique un grupo de seguridad de red a una máquina virtual de carga equilibrada, preste atención a las [etiquetas de servicio](../virtual-network/network-security-groups-overview.md#service-tags) y a las [reglas de seguridad predeterminadas](../virtual-network/network-security-groups-overview.md#default-security-rules). 

Asegúrese de que la VM pueda recibir solicitudes de sondeo de mantenimiento desde Azure Load Balancer.

Si un grupo de seguridad de red bloquea las solicitudes de sondeo de mantenimiento de la etiqueta predeterminada AZURE_LOADBALANCER, se producirá un error en el sondeo de mantenimiento de la máquina virtual y esta se marca como no disponible. El equilibrador de carga dejará de enviar nuevos flujos a esa VM.

## <a name="scenarios-with-outbound-rules"></a>Escenarios con reglas de salida
        

### <a name="outbound-rules-scenarios"></a>Escenarios de reglas de salida


* Configuración de conexiones de salida en un conjunto específico de direcciones IP públicas o prefijos.
* Modificación de la asignación de puertos de [SNAT](load-balancer-outbound-connections.md).
* Habilitación de solo salida.
* NAT de salida solo para VM (no entrada).
* NAT de salida para equilibrador de carga interno estándar.
* Habilitación de los protocolos TCP y UDP para NAT de salida con un equilibrador de carga público estándar.


### <a name="scenario-1-configure-outbound-connections-to-a-specific-set-of-public-ips-or-prefix"></a><a name="scenario1out"></a>Escenario 1: Configuración de conexiones de salida en un conjunto específico de direcciones IP públicas o prefijos


#### <a name="details"></a>Detalles


Use este escenario para adaptar las conexiones de salida de modo que se originen desde un conjunto de direcciones IP públicas. Agregue direcciones IP públicas o prefijos a una lista de permitidos o denegados en función del origen.


Este prefijo o esta dirección IP pública pueden ser los mismos que los usados por una regla de equilibrio de carga. 


Para usar una dirección IP pública o un prefijo distintos a los usados por una regla de equilibrio de carga: 


1. Cree una dirección IP pública o un prefijo de dirección IP pública.
2. Cree un equilibrador de carga público estándar. 
3. Cree un front-end que haga referencia al prefijo de IP pública o la dirección IP pública que quiera usar. 
4. Vuelva a usar un grupo de back-end o cree uno y coloque las VM en un grupo de back-end del equilibrador de carga público.
5. Configure una regla de salida en el equilibrador de carga público para habilitar una NAT de salida para las VM que usan el front-end. No se recomienda usar una regla de equilibrio de carga para la salida ni deshabilitar SNAT de salida en la regla de equilibrio de carga.


### <a name="scenario-2-modify-snat-port-allocation"></a><a name="scenario2out"></a>Escenario 2: Modificación de la asignación de puertos de [SNAT](load-balancer-outbound-connections.md)


#### <a name="details"></a>Detalles


Puede utilizar reglas de salida para ajustar la [asignación de puertos de SNAT automática basada en el tamaño del grupo de back-end](load-balancer-outbound-connections.md#preallocatedports). 


Si experimenta agotamiento de SNAT, aumente el número de puertos de [SNAT](load-balancer-outbound-connections.md) proporcionados a partir del valor predeterminado de 1024. 


Cada dirección IP pública aporta hasta 64 000 puertos efímeros. El número de máquinas virtuales del grupo de back-end determina el número de puertos distribuidos a cada máquina virtual. Una máquina virtual del grupo de back-end tiene acceso al máximo de 64 000 puertos. Para dos VM, se puede proporcionar un máximo de 32 000 puertos de [SNAT](load-balancer-outbound-connections.md) con una regla de salida (2 x 32 000 = 64 000). 


Puede usar reglas de salida para ajustar los puertos de SNAT proporcionados de manera predeterminada. Proporcione más o menos que la asignación de puertos predeterminada de [SNAT](load-balancer-outbound-connections.md). Cada dirección IP pública de un front-end de una regla de salida aporta hasta 64 000 puertos efímeros para su uso como puertos de [SNAT](load-balancer-outbound-connections.md). 


El equilibrador de carga proporciona puertos de [SNAT](load-balancer-outbound-connections.md) en múltiplos de 8. Si proporciona un valor que no se puede dividir por 8, se rechaza la operación de configuración. Cada regla de equilibrio de carga y regla NAT de entrada consume un intervalo de 8 puertos. Si un equilibrio de carga o una regla NAT de entrada comparten el mismo intervalo de 8 que otros, no se consumen puertos adicionales.


Si intenta asignar más puertos de [SNAT](load-balancer-outbound-connections.md) de los disponibles en función del número de direcciones IP públicas, se rechaza la operación de configuración. Por ejemplo, si proporciona 10 000 puertos por máquina virtual y siete máquinas virtuales de un grupo de back-end comparten una única dirección IP pública, se rechaza la configuración. Siete multiplicado por 10 000 supera el límite de puertos de 64 000. Agregue más direcciones IP públicas al front-end de la regla de salida para habilitar el escenario. 


Vuelva a la [asignación de puertos predeterminada](load-balancer-outbound-connections.md#preallocatedports) mediante la especificación de 0 en el número de puertos. Las primeras 50 instancias de máquina virtual obtienen 1024 puertos, las instancias de máquina virtual de la 51 a la 100 obtienen 512 hasta las instancias máximas. Para obtener más información sobre la asignación de puertos de SNAT predeterminada, vea la [tabla de asignación de puertos de SNAT](./load-balancer-outbound-connections.md#preallocatedports).


### <a name="scenario-3-enable-outbound-only"></a><a name="scenario3out"></a>Escenario 3: Habilitación de solo la salida


#### <a name="details"></a>Detalles


Use un equilibrador de carga público estándar para proporcionar NAT de salida para un grupo de VM. En este escenario, use una regla de salida por sí misma, sin reglas adicionales configuradas.


> [!NOTE]
> **Azure Virtual Network NAT** puede proporcionar conectividad de salida para máquinas virtuales sin necesidad de un equilibrador de carga. Para obtener más información, vea [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

### <a name="scenario-4-outbound-nat-for-vms-only-no-inbound"></a><a name="scenario4out"></a>Escenario 4: NAT de salida solo para VM (no de entrada)


> [!NOTE]
> **Azure Virtual Network NAT** puede proporcionar conectividad de salida para máquinas virtuales sin necesidad de un equilibrador de carga. Para obtener más información, vea [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

#### <a name="details"></a>Detalles


En este escenario: Las reglas de salida de Azure Load Balancer y Virtual Network NAT son dos opciones disponibles para la salida desde una red virtual.


1. Cree un prefijo o una dirección IP pública.
2. Cree un equilibrador de carga público estándar. 
3. Cree un front-end asociado a la dirección IP pública o el prefijo dedicado para la salida.
4. Cree un grupo de back-end para las máquinas virtuales.
5. Coloque las máquinas virtuales en el grupo de back-end.
6. Configure una regla de salida para habilitar NAT de salida.



Use un prefijo o una dirección IP pública para escalar puertos de [SNAT](load-balancer-outbound-connections.md). Agregue el origen de conexiones de salida a una lista de permitidos o denegados.



### <a name="scenario-5-outbound-nat-for-internal-standard-load-balancer"></a><a name="scenario5out"></a>Escenario 5: NAT de salida para equilibrador de carga interno estándar


> [!NOTE]
> **Azure Virtual Network NAT** puede proporcionar conectividad de salida para máquinas virtuales que usen un equilibrador de carga interno estándar. Para obtener más información, vea [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

#### <a name="details"></a>Detalles


La conectividad de salida no está disponible para un equilibrador de carga interno estándar hasta que se ha declarado explícitamente por medio de direcciones IP públicas de nivel de instancia o Virtual Network NAT, o mediante la asociación de los miembros del grupo de back-end a una configuración de equilibrador de carga solo de salida. 


Para obtener más información, vea [Configuración del equilibrador de carga solo de salida](./egress-only.md).




### <a name="scenario-6-enable-both-tcp--udp-protocols-for-outbound-nat-with-a-public-standard-load-balancer"></a><a name="scenario6out"></a>Escenario 6: Habilitación de los protocolos TCP y UDP para NAT de salida con un equilibrador de carga público estándar


#### <a name="details"></a>Detalles


Cuando se usa un equilibrador de carga público estándar, la NAT de salida automática proporcionada coincide con el protocolo de transporte de la regla de equilibrio de carga. 


1. Deshabilite [SNAT](load-balancer-outbound-connections.md) de salida en la regla de equilibrio de carga. 
2. Configure una regla de salida en el mismo equilibrador de carga.
3. Vuelva a usar el grupo de back-end que usó para sus VM. 
4. Especifique "protocol": "All" como parte de la regla de salida. 


Si solo se usan reglas de NAT entrantes, no se proporciona ninguna NAT de salida. 


1. Coloque las VM en un grupo de back-end.
2. Defina una o varias configuraciones de IP de front-end con direcciones IP públicas o prefijos de IP pública. 
3. Configure una regla de salida en el mismo equilibrador de carga. 
4. Especifique "protocol": "All" como parte de la regla de salida.


## <a name="limitations"></a>Limitaciones

- El número máximo de puertos efímeros posibles por dirección IP de front-end es 64 000.
- El intervalo de tiempo de espera de inactividad de salida que puede configurar oscila entre 4 y 120 minutos (de 240 a 7200 segundos).
- El equilibrador de carga no es compatible con ICMP para NAT de salida.
- Las reglas de salida solo se pueden aplicar a la configuración de IP principal de una NIC.  No se puede crear una regla de salida para la dirección IP secundaria de una VM o NVA. Se admiten varias NIC.
- Todas las máquinas virtuales de un **conjunto de disponibilidad** se deben agregar al grupo de back-end para la conectividad de salida. 
- Todas las máquinas virtuales de un **conjunto de escalado de máquinas virtuales** se deben agregar al grupo de back-end para la conectividad de salida.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Azure Standard Load Balancer](load-balancer-overview.md).
- Consulte las [preguntas más frecuentes sobre Azure Load Balancer](load-balancer-faqs.md)
