---
title: 'Protocolo de la extensión HTTP: Azure'
description: Azure Video Analyzer permite mejorar sus funcionalidades de procesamiento a través de un nodo de extensión de canalización. El procesador de extensiones HTTP permite escenarios de extensibilidad mediante el protocolo HTTP, donde el rendimiento o el uso óptimo de los recursos no es la principal preocupación.
ms.topic: reference
ms.date: 03/30/2021
ms.openlocfilehash: 1f000ffd6368578ba9a6b5206af4b8ae5bf6cf00
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388489"
---
# <a name="use-the-http-extension-protocol"></a>Uso del protocolo de la extensión HTTP 

Azure Video Analyzer permite mejorar sus funcionalidades de procesamiento a través de un nodo de [extensión de canalización](pipeline-extension.md). El nodo del procesador de extensiones HTTP permite escenarios de extensibilidad mediante el protocolo de la extensión HTTP, donde el rendimiento o el uso óptimo de los recursos no son las principales preocupaciones. En este artículo, aprenderá a usar este protocolo para enviar mensajes entre Video Analyzer y un punto de conexión HTTP REST, que normalmente se encapsularía en torno a un servidor de inferencia de inteligencia artificial.

El contrato HTTP se define entre los dos componentes siguientes:

* Servidor HTTP
* El módulo Video Analyzer actúa como cliente HTTP

## <a name="http-contract"></a>Contrato de HTTP

### <a name="request"></a>Solicitud

Las solicitudes del módulo Video Analyzer al servidor HTTP serían así:

|Clave|Value|
|---|---|
|POST|https://hostname/optional-path?optional-query|
|Accept|application/json|
|Authorization| Básica, Implícita, Portador (por medio de compatibilidad con encabezado personalizado)|
|Content-Type|  image/jpeg<br/>image/png<br/>image/bmp<br/>image/x-raw|
|Longitud de cuerpo longitud de contenido, en bytes.   ||
|User-Agent |Azure Media Services|
|Body|  Bytes de imagen, codificados en binario en uno de los tipos de contenido admitidos.|
```

### Example

```html
POST http://localhost:8080/inference HTTP/1.1
Host: localhost:8080
x-ms-client-request-id: d6050cd4-c9f2-42d3-9adc-53ba7e440f17
Content-Type: image/bmp
Content-Length: 519222

(Image Binary Content)
```

### <a name="response"></a>Response

Las respuestas del servidor de inferencia al módulo Video Analyzer deberían ser así:

|Clave|   Value|
|---|----|
|Códigos de estado|  200 Correcto: Resultados de inferencia encontrados<br/>204 Sin contenido: La IA no ha encontrado ningún resultado<br/>400 Solicitud incorrecta: Inesperado<br/>500 Error interno del servidor: Inesperado<br/>503 Servidor ocupado: Video Analyzer se interrumpe en función del encabezado "Retry-after" o de una cantidad de tiempo predeterminada en caso de que el encabezado no esté preestablecido.|
|Content-Type   |application/json|
|Content-Length |Longitud del cuerpo, en bytes|
|Body|  Objeto JSON con una sola propiedad "inferences".|

### <a name="example"></a>Ejemplo

```html
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 468
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 17 Apr 2021 04:44:01 GMT

{
  "inferences": [
    {
      "type": "entity",
      "entity": {
        "tag": { "value": "car", "confidence": 0.9048132 },
        "box": { "l": 0.42681578, "t": 0.47660735, "w": 0.019501392, "h": 0.020954132 }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": { "value": "car", "confidence": 0.8953932 },
        "box": { "l": 0.55083525, "t": 0.4843858, "w": 0.046550274, "h": 0.046502113 }
      }
    }    
  ]
}
```

Se recomienda que las respuestas se devuelvan mediante documentos JSON válidos y que se siga el esquema preestablecido definido en el [modelo de objetos del esquema de metadatos de inferencia](inference-metadata-schema.md). La conformidad con el esquema garantizará la interoperabilidad con otros componentes de Video Analyzer, como la capacidad de realizar un seguimiento de objetos en vídeo en directo y superponer los metadatos de inferencia sobre el vídeo durante la reproducción, como se muestra [aquí](record-stream-inference-data-with-video.md).

Si el módulo devuelve una respuesta en la que el tipo de contenido no es "application/json", Video Analyzer codifica el mensaje como contenido Base 64 y lo serializa como una carga JSON opaca.

Si el módulo devuelve una respuesta cuyo tipo de contenido es "application/json" pero el esquema JSON no sigue el esquema de metadatos de inferencia anterior, la carga del mensaje se reenvía a través de la canalización, pero se reduce la interoperabilidad.

> [!NOTE]
> Si el servidor de inferencia no genera ningún resultado para una imagen determinada, debe devolver el código de estado HTTP 204 (Sin contenido) con un cuerpo de respuesta vacío. Video Analyzer lo entiende como un resultado vacío y no reenvía el evento a través de la canalización.

## <a name="next-steps"></a>Pasos siguientes

[Más información acerca del protocolo de la extensión gRPC](grpc-extension-protocol.md)


