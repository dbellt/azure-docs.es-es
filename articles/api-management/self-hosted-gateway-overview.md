---
title: Introducción a la puerta de enlace autohospedada | Microsoft Docs
description: Aprenda cómo la característica de puerta de enlace autohospedada de Azure API Management ayuda a las organizaciones a administrar las API en entornos híbridos y multinube.
services: api-management
documentationcenter: ''
author: vlvinogr
manager: gwallace
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 05/25/2021
ms.author: apimpm
ms.openlocfilehash: 946a2707e68e2f3ce6f78df997d102eeb0c7c450
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376976"
---
# <a name="self-hosted-gateway-overview"></a>Introducción a la puerta de enlace autohospedada

En este artículo se explica cómo la característica de puerta de enlace autohospedada de Azure API Management habilita la administración de API híbrida y multinube, presenta su arquitectura de alto nivel y resalta sus capacidades.

## <a name="hybrid-and-multi-cloud-api-management"></a>Administración de API híbrida y multinube

La característica de puerta de enlace autohospedada amplía la compatibilidad de API Management con entornos híbridos y de varias nubes y permite a las organizaciones administrar de forma eficaz y segura las API hospedadas localmente y en diferentes nubes desde un único servicio de API Management en Azure.

Con la puerta de enlace autohospedada, los clientes tienen la flexibilidad de implementar una versión en contenedor del componente de puerta de enlace de API Management en los mismos entornos donde hospedan sus API. Todas las puertas de enlace autohospedadas se administran desde el servicio de API Management con el que están federadas, lo que proporciona a los clientes visibilidad y una experiencia de administración unificada en todas las API internas y externas. La colocación de las puertas de enlace cerca de las API permite a los clientes optimizar los flujos de tráfico de la API y abordar los requisitos de seguridad y cumplimiento.

Cada servicio de API Management consta de los siguientes componentes clave:

-   Plano de administración, expuesto como una API, que se usa para configurar el servicio mediante Azure Portal, PowerShell y otros mecanismos compatibles.
-   La puerta de enlace (o el plano de datos) es responsable de redirigir mediante proxy las solicitudes de API y también de la aplicación de directivas y la recopilación de datos de telemetría
-   Portal para desarrolladores que les permite descubrir, conocer y usar las API

De forma predeterminada, todos estos componentes se implementan en Azure, lo que hace que todo el tráfico de la API (que se muestra como flechas negras en la siguiente imagen) fluya a través de Azure, independientemente de dónde se hospeden los back-ends que implementan las API. La simplicidad operativa de este modelo se traduce en un aumento de latencia, problemas de cumplimiento y, en algunos casos, cargos adicionales de transferencia de datos.

![Flujo de tráfico de API sin puertas de enlace autohospedadas](media/self-hosted-gateway-overview/without-gateways.png)

La implementación de puertas de enlace autohospedadas en los mismos entornos en que las implementaciones de API de back-end se hospedan permiten que el tráfico de la API fluya directamente a las API de back-end, lo que mejora la latencia, optimiza los costos de transferencia de datos y facilita el cumplimiento, a la vez que conserva las ventajas de tener un único punto de administración y detección de todas las API dentro de la organización, independientemente de dónde se hospeden sus implementaciones.

![Flujo de tráfico de API con puertas de enlace autohospedadas](media/self-hosted-gateway-overview/with-gateways.png)

## <a name="packaging-and-features"></a>Empaquetado y características

La puerta de enlace autohospedada es una versión en contenedor, funcionalmente equivalente, de la puerta de enlace administrada implementada en Azure como parte de cada servicio de API Management. La puerta de enlace autohospedada está disponible como un [contenedor](https://aka.ms/apim/sputnik/dhub) de Docker basado en Linux de Microsoft Container Registry. Se puede implementar en Docker, Kubernetes o en cualquier otra solución de orquestación de contenedores que se ejecute en un clúster de servidores local, una infraestructura en la nube o, con fines de evaluación y desarrollo, en un equipo personal. También puede implementar la puerta de enlace autohospedada como una extensión de clúster en un [clúster de Kubernetes habilitado para Azure Arc](./how-to-deploy-self-hosted-gateway-azure-arc.md).

Las siguientes funcionalidades de las puertas de enlace administradas **no están disponibles** en las puertas de enlace autohospedadas:

- Registros de Azure Monitor
- Administración de cifrado y de la versión de TLS ascendente (back-end)
- La validación de los certificados de servidor y cliente mediante [certificados raíz de CA](api-management-howto-ca-certificates.md) cargados en el servicio de API Management. Para más información, consulte [Validación de certificados en una puerta de enlace autohospedada](api-management-howto-mutual-certificates-for-clients.md#certificate-validation-in-self-hosted-gateway).
- Integración con [Service Fabric](../service-fabric/service-fabric-api-management-overview.md).
- Reanudación de la sesión TLS.
- Renegociación del certificado de cliente. Esto significa que, para que la [autenticación de certificados de cliente](api-management-howto-mutual-certificates-for-clients.md) funcione, los consumidores de la API de trabajo deben presentar sus certificados como parte del protocolo de enlace TLS inicial. Para garantizar este escenario, habilite la configuración Negociar certificado de cliente cuando configure un nombre de host personalizado de puerta de enlace autohospedada.
- Memoria caché integrada. Consulte este [documento](api-management-howto-cache-external.md) para aprender a usar la memoria caché externa en las puertas de enlace autohospedadas.

## <a name="connectivity-to-azure"></a>Conectividad a Azure

Las puertas de enlace autohospedadas requieren conectividad TCP/IP saliente a Azure en el puerto 443. Cada puerta de enlace autohospedada debe estar asociada a un único servicio de API Management y configurada a través de su plano de administración. La puerta de enlace autohospedada usa la conectividad a Azure para:

-   Informar de su estado mediante el envío de latidos cada minuto
-   Comprobar regularmente (cada 10 segundos) y aplicar actualizaciones de configuración siempre que estén disponibles
-   Enviar registros y métricas de solicitudes a Azure Monitor, si se configura para ello
-   Enviar eventos a Application Insights, si se establece para ello

Cuando se pierde la conectividad con Azure, la puerta de enlace autohospedada no podrá recibir actualizaciones de configuración, notificar su estado o cargar los datos de telemetría.

La puerta de enlace autohospedada está diseñada para "suspender la estática" y puede sobrevivir a la pérdida temporal de conectividad con Azure. Se puede implementar con o sin la copia de seguridad de configuración local. En el primer caso, las puertas de enlace autohospedadas guardarán con frecuencia una copia de seguridad de la configuración descargada más reciente en un volumen persistente conectado al contenedor o pod.

Cuando se desactiva la copia de seguridad de la configuración y se interrumpe la conectividad a Azure:

-   Las puertas de enlace autohospedadas en ejecución seguirán funcionando con una copia en memoria de la configuración
-   Las puertas de enlace autohospedadas detenidas no podrán iniciarse

Cuando se activa la copia de seguridad de la configuración y se interrumpe la conectividad a Azure:

-   Las puertas de enlace autohospedadas en ejecución seguirán funcionando con una copia en memoria de la configuración
-   Las puertas de enlace autohospedadas detenidas podrán comenzar a usar una copia de seguridad de la configuración

Cuando se restaura la conectividad, cada puerta de enlace autohospedada afectada por la interrupción se volverá a conectar automáticamente con su servicio de API Management asociado y descargará todas las actualizaciones de configuración que se produjeron mientras la puerta de enlace estaba "desconectada".

## <a name="next-steps"></a>Pasos siguientes

-   [Lea las notas del producto para obtener más información sobre este tema](https://aka.ms/hybrid-and-multi-cloud-api-management)
-   [Implementación de una puerta de enlace autohospedada en Docker](how-to-deploy-self-hosted-gateway-docker.md)
-   [Implementación de una puerta de enlace autohospedada en Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md)
-   [Implementación de una puerta de enlace autohospedada en el clúster de Kubernetes habilitado para Azure Arc](how-to-deploy-self-hosted-gateway-azure-arc.md)
