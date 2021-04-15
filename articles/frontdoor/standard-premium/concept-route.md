---
title: ¿Qué es Azure Front Door Estándar/Prémium (versión preliminar)?
description: Este artículo le ayuda a comprender cómo Azure Front Door Estándar/Prémium busca coincidencias con la regla de enrutamiento que se usará con una solicitud entrante.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: article
ms.workload: infrastructure-services
ms.date: 02/18/2021
ms.author: duau
ms.openlocfilehash: db026c4903aa30a0a4c8154af8ad6eeb4b72b706
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "101098233"
---
# <a name="what-is-azure-front-door-standardpremium-preview-route"></a>¿Cuál es la ruta de Azure Front Door Estándar/Prémium (versión preliminar)?

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

La ruta de Azure Front Door Estándar/Prémium define cómo se administra el tráfico cuando la solicitud entrante llega al entorno de Azure Front Door. Mediante la configuración de la ruta, se define una asociación entre un dominio y un grupo de origen de back-end. Al activar las características avanzadas, como Modelo de coincidencia o Conjunto de reglas, se consigue un control más pormenorizado sobre el tráfico.

Una configuración de regla de enrutamiento de Front Door Estándar/Premium se compone de dos partes principales: un "lado izquierdo" y un "lado derecho". La solicitud entrante se relaciona con el lado izquierdo de la ruta mientras que el lado derecho define cómo se procesa la solicitud.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="incoming-match-left-hand-side"></a>Coincidencia de entrada (lado izquierdo)

Las propiedades siguientes determinan si la solicitud entrante coincide con la regla de enrutamiento (o el lado izquierdo):

* **Protocolos HTTP** (HTTP/HTTPS)
* **Hosts** (por ejemplo, www\.foo.com, \*.bar.com)
* **Rutas de acceso** (por ejemplo, /\*, /Users/\*, /file.gif)

Estas propiedades se expanden de forma interna para que cada combinación de protocolo, host y ruta de acceso sea un conjunto de posible coincidencia.

### <a name="route-data-right-hand-side"></a>Datos de ruta (lado derecho)

La decisión de cómo procesar la solicitud depende de si el almacenamiento en caché está habilitado o no para la ruta. Si una respuesta almacenada en caché no está disponible, la solicitud se reenvía al servidor back-end adecuado.

## <a name="route-matching"></a>Búsqueda de coincidencia de ruta

Esta sección se centrará en cómo se busca la coincidencia con una regla de enrutamiento de Front Door determinada. El concepto básico es que siempre coincidimos con la regla **coincidencia más específica primero** fijándonos solo en el "lado izquierdo".  En primer lugar, buscamos la coincidencia en función del protocolo HTTP, el host de front-end y, luego, la ruta de acceso.

### <a name="frontend-host-matching"></a>Coincidencia de host de front-end

Al comparar los hosts de front-end, utilizamos la lógica que se define a continuación:

1. Busque cualquier enrutamiento con una coincidencia exacta en el host.
2. Si no hay una coincidencia de hosts de front-end exacta, rechace la solicitud y envíe un error 400 de solicitud incorrecta.

Para explicar más este proceso, echemos un vistazo a un ejemplo de configuración de rutas de Front Door (solo para el lado izquierdo):

| Regla de enrutamiento | Hosts de front-end | Path |
|-------|--------------------|-------|
| Un | foo.contoso.com | /\* |
| B | foo.contoso.com | /users/\* |
| C | www\.fabrikam.com, foo.adventure-works.com  | /\*, /images/\* |

Si las siguientes solicitudes entrantes se enviaron a Front Door, coincidirían con las siguientes reglas de enrutamiento anteriores:

| Host de front-end entrante | Reglas de enrutamientos que coinciden |
|---------------------|---------------|
| foo.contoso.com | A, B |
| www\.fabrikam.com | C |
| images.fabrikam.com | Error 400: Bad Request |
| foo.adventure-works.com | C |
| contoso.com | Error 400: Bad Request |
| www\.adventure-works.com | Error 400: Bad Request |
| www\.northwindtraders.com | Error 400: Bad Request |

### <a name="path-matching"></a>Coincidencia de ruta de acceso

Después de que Azure Front Door Estándar/Prémium determina el host de front-end específico y filtra las posibles reglas de enrutamiento a solo las rutas con ese host de front-end, filtra las reglas de enrutamiento en función de la ruta de acceso de la solicitud. Luego, Front Door filtra las reglas de enrutamiento en función de la ruta de acceso de la solicitud. Utilizamos una lógica similar que los hosts de front-end:

1. Busque cualquier regla enrutamiento con una coincidencia exacta con la ruta de acceso.
2. Si ninguna ruta de acceso presenta una coincidencia exacta, busque las reglas de enrutamiento con una ruta de acceso con un carácter comodín que coincida.
3. Si no hay reglas de enrutamiento con una ruta coincidente, rechace la solicitud y devuelva una respuesta de error HTTP 400: solicitud incorrecta.

>[!NOTE]
> Las rutas de acceso sin un carácter comodín se consideran con coincidencia exacta. Incluso si la ruta de acceso termina en una barra diagonal, todavía se considera una coincidencia exacta.

Para explicarlo mejor, echemos un vistazo a otra serie de ejemplos:

| Regla de enrutamiento | Host de front-end    | Path     |
|-------|---------|----------|
| Un     | www\.contoso.com | /        |
| B     | www\.contoso.com | /\*      |
| C     | www\.contoso.com | /ab      |
| D     | www\.contoso.com | /abc     |
| E     | www\.contoso.com | /abc/    |
| F     | www\.contoso.com | /abc/\*  |
| G     | www\.contoso.com | /abc/def |
| H     | www\.contoso.com | /path/   |

Dada esa configuración, daría lugar a la tabla de búsqueda de coincidencias de ejemplo siguiente:

| Solicitud entrante    | Ruta coincidente |
|---------------------|---------------|
| www\.contoso.com/            | Un             |
| www\.contoso.com/a           | B             |
| www\.contoso.com/ab          | C             |
| www\.contoso.com/abc         | D             |
| www\.contoso.com/abzzz       | B             |
| www\.contoso.com/abc/        | E             |
| www\.contoso.com/abc/d       | F             |
| www\.contoso.com/abc/def     | G             |
| www\.contoso.com/abc/defzzz  | F             |
| www\.contoso.com/abc/def/ghi | F             |
| www\.contoso.com/path        | B             |
| www\.contoso.com/path/       | H             |
| www\.contoso.com/path/zzz    | B             |

>[!WARNING]
> </br> Si no hay ninguna regla de enrutamiento para un host de front-end con coincidencia exacta con una ruta de acceso de ruta comodín (`/*`), entonces no será una coincidencia con una regla de enrutamiento.
>
> Configuración de ejemplo:
>
> | Enrutar | Host             | Path    |
> |-------|------------------|---------|
> | Un     | profile.contoso.com | /api/\* |
>
> Tabla de búsqueda de coincidencias:
>
> | Solicitud entrante       | Ruta coincidente |
> |------------------------|---------------|
> | profile.domain.com/other | Ninguno. Error 400: Bad Request |

### <a name="routing-decision"></a>Decisión de enrutamiento

Una vez que Azure Front Door Estándar/Prémium ha coincidido con una única regla de enrutamiento, debe elegir cómo procesar la solicitud. Si Azure Front Door Estándar/Prémium tiene una respuesta almacenada en caché disponible para la regla de enrutamiento coincidente, la solicitud se envía de vuelta al cliente. Lo próximo que evalúa Azure Front Door Estándar/Prémium es si tiene un conjunto de reglas para la regla de enrutamiento coincidente. Si no hay un conjunto de reglas definido, la solicitud se reenvía al grupo de back-end como está. De lo contrario, el conjunto de reglas se ejecuta en el orden en que están configuradas.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [crear una instancia de Front Door Estándar/Prémium](create-front-door-portal.md).
