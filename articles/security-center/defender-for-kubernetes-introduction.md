---
title: 'Azure Defender para Kubernetes: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para Kubernetes.
author: memildin
ms.author: memildin
ms.date: 04/07/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: acf038d6e1ead94bdfb2e00ffe225e566faa6c44
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854907"
---
# <a name="introduction-to-azure-defender-for-kubernetes"></a>Introducción a Azure Defender para Kubernetes

Azure Defender para Kubernetes es el plan de Azure Defender que proporciona protecciones para los clústeres de Kubernetes dondequiera que se ejecuten. 

Podemos proteger los clústeres en:

- **Azure Kubernetes Service (AKS)** : servicio administrado por Microsoft para desarrollar, implementar y administrar aplicaciones en contenedores.

- **Entornos locales y de varias nubes:** mediante una [extensión para Kubernetes habilitado para Arc](defender-for-kubernetes-azure-arc.md)

Azure Security Center y AKS forman una oferta de seguridad de Kubernetes nativa en la nube de protección del entorno, protección de la carga de trabajo y protección en tiempo de ejecución, tal y como se describe en [Seguridad de los contenedores en Security Center](container-security.md).

La detección de amenazas a nivel de host en los nodos de AKS de Linux está disponible si habilita [Azure Defender para servidores](defender-for-servers-introduction.md) y su agente de Log Analytics. Sin embargo, si el clúster se implementa en un conjunto de escalado de máquinas virtuales, el agente de Log Analytics no se admite actualmente.



## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|**Azure Defender para Kubernetes** se factura como se indica en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).|
|Roles y permisos necesarios:|**Administración de seguridad** puede descartar las alertas.<br>**El lector de seguridad** puede ver los resultados.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-kubernetes"></a>¿Cuáles son las ventajas de Azure Defender para Kubernetes?

Azure Defender para Kubernetes proporciona **protección frente a amenazas en el nivel de clúster** mediante la supervisión de los registros del clúster.

Entre los ejemplos de eventos de seguridad que Azure Defender para Kubernetes supervisa se incluyen los paneles de Kubernetes expuestos y la creación de roles con privilegios elevados y de montajes confidenciales. Para la lista completa de las alertas de nivel de clúster, consulte la [tabla de referencia de alertas](alerts-reference.md#alerts-k8scluster).

> [!TIP]
> Puede simular las alertas de contenedor al seguir las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).

Además, nuestro equipo global de investigadores de seguridad supervisa constantemente el panorama de las amenazas. Agregan alertas específicas del contenedor y vulnerabilidades a medida que se detectan.

>[!NOTE]
> Azure Defender genera alertas de seguridad para las acciones e implementaciones que se producen después de habilitar el plan de Defender para Kubernetes en la suscripción.




## <a name="faq---azure-defender-for-kubernetes"></a>Preguntas frecuentes: Azure Defender para Kubernetes

### <a name="can-i-still-get-cluster-protections-without-the-log-analytics-agent"></a>¿Puedo obtener protecciones del clúster aún sin el agente de Log Analytics?

El plan de **Azure Defender para Kubernetes** proporciona protecciones a nivel de clúster. Si también implementa el agente de Log Analytics de **Azure Defender para servidores**, obtendrá protección contra amenazas para los nodos que se proporcionan con ese plan. Más información en [Introducción a Azure Defender para servidores](defender-for-servers-introduction.md).

Se recomienda implementar ambos para obtener la protección más completa posible.

Si decide no instalar el agente en los hosts, solo recibirá un subconjunto de las ventajas de la protección contra amenazas y las alertas de seguridad. Aún así, seguirá recibiendo alertas relacionadas con el análisis de redes y las comunicaciones con servidores malintencionados.

### <a name="does-aks-allow-me-to-install-custom-vm-extensions-on-my-aks-nodes"></a>¿Me permite AKS instalar extensiones de máquina virtual personalizadas en mis nodos de AKS?
Para que Azure Defender supervise sus nodos de AKS, deben ejecutar el agente de Log Analytics. 

AKS es un servicio administrado y, como el agente de Log Analytics es una extensión administrada por Microsoft, también es compatible con los clústeres de AKS.

### <a name="if-my-cluster-is-already-running-an-azure-monitor-for-containers-agent-do-i-need-the-log-analytics-agent-too"></a>Si el clúster ya está ejecutando un agente de Azure Monitor para contenedores, ¿necesito también el agente de Log Analytics?
Para que Azure Defender supervise los nodos, estos deben ejecutar el agente de Log Analytics.

Si los clústeres ya ejecutan el agente de Azure Monitor para contenedores, puede instalar el agente de Log Analytics y los dos agentes pueden trabajar junto con otros sin problemas.

[Más información sobre el agente de Azure Monitor para contenedores](../azure-monitor/containers/container-insights-manage-agent.md).


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre la protección de Kubernetes de Security Center, incluido Azure Defender para Kubernetes. 

> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](enable-azure-defender.md)

Para obtener material relacionado, consulte los siguientes artículos: 

- [Transmisión de alertas a una solución de administración de servicios de TI, SIEM o SOAR](export-to-siem.md)
- [Tabla de referencia de alertas](alerts-reference.md)
