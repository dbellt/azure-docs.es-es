---
author: baanders
description: incluir archivo para límites de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/8/2021
ms.author: baanders
ms.openlocfilehash: 2a6cff1aa93c4f3ac4f95fd176e1b1e42736d8f4
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108763365"
---
### <a name="functional-limits"></a>Límites funcionales

En la tabla siguiente se enumeran los límites funcionales de Azure Digital Twins. 

> [!TIP]
> Para conocer las recomendaciones de modelado que operan dentro de estos límites funcionales, consulte [Procedimientos recomendados para diseñar modelos](../articles/digital-twins/concepts-models.md#best-practices-for-designing-models).

| Área | Capacidad | Límite predeterminado | ¿Ajustable? |
| --- | --- | --- | --- |
| Recurso de Azure | Número de instancias de Azure Digital Twins en una región por suscripción | 10 | Sí |
| Gemelos digitales | Número de gemelos de una instancia de Azure Digital Twins | 200 000 | Sí |
| Gemelos digitales | Número de relaciones entrantes en un solo gemelo | 5\.000 | No |
| Gemelos digitales | Número de relaciones de salida de un solo gemelo | 5\.000 | No |
| Gemelos digitales | Tamaño máximo (del cuerpo JSON en una solicitud PUT o PATCH) de un solo gemelo | 32 KB | No |
| Gemelos digitales | Tamaño máximo de carga de solicitud | 32 KB | No | 
| Enrutamiento | Número de puntos de conexión de una instancia de Azure Digital Twins | 6 | No |
| Enrutamiento | Número de rutas de una instancia de Azure Digital Twins | 6 | Sí |
| Modelos | Número de modelos de una instancia de Azure Digital Twins | 10 000 | Sí |
| Modelos | Número de modelos que se pueden cargar en una única llamada a la API | 250 | No |
| Modelos | Tamaño máximo (del cuerpo JSON en una solicitud PUT o PATCH) de un solo modelo | 1 MB | No |
| Modelos | Número de elementos devueltos en una única página | 100 | No |
| Consultar | Número de elementos devueltos en una única página | 100 | Sí |
| Consultar | Número de expresiones `AND` / `OR` de una consulta | 50 | Sí |
| Consultar | Número de elementos de matriz de una cláusula `IN` / `NOT IN` | 50 | Sí |
| Consultar | Número de caracteres de una consulta | 8,000 | Sí |
| Consultar | Número de `JOINS` de una consulta | 5 | Sí |

### <a name="rate-limits"></a>Límites de frecuencia

En la tabla siguiente se reflejan los límites de frecuencia de distintas API.

| API | Capacidad | Límite predeterminado | ¿Ajustable? |
| --- | --- | --- | --- |
| API de modelos | Número de solicitudes por segundo | 100 | Sí |
| API de Digital Twins | Número de solicitudes de lectura por segundo | 1,000 | Sí |
| API de Digital Twins | Número de solicitudes de revisión por segundo | 1,000 | Sí |
| API de Digital Twins | Número de operaciones de creación y eliminación por segundo en **todos los gemelos y las relaciones** | 50 | Sí |
| API de Digital Twins | Número de operaciones de creación, actualización y eliminación por segundo en un **único gemelo** o sus relaciones | 10 | No |
| API de consulta | Número de solicitudes por segundo | 500 | Sí |
| API de consulta | [Unidades de consulta](../articles/digital-twins/concepts-query-units.md) por segundo | 4\.000 | Sí |
| API de rutas de eventos | Número de solicitudes por segundo | 100 | Sí |

### <a name="other-limits"></a>Otros límites

Los límites de los tipos de datos y los campos de los documentos de DTDL para los modelos de Azure Digital Twins se pueden encontrar en la documentación de la especificación en GitHub: [Lenguaje de definición de Digital Twins (DTDL): versión 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).
 
Los detalles de la latencia de consulta se describen en [Conceptos: Lenguaje de consulta](../articles/digital-twins/concepts-query-language.md#considerations-for-querying). Las limitaciones de características concretas del lenguaje de consulta se pueden encontrar en la [documentación de referencia de consulta](../articles/digital-twins/concepts-query-language.md#reference-documentation).
