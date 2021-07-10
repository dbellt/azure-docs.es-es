---
title: Preparación de la red de una organización para Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los requisitos de red para las llamadas de voz y vídeo de Azure Communication Services
author: nmurav
manager: jken
services: azure-communication-services
ms.author: nmurav
ms.date: 3/23/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 2fa9cbba571fa5603607c5efea7f49a5822726e9
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089423"
---
# <a name="ensure-high-quality-media-in-azure-communication-services"></a>Garantía de multimedia de alta calidad en Azure Communication Services

En este documento se proporciona información general sobre los factores y procedimientos recomendados que se deben tener en cuenta al crear experiencias de comunicación multimedia de alta calidad con Azure Communication Services.

## <a name="factors-that-affect-media-quality-and-reliability"></a>Factores que afectan a la calidad y confiabilidad de los elementos multimedia

Hay muchos factores que contribuyen a la calidad de los elementos multimedia en tiempo real (audio, vídeo y uso compartido de aplicaciones) de Azure Communication Services, entre los que se incluyen la calidad de la red, el ancho de banda, el firewall, el host y la configuración de los dispositivos.


### <a name="network-quality"></a>Calidad de la red

La calidad de los elementos multimedia en tiempo real a través de IP se ve considerablemente afectada por la calidad de la conectividad de red subyacente, pero especialmente por la cantidad de:
* **Latencia**. Es el tiempo que tarda en llegar un paquete IP desde un punto A a un punto B en la red. Este retraso en la propagación por la red viene determinado por la distancia física entre los dos puntos y cualquier sobrecarga adicional que provoquen los dispositivos a través de los que fluye el tráfico. La latencia se mide no solo como el tiempo que se tarda en realizar un recorrido unidireccional, sino también el tiempo de ida y vuelta (RTT).
* **Pérdida de paquetes**. Porcentaje de paquetes que se pierden en un período determinado. La pérdida de paquetes afecta directamente a la calidad del audio (desde la pérdida de paquetes pequeños, lo que no tiene apenas impacto, hasta pérdidas de ráfagas completas que provocan una interrupción completa del audio).
* **Inestabilidad de llegada entre paquetes o simplemente inestabilidad**. Es el cambio medio, en lo que se refiere al retraso, entre paquetes sucesivos. Azure Communication Services puede adaptarse a algunos niveles de inestabilidad a través del almacenamiento en búfer. Los participantes no notarán los efectos de la inestabilidad, salvo que esta supere el almacenamiento en búfer.

### <a name="network-bandwidth"></a>Ancho de banda de red

Asegúrese de que la red está configurada para admitir el ancho de banda que requieren las sesiones de elementos multimedia de Azure Communication Services simultáneas y otras aplicaciones empresariales. Para que cualquier solución multimedia de Communication Services se implemente correctamente, es fundamental probar la ruta de acceso de red de un extremo a otro, con el fin de saber si hay cuellos de botella en el ancho de banda.

A continuación se muestran los requisitos de ancho de banda de los SDK de JavaScript:

|Ancho de banda|Escenarios|
|:--|:--|
|40 kbps|Llamadas de audio punto a punto|
|500 kbps|Llamadas de audio punto a punto y uso compartido de pantalla|
|500 kbps|Vídeollamadas de calidad estándar punto a punto con una resolución de 360 puntos a 30 fps|
|1,2 MBps|Vídeollamadas de alta definición punto a punto con una resolución de 720 puntos a 30 fps|
|500 kbps|Vídeollamadas grupales con una resolución de 360 puntos a 30 fps|
|1,2 MBps|Vídeollamadas grupales de alta definición con una resolución de 720 puntos a 30 fps| 

A continuación, se muestran los requisitos de ancho de banda para los SDK de Windows, Android e iOS nativos:

|Ancho de banda|Escenarios|
|:--|:--|
|30 kbps|Llamadas de audio punto a punto |
|130 kbps|Llamadas de audio punto a punto y uso compartido de pantalla|
|500 kbps|Vídeollamadas de calidad estándar punto a punto con una resolución de 360 puntos a 30 fps|
|1,2 MBps|Vídeollamadas de alta definición punto a punto con una resolución de 720 puntos a 30 fps|
|1,5 Mbps|Vídeollamadas de alta definición punto a punto con una resolución de 1080 puntos a 30 fps |
|500 kbps/1 Mbps|Vídeollamadas grupales|
|1 Mbps/2 Mbps|Vídeollamadas grupales de alta definición (vídeos de 540 puntos en pantalla de 1080 puntos)|

### <a name="firewalls-configuration"></a>Configuración de firewalls

Las conexiones de Azure Communication Services requieren conectividad a Internet en puertos y direcciones IP específicos, con el fin de ofrecer experiencias multimedia de alta calidad. Azure Communication Services también funciona aunque no haya acceso a estos puertos y direcciones IP. Sin embargo, la experiencia óptima se obtiene cuando los puertos e intervalos IP recomendados están abiertos.

| Categoría | Intervalos IP o nombre de dominio completo | Puertos | 
| :-- | :-- | :-- |
| Tráfico multimedia | [Intervalo de direcciones IP de la nube pública de Azure](https://www.microsoft.com/download/confirmation.aspx?id=56519) | UDP 3478 a 3481, puertos TCP 443 |
| Señalización, telemetría, registro| *.skype.com, *.microsoft.com, *.azure.net, *.azureedge.net, *.office.com y *.trouter.io | TCP 443, 80 |

### <a name="network-optimization"></a>Optimización de la red

Las siguientes tareas son opcionales, no son necesarias para implementar Azure Communication Services. Use esta guía para optimizar el rendimiento de la red y de Azure Communication Services, o si sabe que tiene algunas limitaciones de red.
Es posible que desee optimizarlo más si:
* Azure Communication Services funciona con lentitud (puede que el ancho de banda no sea suficiente)
* Las llamadas se siguen perdiendo (puede deberse a bloqueos del firewall o del proxy)
* Las llamadas tienen estática y se cortan, o bien las voces suenan como robots (puede ser debido a la inestabilidad o pérdida de paquetes)

| Tarea de optimización de la red | Detalles |
| :-- | :-- |
| Planeamiento de la red | En esta documentación puede encontrar los requisitos mínimos de red para poder realizar y recibir llamadas. Consulte el [ejemplo de Teams para planear una red](/microsoftteams/tutorial-network-planner-example) |
| Resolución de nombres externos | Asegúrese de que todos los equipos en los que se ejecutan los SDK de Azure Communication Services pueden resolver consultas de DNS externas para detectar los servicios que proporciona Azure Communication Services y que los firewalls no impiden el acceso. Asegúrese de que los SDK pueden resolver direcciones *.skype.com, *.microsoft.com, *.azure.net, *.azureedge.net, *.office.com, *.trouter.io  |
| Conservación de la persistencia de la sesión | Asegúrese de que el firewall no cambia las direcciones NAT (traducción de direcciones de red) asignadas ni los puertos de UDP.
Validación del tamaño del grupo NAT | Valide el tamaño del grupo de traducción de direcciones de red (NAT) necesario para la conectividad de los usuarios. Si varios usuarios y dispositivos acceden a Azure Communication Services mediante la [traducción de direcciones de red (NAT) o la traducción de direcciones de puerto (PAT)](/office365/enterprise/nat-support-with-office-365), asegúrese de que los dispositivos que se ocultan detrás de cada dirección IP enrutable públicamente no superan el número admitido. Para evitar el agotamiento de los puertos, asegúrese de que las direcciones IP públicas adecuadas están asignadas a los grupos NAT. El agotamiento de puertos contribuirá a que los usuarios y dispositivos internos no puedan conectarse a Azure Communication Services. |
| Guía para la detección y prevención de intrusiones | Si su entorno tiene implementado un sistema de prevención o [detección de intrusiones](../../../network-watcher/network-watcher-intrusion-detection-open-source-tools.md) (IDS/IPS), con el fin de proporcionar una capa adicional de seguridad para las conexiones salientes, conceda el permiso pertinente a todas las direcciones URL de Azure Communication Services. |
| Configuración de VPN de túnel dividido | Se recomienda especificar una ruta de acceso alternativa para el tráfico de Teams que sortee la red privada virtual (VPN), lo que se conoce comúnmente como [VPN de túnel dividido](/windows/security/identity-protection/vpn/vpn-routing). La tunelización dividida significa que el tráfico de Azure Communications Services no pasa por la VPN, sino que va directamente a Azure. Le hecho de omitir la VPN afecta positivamente a la calidad de los elementos multimedia y reduce la carga de los dispositivos VPN y de la red de la organización. Para implementar una VPN de túnel dividido, póngase en contacto con el proveedor de VPN. Otras razones por las que se recomienda sortear la VPN: <ul><li> Normalmente, las VPN no están diseñadas ni configuradas para admitir elementos multimedia en tiempo real.</li><li> También es posible que las VPN no admitan UDP (que es necesario para Azure Communication Services)</li><li>Además, las VPN aportan una capa adicional de cifrado sobre el tráfico multimedia que ya está cifrado.</li><li>Es posible que la conectividad con Azure Communication Services no sea eficaz debido al tráfico de conexiones entre nodos a través de un dispositivo VPN.</li></ul>|
| Implementación de QoS | [Utilice Calidad de servicio (QoS)](/microsoftteams/qos-in-teams) para configurar la clasificación de un orden de prioridad a los paquetes, ya que así mejorará la calidad de las llamadas y le resultará más fácil supervisar y solucionar problemas de calidad de las llamadas. QoS se debe implementar en todos los segmentos de cualquier red administrada. Incluso cuando una red se ha aprovisionado adecuadamente para el ancho de banda, QoS proporciona mitigación de riesgos en caso de eventos de red imprevistos. Con QoS, el tráfico de voz tiene la máxima prioridad, con el fin de que estos eventos imprevistos no afecten negativamente a la calidad. | 
| Optimización de Wi-Fi | De forma similar a la VPN, las redes Wi-Fi no están necesariamente diseñadas o configuradas para admitir elementos multimedia en tiempo real. El planeamiento, o la optimización, de una red Wi-Fi para que admita Azure Communication Services es una consideración importante para una implementación de alta calidad. Tenga en cuenta estos factores: <ul><li>Implemente QoS o Wi-Fi Multimedia (WMM) para asegurarse de que el tráfico de elementos multimedia se prioriza correctamente en las redes Wi-Fi.</li><li>Planee y optimice las bandas de Wi-Fi y la ubicación de los puntos de acceso. El intervalo de 2,4 GHz puede proporcionarle una experiencia adecuada en función de la ubicación de los puntos de acceso, pero a menudo estos acceso suelen verse afectados por otros dispositivos que operan en ese intervalo. El intervalo de 5 GHz es más adecuado para los elementos multimedia en tiempo real debido a su denso rango, pero requiere más puntos de acceso para obtener una cobertura suficiente. Los puntos de conexión también necesitan admitir ese intervalo y configurarse para aprovechar esas bandas en consecuencia.</li><li>Si usa redes Wi-Fi de banda dual, considere la posibilidad de implementar el direccionamiento de banda. El direccionamiento de banda es una técnica que implementan los proveedores de Wi-Fi para que los clientes de doble banda usen usar el intervalo de 5 GHz.</li><li>Cuando los puntos de acceso del mismo canal están demasiado próximos, pueden provocar la superposición de la señal y competir involuntariamente, lo que empeora la experiencia del usuario. Asegúrese de que los puntos de acceso que están próximos entre sí se encuentran en canales que no se superponen.</li></ul> Cada proveedor tiene sus propias recomendaciones para implementar su solución inalámbrica. Póngase en contacto con el proveedor de la red Wi-Fi para obtener instrucciones específicas.|



### <a name="operating-system-and-browsers-for-javascript-sdks"></a>Sistema operativo y exploradores (para los SDK de JavaScript)

Los SDK de voz y vídeo de Azure Communication Services admiten determinados sistemas operativos y exploradores.
Obtenga información sobre los sistemas operativos y los exploradores que admiten los SDK de llamada en la [documentación conceptual de llamada](./calling-sdk-features.md).

## <a name="next-steps"></a>Pasos siguientes

Puede que los siguientes documentos le resulten interesantes:

- Más información sobre [las bibliotecas de llamadas](./calling-sdk-features.md)
- Información sobre la [arquitectura de cliente y servidor](../client-and-server-architecture.md)
- Información sobre las [topologías de flujo de llamadas](../call-flows.md)