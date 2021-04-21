---
author: memildin
ms.author: memildin
manager: rkarlin
ms.date: 04/07/2021
ms.topic: include
ms.openlocfilehash: 73886b966676af75cc74484ccdc0632f080b041a
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029157"
---
Azure Defender proporciona protección contra amenazas en tiempo real para los entornos en contenedores y genera alertas de actividades sospechosas. Puede usar esta información para corregir problemas de seguridad y mejorar la seguridad de los contenedores rápidamente.

Azure Defender proporciona protección contra amenazas en diferentes niveles: 

* **Nivel de host (proporcionado por Azure Defender para servidores)** : con el mismo agente de Log Analytics que Security Center usa en otras máquinas virtuales, Azure Defender supervisa los nodos de Kubernetes en Linux en busca de actividades sospechosas, como la detección de shell web y la vinculación con direcciones IP sospechosas conocidas. El agente también supervisa análisis específicos de contenedores, como la creación de contenedores con privilegios, las actividades sospechosas de acceso a los servidores de API y los servidores de Secure Shell (SSH) que se ejecutan dentro de un contenedor de Docker.

    Si decide no instalar los agentes en los hosts, solo recibirá un subconjunto de las ventajas de la protección contra amenazas y las alertas de seguridad. Aún así, seguirá recibiendo alertas relacionadas con el análisis de redes y las comunicaciones con servidores malintencionados.

    >[!IMPORTANT]
    > Actualmente no se admite la instalación del agente de Log Analytics en los clústeres de Azure Kubernetes Service que se ejecutan en conjuntos de escalado de máquinas virtuales.

    Para obtener una lista de las alertas del nivel de clúster, consulte la [tabla de referencia de alertas](../articles/security-center/alerts-reference.md#alerts-containerhost).


* **Nivel de clúster (proporcionado por Azure Defender para Kubernetes)** : en el nivel de clúster, la protección contra amenazas consiste en analizar los registros de auditoría de Kubernetes. Para habilitar esta supervisión **sin agente**, habilite Azure Defender. Si su clúster es local o reside en otro proveedor de nube, utilice [Kubernetes habilitado para Arc y la extensión de Azure Defender](../articles/security-center/defender-for-kubernetes-azure-arc.md).

    Para generar alertas en este nivel, Azure Defender supervisa los registros de los clústeres. Entre los ejemplos de eventos en este nivel se incluyen los paneles de Kubernetes expuestos y la creación de roles con privilegios elevados y de montajes confidenciales.

    >[!NOTE]
    > Azure Defender genera alertas de seguridad para las acciones e implementaciones que se producen después de habilitar el plan de Defender para Kubernetes en la suscripción. 

    Para obtener una lista de las alertas del nivel de clúster de AKS, consulte la [tabla de referencia de alertas](../articles/security-center/alerts-reference.md#alerts-akscluster).

Además, nuestro equipo global de investigadores de seguridad supervisa constantemente el panorama de las amenazas. Agregan alertas específicas del contenedor y vulnerabilidades a medida que se detectan.

> [!TIP]
> Puede simular las alertas de contenedor al seguir las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).