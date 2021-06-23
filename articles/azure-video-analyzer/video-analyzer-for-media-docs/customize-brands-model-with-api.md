---
title: Personalización de un modelo de marcas con la API de Azure Video Analyzer for Media (anteriormente, Video Indexer)
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo personalizar un modelo de marcas con la API de Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
author: anikaz
manager: johndeu
ms.topic: article
ms.date: 01/14/2020
ms.author: kumud
ms.openlocfilehash: 3f3e2356a9c9101763f9b944993b9a96b22b8a0e
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386689"
---
# <a name="customize-a-brands-model-with-the-video-analyzer-for-media-api"></a>Personalización de un modelo de marcas con la API de Video Analyzer for Media

Azure Video Analyzer for Media (anteriormente Video Indexer) permite la detección de marcas por voz y texto visual durante la indexación y la reindexación de contenido de audio y vídeo. La característica de detección de marcas identifica menciones de productos, servicios y compañías sugeridas por la base de datos de marcas de Bing. Por ejemplo, si se menciona Microsoft en contenido de audio o vídeo, o si se muestra en texto visual en un vídeo, Video Analyzer for Media lo detectará como una marca en el contenido. Un modelo de marcas personalizado permite excluir determinadas marcas de la detección e incluir otras que deben ser parte del modelo que podrían no estar en la base de datos de marcas de Bing. Para más información, consulte [Información general](customize-brands-model-overview.md).

> [!NOTE]
> Si el vídeo se ha indexado antes de agregar una marca, debe reindexarlo.

Puede usar las API de Video Analyzer for Media para crear, usar y editar modelos de marcas personalizados detectados en un vídeo, como se describe en este tema. También puede usar el sitio web de Video Analyzer for Media, como se describe en [Personalización de un modelo de marcas con el sitio web de Video Analyzer for Media](customize-brands-model-with-api.md).

## <a name="create-a-brand"></a>Creación de una marca

La API para [crear una marca](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Create-Brand) crea una marca personalizada y la agrega al modelo de marcas personalizado de la cuenta especificada.

> [!NOTE]
> Al establecer `enabled` (en el cuerpo) en true, la marca se coloca en la lista *Include* (Incluir) para que Video Analyzer for Media la detecte. Al establecer `enabled` en false, la marca se coloca en la lista *Exclude* (de exclusión), así que Video Analyzer for Media no la detectará.

Otros parámetros que se pueden establecer en el cuerpo:

* El valor `referenceUrl` puede ser cualquier sitio web de referencia para la marca, como un vínculo a su página de Wikipedia.
* El valor `tags` es una lista de etiquetas de la marca. Esta etiqueta se muestra en el campo *Categoría* de la marca del sitio web de Video Analyzer for Media. Por ejemplo, la marca "Azure" se puede etiquetar o clasificar como "Nube".

### <a name="response"></a>Response

La respuesta proporciona información sobre la marca que acaba de crear, siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T14:59:52.7433333",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

## <a name="delete-a-brand"></a>Eliminación de una marca

La API para [eliminar una marca](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Delete-Brand) quita una marca del modelo de marcas personalizado de la cuenta especificada. La cuenta se especifica en el parámetro `accountId`. Una vez que se llama correctamente, la marca ya no está en las listas de marcas *Include* (Incluir) o *Exclude* (Excluir).

### <a name="response"></a>Response

No se devuelve ningún contenido cuando la marca se elimina correctamente.

## <a name="get-a-specific-brand"></a>Obtención de una hoja específica

La API para [obtener una marca](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brand) permite buscar los detalles de una marca en el modelo de marcas personalizado de la cuenta especificada mediante el identificador de marca.

### <a name="response"></a>Response

La respuesta proporciona información sobre la marca que ha buscado (mediante el identificador de marca), siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 128846,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-01-06T13:51:38.3666667",
  "lastModified": "2018-01-11T13:51:38.3666667",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

> [!NOTE]
> Si `enabled` se establece en `true`, la marca está en la lista *Include* (Incluir) para que Video Analyzer for Media la detecte. Si `enabled` es false, significa que la marca está en la lista *Exclude* (Excluir), así que Video Analyzer for Media no la detectará.

## <a name="update-a-specific-brand"></a>Actualización de una marca específica

La API para [actualizar una marca](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Brand) permite buscar los detalles de una marca en el modelo de marcas personalizado de la cuenta especificada mediante el identificador de marca.

### <a name="response"></a>Response

La respuesta proporciona la información actualizada sobre la marca que actualizó, siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": null,
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "Created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T15:37:50.67",
  "enabled": false,
  "description": "This is an update example",
  "tags": [
    "Tag1",
    "NewTag2"
  ]
}
```

## <a name="get-all-of-the-brands"></a>Obtención de todas las marcas

La API para [devolver todas las marcas](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brands) devuelve todas las marcas del modelo de marcas personalizado de la cuenta especificada, independientemente de si se quiere que la marca esté en las listas *Include* (Incluir) o *Exclude* (Excluir).

### <a name="response"></a>Response

La respuesta proporciona una lista de todas las marcas de su cuenta y cada uno de sus detalles, siguiendo el formato del ejemplo siguiente.

```json
[
    {
        "ReferenceUrl": null,
        "id": 97974,
        "name": "Example",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-25T14:59:52.7433333",
        "LastModified": "2018-04-25T14:59:52.7433333",
        "enabled": true,
        "description": "This is an example",
        "tags": ["Tag1", "Tag2"]
    },
    {
        "ReferenceUrl": null,
        "id": 97975,
        "name": "Example2",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-26T14:59:52.7433333",
        "LastModified": "2018-04-26T14:59:52.7433333",
        "enabled": false,
        "description": "This is another example",
        "tags": ["Tag1", "Tag2"]
    },
]
```

> [!NOTE]
> La marca llamada *Example* está en la lista *Include* (Incluir) para que Video Analyzer for Media la detecte, y la marca llamada *Example2* está en la lista *Exclude* (Excluir), así que Video Analyzer for Media no la detectará.

## <a name="get-brands-model-settings"></a>Obtención de la configuración del modelo de marcas

La API para [obtener la configuración de las marcas](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brands) devuelve la configuración del modelo de marcas de la cuenta especificada. La configuración del modelo de marcas representa si la detección desde la base de datos de Bing está o no habilitada. Si las marcas de Bing no están habilitadas, Video Analyzer for Media solo detectará las marcas del modelo de marcas personalizado de la cuenta especificada.

### <a name="response"></a>Response

La respuesta muestra si las marcas de Bing están habilitadas, siguiendo el formato del ejemplo siguiente.

```json
{
  "state": true,
  "useBuiltIn": true
}
```

> [!NOTE]
> Si `useBuiltIn` se establece en true, representa que las marcas de Bing están habilitadas. Si `useBuiltin` es false, las marcas de Bing están deshabilitadas. El valor `state` se puede omitir, porque ya está en desuso.

## <a name="update-brands-model-settings"></a>Actualización de la configuración del modelo de marcas

La API para [actualizar marcas](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Brands-Model-Settings) actualiza la configuración del modelo de marcas de la cuenta especificada. La configuración del modelo de marcas representa si la detección desde la base de datos de Bing está o no habilitada. Si las marcas de Bing no están habilitadas, Video Analyzer for Media solo detectará las marcas del modelo de marcas personalizado de la cuenta especificada.

Si la marca `useBuiltIn` se establece en true, las marcas de Bing están habilitadas. Si `useBuiltin` es false, las marcas de Bing están deshabilitadas.

### <a name="response"></a>Response

No se devuelve ningún contenido cuando la configuración del modelo de marcas se actualiza correctamente.

## <a name="next-steps"></a>Pasos siguientes

[Customize Brands model using website](customize-brands-model-with-website.md) (Personalización del modelo de marcas mediante el sitio web)
