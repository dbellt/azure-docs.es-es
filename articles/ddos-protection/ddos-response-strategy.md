---
title: Componentes de una estrategia de respuesta a DDoS
description: Aprenda a usar Azure DDoS Protection Estándar para responder a ataques DDoS.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: c028185a0753bc4da966ebc56bb792e57cac1f77
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103052"
---
# <a name="components-of-a-ddos-response-strategy"></a>Componentes de una estrategia de respuesta a DDoS

Un ataque DDoS destinado a recursos de Azure normalmente requiere una intervención mínima desde la perspectiva del usuario. No obstante, incorporar la mitigación de DDoS como parte de la estrategia de respuesta a incidentes ayuda a minimizar el impacto en la continuidad del negocio.

## <a name="microsoft-threat-intelligence"></a>Información sobre amenazas de Microsoft

Microsoft cuenta con una extensa red de información sobre amenazas. Esta red usa los conocimientos colectivos de una amplia comunidad de seguridad para dar soporte a los servicios en línea de Microsoft, los asociados de Microsoft y las relaciones dentro de la comunidad de seguridad de Internet. 

Como importante proveedor de infraestructura, Microsoft recibe alertas tempranas acerca de las amenazas. Microsoft recopila la información sobre amenazas obtenida de otros servicios en línea y de la base internacional de clientes. Microsoft incorpora toda esta información sobre amenazas a los productos de Azure DDoS Protection.

Además, Microsoft Digital Crimes Unit (DCU) adopta estrategias ofensivas frente a las redes de robots. Las redes de robots son una causa común de comando y control de los ataques de DDoS.

## <a name="risk-evaluation-of-your-azure-resources"></a>Evaluación del riesgo de los recursos de Azure

Es fundamental conocer el alcance del riesgo de un ataque de DDoS de forma continuada. Periódicamente, pregúntese lo siguiente:

- ¿Qué nuevos recursos de Azure que están públicamente disponibles necesitan protección?

- ¿Hay un único punto de error en el servicio? 

- ¿Cómo pueden aislarse los servicios para limitar el impacto de un ataque, manteniendo al mismo tiempo los servicios disponibles para los clientes legítimos?

- ¿Hay redes virtuales en las que DDoS Protection Standard debería estar habilitado y no lo está? 

- ¿Estás mis servicios activos o activos con conmutación por error en varias regiones?

Es fundamental que comprenda el comportamiento normal de una aplicación y se prepare para actuar si la aplicación no se comporta de la manera esperada durante un ataque DDoS. Tenga monitores configurados para las aplicaciones críticas para la empresa que imitan el comportamiento del cliente y le notifican cuando se detectan anomalías relevantes. Consulte [Procedimientos recomendados de supervisión y diagnósticos](/azure/architecture/best-practices/monitoring#monitoring-and-diagnostics-scenarios) para obtener información sobre el estado de la aplicación.

[Azure Application Insights](../azure-monitor/app/app-insights-overview.md) es un servicio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas. Use Application Insights para supervisar la aplicación web en vivo. Se detectan automáticamente las anomalías de rendimiento. Incluye herramientas de análisis que le ayudan a diagnosticar problemas y a comprender lo que hacen los usuarios con la aplicación. Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso.

## <a name="customer-ddos-response-team"></a>Equipo de respuesta a DDoS del cliente

Crear un equipo de respuesta a DDoS es un paso clave para garantizar una respuesta rápida y eficaz a un ataque. Debe identificar los contactos de la organización que supervisarán el planeamiento y la ejecución. Este equipo de respuesta de DDoS debe conocer perfectamente el servicio Azure DDoS Protection Standard. Asegúrese de que el equipo puede identificar y mitigar un ataque en coordinación con clientes internos y externos, incluido el equipo de soporte técnico de Microsoft. 

Es recomendable que utilice ejercicios de simulación como parte normal de la disponibilidad del servicio y el planeamiento de la continuidad. Estos ejercicios deben incluir pruebas de escalado. Consulte [Pruebas mediante simulaciones](test-through-simulations.md) para aprender a simular el tráfico de prueba de DDoS en los puntos de conexión públicos de Azure.

## <a name="alerts-during-an-attack"></a>Alertas durante un ataque

Azure DDoS Protection Standard identifica y mitiga los ataques de DDoS sin intervención del usuario. Para recibir una notificación cuando haya una mitigación activa en una dirección IP pública protegida, puede [configurar alertas](alerts.md).

### <a name="when-to-contact-microsoft-support"></a>Cuándo ponerse en contacto con el soporte técnico de Microsoft

Los clientes de Azure DDoS Protection Estándar tienen acceso al equipo de Respuesta rápida de DDoS (DRR), que puede ayudar con la investigación de ataques durante un ataque, así como con el análisis posterior al ataque. Consulte [Respuesta rápida de DDoS](ddos-rapid-response.md) para obtener más información, incluido cuándo debe ponerse en contacto con el equipo de DRR.

## <a name="post-attack-steps"></a>Pasos posteriores al ataque

Siempre es una buena estrategia hacer una autopsia después de un ataque y ajustar la estrategia de respuesta a DDoS según sea necesario. Puntos que se deben tener en cuenta:

- ¿Se produjo una interrupción en el servicio o en la experiencia del usuario debido a la falta de una arquitectura escalable?

- ¿Qué aplicaciones o servicios sufrieron más?

- ¿Fue eficaz la estrategia de respuesta a DDoS y cómo se podría mejorar aún más?

Si sospecha que está sufriendo un ataque de DDoS, notifíquelo a través de los canales de soporte técnico de Azure habituales.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un plan de protección contra DDoS](manage-ddos-protection.md).