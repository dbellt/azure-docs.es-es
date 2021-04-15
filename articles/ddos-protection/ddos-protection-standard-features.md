---
title: Características de Azure DDoS Protection
description: Más información sobre las características de Azure DDoS Protection.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 490a65c7d20194956350166d23e2d3450b94b12e
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103103"
---
# <a name="azure-ddos-protection-standard-features"></a>Características de Azure DDoS Protection Estándar

Las siguientes secciones describen las características clave del servicio Azure DDoS Protection Standard.

## <a name="always-on-traffic-monitoring"></a>Supervisión continua del tráfico

El servicio Protección contra DDoS estándar supervisa el uso de tráfico real y lo compara constantemente con los umbrales definidos en la directiva de DDoS. Cuando se supera el umbral de tráfico, se inicia automáticamente la mitigación de DDoS. Cuando el tráfico vuelve a estar por debajo del umbral, se detiene la mitigación.

![Mitigación mediante Azure DDoS Protection Estándar](./media/ddos-protection-overview/mitigation.png)

Durante la mitigación, el servicio Protección contra DDoS redirige el tráfico enviado al recurso protegido y realiza varias comprobaciones, por ejemplo:

- Asegurarse de que los paquetes se ajustan a las especificaciones de Internet y no tienen un formato incorrecto.
- Interactúe con el cliente para determinar si el tráfico es un posible paquete falsificado (por ejemplo, Autorización de SYN o Cookie de SYN o colocando un paquete para que el origen lo retransmita).
- Limitar la velocidad de los paquetes si no se puede realizar ningún otro método de cumplimiento.

El servicio Protección contra DDoS elimina el tráfico del ataque y reenvía el tráfico restante al destino previsto. En un intervalo de pocos minutos tras la detección del ataque, recibe una notificación mediante las métricas de Azure Monitor. Si configura el registro de datos de telemetría de Protección contra DDoS estándar, puede escribir los registros en opciones disponibles para su posterior análisis. Los datos métricos de Azure Monitor para Protección contra DDoS estándar se conservan actualmente durante 30 días.

## <a name="adaptive-real-time-tuning"></a>Ajuste adaptable en tiempo real

El servicio Azure DDoS Protection Basic le ayuda a proteger a los clientes y a evitar que los ataques afecten a otros clientes. Por ejemplo, si un servicio se aprovisiona para un volumen normal de tráfico de entrada legítimo que es menor que la *tasa desencadenante* de la directiva de DDoS Protection para toda la infraestructura, un ataque de DDoS en los recursos de ese cliente podría pasar desapercibido. Por lo general, la complejidad de los ataques recientes (por ejemplo, DDoS multivector), así como los comportamientos de los inquilinos, específicos de cada aplicación, hacen necesarias directivas de protección personalizadas para cada cliente. El servicio lo lleva a cabo mediante el uso de dos tipos de información:

- Aprendizaje automático de los patrones de tráfico de las capas 3 y 4 de cada cliente (por dirección IP pública).

- Reducción de los falsos positivos ya que el escalado de Azure le permite absorber una cantidad importante de tráfico.

![Diagrama de funcionamiento de DDoS Protection Standard, con "Generación de directiva" rodeada por un círculo](./media/ddos-best-practices/image-5.png)

## <a name="ddos-protection-telemetry-monitoring-and-alerting"></a>Telemetría, alertas y supervisión de DDoS Protection

DDoS Protection Estándar expone datos de telemetría detallados mediante [Azure Monitor](../azure-monitor/overview.md). También puede configurar alertas para todas las métricas de Azure Monitor que usa DDoS Protection. El registro se puede integrar con Splunk (Azure Event Hubs), registros de Azure Monitor y Azure Storage para realizar análisis avanzados con la interfaz de Azure Monitor Diagnostics.

### <a name="ddos-mitigation-policies"></a>Directivas de mitigación de DDoS

En Azure Portal, seleccione **Monitor** > **Métricas**. En el panel **Métricas**, seleccione el grupo de recursos, un tipo de recurso de **Dirección IP pública** y la dirección IP pública de Azure. Las métricas de DDoS estarán visibles en el panel **Métricas disponibles**.

DDoS Protection Standard aplica tres directivas de mitigación de ajuste automático (TCP SYN, TCP y UDP) a cada dirección IP pública del recurso protegido, en la red virtual que tiene habilitado DDoS. Para ver los umbrales de la directiva, seleccione la métrica "**Inbound packets to trigger DDoS mitigation**" (Paquetes de entrada para desencadenar la mitigación de DDoS).

![Gráficos de métricas y de métricas disponibles](./media/ddos-best-practices/image-7.png)

Los umbrales de las directivas se configuran automáticamente con el sistema de generación de perfiles de tráfico de red basado en aprendizaje automático. La mitigación de DDoS se produce para una dirección IP que está siendo atacada solo cuando se supera el umbral de la directiva.

### <a name="metric-for-an-ip-address-under-ddos-attack"></a>Métricas de una dirección IP sometida a un ataque de DDoS

Si la dirección IP pública está siendo atacada, el valor de la métrica **Under DDoS attack or not** (Bajo ataque de DDoS o no) cambia a 1 cuando aplicamos la mitigación al tráfico de ataque.

![Métrica y gráfico "Under DDoS attack or not" (Bajo ataque de DDoS o no)](./media/ddos-best-practices/image-8.png)

Se recomienda configurar una alerta en esta métrica. Si lo hace, recibirá una notificación cuando se esté aplicando una mitigación de DDoS en su dirección IP pública.

Para más información, consulte [Administración de Azure DDoS Protection Standard mediante Azure Portal](manage-ddos-protection.md).

## <a name="web-application-firewall-for-resource-attacks"></a>Firewall de aplicaciones web para ataques a recursos

En el caso de los ataques a recursos en el nivel de aplicación, los clientes deben configurar un firewall de aplicaciones web (WAF) para ayudar a proteger las aplicaciones web. Un WAF inspecciona el tráfico web entrante para bloquear las inyecciones de SQL, los scripts entre sitios, DDoS y otros ataques al nivel 7. Azure ofrece [WAF como una característica de Application Gateway](../web-application-firewall/ag/ag-overview.md) para ofrecer una protección centralizada de las aplicaciones web contra las vulnerabilidades de seguridad comunes. En [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?search=WAF&page=1) encontrará ofertas de WAF de asociados de Azure que podrían ser más adecuadas para sus necesidades.

Incluso los firewalls de aplicaciones web son susceptibles de sufrir ataques de agotamiento del estado y volumétricos. Es muy recomendable activar DDoS Protection Standard en la red virtual del WAF para impedir los ataques a protocolos y volumétricos. Para más información, consulte la sección [Arquitecturas de referencia de DDoS Protection](ddos-protection-reference-architectures.md).

## <a name="protection-planning"></a>Planeamiento de la protección

El planeamiento y la preparación son cruciales para entender cómo se comportará un sistema durante un ataque de DDoS. Diseñar un plan de respuesta de administración de incidentes forma parte de este esfuerzo.

Si dispone de DDoS Protection Standard, asegúrese de que esté habilitado en la red virtual de los puntos de conexión accesibles desde Internet. La configuración de alertas de DDoS le ayuda a vigilar constantemente los posibles ataques contra su infraestructura. 

Supervise las aplicaciones de forma independiente. Debe conocer el comportamiento normal de una aplicación. Debe estar preparado para actuar si la aplicación no se comporta como está previsto durante un ataque de DDoS. 

Obtenga información sobre cómo los servicios responderán a un ataque a través de las [pruebas mediante simulaciones](test-through-simulations.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un plan de protección contra DDoS](manage-ddos-protection.md).
