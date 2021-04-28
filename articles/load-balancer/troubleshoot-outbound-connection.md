---
title: Solución de problemas de las conexiones salientes en Azure Load Balancer
description: Resoluciones para problemas habituales con la conectividad saliente a través de Azure Load Balancer.
services: load-balancer
author: erichrt
ms.service: load-balancer
ms.topic: troubleshooting
ms.date: 05/7/2020
ms.author: errobin
ms.openlocfilehash: 1f52900086afef09d69b80bf44474d5514e25235
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107748887"
---
# <a name="troubleshooting-outbound-connections-failures"></a><a name="obconnecttsg"></a> Solución de errores de conexiones salientes

Este artículo está diseñado para proporcionar soluciones a los problemas habituales que pueden producirse con conexiones salientes de una instancia de Azure Load Balancer. La mayoría de los problemas con la conectividad saliente que experimentan los clientes se deben al agotamiento de puertos SNAT y a tiempos de espera de conexión agotados que generan pérdida de paquetes. En este artículo se proporcionan los pasos para mitigar cada uno de estos problemas.

## <a name="managing-snat-pat-port-exhaustion"></a><a name="snatexhaust"></a>Administración del agotamiento de puertos SNAT (PAT)
Los [puertos efímeros](load-balancer-outbound-connections.md) usados para [PAT](load-balancer-outbound-connections.md) son un recurso agotable, como se describe en [Máquina virtual independiente sin una dirección IP pública](load-balancer-outbound-connections.md) y [Máquina virtual de carga equilibrada sin dirección IP pública](load-balancer-outbound-connections.md). Puede supervisar el uso de puertos efímeros y compararlos con su asignación actual para determinar la probabilidad o confirmar el agotamiento de SNAT mediante [esta guía](./load-balancer-standard-diagnostics.md#how-do-i-check-my-snat-port-usage-and-allocation).

Si sabe que se van a iniciar muchas conexiones TCP o UDP salientes a la misma dirección IP y puerto de destino, y observa errores en las conexiones salientes, o el soporte técnico está avisando del agotamiento de los puertos SNAT ([puertos efímeros](load-balancer-outbound-connections.md#preallocatedports) asignados previamente usados por [PAT](load-balancer-outbound-connections.md)), dispone de varias opciones generales para mitigar este problema. Revise estas opciones y vea cuál está disponible y resulta mejor para su escenario. Es posible que una o varias de ellas puedan ayudar a administrar este escenario.

Si tiene problemas para entender el comportamiento de las conexiones de salida, puede usar las estadísticas de pila IP (netstat). O bien, puede ser útil observar el comportamiento de la conexión mediante capturas de paquetes. Estas capturas de paquetes se pueden realizar en el sistema operativo invitado de la instancia o se puede usar [Network Watcher para la captura de paquetes](../network-watcher/network-watcher-packet-capture-manage-portal.md). 

## <a name="manually-allocate-snat-ports-to-maximize-snat-ports-per-vm"></a><a name ="manualsnat"></a>Asignación manual de puertos SNAT para maximizar puertos SNAT por máquina virtual
Como se define en los [puertos preasignados](load-balancer-outbound-connections.md#preallocatedports), el equilibrador de carga asignará automáticamente los puertos según el número de máquinas virtuales en el back-end. De forma predeterminada, esto se hace con cautela para garantizar la escalabilidad. Si conoce el número máximo de máquinas virtuales que tendrá en el back-end, puede asignar manualmente puertos SNAT en cada regla de salida. Por ejemplo, si sabe que va a tener un máximo de 10 máquinas virtuales, puede asignar 6400 puertos SNAT por máquina virtual en lugar del valor predeterminado 1024. 

## <a name="modify-the-application-to-reuse-connections"></a><a name="connectionreuse"></a>Modificación de la aplicación para reutilizar las conexiones 
Puede reducir la demanda de puertos efímeros utilizados para SNAT mediante la reutilización de conexiones en la aplicación. Esta es especialmente importante para protocolos como HTTP/1.1, en los que la reutilización de las conexiones es la opción predeterminada. Y, a su vez, se pueden beneficiar otros protocolos que usan HTTP como transporte (por ejemplo, REST). 

La reutilización es siempre preferible a conexiones individuales atómicas para cada solicitud. La reutilización tiene como resultado transacciones TCP muy eficientes y con un mayor rendimiento.

## <a name="modify-the-application-to-use-connection-pooling"></a><a name="connection pooling"></a>Modificación de la aplicación para usar la agrupación de conexiones
Puede emplear un esquema de agrupación de conexiones en la aplicación, donde las solicitudes se distribuyan internamente a través de un conjunto fijo de conexiones (cada una reutilizada siempre que sea posible). Este esquema limita el número de puertos efímeros en uso y crea un entorno más predecible. También puede aumentar el rendimiento de las solicitudes al permitir varias operaciones simultáneas cuando una sola conexión bloquea la respuesta de una operación.  

La agrupación de conexiones puede existir ya en el marco de trabajo que se use para desarrollar la aplicación o su configuración. Se puede combinar la agrupación de conexiones con la reutilización de conexiones. Así, sus múltiples conexiones consumen un número fijo y predecible de puertos para la misma dirección IP y el mismo puerto de destino. Las solicitudes también se benefician del uso eficiente de transacciones TCP que reducen la latencia y la utilización de recursos. Las transacciones UDP también pueden beneficiarse, dado que administrar el número de flujos de UDP puede evitar, a su vez, condiciones de administración del agotamiento de los puertos SNAT.

## <a name="modify-the-application-to-use-less-aggressive-retry-logic"></a><a name="retry logic"></a>Modificación de la aplicación para utilizar lógica de reintento menos agresiva
Cuando se agotan los [puertos efímeros asignados previamente](load-balancer-outbound-connections.md#preallocatedports) usados para [PAT](load-balancer-outbound-connections.md) o se producen errores de la aplicación, los reintentos agresivos o por fuerza bruta sin lógica de retroceso y decadencia causan que se produzca el agotamiento o su persistencia. Puede reducir la demanda de puertos efímeros mediante el uso de una lógica de reintento menos agresiva. 

Los puertos efímeros tienen un tiempo de inactividad de 4 minutos (no ajustable). Si los reintentos son demasiado agresivos, el agotamiento no tiene ninguna oportunidad de solucionarlo por sí mismo. Por lo tanto, una parte fundamental del diseño es considerar cómo y con qué frecuencia la aplicación vuelve a intentar las transacciones.

## <a name="assign-a-public-ip-to-each-vm"></a><a name="assignilpip"></a>Asignación de una IP pública a cada máquina virtual
La asignación de una dirección IP pública cambia el escenario a [IP pública a una máquina virtual](load-balancer-outbound-connections.md). Todos los puertos efímeros de la dirección IP pública que se usan para cada máquina virtual están disponibles para la máquina virtual. (En contraste con los escenarios donde los puertos efímeros de una dirección IP pública se comparten con todas las máquinas virtuales asociadas al grupo back-end respectivo). Existen ventajas e inconvenientes que deben considerarse, como el costo adicional de las direcciones IP públicas y el posible impacto del filtrado de un gran número de direcciones IP individuales.

>[!NOTE] 
>Esta opción no está disponible para roles de trabajo web.

## <a name="use-multiple-frontends"></a><a name="multifesnat"></a> Uso de varios servidores front-end
Cuando se usa Load Balancer estándar público, se asignan [varias direcciones IP de servidor front-end para las conexiones salientes](load-balancer-outbound-connections.md) y [se multiplica el número de puertos SNAT disponibles](load-balancer-outbound-connections.md#preallocatedports).  Cree una configuración de IP de servidor front-end, una regla y el grupo de servidores back-end para desencadenar la programación de SNAT para la dirección IP pública del servidor front-end.  La regla no tiene que funcionar y no es necesario que el sondeo de estado sea correcto.  Si usa varios servidores front-end para la entrada también (en lugar de simplemente para la salida), debe usar además sondeos de estado personalizados para garantizar la confiabilidad.

>[!NOTE]
>En la mayoría de los casos, el agotamiento de puertos SNAT indica que el diseño es incorrecto.  Asegúrese de saber por qué está agotando los puertos antes de usar más servidores front-end para agregar puertos SNAT.  Puede estar enmascarando un problema que podría provocar un error más adelante.

## <a name="scale-out"></a><a name="scaleout"></a>Escalado horizontal
Los [puertos preasignados](load-balancer-outbound-connections.md#preallocatedports) se asignan según el tamaño del grupo de back-end y se agrupan en niveles para minimizar la interrupción cuando algunos de los puertos tengan que reasignarse para alojar el siguiente nivel de tamaño superior de grupo de back-end.  Podría tener la opción de aumentar la utilización de puertos SNAT para un front-end determinado si ajusta el grupo de back-end al tamaño máximo de un nivel determinado.  Tenga en cuenta que la asignación de puertos predeterminada es necesaria para que la aplicación se escale horizontalmente de forma eficaz sin que haya riesgo de agotamiento de SNAT.

Por ejemplo, dos máquinas virtuales en el grupo back-end tendrían 1024 puertos SNAT disponibles por cada configuración de IP, lo que permite un total de 2048 puertos SNAT para la implementación.  Si la implementación fuera a aumentarse hasta 50 máquinas virtuales, incluso si el número de puertos preasignados permanece constante por máquina virtual, se puede utilizar un total de 51 200 (50 x 1024) puertos SNAT por la implementación.  Si quiere escalar horizontalmente la implementación, compruebe el número de [puertos preasignados](load-balancer-outbound-connections.md#preallocatedports) por cada nivel para asegurarse de adaptar el escalado horizontal al máximo del nivel correspondiente.  En el ejemplo anterior, si hubiera elegido escalar horizontalmente a 51 en lugar de a 50 instancias, avanzaría al nivel siguiente y terminaría con menos puertos SNAT por VM, así como en total.

Si se escala horizontalmente al siguiente nivel de grupo de back-end de mayor tamaño, algunas de las conexiones salientes podrían agotar el tiempo de espera si hubiera que reasignar los puertos.  Si solo usa algunos de los puertos SNAT, no importa si se escala horizontalmente al siguiente mayor tamaño de grupo de back-end.  La mitad de los puertos existentes se volverá a asignar cada vez que cambie al siguiente nivel de grupo de back-end.  Si no quiere que suceda esto, tendrá que adaptar la implementación al tamaño del nivel.  O bien, asegúrese de que la aplicación puede detectar y volver a intentar según se requiera.  Las conexiones persistentes de TCP pueden ayudar a detectar cuándo dejan de funcionar los puertos SNAT por una reasignación.

## <a name="use-keepalives-to-reset-the-outbound-idle-timeout"></a><a name="idletimeout"></a>Uso de conexiones persistentes para restablecer el tiempo de espera de inactividad saliente
Las conexiones salientes tienen un tiempo de espera de inactividad de 4 minutos. Este tiempo de espera se ajusta desde [Reglas de salida](outbound-rules.md). Pero también puede usar conexiones persistentes de transporte (por ejemplo, TCP) o de capa de aplicación para actualizar un flujo de inactividad y restablecer este tiempo de espera de inactividad en caso necesario.  

Al utilizar conexiones persistentes de TCP, es suficiente con habilitarlas en un lado de la conexión. Por ejemplo, es suficiente habilitarlas solo en el servidor para restablecer el temporizador de inactividad del flujo y no se necesita para ambos lados en conexiones persistentes de TCP iniciadas.  Existen conceptos similares existen para la capa de aplicación, incluidas las configuraciones de cliente/servidor de base de datos.  Compruebe el lado del servidor para ver qué opciones existen para las conexiones persistentes específicas de la aplicación.

## <a name="next-steps"></a>Pasos siguientes
Siempre buscamos mejorar la experiencia de nuestros clientes. Si tiene problemas con la conectividad saliente que no aparecen o no se resuelven en este artículo, envíe sus comentarios a través de GitHub desde la parte inferior de la página y abordaremos sus comentarios lo antes posible.
