---
title: Topologías de flujo de llamadas en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Aprenda sobre las topologías de flujo de llamadas en Azure Communication Services.
author: nmurav
services: azure-communication-services
ms.author: nmurav
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: b2a9f53dd2513baaa60ba39b2eaae7b11fe9f4b8
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113112714"
---
# <a name="call-flow-topologies"></a>Topologías de flujos de llamadas
En este artículo se describen las topologías de flujo de llamadas de Azure Communication Services. Este artículo es excelente si es un cliente de empresa que va a integrar Communication Services en una red de las que administra. Para una introducción a los flujos de llamadas de Communication Services, visite la [documentación conceptual sobre los flujos de llamadas](./call-flows.md).

## <a name="background"></a>Fondo

### <a name="network-concepts"></a>Conceptos de red

Antes de revisar las topologías de flujo de llamadas, definiremos algunos términos a los que se hace referencia en todo el documento.

Una **red de cliente** contiene los segmentos de red que usted administra. Podría incluir redes cableadas e inalámbricas en su oficina o entre oficinas, centros de datos y proveedores de servicios de Internet.

Normalmente, una red de cliente tiene varios perímetros de red con firewalls o servidores proxy que aplican las directivas de seguridad de la organización. Se recomienda realizar una [evaluación de red completa](/microsoftteams/3-envision-evaluate-my-environment) para garantizar el rendimiento y la calidad óptimos de la solución de comunicación.

La **red de Communication Services** es el segmento de red que admite Azure Communication Services. Esta red está administrada por Microsoft y se distribuye por todo el mundo con bordes cercanos a la mayoría de las redes de cliente. Esta red es la responsable del relé de transporte, el procesamiento de los elementos multimedia para las llamadas grupales y otros componentes que admiten la comunicación multimedia enriquecida en tiempo real.

### <a name="types-of-traffic"></a>Tipos de tráfico

Communication Services se basa principalmente en dos tipos de tráfico: **elementos multimedia en tiempo real** y **señalización**.

Los **elementos multimedia en tiempo real** se transmiten mediante el protocolo de transporte en tiempo real (RTP). Este protocolo es compatible con la transmisión de datos de audio, vídeo y pantalla compartida. Estos datos pueden sufrir problemas de latencia de red. Aunque es posible transmitir elementos multimedia en tiempo real mediante TCP o HTTP, se recomienda el uso de UDP como protocolo de capa de transporte para admitir experiencias de usuario final de alto rendimiento. Las cargas de los elementos multimedia transmitidas a través de RTP se protegen mediante SRTP.

Los usuarios de la solución de Communication Services se conectarán a los servicios desde sus dispositivos cliente. La comunicación entre estos dispositivos y los servidores se controla con **señalización**. Por ejemplo, se permite iniciar una llamada y el chat en tiempo real mediante la señalización entre los dispositivos y el servicio. La mayoría del tráfico de señalización usa REST de HTTPS, aunque en algunos casos se puede usar SIP como protocolo para el tráfico de señalización. Aunque este tipo de tráfico es menos sensible a la latencia, la señalización de baja latencia permitirá una experiencia de usuario final agradable para los usuarios de la solución.

### <a name="interoperability-restrictions"></a>Restricciones de interoperabilidad

El flujo de elementos multimedia mediante Communication Services está restringido de la siguiente manera:

**Relés multimedia de terceros.** No se admite la interoperabilidad con relés multimedia de terceros. Si uno de los puntos de conexión multimedia es Communication Services, los elementos multimedia solo pueden atravesar relés multimedia nativos de Microsoft, incluidos los que admiten Microsoft Teams y Skype Empresarial.

Un controlador de borde de sesión (SBC) de terceros en el límite con RTC debe finalizar la transmisión RTP/RTCP, protegida mediante SRTP, y no retransmitirla al próximo salto. Si retransmite el flujo al próximo salto, es posible que no se entienda.

**Servidores proxy SIP de terceros.** Un cuadro de diálogo de SIP de señalización de Communication Services con una puerta de enlace o SBC de terceros puede atravesar los servidores proxy SIP nativos de Microsoft (como Teams). No se admite la interoperabilidad con servidores proxy SIP de terceros.

**B2BUA de terceros (o SBC).** El flujo de los elementos multimedia de Communication Services hacia la RTC y desde aquí lo termina un SBC de terceros. No se admite la interoperabilidad con un SBC de terceros dentro de la red de Communication Services (donde un SBC de terceros medie entre dos puntos de conexión de Communication Services).

### <a name="unsupported-technologies"></a>Tecnologías no admitidas

**Red VPN.** Communication Services no admite la transmisión multimedia a través de VPN. Si los usuarios utilizan clientes VPN, el cliente debe dividir y enrutar el tráfico multimedia a través de una conexión que no sea VPN, como se especifica en [Permiso de los elementos multimedia de Lync para omitir un túnel VPN](https://techcommunity.microsoft.com/t5/skype-for-business-blog/enabling-lync-media-to-bypass-a-vpn-tunnel/ba-p/620210).

*Nota. Aunque el título indica Lync, es aplicable a Azure Communication Services y Teams.*

**Moldeadores de paquetes.** No se admiten dispositivos de recorte, inspección o moldeado de paquetes, ya que pueden degradar considerablemente la calidad.

### <a name="call-flow-principles"></a>Principios del flujo de llamadas

Hay cuatro principios generales que sustentan los flujos de llamadas de Communication Services:

* **El primer participante de una llamada de grupo de Communication Services determina la región donde se hospeda la llamada**. En algunas topologías existen excepciones a esta regla, que se describen a continuación.
* **El punto de conexión multimedia que se usa para las llamadas de Communication Services se selecciona en función de las necesidades de procesamiento multimedia** y no se ve afectado por el número de participantes de la llamada. Por ejemplo, una llamada de punto a punto puede utilizar un punto de conexión multimedia en la nube para procesar elementos multimedia para la transcripción o la grabación, mientras que una llamada con dos participantes quizá no use ningún punto de conexión multimedia. Las llamadas grupales usarán un punto de conexión multimedia para la combinación y el enrutamiento. Este punto de conexión se selecciona en función de la región donde se hospeda la llamada. El tráfico multimedia enviado desde un cliente al punto de conexión multimedia se puede enrutar directamente, o bien, se puede usar un relé de transporte en Azure si las restricciones del firewall de red del cliente así lo requieren.
* **El tráfico multimedia para las llamadas del mismo nivel toma la ruta más directa disponible**, suponiendo que la llamada no necesita un punto de conexión multimedia en la nube. La ruta preferida es la directa al elemento remoto del mismo nivel (cliente). Si no hay una ruta directa disponible, uno o varios relés de transporte retransmiten el tráfico. El tráfico multimedia no debe atravesar servidores que actúen como moldeadores de paquetes, servidores VPN u otras funciones que puedan retrasar el procesamiento y reducir la calidad de la experiencia del usuario final.
* **El tráfico de señalización siempre se dirige al servidor más cercano al usuario.**

Para más información sobre la ruta de acceso a los elementos multimedia elegida, consulte la [documentación conceptual sobre los flujos de llamadas](./call-flows.md).


## <a name="call-flows-in-various-topologies"></a>Flujos de llamadas en varias topologías

### <a name="communication-services-internet"></a>Communication Services (Internet)

Esta topología la usan los clientes que emplean Communication Services desde la nube sin implementación local, como enrutamiento directo de Azure. En esta topología, el tráfico hacia Communication Services y desde aquí fluye a través de Internet.

:::image type="content" source="./media/call-flows/detailed-flow-general.png" alt-text="Topología de Azure Communication Services.":::

*Figura 1. Topología Communication Services*

La dirección de las flechas del diagrama anterior refleja la dirección de inicio de la comunicación que afecta a la conectividad en los perímetros empresariales. En el caso de UDP para los elementos multimedia, los primeros paquetes pueden fluir en sentido inverso, pero podrían bloquearse hasta que fluyan paquetes en la otra dirección.

Descripciones de flujo:
* Flujo 2*: representa un flujo iniciado por un usuario en la red del cliente a Internet como parte de la experiencia con Communication Services del usuario. Algunos ejemplos de estos flujos son la transmisión de elementos multimedia de DNS y del mismo nivel.
* Flujo 2: representa un flujo iniciado por un usuario remoto de Communication Services móvil, con VPN en la red del cliente.
* Flujo 3: representa un flujo iniciado por un usuario remoto de Communication Services móvil a puntos de conexión de Communication Services.
* Flujo 4: representa un flujo iniciado por un usuario en la red del cliente a Communication Services.
* Flujo 5: representa un flujo de elementos multimedia del mismo nivel entre un usuario de Communication Services y otro dentro de la red del cliente.
* Flujo 6: representa un flujo de elementos multimedia del mismo nivel entre un usuario remoto de Communication Services móvil y otro usuario remoto de Communication Services móvil a través de Internet.

### <a name="use-case-one-to-one"></a>Caso de uso: Uno a uno

Las llamadas entre dos personas usan un modelo común en el que el autor de la llamada obtendrá un conjunto de candidatos que consta de direcciones IP/puertos, incluidos los candidatos local, de relé y reflexivo (dirección IP pública del cliente como la ve el relé). El autor de la llamada envía estos candidatos a la entidad a la que se llama, quien también obtiene un conjunto similar de candidatos y los envía al autor de la llamada. Los mensajes de comprobación de la conectividad de STUN se usan para encontrar las rutas de acceso a los elementos multimedia del autor o receptor de la llamada funcionan y se selecciona la ruta que mejor funciona. Después, los elementos multimedia (es decir, los paquetes RTP/RTCP protegidos mediante SRTP) se envían mediante el par candidato seleccionado. El relé de transporte se implementa como parte de Azure Communication Services.

Si los candidatos de puerto o dirección IP local o los candidatos reflexivos tienen conectividad, para los elementos multimedia se seleccionará la ruta de acceso directa entre los clientes (o que use NAT). Si los dos clientes están en la red del cliente, se debe seleccionar la ruta de acceso directa. Esto requiere conectividad UDP directa dentro de la red del cliente. Si los clientes son usuarios de la nube Nomadic, los elementos multimedia usarán conectividad directa en función de la NAT o el firewall.

Si un cliente es interno en la red del cliente y el otro es externo (por ejemplo, un usuario de la nube móvil), es improbable que funcione la conectividad directa entre los candidatos locales o reflexivos. En este caso, una opción es usar uno de los candidatos de relé de transporte de cualquier cliente (por ejemplo, el cliente interno obtuvo un candidato de relé del relé de transporte en Azure; el externo debe poder enviar paquetes STUN/RTP/RTCP al relé de transporte). Otra opción es que el cliente interno realice el envío al candidato de relé obtenido por el cliente de la nube móvil. Aunque para los elementos multimedia se recomienda encarecidamente la conectividad UDP, se admite TCP.

**Pasos generales:**
1.  El usuario de Communication Services A resuelve el nombre de dominio de la dirección URL (DNS) con el flujo 2.
2.  El usuario A asigna un puerto de relé multimedia en el relé de transporte de Teams mediante el flujo 4.
3.  El usuario de Communication Services A envía una "invitación" con los candidatos ICE mediante el flujo 4 a Communication Services.
4.  Communication Services notifica al usuario B mediante el flujo 4.
5.  El usuario B asigna un puerto de relé multimedia en el relé de transporte de Teams mediante el flujo 4.
6.  El usuario B envía una "respuesta" con los candidatos ICE mediante el flujo 4, que se reenvía de vuelta al usuario A mediante el flujo 4.
7.  El usuario A y el usuario B invocan las pruebas de conectividad por ICE y se selecciona la mejor ruta de acceso a los elementos multimedia disponibles (consulte los diagramas siguientes para varios casos de uso).
8.  Ambos usuarios envían telemetría a Communication Services mediante el flujo 4.

### <a name="customer-network-intranet"></a>Red del cliente (intranet)

:::image type="content" source="./media/call-flows/one-to-one-internal.png" alt-text="Flujo de tráfico dentro de la red del cliente.":::

*Figura 2. En la red del cliente*

En el paso 7, se selecciona el flujo 5 de elementos multimedia del mismo nivel.

Esta transmisión multimedia es bidireccional. La dirección del flujo 5 indica que un lado inicia la comunicación desde una perspectiva de conectividad. En este caso no importa qué dirección se use, porque ambos puntos de conexión forman parte de la red del cliente.

### <a name="customer-network-to-external-user-media-relayed-by-teams-transport-relay"></a>Red del cliente a usuario externo (elementos multimedia retransmitidos por el relé de transporte de Teams)

:::image type="content" source="./media/call-flows/one-to-one-via-relay.png" alt-text="Flujo de llamada entre dos personas a través de un relé.":::

*Figura 3. Red del cliente a usuario externo (elementos multimedia retransmitidos por el relé de transporte de Azure)*

En el paso 7, se seleccionan el flujo 4 (de la red del cliente a Communication Services) y el flujo 3 (de un usuario remoto de Communication Services móvil a Azure Communication Services).

Estos flujos los retransmite el relé de transporte de Teams en Azure.

Esta transmisión multimedia es bidireccional. La dirección indica qué lado inicia la comunicación desde una perspectiva de conectividad. En este caso, estos flujos se usan para la señalización y los elementos multimedia, con diferentes direcciones y protocolos de transporte.

### <a name="customer-network-to-external-user-direct-media"></a>Red del cliente a usuario externo (elementos multimedia directos)

:::image type="content" source="./media/call-flows/one-to-one-with-extenal.png" alt-text="Flujo de llamadas entre dos personas con un usuario externo.":::

*Figura 4. Red del cliente a usuario externo (elementos multimedia directos)*

En el paso 7 se selecciona el flujo 2 (desde la red del cliente al elemento del mismo nivel a través de Internet).

Los elementos multimedia directos con un usuario remoto móvil (no retransmitido mediante Azure) son opcionales. En otras palabras, puede bloquear esta ruta de acceso para aplicar una ruta de acceso a los elementos multimedia mediante un relé transporte en Azure.

Esta transmisión multimedia es bidireccional. La dirección del flujo 2 a un usuario remoto móvil indica que un lado inicia la comunicación desde una perspectiva de conectividad.

### <a name="vpn-user-to-internal-user-media-relayed-by-teams-transport-relay"></a>Usuario de VPN a usuario interno (elementos multimedia retransmitidos por el relé de transporte de Teams)

:::image type="content" source="./media/call-flows/vpn-to-internal-via-relay.png" alt-text="Flujo de llamadas entre dos personas con un usuario de VPN a través de Relay":::

*Figura 5. Usuario de VPN a usuario interno (elementos multimedia retransmitidos por Azure Relay)*

La señalización entre la VPN y la red del cliente sigue el flujo 2*. La señalización entre la red del cliente y Azure sigue el flujo 4. Sin embargo, los elementos multimedia omiten la VPN y se enrutan mediante los flujos 3 y 4 mediante el relé multimedia de Azure.

### <a name="vpn-user-to-internal-user-direct-media"></a>Usuario de VPN a usuario interno (elementos multimedia directos)

:::image type="content" source="./media/call-flows/vpn-to-internal-direct-media.png" alt-text="Flujo de llamadas uno a uno (usuario interno) con una VPN con elementos multimedia directos":::

*Figura 6. Usuario de VPN a usuario interno (elementos multimedia directos)*

La señalización entre la VPN y la red del cliente sigue el flujo 2'. La señalización entre la red del cliente y Azure sigue el flujo 4. Sin embargo, los elementos multimedia omiten la VPN y se enrutan mediante el flujo 2 desde la red del cliente a Internet.

Esta transmisión multimedia es bidireccional. La dirección del flujo 2 a un usuario remoto móvil indica que un lado inicia la comunicación desde una perspectiva de conectividad.

### <a name="vpn-user-to-external-user-direct-media"></a>Usuario de VPN a usuario externo (elementos multimedia directos)

:::image type="content" source="./media/call-flows/vpn-user-to-external-user.png" alt-text="Flujo de llamadas uno a uno (usuario externo) con una VPN con elementos multimedia directos":::

*Figura 7. Usuario de VPN a usuario externo (elementos multimedia directos)*

La señalización entre el usuario de VPN y la red del cliente sigue el flujo 2* y el flujo 4 a Azure. Sin embargo, los elementos multimedia omiten la VPN y se enrutan mediante el flujo 6.

Esta transmisión multimedia es bidireccional. La dirección del flujo 6 a un usuario remoto móvil indica que un lado inicia la comunicación desde una perspectiva de conectividad.

### <a name="use-case-communication-services-client-to-pstn-through-communication-services-trunk"></a>Caso de uso: Cliente de Communication Services a RTC mediante el tronco de Communication Services

Communication Services permite realizar llamadas desde la red telefónica conmutada (RTC) y recibirlas aquí. Si el tronco de RTC está conectado mediante números de teléfono proporcionados por Communication Services, no hay requisitos de conectividad especiales para este caso de uso. Si quiere conectar su propio tronco de RTC local a Azure Communication Services, puede usar el enrutamiento directo de Azure (disponible en CY2021).

:::image type="content" source="./media/call-flows/acs-to-pstn.png" alt-text="Llamada entre dos personas con un participante de RTC":::

*Figura 8. Usuario de Communication Services a RTC mediante el tronco de Azure*

En este caso, la señalización y los elementos multimedia de la red del cliente a Azure siguen el flujo 4.

### <a name="use-case-communication-services-group-calls"></a>Caso de uso: Llamadas grupales de Communication Services

El servicio de uso compartido de audio/vídeo/pantalla (VBSS) forma parte de Azure Communication Services. Tiene una dirección IP pública que debe ser accesible desde la red del cliente y desde un cliente de la nube Nomadic. Los clientes o puntos de conexión deben poder conectarse al servicio.

Los clientes internos obtendrán candidatos locales, reflexivos y de relé, tal y como se describen para las llamadas entre dos personas. Los clientes enviarán estos candidatos al servicio en una invitación. El servicio no utiliza un relé, ya que tiene una dirección IP accesible públicamente, por lo que responde con su candidato de dirección IP local. El cliente y el servicio comprobarán la conectividad tal y como se describe para las llamadas entre dos personas.

:::image type="content" source="./media/call-flows/acs-group-calls.png" alt-text="Llamada grupal de OACS":::

*Figura 9. Llamadas grupales de Communication Services*

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las llamadas](../quickstarts/voice-video-calling/getting-started-with-calling.md)

Puede que los siguientes documentos le resulten interesantes:

- Más información sobre los [tipos de llamada](../concepts/voice-video-calling/about-call-types.md)
- Información sobre la [arquitectura de cliente y servidor](./client-and-server-architecture.md)