---
title: Traducción de direcciones de red de origen (SNAT) para conexiones salientes
titleSuffix: Azure Load Balancer
description: Aprenda a usar Azure Load Balancer para la conectividad saliente de Internet (SNAT).
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 10/13/2020
ms.author: allensu
ms.openlocfilehash: 3b92ef3ce195a2eee9bce53e08d977593a9f1fc2
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107477713"
---
# <a name="using-source-network-address-translation-snat-for-outbound-connections"></a>Uso de la Traducción de direcciones de red de origen (SNAT) para conexiones salientes

Las direcciones IP de front-end de un equilibrador de carga público de Azure se pueden usar para proporcionar conectividad saliente a Internet para las instancias de back-end. Esta configuración usa **traducción de direcciones de red de origen (SNAT)** . SNAT reescribe la dirección IP del back-end en la IP pública del equilibrador de carga. 

SNAT habilita el **enmascaramiento de IP** de la instancia de back-end. Esta enmascaramiento impide que los orígenes externos tengan una dirección directa a las instancias de back-end. Una dirección IP compartida entre las instancias de back-end reduce el costo de las direcciones IP públicas estáticas. Una dirección IP conocida es compatible con escenarios como la simplificación de la lista de direcciones IP permitidas con tráfico procedente de direcciones IP públicas conocidas. 

>[!Note]
> En el caso de las aplicaciones que requieren un gran número de conexiones salientes, o de los clientes empresariales que requieren un único conjunto de direcciones IP para usarse desde una red virtual determinada, [Virtual Network NAT](../virtual-network/nat-overview.md) es la solución recomendada. La asignación dinámica permite realizar una configuración simple y usar de forma eficaz los puertos SNAT desde cada dirección IP. También permite que todos los recursos de la red virtual compartan un conjunto de direcciones IP sin necesidad de compartir un equilibrador de carga.

>[!Important]
> Incluso sin la configuración de SNAT saliente, aún podrá acceder a las cuentas de Azure Storage dentro de la misma región, y los recursos de back-end seguirán teniendo acceso a los servicios de Microsoft, como las actualizaciones de Windows.

>[!NOTE] 
>Este artículo trata solo sobre las implementaciones de Azure Resource Manager. Revise [Conexiones salientes (clásico)](/previous-versions/azure/load-balancer/load-balancer-outbound-connections-classic) para todos los escenarios de implementación clásicos de Azure.

## <a name="sharing-frontend-ip-address-across-backend-resources"></a><a name ="snat"></a> Uso compartido de la dirección IP de front-end entre recursos de back-end

Si los recursos de back-end de un equilibrador de carga no tienen direcciones IP públicas a nivel de instancia (ILPIP), establecen conectividad saliente a través de la IP de front-end del equilibrador de carga público. Se usan puertos para generar identificadores únicos que se emplean para mantener flujos distintos. Internet usa una tupla de cinco elementos para proporcionar esta distinción.

La tupla de cinco elementos consta de:

* IP de destino
* Puerto de destino
* IP de origen
* Puerto y protocolo de origen para proporcionar esta distinción.

Si se usa un puerto para las conexiones entrantes, tendrá un **agente de escucha** para las solicitudes de conexión entrantes en ese puerto. Recuerde que ese puerto no se puede usar para las conexiones salientes. Para establecer una conexión saliente, se usa un **puerto efímero** para proporcionar al destino un puerto con el que comunicarse y mantener un flujo de tráfico distinto. Cuando estos puertos efímeros se usan para realizar la traducción SNAT, se denominan **puertos SNAT**. 

Por definición, todas las direcciones IP tienen 65 535 puertos. Cada puerto puede usarse para conexiones de entrada o salida para TCP (protocolo de control de transmisión) y UDP (protocolo de datagramas de usuario). 

Cuando se agrega una dirección IP pública como una dirección IP de front-end a un equilibrador de carga, Azure ofrece 64 000 puertos válidos para SNAT.

>[!NOTE]
> Cada puerto que se usa para una regla NAT de entrada o de equilibrio de carga consumirá un intervalo de ocho puertos de estos 64 000 puertos, lo que reduce el número de puertos que son válidos para SNAT. Si una regla NAT o de equilibrio de carga está en el mismo intervalo de ocho puertos que otra regla, no consumirá puertos adicionales. 

A través de las [reglas de salida](./outbound-rules.md) y de equilibrio de carga, estos puertos SNAT se pueden distribuir a instancias de back-end para permitirles compartir las IP públicas del equilibrador de carga para las conexiones salientes.

Cuando el [escenario 2](#scenario2) que tiene a continuación esté configurado, el host de cada instancia de back-end realizará la traducción SNAT en los paquetes que forman parte de una conexión saliente. 

Al realizar la traducción SNAT en una conexión de salida desde una instancia de back-end, el host reescribe la dirección IP de origen en una de las direcciones IP de front-end. 

Para mantener flujos únicos, el host reescribe el puerto de origen de cada paquete de salida en un puerto SNAT de la instancia de back-end.

## <a name="outbound-connection-behavior-for-different-scenarios"></a>Comportamiento de la conexión de salida en diferentes escenarios
  * Máquina virtual con dirección IP pública.
  * Máquina virtual sin dirección IP pública.
  * Máquina virtual sin dirección IP pública y sin Standard Load Balancer.
        
 ### <a name="scenario-1-virtual-machine-with-public-ip-either-with-or-without-a-load-balancer"></a><a name="scenario1"></a> Escenario 1: Máquina virtual con dirección IP pública con o sin un equilibrador de carga.

 | Asociaciones | Método | Protocolos IP |
 | ---------- | ------ | ------------ |
 | Equilibrador de carga público o independiente | [SNAT (traducción de direcciones de red de origen)](#snat) </br> no se usa. | TCP (protocolo de control de transmisión) </br> UDP (protocolo de datagramas de usuario) </br> ICMP (protocolo de mensajes de control de Internet) </br> ESP (carga de seguridad encapsuladora) |

 #### <a name="description"></a>Descripción

 Todo el tráfico volverá al cliente solicitante desde la dirección IP pública de la máquina virtual (IP de nivel de instancia).
 
 Azure usa la dirección IP pública asignada a la configuración IP de la NIC de la instancia para todos los flujos de salida. La instancia tiene disponibles todos los puertos efímeros. Es irrelevante si la máquina virtual es de carga equilibrada o no. Este escenario tiene prioridad sobre las demás. 

 Una dirección IP pública asignada a una máquina virtual es una relación 1:1 (en lugar de 1:muchos) y se implementa como NAT de 1:1 sin estado.

 ### <a name="scenario-2-virtual-machine-without-public-ip-and-behind-standard-public-load-balancer"></a><a name="scenario2"></a>Escenario 2: Máquina virtual sin dirección IP pública y detrás de una instancia pública de Standard Load Balancer

 | Asociaciones | Método | Protocolos IP |
 | ------------ | ------ | ------------ |
 | Equilibrador de carga público estándar | Uso de direcciones IP de front-end del equilibrador de carga para [SNAT](#snat).| TCP </br> UDP |

 #### <a name="description"></a>Descripción

 El recurso del equilibrador de carga se configura con una regla de salida o una regla de equilibrio de carga que habilita la traducción SNAT. Esta regla se usa para crear un vínculo entre el front-end de la dirección IP pública y el grupo de back-end. 

 Si no configura esta regla, el comportamiento será el que se describe en el escenario 3. 

 No se necesita una regla con un agente de escucha para que el sondeo de estado se realice correctamente.

 Cuando una máquina virtual crea un flujo de salida, Azure traduce la dirección IP de origen a la dirección IP pública del front-end público del equilibrador de carga. Esta traducción se realiza a través de [SNAT](#snat). 

 Para distinguir los flujos individuales que se originan en la máquina virtual, se usan puertos efímeros de la dirección IP pública de front-end del equilibrador de carga. Cuando se crean flujos de salida, SNAT usa dinámicamente los [puertos efímeros asignados previamente](#preallocatedports). 

 En este contexto, los puertos efímeros usados para SNAT se conocen como puertos SNAT. Se recomienda encarecidamente configurar explícitamente una [regla de salida](./outbound-rules.md). Si usa el valor predeterminado de SNAT en una regla de equilibrio de carga, los puertos SNAT se asignan previamente, como se describe en la [tabla de asignación de puertos SNAT predeterminados](#snatporttable).

> [!NOTE]
> **Azure Virtual Network NAT** puede proporcionar conectividad de salida para máquinas virtuales sin necesidad de un equilibrador de carga. Para obtener más información, vea [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

 ### <a name="scenario-3-virtual-machine-without-public-ip-and-behind-standard-internal-load-balancer"></a><a name="scenario3"></a>Escenario 3: Máquina virtual sin dirección IP pública y detrás de una instancia interna estándar de Load Balancer

 | Asociaciones | Método | Protocolos IP |
 | ------------ | ------ | ------------ |
 | Equilibrador de carga público estándar | No hay conectividad de Internet.| None |

 #### <a name="description"></a>Descripción
 
Cuando se usa un equilibrador de carga interno estándar, no se usan direcciones IP efímeras para SNAT. Esta característica admite las opciones de seguridad de forma predeterminada. Asimismo, esta característica garantiza que todas las direcciones IP que usen los recursos sean configurables y se puedan reservar. 

Para lograr la conectividad saliente a Internet cuando se usa un equilibrador de carga interno estándar, debe configurar una dirección IP pública de nivel de instancia para seguir el comportamiento del [escenario 1](#scenario1). 

Otra opción consiste en agregar las instancias de back-end a un equilibrador de carga público estándar con una regla de salida configurada. Las instancias de back-end se agregan a un equilibrador de carga interno para el equilibrio de carga interno. Esta implementación sigue el comportamiento del [escenario 2](#scenario2). 

> [!NOTE]
> **Azure Virtual Network NAT** puede proporcionar conectividad de salida para máquinas virtuales sin necesidad de un equilibrador de carga. Para obtener más información, vea [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

 ### <a name="scenario-4-virtual-machine-without-public-ip-and-behind-basic-load-balancer"></a><a name="scenario4"></a>Escenario 4: Máquina virtual sin dirección IP pública y detrás de una versión Básica de Load Balancer

 | Asociaciones | Método | Protocolos IP |
 | ------------ | ------ | ------------ |
 |Ninguno </br> Versión Básica de Load Balancer | [SNAT](#snat) con una dirección IP dinámica en el nivel de instancia| TCP </br> UDP | 

 #### <a name="description"></a>Descripción

 Cuando la VM crea un flujo de salida, Azure traduce la dirección IP de origen a una dirección IP de origen pública asignada dinámicamente. Esta dirección IP pública **no se puede configurar** y no se puede reservar. Esta dirección no cuenta para el límite de recursos de dirección IP pública de la suscripción. 

Se liberará la dirección IP pública y se solicitará una nueva dirección IP pública si vuelve a implementar alguno de los siguientes: 

 * Máquina virtual
 * Conjunto de disponibilidad
 * Conjunto de escalado de máquina virtual 

 No use este escenario para agregar direcciones IP a una lista de permitidos. Use el escenario 1 o 2, donde se declara explícitamente el comportamiento de salida. Los puertos [SNAT](#snat) se asignan previamente, como se describe en la [tabla de asignación de puertos SNAT predeterminados](#snatporttable).

## <a name="exhausting-ports"></a><a name="scenarios"></a> Agotamiento de puertos

Cada conexión a la misma dirección IP de destino y puerto de destino usará un puerto SNAT. Esta conexión mantiene un **flujo de tráfico** distinto desde la instancia de back-end o **cliente** a un **servidor**. Este proceso proporciona al servidor un puerto distinto en el que dirigir el tráfico. Sin este proceso, el equipo cliente no sabe a qué flujo pertenece un paquete.

Imagine que varios exploradores van a https://www.microsoft.com, que tiene los valores siguientes:

* Dirección IP de destino = 23.53.254.142
* Puerto de destino = 443
* Protocolo = TCP

Sin puertos de destino diferentes para el tráfico de retorno (el puerto SNAT usado para establecer la conexión), el cliente no tendrá forma de separar el resultado de una consulta de otro.

Las conexiones salientes pueden aumentar. Una instancia de back-end podría tener asignados puertos insuficientes. Si la **reutilización de conexiones** no está habilitada, aumenta el riesgo de **agotamiento de puertos** SNAT.

Se producirá un error en las nuevas conexiones salientes a una dirección IP de destino cuando se agoten los puertos. Las conexiones se realizarán correctamente cuando esté disponible un puerto. Este agotamiento se produce cuando los 64 000 puertos de una dirección IP se distribuyen entre muchas instancias de back-end. Para obtener instrucciones sobre la mitigación del agotamiento de puertos SNAT, consulte la [guía de solución de problemas](./troubleshoot-outbound-connection.md).  

En el caso de las conexiones TCP, el equilibrador de carga usará un solo puerto SNAT para cada IP y puerto de destino. Este uso múltiple permite varias conexiones a la misma dirección IP de destino con el mismo puerto SNAT, pero se limita si la conexión no es a puertos de destino diferentes.

En el caso de las conexiones UDP, el equilibrador de carga usa un algoritmo de **NAT de cono con restricción de puerto**, que consume un puerto SNAT por cada IP de destino, sea cual sea el puerto de destino. 

Un puerto se reutiliza para un número ilimitado de conexiones. El puerto solo se reutiliza si la dirección IP o puerto de destino son diferentes.

## <a name="default-port-allocation"></a><a name="preallocatedports"></a> Asignación de puertos predeterminados

A cada dirección IP pública asignada como IP de front-end del equilibrador de carga se le proporcionan 64 000 puertos SNAT para sus miembros del grupo de back-end. Los puertos no se pueden compartir con miembros del grupo de back-end. Un intervalo de puertos SNAT solo lo puede usar una única instancia de back-end para garantizar que los paquetes devueltos se enruten correctamente. 

Se recomienda usar una regla de salida explícita para configurar la asignación de puertos SNAT. Esta regla maximizará el número de puertos SNAT que cada instancia de back-end tiene disponible para las conexiones salientes. 

Si usa la asignación automática de SNAT saliente a través de una regla de equilibrio de carga, la tabla de asignación definirá la asignación de los puertos.

En la <a name="snatporttable"></a>tabla siguiente se muestran las asignaciones previas de puertos SNAT para los niveles de tamaño del grupo de back-end:

| Tamaño del grupo (instancias de máquina virtual) | Puertos SNAT asignados previamente por configuración IP |
| --- | --- |
| 1-50 | 1024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1000 | 32 | 

>[!NOTE]
> Si tiene un grupo de back-end con un tamaño máximo de 10, cada instancia puede tener 64 000/10 = 6400 puertos si define una regla de salida explícita. Según la tabla anterior, cada uno solo tendrá 1024 si elige la asignación automática.

## <a name="outbound-rules-and-virtual-network-nat"></a><a name="outboundrules"></a> Reglas de salida y Virtual Network NAT

Las reglas de salida de Azure Load Balancer y Virtual Network NAT son dos opciones disponibles para la salida desde una red virtual.

Para obtener más información sobre las reglas de salida, consulte [Reglas de salida](outbound-rules.md).

Para obtener más información sobre Azure Virtual Network NAT, consulte [¿Qué es Azure Virtual Network NAT?](../virtual-network/nat-overview.md)

## <a name="constraints"></a>Restricciones

*   Cuando una conexión está inactiva y no se envían paquetes nuevos, los puertos se liberarán después de un intervalo entre 4 y 120 minutos.
  * Este umbral se puede configurar a través de reglas de salida.
*   Cada dirección IP proporciona 64 000 puertos que se pueden usar para SNAT.
*   Cada puerto se puede usar para conexiones TCP y UDP con una dirección IP de destino.
  * Se necesita un puerto SNAT UDP tanto si el puerto de destino es único como si no. Para cada conexión UDP con una dirección IP de destino, se usa un puerto SNAT UDP.
  * Se puede usar un puerto SNAT TCP para varias conexiones con la misma dirección IP de destino, siempre que los puertos de destino sean diferentes.
*   El agotamiento de SNAT se produce cuando una instancia de back-end se ejecuta fuera de los puertos SNAT establecidos. Un equilibrador de carga puede tener puertos SNAT sin usar. Si los puertos SNAT usados de una instancia de back-end superan sus puertos SNAT, no podrá establecer nuevas conexiones salientes.

## <a name="next-steps"></a>Pasos siguientes

*   [Solución de errores de conexiones salientes debidos a un agotamiento de SNAT](./troubleshoot-outbound-connection.md)
*   [Revise las métricas de SNAT](./load-balancer-standard-diagnostics.md#how-do-i-check-my-snat-port-usage-and-allocation) y familiarícese con la manera correcta de filtrarlas, dividirlas y visualizarlas.
