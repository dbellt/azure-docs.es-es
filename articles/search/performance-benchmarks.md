---
title: Banco de pruebas de rendimiento
titleSuffix: Azure Cognitive Search
description: Obtenga información acerca del rendimiento de Azure Cognitive Search a través de varios puntos de referencia de rendimiento
author: dereklegenzoff
manager: luisca
ms.author: delegenz
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 9f4473d6c8a584bf60e5c8fe2d69d6a56a55e71d
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108064"
---
# <a name="azure-cognitive-search-performance-benchmarks"></a>Puntos de referencia de rendimiento de Azure Cognitive Search

El rendimiento de Azure Cognitive Search depende de [diversos factores,](search-performance-tips.md), incluido el tamaño del servicio de búsqueda y los tipos de consultas que envía. Para ayudarle a calcular el tamaño del servicio de búsqueda necesario para la carga de trabajo, hemos realizado varios puntos de referencia para documentar el rendimiento de diferentes servicios de búsqueda y configuraciones. Estos puntos de referencia de ninguna manera garantizan un cierto nivel de rendimiento del servicio, pero pueden darle una idea del rendimiento que puede esperar.

Para cubrir una variedad de casos de uso diferentes, hemos realizado puntos de referencia para dos escenarios principales:

* **Búsqueda de comercio electrónico:** este punto de referencia emula un escenario real de comercio electrónico y se basa en la empresa de comercio electrónico escandinava [CDON](https://cdon.com).
* **Búsqueda de documentos:** este escenario se compone de una búsqueda de palabras clave en documentos de texto completo de [Semantic Scholar](http://s2-public-api-prod.us-west-2.elasticbeanstalk.com/corpus/download/). Esto emula una solución de búsqueda de documentos típica.

Aunque estos escenarios reflejan distintos casos de uso, cada escenario es diferente, por lo que siempre se recomienda realizar pruebas de rendimiento de la carga de trabajo individual. Publicamos una [solución de pruebas de rendimiento con JMeter](https://github.com/Azure-Samples/azure-search-performance-testing) para que pueda ejecutar pruebas similares en su propio servicio.

## <a name="testing-methodology"></a>Metodología de prueba

Para realizar puntos de referencia del rendimiento de Azure Cognitive Search, llevamos a cabo pruebas para dos escenarios diferentes con distintos niveles y combinaciones réplica-partición.

Para crear estos puntos de referencia, se usó la siguiente metodología:

1. La prueba comienza en `X` consultas por segundo (QPS) durante 180 segundos. Normalmente, la cifra es de 5 o 10 QPS.
2. A continuación, las QPS se aumentan en `X` y se ejecutan durante otros 180 segundos.
3. Cada 180 segundos, la prueba aumenta en `X` QPS hasta que la latencia media supera los 1000 ms o menos del 99 % de las consultas son correctas.

En el grafo siguiente se muestra el aspecto de la carga de consulta de la prueba:

![Prueba de ejemplo](./media/performance-benchmarks/example-test.png)

Cada escenario usó al menos 10 000 consultas únicas para evitar que las pruebas resultaran demasiado asimétricas por el almacenamiento en caché.

> [!IMPORTANT]
> Estas pruebas solo incluyen cargas de trabajo de consulta. Si espera tener un gran volumen de operaciones de indexación, asegúrese de tenerlo en cuenta durante las pruebas de estimación y rendimiento. Puede encontrar código de ejemplo para simular la indexación en este [tutorial](tutorial-optimize-indexing-push-api.md).

### <a name="definitions"></a>Definiciones

- **QPS máximo**: los siguientes números máximos de QPS se basan en el QPS más alto obtenido en una prueba en la que el 99 % de las consultas se completaron correctamente sin limitación y la latencia media se quedó por debajo de 1000 ms.

- **Porcentaje de QPS máximo:** porcentaje de QPS máximo obtenido para una prueba determinada. Por ejemplo, si una prueba determinada alcanzó un máximo de 100 QPS, el 20 % de QPS máximo sería 20 QPS.

- **Latencia:** latencia del servidor para una consulta; estos números no incluyen el [tiempo de ida y vuelta (RTT)](https://en.wikipedia.org/wiki/Round-trip_delay). Los valores siguientes se encuentran en milisegundos (ms).

### <a name="disclaimer"></a>Declinación de responsabilidades

El código que usamos para ejecutar estas pruebas comparativas está disponible [aquí](https://github.com/Azure-Samples/azure-search-performance-testing/tree/main/other_tools). Cabe señalar que observamos niveles de QPS ligeramente inferiores con la [solución de pruebas de rendimiento de JMeter](https://github.com/Azure-Samples/azure-search-performance-testing) que con los siguientes puntos de referencia. Esta distinción se puede atribuir a diferencias en el estilo de las pruebas. Esto confirma la importancia de hacer que las pruebas de rendimiento se asemejen lo más posible a su carga de trabajo de producción.

Estos puntos de referencia de ninguna manera garantizan un cierto nivel de rendimiento del servicio, pero pueden darle una idea del rendimiento que puede esperar en función de su escenario.

Si tiene comentarios o preguntas, póngase en contacto con nosotros en azuresearch_contact@microsoft.com.

## <a name="benchmark-1-e-commerce-search"></a>Punto de referencia 1: búsqueda de comercio electrónico

:::row:::
   :::column span="1":::
      ![Logotipo de CDON](./media/performance-benchmarks/cdon-logo-160px2.png)
   :::column-end:::
   :::column span="3":::
      Este punto de referencia se creó en colaboración con la empresa de comercio electrónico [CDON,](https://cdon.com) el marketplace en línea más grande de la región nórdica, con operaciones en Suecia, Finlandia, Noruega y Dinamarca. A través de sus 1500 comerciantes, CDON ofrece un amplio catálogo que incluye más de 8 millones de productos. En 2020, CDON tuvo más de 120 millones de visitantes y dos millones de clientes activos. Puede obtener más información acerca de cómo CDON usa Azure Cognitive Search en [este artículo](https://pulse.microsoft.com/transform/na/fa1-how-cdon-has-been-using-technology-to-become-the-leading-marketplace-in-the-nordics/).
   :::column-end:::
:::row-end:::

Para ejecutar estas pruebas, hemos usado una instantánea del índice de búsqueda de producción de CDON y miles de consultas únicas de su [sitio web](https://cdon.com).

### <a name="scenario-details"></a>Detalles del escenario

- **Número de documentos**: 6 000 000 
- **Tamaño del índice**: 20 GB
- **Esquema de índice**: índice amplio con 250 campos en total, 25 campos que permiten búsquedas y 200 campos que permiten facetas o filtros.
- **Tipos de consulta**: consultas de búsqueda de texto completo que incluyen facetas, filtros, ordenación y perfiles de puntuación

### <a name="s1-performance"></a>Rendimiento S1

#### <a name="queries-per-second"></a>Consultas por segundo

En el gráfico siguiente se muestra la carga de consulta más alta que un servicio podría controlar durante un largo período de tiempo en términos de consultas por segundo (QPS).

![QPS más alto que se puede mantener, comercio electrónico s1](./media/performance-benchmarks/s1-ecom-qps.png)

#### <a name="query-latency"></a>Latencia de consultas

La latencia de las consultas varía en función de la carga del servicio. Los servicios con cargas elevadas tendrán una latencia de consulta media mayor. En la tabla siguiente se muestran los percentiles 25, 50, 75, 90, 95 y 99 de latencia de consulta para tres niveles de uso diferentes.

| Porcentaje de QPS máximo  | Latencia promedio | 25 % | 75 % | 90% | 95 % | 99%|
|---|---|---|---| --- | --- | --- | 
| 20%  | 104 ms  | 35 ms  | 115 ms   | 177 ms | 257 ms | 738 ms |
| 50 %  | 140 ms  | 47 ms  | 144 ms   | 241 ms | 400 ms | 1175 ms |
| 80 %  | 239 ms  | 77 ms  | 248 ms   | 466 ms | 763 ms | 1752 ms | 


### <a name="s2-performance"></a>Rendimiento S2

#### <a name="queries-per-second"></a>Consultas por segundo

En el gráfico siguiente se muestra la carga de consulta más alta que un servicio podría controlar durante un largo período de tiempo en términos de consultas por segundo (QPS).

![QPS más alto que se puede mantener, comercio electrónico s2](./media/performance-benchmarks/s2-ecom-qps.png)

#### <a name="query-latency"></a>Latencia de consultas

La latencia de las consultas varía en función de la carga del servicio. Los servicios con cargas elevadas tendrán una latencia de consulta media mayor. En la tabla siguiente se muestran los percentiles 25, 50, 75, 90, 95 y 99 de latencia de consulta para tres niveles de uso diferentes.

| Porcentaje de QPS máximo  | Latencia promedio | 25 % | 75 % | 90% | 95 % | 99%|
|---|---|---|---| --- | --- | --- | 
| 20%  | 56 ms | 21 ms | 68 ms  | 106 ms  | 132 ms | 210 ms | 
| 50 %  | 71 ms  | 26 ms  | 83 ms   | 132 ms | 177 ms | 329 ms |
| 80 %  | 140 ms  | 47 ms  | 153 ms   | 293 ms | 452 ms | 924 ms | 

### <a name="s3-performance"></a>Rendimiento S3

#### <a name="queries-per-second"></a>Consultas por segundo

En el gráfico siguiente se muestra la carga de consulta más alta que un servicio podría controlar durante un largo período de tiempo en términos de consultas por segundo (QPS).

![QPS más alto que se puede mantener, comercio electrónico s3](./media/performance-benchmarks/s3-ecom-qps.png)

En este caso, vemos que agregar una segunda partición aumenta significativamente el número máximo de QPS, pero agregar una tercera partición disminuye los rendimientos marginales. Es probable que la mejora menor se deba a que todos los datos ya se están incorporando en la memoria activa del nivel S3 con solo dos particiones.

#### <a name="query-latency"></a>Latencia de consultas

La latencia de las consultas varía en función de la carga del servicio. Los servicios con cargas elevadas tendrán una latencia de consulta media mayor. En la tabla siguiente se muestran los percentiles 25, 50, 75, 90, 95 y 99 de latencia de consulta para tres niveles de uso diferentes.

| Porcentaje de QPS máximo  | Latencia promedio | 25 % | 75 % | 90% | 95 % | 99%|
|---|---|---|---| --- | --- | --- |
| 20%  | 50 ms  | 20 ms  | 64 ms   | 83 ms | 98 ms | 160 ms |
| 50 %  | 62 ms  | 24 ms  | 80 ms   | 107 ms | 130 ms | 253 ms |
| 80 %  | 115 ms  | 38 ms  | 121 ms   | 218 ms | 352 ms | 828 ms |

## <a name="benchmark-2-document-search"></a>Punto de referencia 2: búsqueda de documentos

### <a name="scenario-details"></a>Detalles del escenario

- **Número de documentos**: 7,5 millones
- **Tamaño del índice**: 22 GB
- **Esquema de índice**: 23 campos; 8 que permiten búsquedas, 10 que permiten facetas o filtros
- **Tipos de consulta**: búsquedas de palabras clave con facetas y resaltado de coincidencias

### <a name="s1-performance"></a>Rendimiento S1

#### <a name="queries-per-second"></a>Consultas por segundo

En el gráfico siguiente se muestra la carga de consulta más alta que un servicio podría controlar durante un largo período de tiempo en términos de consultas por segundo (QPS).

![QPS más alto que se puede mantener, búsqueda de documentos s1](./media/performance-benchmarks/s1-docsearch-qps.png)

#### <a name="query-latency"></a>Latencia de consultas

La latencia de las consultas varía en función de la carga del servicio. Los servicios con cargas elevadas tendrán una latencia de consulta media mayor. En la tabla siguiente se muestran los percentiles 25, 50, 75, 90, 95 y 99 de latencia de consulta para tres niveles de uso diferentes.

| Porcentaje de QPS máximo  | Latencia promedio | 25 % | 75 % | 90% | 95 % | 99%|
|---|---|---|---| --- | --- | --- |
| 20%  | 67 ms  | 44 ms  | 77 ms   | 103 ms | 126 ms | 216 ms |
| 50 %  | 93 ms  | 59 ms  | 110 ms   | 150 ms | 184 ms | 304 ms |
| 80 %  | 150 ms  | 96 ms  | 184 ms   | 248 ms | 297 ms | 424 ms |

### <a name="s2-performance"></a>Rendimiento S2

#### <a name="queries-per-second"></a>Consultas por segundo

En el gráfico siguiente se muestra la carga de consulta más alta que un servicio podría controlar durante un largo período de tiempo en términos de consultas por segundo (QPS).

![QPS más alto que se puede mantener, búsqueda de documentos s2](./media/performance-benchmarks/s2-docsearch-qps.png)

#### <a name="query-latency"></a>Latencia de consultas

La latencia de las consultas varía en función de la carga del servicio. Los servicios con cargas elevadas tendrán una latencia de consulta media mayor. En la tabla siguiente se muestran los percentiles 25, 50, 75, 90, 95 y 99 de latencia de consulta para tres niveles de uso diferentes.

| Porcentaje de QPS máximo  | Latencia promedio | 25 % | 75 % | 90% | 95 % | 99%|
|---|---|---|---| --- | --- | --- |
| 20%  | 45 ms  | 31 ms  | 55 ms   | 73 ms | 84 ms | 109 ms |
| 50 %  | 63 ms  | 39 ms  | 81 ms   | 106 ms | 123 ms | 163 ms |
| 80 %  | 115 ms  | 73 ms  | 145 ms   | 191 ms | 224 ms | 291 ms |

## <a name="takeaways"></a>Puntos clave

A través de estos puntos de referencia, puede obtener una idea del rendimiento que ofrece Azure Cognitive Search. También puede ver la diferencia entre los servicios en distintos niveles.

Algunas de las conclusiones principales de estos puntos de referencia son:

* Un nivel S2 normalmente puede controlar al menos cuatro veces el volumen de consultas de un nivel S1
* Un nivel S2 normalmente tendrá una latencia menor que un nivel S1 en volúmenes de consulta comparables
* A medida que agregue réplicas, el QPS que un servicio puede controlar normalmente escalará linealmente (por ejemplo, si una réplica puede controlar 10 QPS, cinco réplicas normalmente podrán controlar 50 QPS).
* Cuanto mayor sea la carga en el servicio, mayor será la latencia media.

También puede ver que el rendimiento podría variar drásticamente entre escenarios. Si no está logrando el rendimiento esperado, consulte las [sugerencias para mejorar el rendimiento](search-performance-tips.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto los puntos de referencia de rendimiento, puede obtener más información acerca de cómo analizar el rendimiento de Cognitive Search y los factores clave que influyen en el rendimiento.

> [!div class="nextstepaction"]
> [Análisis de rendimiento](search-performance-analysis.md)
> [Sugerencias para mejorar el rendimiento](search-performance-tips.md)