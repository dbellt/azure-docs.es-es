---
title: 'Azure Event Grid en Kubernetes: introducción'
description: En este artículo se proporciona información general sobre Event Grid en Kubernetes con Azure Arc.
author: jfggdl
ms.author: jafernan
ms.subservice: kubernetes
ms.date: 05/25/2021
ms.topic: overview
ms.openlocfilehash: a958e42f3afab3dccd2a989c9d824a37ed9d9a6c
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536818"
---
# <a name="event-grid-on-kubernetes-with-azure-arc-preview---overview"></a>Event Grid en Kubernetes con Azure Arc (versión preliminar): introducción
En este artículo se proporciona información general sobre Event Grid en Kubernetes, los casos de uso, las características que ofrece y en qué se diferencia de Azure Event Grid.

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]

## <a name="what-is-event-grid"></a>¿Qué es Event Grid?
Event Grid es un agente de eventos que se usa para integrar cargas de trabajo que usan arquitecturas controladas por eventos. Una arquitectura controlada por eventos usa eventos para comunicar los hechos en los cambios de estado del sistema y es un enfoque de integración común en arquitecturas desacopladas como las que usan microservicios. Event Grid ofrece un modelo de publicación y suscripción, que también se describe como un modelo de comunicación de inserción-inserción, en el que se envían (insertan) eventos a los suscriptores y esos suscriptores no son necesariamente conscientes del publicador que envía los eventos. Este modelo contrasta con los modelos de inserción-extracción clásicos, como los usados por Azure Service Bus o Azure Event Hubs, en los que los clientes extraen mensajes de los agentes de mensajes y, como consecuencia, hay un acoplamiento más fuerte entre los agentes de mensajes y los clientes de consumo.

Event Grid se ofrece en dos ediciones: **Azure Event Grid**, un servicio PaaS totalmente administrado en Azure, y Event Grid en Kubernetes con Azure Arc, que le permite usar Event Grid en el clúster de Kubernetes dondequiera que se implemente, localmente o en la nube. 

Para mayor claridad, en este artículo usamos el término **Event Grid** al hacer referencia a las funcionalidades generales del servicio, independientemente de la edición usada. Nos referimos a **Azure Event Grid** para hacer referencia al servicio administrado hospedado en Azure. Para ser concisos, también nos referimos a **Event Grid en Kubernetes con Azure Arc** como **Event Grid en Kubernetes**.

Independientemente de la edición de Event Grid que use, hay un **publicador de eventos** que envía eventos a Event Grid y uno o varios **suscriptores de eventos** que exponen puntos de conexión en los que reciben los eventos entregados por Event Grid. No es necesario entregar todos los eventos publicados en Event Grid a todos los suscriptores de eventos. Event Grid permite seleccionar los eventos que se deben enrutar a destinos específicos mediante un conjunto de opciones de configuración definidas en una **suscripción de eventos**. Puede usar filtros en las suscripciones de eventos para enrutar eventos específicos a un punto de conexión o realizar una multidifusión a varios puntos de conexión. Event Grid ofrece un mecanismo de entrega confiable con lógica de reintento. Event Grid también se basa en estándares abiertos y admite la [especificación de esquema de Cloud Events 1.0](https://github.com/cloudevents/spec/blob/master/spec.md).


## <a name="event-grid-on-kubernetes-with-azure-arc"></a>Event Grid en Kubernetes con Azure Arc
Event Grid en Kubernetes con Azure Arc es una oferta que le permite ejecutar Event Grid en su propio clúster de Kubernetes. Esta funcionalidad se habilita mediante el uso de [Kubernetes habilitado para Azure Arc](../../azure-arc/kubernetes/overview.md). Mediante Kubernetes habilitado para Azure Arc, se conecta a Azure un [clúster de Kubernetes compatible](install-k8s-extension.md#supported-kubernetes-distributions). Una vez conectado, puede [instalar Event Grid](install-k8s-extension.md) en él. 

### <a name="use-case"></a>Caso de uso
Event Grid en Kubernetes admite varios escenarios de integración controlados por eventos. Sin embargo, el escenario principal que abarca se admite y se expresa como un caso de usuario es:

"Como propietario de un sistema implementado en un clúster de Kubernetes, quiero comunicar los cambios de estado de mi sistema mediante la publicación de eventos y la configuración del enrutamiento de esos eventos para que los controladores de eventos, bajo mi control o de otro modo, puedan procesar los eventos del sistema de la mejor manera".

**Característica** que le ayuda a llevar a cabo los requisitos anteriores: [temas de Event Grid](/rest/api/eventgrid/version2020-10-15-preview/topics).

### <a name="event-grid-on-kubernetes-at-a-glance"></a>Event Grid en Kubernetes de un vistazo
Desde la perspectiva del usuario, Event Grid en Kubernetes se compone de los siguientes recursos indicados en color azul:

:::image type="content" source="./media/overview/event-grid-topics.png" alt-text="Recursos" lightbox="./media/overview/event-grid-topics.png":::

* Un **tema** es un tipo de canal de entrada que expone un punto de conexión en el que los publicadores envían eventos a Event Grid.
* Una **suscripción de eventos** es un recurso que contiene opciones de configuración para filtrar y enrutar eventos a un destino en el que se entregan los eventos.
* Un **evento** es el anuncio de un cambio de estado.
* Un **controlador de eventos** es una aplicación o servicio que recibe eventos y reacciona o procesa los eventos de alguna manera. A veces también hacemos referencia a los controladores de eventos como **suscriptores de eventos**. En el diagrama anterior, los controladores de eventos son la API implementada en un clúster de Kubernetes (K8s) y el servicio Azure Service Bus.

Para más información acerca de estos conceptos, consulte [Conceptos en Azure Event Grid](concepts.md).

### <a name="sample-workload-integration-scenarios-and-destinations"></a>Escenarios y destinos de integración de cargas de trabajo de ejemplo

Puede integrar las cargas de trabajo que se ejecutan en el clúster. El publicador puede ser cualquier servicio que se ejecute en el clúster de Kubernetes o cualquier carga de trabajo que tenga acceso al punto de conexión del tema (hospedado por el agente de Event Grid) al que el publicador envía eventos.

:::image type="content" source="./media/overview/event-grid-intra-cluster-integration.png" alt-text="Integración entre clústeres" lightbox="./media/overview/event-grid-intra-cluster-integration.png":::


También puede tener un publicador implementado en otra parte de la red que envíe eventos a la instancia de Event Grid implementada en uno de los clústeres de Kubernetes:

:::image type="content" source="./media/overview/event-grid-in-network-integration.png" alt-text="Integración en la red" lightbox="./media/overview/event-grid-in-network-integration.png":::

Con Event Grid en Kubernetes, puede reenviar eventos a Azure para su posterior procesamiento, almacenamiento o visualización:

:::image type="content" source="./media/overview/event-grid-forward-events.png" alt-text="Reenvío de eventos a Azure":::

#### <a name="destinations"></a>Destinations
Los destinos del controlador de eventos pueden ser cualquier punto de conexión HTTPS o HTTP al que Event Grid pueda acceder mediante la red, pública o privada, y tenga acceso (no protegido con algún mecanismo de autenticación). Los destinos de entrega de los eventos se definen al crear una suscripción de eventos. Para más información, vea [Controladores de eventos](event-handlers.md). 

## <a name="features"></a>Características
Event Grid en Kubernetes admite [temas de Event Grid](/rest/api/eventgrid/version2020-10-15-preview/topics), que es una característica que también ofrece [Azure Event Grid](../custom-topics.md). Los temas de Event Grid temas le ayudarán a llevar a cabo el [caso de uso de integración principal](#use-case), en el que los requisitos piden integrar el sistema con otra carga de trabajo, ya sea de su propiedad o que se hace accesible para el sistema de algún otro modo.

Algunas de las funcionalidades que obtiene con Azure Event Grid en Kubernetes son:

* **[Filtrado avanzado](filter-events.md)** : filtre por tipo de evento, asunto del evento o datos del evento para asegurarse de que los controladores de eventos solo reciban los eventos pertinentes.
* **Distribución ramificada**: suscriba varios puntos de conexión al mismo evento para enviar copias del evento a varios lugares.
* **Basado en estándares abiertos**: defina los eventos mediante la [especificación de esquema de Cloud Events 1.0](https://github.com/cloudevents/spec/blob/master/spec.md) de CNCF.
* **Confiabilidad**: Event Grid dispone de una lógica de entrega de eventos con reintento que garantiza que los eventos lleguen a su destino.

Para más información, consulte [Características admitidas por Event Grid en Kubernetes](features.md).

## <a name="pricing"></a>Precios 
Event Grid en Kubernetes con Azure Arc se ofrece sin cargo durante su versión preliminar.

## <a name="next-steps"></a>Pasos siguientes
Siga estos pasos para empezar a enrutar eventos mediante Event Grid en Kubernetes.

1. [Conexión del clúster a Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md).
1. [Instalación de una extensión de Event Grid](install-k8s-extension.md), que es el recurso real que implementa Event Grid en un clúster de Kubernetes. Para más información sobre la extensión, consulte la sección [Extensión de Event Grid](install-k8s-extension.md#event-grid-extension). 
1. [Creación de una ubicación personalizada](../../azure-arc/kubernetes/custom-locations.md). Una ubicación personalizada representa un espacio de nombres del clúster y es el lugar donde se implementan los temas y las suscripciones de eventos.
1. [Creación de un tema y una o varias suscripciones de eventos](create-topic-subscription.md).
1. [Publicación de eventos](create-topic-subscription.md).

Estos son más recursos que puede usar:

* [SDK del plano de datos](../sdk-overview.md#data-plane-sdks).
* [Publicación de ejemplos de eventos mediante los SDK del plano de datos](https://devblogs.microsoft.com/azure-sdk/event-grid-ga/).
* [CLI de Event Grid](/cli/azure/eventgrid).
* [SDK de administración](../sdk-overview.md#management-sdks).