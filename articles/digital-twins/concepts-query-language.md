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
ms.openlocfilehash: e311a39a68a5fb03c68a1685996a8e0fbde206e8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108734352"
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

Para enviar una consulta al servicio desde una aplicación cliente, usará la [API Query](/rest/api/digital-twins/dataplane/query) de Azure Digital Twins. Una manera de usar la API es a mediante uno de los [SDK](concepts-apis-sdks.md#overview-data-plane-apis) de Azure Digital Twins.

[!INCLUDE [digital-twins-query-reference.md](../../includes/digital-twins-query-reference.md)]

## <a name="considerations-for-querying"></a>Consideraciones para la consulta

Al escribir consultas para Azure Digital Twins, tenga en cuenta las consideraciones siguientes:
* **Recuerde la distinción entre mayúsculas y minúsculas**: todas las operaciones de consulta de Azure Digital Twins distinguen mayúsculas de minúsculas, por lo que debe tener cuidado de usar los nombres exactos definidos en los modelos. Si los nombres de propiedad están mal escritos o usan las mayúsculas de forma incorrecta, el conjunto de resultados está vacío y no se devuelven errores.
* **Comillas simples de escape**: si el texto de la consulta incluye un carácter de comilla simple en los datos, la comilla deberá utilizar el carácter `\` como escape. Este es un ejemplo que trata sobre el valor de una propiedad de *D'Souza*:

  :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="EscapedSingleQuote":::

* **Considere la posible latencia**: después de realizar un cambio en los datos del gráfico, puede haber una latencia de hasta 10 segundos antes de que los cambios se reflejen en las consultas. La [API GetDigitalTwin](how-to-manage-twin.md#get-data-for-a-digital-twin) no experimenta este retraso, por lo que debe usar la llamada a la API en lugar de realizar una consulta para ver los cambios reflejados inmediatamente si necesita una respuesta instantánea.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo escribir consultas y ver ejemplos de código de cliente en [Procedimientos para consultar el grafo de gemelos](how-to-query-graph.md).