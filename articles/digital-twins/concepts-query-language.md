---
title: Lenguaje de consulta
titleSuffix: Azure Digital Twins
description: Información sobre los aspectos básicos del lenguaje de consulta de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 4/22/2021
ms.topic: conceptual
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: 13bf3b8b8756fcc83211b02379980972629d9438
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290549"
---
# <a name="about-the-query-language-for-azure-digital-twins"></a>Acerca del lenguaje de consulta para Azure Digital Twins

Recuerde que el centro de Azure Digital Twins es el [grafo de gemelos](concepts-twins-graph.md), que se crea partir de gemelos digitales y relaciones. 

Este grafo se puede consultar para obtener información sobre los gemelos digitales y las relaciones que contiene. Estas consultas se escriben en un lenguaje de consulta personalizado similar a SQL al que se conoce como **lenguaje de consulta de Azure Digital Twins**. Es similar al [lenguaje de consulta de IoT Hub](../iot-hub/iot-hub-devguide-query-language.md) con muchas características comparables.

En este artículo se describen los aspectos básicos del lenguaje de consulta y sus funcionalidades. Para ver ejemplos más detallados de la sintaxis de consulta y cómo ejecutar solicitudes de consulta, consulte  [Consulta del grafo gemelo de Azure Digital Twins](how-to-query-graph.md).

## <a name="about-the-queries"></a>Acerca de las consultas

Puede usar el lenguaje de consulta de Azure Digital Twins para recuperar gemelos digitales según sus...
* propiedades (incluidas [propiedades de etiqueta](how-to-use-tags.md))
* modelos
* relationships
  - propiedades de las relaciones

Para enviar una consulta al servicio desde una aplicación cliente, usará la [API Query](/rest/api/digital-twins/dataplane/query) de Azure Digital Twins. Una manera de usar la API es a mediante uno de los [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) de Azure Digital Twins.

[!INCLUDE [digital-twins-query-reference.md](../../includes/digital-twins-query-reference.md)]

## <a name="considerations-for-querying"></a>Consideraciones para la consulta

Al escribir consultas para Azure Digital Twins, tenga en cuenta las consideraciones siguientes:
* **Recuerde la distinción entre mayúsculas y minúsculas**: todas las operaciones de consulta de Azure Digital Twins distinguen mayúsculas de minúsculas, por lo que debe tener cuidado de usar los nombres exactos definidos en los modelos. Si los nombres de propiedad están mal escritos o usan las mayúsculas de forma incorrecta, el conjunto de resultados está vacío y no se devuelven errores.
* **Comillas simples de escape**: si el texto de la consulta incluye un carácter de comilla simple en los datos, la comilla deberá utilizar el carácter `\` como escape. Este es un ejemplo que trata sobre el valor de una propiedad de *D'Souza*:

  :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="EscapedSingleQuote":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo escribir consultas y ver ejemplos de código de cliente en [Procedimientos para consultar el grafo de gemelos](how-to-query-graph.md).