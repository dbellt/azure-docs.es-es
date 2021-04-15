---
title: Asociación con Azure DDoS Protection estándar
description: Conozca las oportunidades de asociación habilitadas por Azure DDoS Protection estándar.
ms.service: ddos-protection
documentationcenter: na
author: aletheatoh
mms.devlang: na
ms.topic: how-to
ms.date: 08/28/2020
ms.author: kumud
ms.openlocfilehash: 625f77a6d9dbeaf612df884434ab39b943587791
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103205"
---
# <a name="partnering-with-azure-ddos-protection-standard"></a>Asociación con Azure DDoS Protection estándar
En este artículo se describen las oportunidades de asociación habilitadas por Azure DDoS Protection estándar. Este artículo está diseñado para ayudar a los administradores de productos y a los roles de desarrollo empresarial a comprender las rutas de inversión, y para ofrecer información sobre las propuestas de valor de la asociación.

## <a name="background"></a>Información previa
Los ataques por denegación de servicio distribuido (DDoS) son uno de los principales problemas de seguridad y disponibilidad que notifican los clientes que mueven sus aplicaciones a la nube. La extorsión y el hacktivismo son motivaciones comunes de los ataques DDoS, que han aumentado de forma constante en el tipo, la escala y la frecuencia de repetición, ya que cometerlos resulta relativamente fácil y económico.

Azure DDoS Protection ofrece técnicas defensivas contra las amenazas de DDoS más sofisticadas, en las que usa la escala global de Redes de Azure. El servicio proporciona funcionalidades mejoradas de mitigación de DDoS para las aplicaciones y los recursos implementados en redes virtuales.

Los asociados tecnológicos pueden proteger los recursos de sus clientes de forma nativa con Azure DDoS Protection estándar para abordar los problemas de disponibilidad y confiabilidad debido a ataques DDoS.

## <a name="introduction-to-azure-ddos-protection-standard"></a>Introducción a Azure DDoS Protection estándar
Azure DDoS Protection estándar proporciona funcionalidades mejoradas de mitigación de DDoS contra ataques DDoS de nivel 3 y 4. A continuación se describen las características clave del servicio DDoS Protection estándar.

### <a name="adaptive-real-time-tuning"></a>Ajuste adaptable en tiempo real
Para cada aplicación protegida, Azure DDoS Protection estándar ajusta automáticamente los umbrales de la directiva de mitigación de DDoS en función de los patrones de perfil de tráfico de la aplicación. El servicio consigue esta personalización basándose en dos tipos de información:

- Aprendizaje automático de los patrones de tráfico de los niveles 3 y 4 (por IP) de cada cliente.
- Reducción de los falsos positivos ya que el escalado de Azure le permite absorber una cantidad importante de tráfico.

![Ajuste adaptable en tiempo real](./media/ddos-protection-partner-onboarding/real-time-tuning.png)

### <a name="attack-analytics-telemetry-monitoring-and-alerting"></a>Análisis, telemetría, supervisión y alertas de los ataques
Azure DDoS Protection identifica y mitiga los ataques DDoS sin intervención del usuario.

- Si el recurso protegido está en la suscripción que se trata en Azure Security Center, DDoS Protection estándar envía automáticamente una alerta a Security Center cada vez que se detecta y mitiga un ataque DDoS en la aplicación protegida.
- De forma alternativa, para recibir una notificación cuando haya una mitigación activa en una dirección IP pública protegida, puede [configurar una alerta](alerts.md) en la métrica Under DDoS attack or not (Bajo ataque de DDoS o no).
- También puede optar por crear alertas para las otras métricas de DDoS y [configurar la telemetría de ataques](telemetry.md) para entender la magnitud del ataque, el tráfico que se va a quitar, los vectores de ataque, los principales colaboradores, etc.

![Métricas de DDoS](./media/ddos-protection-partner-onboarding/ddos-metrics.png)

### <a name="ddos-rapid-response-drr"></a>Respuesta rápida de DDoS (DRR)
Los clientes de DDoS Protection estándar ahora tienen acceso al [equipo de Rapid Response](ddos-rapid-response.md) durante un ataque activo. DDR puede ayudar con investigación de ataques cuando se producen, así como con el análisis posterior a estos.

### <a name="sla-guarantee-and-cost-protection"></a>Protección de costos y garantía del contrato de nivel de servicio
El servicio DDoS Protection estándar está cubierto por un contrato de nivel de servicio del 99,99 % y la protección de costos proporciona créditos de recursos para el escalado horizontal durante un ataque documentado. Para más información, consulte [Contrato de nivel de servicio para Azure DDoS Protection](https://azure.microsoft.com/support/legal/sla/ddos-protection/v1_0/).

## <a name="featured-partner-scenarios"></a>Escenarios de asociación destacados
A continuación se enumeran las principales ventajas que se pueden derivar mediante la integración con Azure DDoS Protection estándar:

- Los servicios ofrecidos por los asociados (equilibrador de carga, firewall de aplicaciones web, firewall, etc.) a sus clientes se protegen automáticamente (etiqueta blanca) con Azure DDoS Protection estándar en el back-end.
- Los asociados tienen acceso a datos de telemetría y análisis de ataques de Azure DDoS Protection que pueden integrar con sus propios productos para ofrecer una experiencia de cliente unificada.  
- Los asociados tienen acceso al soporte técnico de respuesta rápida de DDoS, incluso en ausencia de Azure Rapid Response, para problemas relacionados con DDoS.
- Las aplicaciones protegidas de los asociados están respaldados por la protección de costos y la garantía de un contrato de nivel de servicio de DDoS en caso de producirse ataques DDoS.

## <a name="technical-integration-overview"></a>Información general sobre la integración técnica
Las oportunidades de asociación de Azure DDoS Protection estándar están disponibles a través de Azure Portal, de las API y de la CLI/PS.

### <a name="integrate-with-ddos-protection-standard"></a>Integración con DDoS Protection estándar
Los siguientes pasos son necesarios para que los asociados configuren la integración con Azure DDoS Protection estándar:
1. Cree un plan de DDoS Protection en la suscripción (de asociado) que desee. Para obtener instrucciones paso a paso, consulte el tema sobre la [creación de un plan de DDoS Protection estándar](manage-ddos-protection.md#create-a-ddos-protection-plan).
   > [!NOTE]
   > Solo se debe crear un plan de DDoS Protection para un inquilino determinado. 
2. Implemente un servicio con un punto de conexión público en sus suscripciones (de asociado), como el equilibrador de carga, los firewalls y el firewall de aplicaciones web. 
3. Habilite Azure DDoS Protection estándar en la red virtual del servicio que tiene puntos de conexión públicos mediante el plan de DDoS Protection creado en el primer paso. Para obtener instrucciones detalladas, consulte el tema sobre la [habilitación del plan de DDoS Protection estándar](manage-ddos-protection.md#enable-ddos-protection-for-an-existing-virtual-network).
   > [!IMPORTANT] 
   > Una vez habilitado Azure DDoS Protection estándar en una red virtual, todas las direcciones IP públicas de esa red virtual se protegen automáticamente. El origen de estas direcciones IP públicas puede estar en Azure (suscripción de cliente) o fuera de Azure. 
4. Opcionalmente, integre la telemetría y los análisis de ataques de Azure DDoS Protection estándar en el panel orientado al cliente específico de la aplicación. Para obtener más información sobre el uso de la telemetría, consulte [Visualización y configuración de la telemetría de DDoS Protection](telemetry.md). 

### <a name="onboarding-guides-and-technical-documentation"></a>Guías de incorporación y documentación técnica

- [Página del producto Azure DDoS Protection](https://azure.microsoft.com/services/ddos-protection/)
- [Documentación de Azure DDoS Protection](ddos-protection-overview.md)
- [Referencia de la API de Azure DDoS Protection](/rest/api/virtualnetwork/ddosprotectionplans)
- [Referencia de la API de Azure Virtual Network](/rest/api/virtualnetwork/virtualnetworks)

### <a name="get-help"></a>Obtener ayuda

- Si tiene alguna pregunta sobre las integraciones de productos, aplicaciones o servicios con Azure DDoS Protection estándar, puede ponerse en contacto con la [comunidad de seguridad de Azure](https://techcommunity.microsoft.com/t5/security-identity/bd-p/Azure-Security).
- Siga las discusiones sobre [Stack Overflow](https://stackoverflow.com/tags/azure-ddos/).

### <a name="get-to-market"></a>Comercialización de productos

- El programa principal para la asociación con Microsoft es [Microsoft Partner Network](https://partner.microsoft.com/). – Las integraciones de seguridad de Microsoft Graph se incluyen en el seguimiento de [fabricante de software independiente (ISV) de MPN](https://partner.microsoft.com/saas-solution-guide).
- [Asociación de seguridad inteligente de Microsoft](https://www.microsoft.com/security/business/intelligent-security-association?rtc=1) es el programa específico para que los asociados de seguridad de Microsoft le ayuden a enriquecer sus productos de seguridad y a mejorar la capacidad de detección de los clientes de sus integraciones con productos de seguridad de Microsoft.

## <a name="next-steps"></a>Pasos siguientes
Ver las integraciones de asociados existentes:

- [Barracuda WAF-as-a-Service](https://www.barracuda.com/waf-as-a-service)
- [Azure Cloud WAF de Radware](https://www.radware.com/resources/microsoft-azure/)
