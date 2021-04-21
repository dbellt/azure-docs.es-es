---
title: 'Inicio rápido: Envío de consultas a la API en Python: Bing Local Business Search'
titleSuffix: Azure Cognitive Services
description: Use este inicio rápido para empezar a usar Bing Local Business Search API en Python.
services: cognitive-services
author: aahill
ms.author: aahi
manager: nitinme
ms.date: 05/12/2020
ms.topic: quickstart
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: adbf3d9abddf01ba67046cfa433ffd46f713ff83
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107536668"
---
# <a name="quickstart-send-a-query-to-the-bing-local-business-search-api-in-python"></a>Inicio rápido: Envío de una consulta a Bing Local Business Search API en Python

> [!WARNING]
> Las Bing Search API se mueven de Cognitive Services a Bing Search Services. A partir del **30 de octubre de 2020**, las nuevas instancias de Bing Search deben aprovisionarse siguiendo el proceso documentado [aquí](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> El aprovisionamiento de Bing Search APIs con Cognitive Services será posible durante los próximos tres años o hasta que finalice el Contrato Enterprise, lo que suceda primero.
> Puede encontrar instrucciones sobre la migración en [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Use este inicio rápido para aprender a enviar solicitudes a la API Bing Local Business Search, que es un servicio de Azure Cognitive Services. Aunque esta aplicación sencilla está escrita en Python, la API es un servicio web RESTful compatible con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar JSON.

Esta aplicación de ejemplo obtiene los datos de respuesta local de la API para una consulta de búsqueda.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* [Python](https://www.python.org/) 2.x o 3.x.
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search</a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.

## <a name="run-the-complete-application"></a>Ejecución de la aplicación completa

En el ejemplo siguiente se obtienen resultados localizados que se implementan en los pasos siguientes:
1. Declare las variables para especificar el punto de conexión por host y ruta de acceso.
2. Especifique el parámetro de consulta. 
3. Defina la función de búsqueda que crea la solicitud y agrega el encabezado `Ocp-Apim-Subscription-Key`.
4. Establezca el encabezado `Ocp-Apim-Subscription-Key`. 
5. Realice la conexión y envíe la solicitud.
6. Imprima los resultados JSON.

Este es el código completo de esta demostración es el siguiente:

```python
import http.client, urllib.parse
import json

# Replace the subscriptionKey string value with your valid subscription key.

> [!WARNING]
> Bing Search APIs are moving from Cognitive Services to Bing Search Services. Starting **October 30, 2020**, any new instances of Bing Search need to be provisioned following the process documented [here](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Bing Search APIs provisioned using Cognitive Services will be supported for the next three years or until the end of your Enterprise Agreement, whichever happens first.
> For migration instructions, see [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
subscriptionKey = 'YOUR-SUBSCRIPTION-KEY'

host = 'api.cognitive.microsoft.com'
path = '/bing/v7.0/localbusinesses/search'

query = 'restaurant in Bellevue'

params = '?q=' + urllib.parse.quote (query) + '&mkt=en-us'

def get_local():
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection (host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read ()

result = get_local()
print (json.dumps(json.loads(result), indent=4))

```

## <a name="next-steps"></a>Pasos siguientes
- [Guía de inicio rápido de Local Business Search para Java](local-search-java-quickstart.md)
- [Inicio rápido de Local Business Search para C#](local-quickstart.md)
- [Inicio rápido de Local Business Search para Node](local-search-node-quickstart.md)
