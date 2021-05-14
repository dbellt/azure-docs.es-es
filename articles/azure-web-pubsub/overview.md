---
title: ¿Qué es el servicio Azure Web PubSub?
description: Conozca mejor qué escenarios de casos de uso habituales utilizan Azure Web PubSub y las principales ventajas de Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 01/27/2021
ms.openlocfilehash: 920c55746ab78f9479c37027b479af7cf6604865
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167081"
---
# <a name="what-is-azure-web-pubsub-service"></a>¿Qué es el servicio Azure Web PubSub? 

El servicio Azure Web PubSub le ayuda a crear aplicaciones web de mensajería en tiempo real mediante WebSockets y el patrón de publicación y suscripción fácilmente. Esta funcionalidad en tiempo real permite publicar actualizaciones de contenido entre el servidor y los clientes conectados (por ejemplo, una aplicación web de página única o una aplicación móvil). Los clientes no necesitan buscar las actualizaciones más recientes ni enviar nuevas solicitudes HTTP para las actualizaciones.

En este artículo se proporciona una introducción a Azure Web PubSub.

## <a name="what-is-azure-web-pubsub-service-used-for"></a>¿Para qué se usa el servicio Azure Web PubSub?

Cualquier escenario que requiera mensajería de publicación y suscripción en tiempo real entre el servidor y los clientes o entre clientes, puede usar el servicio Azure Web PubSub. Las características tradicionales en tiempo real que a menudo requieren tener que sondear desde el servidor o enviar solicitudes HTTP, también pueden usar el servicio Azure Web PubSub.

Azure Web PubSub se ha utilizado en una amplia variedad de sectores, para cualquier tipo de aplicación que requiera actualizaciones de contenido en tiempo real. A continuación se enumeran algunos ejemplos en los que conviene usar Azure Web PubSub:

* **Actualizaciones de datos de alta frecuencia:** juegos, votos, sondeos, subastas.
* **Paneles activos y supervisión:** panel de empresas, datos de mercados financieros, actualización instantánea de ventas, panel de líderes de juegos multijugador y supervisión de IoT.
* **Chat en directo entre plataformas:** salón de chat en directo, bot de chat, asistencia al cliente, asistente para la compra en tiempo real, Messenger, chat en juegos, etc.
* **Ubicación en tiempo real en el mapa:** seguimiento de la logística, seguimiento del estado de entrega, actualizaciones del estado de transporte, aplicaciones GPS.
* **Publicidad dirigida en tiempo real:** ofertas y anuncios insertados personalizados en tiempo real y anuncios interactivos.
* **Aplicaciones de colaboración:** las aplicaciones de pizarra y coautoría, y el software de reuniones de equipo.
* **Notificaciones instantáneas de inserción:** redes sociales, correo electrónico, juegos, alertas de viaje.
* **Difusión en tiempo real:** difusión de audio y vídeo en directo, subtítulos (CC), traducción, difusión de eventos y noticias.
* **IoT y dispositivos conectados:** métricas de IoT en tiempo real, control remoto, estado en tiempo real y seguimiento de la ubicación.
* **Automatización:** desencadenador en tiempo real de los eventos de nivel superior.

## <a name="what-are-the-benefits-using-azure-web-pubsub-service"></a>¿Cuáles son las ventajas de usar Azure Web PubSub?

**Compatibilidad integrada con conexiones de cliente a gran escala y arquitecturas de alta disponibilidad:**

Azure Web PubSub está diseñado para aplicaciones en tiempo real a gran escala. El servicio permite que varias instancias funcionen conjuntamente y escalen a millones de conexiones de cliente. Al mismo tiempo, admite también varias regiones globales con fines de particionamiento, alta disponibilidad o recuperación ante desastres.

**Compatibilidad con una amplia variedad de SDK de cliente y lenguajes de programación:**

El servicio Azure Web PubSub funciona con una amplia gama de clientes, como los exploradores móviles y web, las aplicaciones de escritorio, las aplicaciones móviles, el proceso de servidor, los dispositivos IoT y las consolas de juegos. Dado que este servicio admite WebSocket sin procesar con un patrón de publicación y suscripción, es fácil usar cualquier SDK de cliente de WebSocket estándar en distintos lenguajes con este servicio. 

**Se ofrecen API enriquecidas para diferentes patrones de mensajes:**

El servicio Azure Web PubSub es un servicio de mensajería bidireccional que permite que se usen diferentes patrones de mensajería entre el servidor y los clientes, por ejemplo:

* El servidor envía mensajes a una conexión concreta, a todas las conexiones o a un subconjunto de conexiones que pertenecen a un usuario específico o que se han colocado en un grupo arbitrario. 
* El cliente envía mensajes a una conexión concreta, a todas las conexiones o a un subconjunto de conexiones que pertenecen a un usuario específico o que se han colocado en un grupo arbitrario.
* Los clientes envían mensajes al servidor.


## <a name="how-to-use-the-azure-web-pubsub-service"></a>¿Cómo usar el servicio Azure Web PubSub?

Hay muchas maneras diferentes de programar con el servicio Azure Web PubSub, como algunas de las muestras enumeradas aquí:

- **Compilar aplicaciones en tiempo real sin servidor**: use la integración de Azure Functions con Azure Web PubSub para crear aplicaciones en tiempo real sin servidor en lenguajes como JavaScript, C#, Java y Python. 
- **Enviar mensajes desde el servidor a los clientes a través de la API REST**: Azure Web PubSub proporciona la API REST para permitir que las aplicaciones envíen mensajes a los clientes conectados, en cualquier lenguaje de programación compatible con REST.