---
title: 'Preguntas más frecuentes (P+F): Bing Image Search API'
titleSuffix: Azure Cognitive Services
description: Encuentre respuestas a preguntas habituales sobre conceptos, código y escenarios relacionados con Bing Image Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: aahi
ms.openlocfilehash: 4e62aa434efade2cb1fa3d82265eb44d523dd869
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108773366"
---
# <a name="frequently-asked-questions-faq-about-the-bing-image-search-api"></a>Preguntas más frecuentes (P+F) sobre Bing Image Search API

> [!WARNING]
> Bing Search APIs se mueve de Cognitive Services a Bing Search Services. A partir del **30 de octubre de 2020**, las nuevas instancias de Bing Search deben aprovisionarse siguiendo el proceso documentado [aquí](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> El aprovisionamiento de Bing Search APIs con Cognitive Services será posible durante los próximos tres años o hasta que finalice el Contrato Enterprise, lo que suceda primero.
> Puede encontrar instrucciones sobre la migración en [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Encuentre respuestas a preguntas habituales sobre conceptos, código y escenarios relacionados con Bing Image Search API para Azure Cognitive Services en Azure.

## <a name="response-headers-in-javascript"></a>Encabezados de respuesta en JavaScript

Los encabezados siguientes pueden producirse en respuestas de Bing Image Search API.

| Atributo           | Descripción   |
| ------------------- | ------------- |
| `X-MSEdge-ClientID` |El identificador único que Bing ha asignado al usuario |
| `BingAPIs-Market`   |El mercado que se usó para cumplir la solicitud |
| `BingAPIs-TraceId`  |La entrada de registro en el servidor de Bing API para esta solicitud (para soporte técnico) |

Es especialmente importante conservar el identificador de cliente y devolverlo con las sucesivas solicitudes. Al hacerlo, la búsqueda usa el contexto pasado en los resultados de búsqueda de clasificación y también se proporciona una experiencia coherente al usuario.

Sin embargo, cuando se llama a Bing Image Search API desde JavaScript, las características de seguridad integradas de su explorador (CORS) podrían impedirle acceder a los valores de estos encabezados.

Para acceder a los encabezados, puede realizar la solicitud de Bing Image Search API mediante un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que filtra los encabezados de respuesta y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que nuestra [aplicación de tutorial](tutorial-bing-image-search-single-page-app.md) obtenga acceso a los encabezados de cliente opcionales. En primer lugar, si aún no lo tiene, [instale Node.js](https://nodejs.org/en/download/). A continuación, escriba el siguiente comando en un símbolo del sistema.

```console
npm install -g cors-proxy-server
```

Luego, cambie el punto de conexión de Bing Image Search API del archivo HTML a:
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

Por último, inicie el proxy CORS con el siguiente comando:

```console
cors-proxy-server
```

Deje abierta la ventana de comandos mientras usa la aplicación del tutorial, ya que si la cierra, se detendrá el proxy. En la sección de encabezados HTTP expandibles situada bajo los resultados de la búsqueda, puede ver el encabezado `X-MSEdge-ClientID` (entre otras cosas) y comprobar que es el mismo en todas las solicitudes.

## <a name="response-headers-in-production"></a>Encabezados de respuesta en producción

El enfoque de proxy CORS que se describe en la respuesta anterior es adecuado para desarrollo, pruebas y aprendizaje.

Sin embargo, en un entorno de producción, debe hospedar un script del lado servidor en el mismo dominio que la página web que usa Bing Web Search API. Este script debería realizar en realidad las llamadas API tras la solicitud del código de JavaScript de la página web y pasar nuevamente al cliente todos los resultados, incluidos los encabezados. Dado que los dos recursos (página y script) comparten un origen, no se aplica CORS y el código de JavaScript puede acceder a los encabezados especiales en la página web.

Este enfoque también protege la clave de API de una exposición pública, ya que solo la necesita el script del lado servidor. El script puede usar otro método (por ejemplo, el origen de referencia de HTTP) para asegurarse de que la solicitud está autorizada.

## <a name="next-steps"></a>Pasos siguientes

¿Es su pregunta acerca de una característica o funcionalidad que falta? Considere la posibilidad de solicitarla o votarla mediante la [herramienta de comentarios](https://feedback.azure.com/forums/932041-azure-cognitive-services?category_id=395749).

## <a name="see-also"></a>Consulte también

 [Stack Overflow: Cognitive Services](https://stackoverflow.com/questions/tagged/bing-api)
