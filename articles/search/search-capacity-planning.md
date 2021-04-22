---
title: Estimación de la capacidad para cargas de trabajo de consultas e índices
titleSuffix: Azure Cognitive Search
description: Ajuste los recursos de proceso de réplica y partición en Azure Cognitive Search, donde el precio de cada recurso se basa en unidades de búsqueda facturables.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/06/2021
ms.openlocfilehash: b1f742c1de259f6c1c06d9b31a8788699f0b8426
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580027"
---
# <a name="estimate-and-manage-capacity-of-an-azure-cognitive-search-service"></a>Estimación y administración de la capacidad de un servicio de Azure Cognitive Search

Antes de [aprovisionar un servicio de búsqueda](search-create-service-portal.md) y de adquirir un plan de tarifa específico, dedique unos minutos a comprender cómo funciona la capacidad y cómo se pueden ajustar las réplicas y las particiones para acomodar la fluctuación de cargas de trabajo.

La capacidad es una función del [nivel de servicio](search-sku-tier.md) que establece el almacenamiento máximo por servicio, por partición y los límites máximos en el número de objetos que se pueden crear. El nivel Básico está diseñado para aplicaciones que tienen unos requisitos de almacenamiento modestos (solo una partición), pero con la capacidad de ejecutarse en una configuración de alta disponibilidad (3 réplicas). Otros niveles están diseñados para cargas de trabajo o patrones específicos, como la arquitectura multiinquilino. Internamente, los servicios creados en esos niveles se benefician del hardware que ayuda a esos escenarios.

La arquitectura de escalabilidad de Azure Cognitive Search se basa en combinaciones flexibles de réplicas y particiones para que pueda variar la capacidad en función de si necesita una mayor potencia de indexación o consulta. Una vez creado un servicio, puede aumentar o reducir el número de réplicas o particiones de forma independiente. Los costos aumentarán con cada recurso físico adicional, pero una vez finalizadas las cargas de trabajo de gran tamaño, puede reducir la escala para reducir la factura. Según el plan y el tamaño del ajuste, el aumento o reducción de la capacidad puede tardar desde 15 minutos a varias horas.

Al modificar la asignación de réplicas y particiones, se recomienda usar Azure Portal. El portal aplica límites a las combinaciones permitidas que se mantengan por debajo de los límites máximos de un plan. No obstante, si necesita un enfoque de aprovisionamiento basado en script o en código, [Azure PowerShell](search-manage-powershell.md) o la [API REST de administración](/rest/api/searchmanagement/services) son soluciones alternativas.

## <a name="concepts-search-units-replicas-partitions-shards"></a>Conceptos: unidades de búsqueda, réplicas, particiones y particiones de base de datos

La capacidad se expresa en *unidades de búsqueda*, que se pueden asignar en combinaciones de *particiones* y *réplicas* mediante un mecanismo de *particionamiento* subyacente para admitir configuraciones flexibles:

| Concepto  | Definición|
|----------|-----------|
|*Unidad de búsqueda* | Un único incremento de la capacidad total disponible (36 unidades). También es la unidad de facturación de un servicio de Azure Cognitive Search. Se requiere un mínimo de una unidad para ejecutar el servicio.|
|*Réplica* | Instancias del servicio de búsqueda, que se utilizan principalmente para equilibrar la carga de las operaciones de consulta. Cada réplica hospeda una copia de un índice. Si asigna tres réplicas, tendrá tres copias de un índice disponibles para atender las solicitudes de consulta.|
|*Partición* | Almacenamiento físico y E/S para operaciones de lectura y escritura (por ejemplo, al volver a compilar o actualizar un índice). Cada partición tiene un segmento del índice total. Si asigna tres particiones, el índice se divide en tercios. |
|*Partición de base de datos* | Un fragmento de un índice. Azure Cognitive Search divide cada índice en particiones de base de datos para que el proceso de agregar particiones sea más rápido (al mover las particiones de base de datos a nuevas unidades de búsqueda).|

En el siguiente diagrama se muestra la relación entre réplicas, particiones, particiones de base de datos y unidades de búsqueda. Muestra un ejemplo de cómo se distribuye un solo índice entre cuatro unidades de búsqueda de un servicio con dos réplicas y dos particiones. Cada una de las cuatro unidades de búsqueda almacena solo la mitad de las particiones de base de datos del índice. Las unidades de búsqueda de la columna izquierda almacenan la primera mitad de las particiones de base de datos, que comprende la primera partición, mientras que las de la columna derecha almacenan la segunda mitad de las particiones de base de datos, que comprende la segunda partición. Dado que hay dos réplicas, hay dos copias de cada partición de base de datos del índice. Las unidades de búsqueda de la fila superior almacenan una copia, que comprende la primera réplica, mientras que las de la fila inferior almacenan otra copia, que comprende la segunda réplica.

:::image type="content" source="media/search-capacity-planning/shards.png" alt-text="Los índices de búsqueda se particionan entre particiones.":::

El diagrama anterior es solo un ejemplo. Hay muchas combinaciones de particiones y réplicas posibles, hasta un máximo de 36 unidades de búsqueda totales.

En Cognitive Search, la administración de particiones de base de datos es un detalle de implementación y no es configurable, pero el saber que un índice está particionado ayuda a comprender las anomalías ocasionales en los comportamientos de clasificación y Autocompletar:

+ Anomalías de clasificación: las puntuaciones de búsqueda se calculan primero en el nivel de partición de base de datos y luego se suman en un único conjunto de resultados. En función de las características del contenido de la partición de base de datos, las coincidencias de una partición de base de datos pueden tener una clasificación mayor que las de otra. Si observa clasificaciones intuitivas del contador en los resultados de búsqueda, lo más probable es que se deba a los efectos del particionamiento, especialmente si los índices son pequeños. Puede evitar estas anomalías de clasificación si opta por [calcular las puntuaciones de forma global en todo el índice](index-similarity-and-scoring.md#scoring-statistics-and-sticky-sessions), aunque esto conlleva una penalización de rendimiento.

+ Anomalías de Autocompletar: las consultas de tipo Autocompletar, donde las coincidencias se realizan según los primeros caracteres de un término especificado parcialmente, aceptan un parámetro aproximado que perdona pequeñas desviaciones de ortografía. En Autocompletar, la coincidencia aproximada se restringe a los términos de la partición de base de datos actual. Por ejemplo, si una partición de base de datos contiene "Microsoft" y se escribe un término parcial "micor", el motor de búsqueda combinará con "Microsoft" en esa partición de base de datos, pero no en otras particiones de base de datos que contengan las partes restantes del índice.

## <a name="approaching-estimation"></a>Aproximándose a la estimación

La capacidad y los costos de ejecución del servicio están relacionados. Los niveles imponen límites en dos niveles: contenido (recuento de índices de un servicio, por ejemplo) y almacenamiento. Es importante tener en cuenta estos dos aspectos, porque el límite real será el que se alcance primero.

Los recuentos de índices y otros objetos normalmente vienen determinados por los requisitos empresariales y de ingeniería. Por ejemplo, podría tener varias versiones del mismo índice para desarrollo, pruebas y producción activos.

Las necesidades de almacenamiento vienen determinadas por el tamaño de los índices que espera compilar. No hay ninguna heurística o generalización sólida que ayude con las estimaciones. La única manera de determinar el tamaño de un índice es [compilar uno](search-what-is-an-index.md). Su tamaño se basará en los datos importados, en el análisis de texto y en la configuración del índice, por ejemplo, si habilita los proveedores de sugerencias, el filtrado y la ordenación.

Para la búsqueda de texto completo, la estructura de datos principal es una estructura de [índice invertido](https://en.wikipedia.org/wiki/Inverted_index), que tiene características diferentes de los datos de origen. Para un índice invertido, el tamaño y la complejidad vienen determinados por el contenido, y no necesariamente por la cantidad de datos que se incorporan. Un origen de datos de gran tamaño con mucha redundancia podría dar lugar a un índice más pequeño que un conjunto de datos más pequeño que incluya contenido muy variable. Así que es poco probable deducir el tamaño del índice en función del tamaño del conjunto de datos original.

Los atributos del índice, como la habilitación de filtros y la ordenación, afectarán a los requisitos de almacenamiento. El uso de proveedores de sugerencias también tiene implicaciones de almacenamiento. Para obtener más información, consulte [Atributos y tamaño de índice](search-what-is-an-index.md#index-size).

> [!NOTE]
> Aunque el cálculo de las necesidades futuras de índices y almacenamiento se hace a partir de suposiciones, vale la pena hacerlo. Si la capacidad de un nivel resulta demasiado baja, tendrá que aprovisionar un nuevo servicio en un nivel superior y luego [volver a cargar los índices](search-howto-reindex.md). No existe ninguna actualización local de un servicio de un nivel a otro.
>

### <a name="estimate-with-the-free-tier"></a>Estimación con el nivel Gratis

Un enfoque para calcular la capacidad es empezar con el nivel Gratis. Recuerde que el servicio Gratis ofrece hasta 3 índices, 50 MB de almacenamiento y 2 minutos de tiempo de indexación. Puede resultar complicado calcular un tamaño de índice proyectado con estas restricciones, pero estos son los pasos:

+ [Cree un servicio gratuito](search-create-service-portal.md).
+ Prepare un conjunto de datos pequeño y representativo.
+ Cree un índice y cargue los datos. Si el conjunto de datos se puede hospedar en el origen de datos de Azure compatible con los indexadores, puede usar el [asistente para la importación de datos en el portal](search-get-started-portal.md) para crear y cargar el índice. De lo contrario, debe usar REST y [Postman](search-get-started-rest.md) o [Visual Studio Code](search-get-started-vs-code.md) para crear el índice e insertar los datos. El modelo de inserción requiere que el formato de los datos sea el de los documentos JSON, donde los campos del documento se corresponden con los campos del índice.
+ Recopile información sobre el índice, como el tamaño. Las características y los atributos afectan al almacenamiento. Por ejemplo, al agregar proveedores de sugerencias (consultas al escribir), aumentan los requisitos de almacenamiento. 

  Con el mismo conjunto de datos, puede intentar crear varias versiones de un índice, con atributos diferentes en cada campo, para ver cómo varían los requisitos de almacenamiento. Para obtener más información, vea ["Implicaciones de almacenamiento" en Creación de un índice básico](search-what-is-an-index.md#index-size).

Con una estimación aproximada, podría doblar esa cantidad a fin de presupuestar dos índices (desarrollo y producción) y luego elegir el nivel en consecuencia.

### <a name="estimate-with-a-billable-tier"></a>Estimación con un nivel facturable

Los recursos dedicados pueden adaptarse a mayores tiempos de muestreo y procesamiento para realizar estimaciones más realistas de cantidad de índices, tamaño y volúmenes de consulta durante el desarrollo. Algunos clientes pasan directamente a un nivel facturable y luego reevalúan a medida que el proyecto de desarrollo madura.

1. [Revise los límites del servicio en cada nivel](./search-limits-quotas-capacity.md#index-limits) para determinar si los niveles más bajos pueden admitir la cantidad de índices que necesita. En los niveles Básico, S1 y S2, los límites de índices son 15, 50 y 200 respectivamente. El nivel Almacenamiento optimizado tiene un límite de 10 índices porque se ha diseñado para admitir un número reducido de índices muy grandes.

1. [Cree un servicio en un nivel facturable](search-create-service-portal.md):

    + Comience por abajo, en Básico o S1, si no está seguro de la carga proyectada.
    + Comience alto, en S2 o incluso S3, si las pruebas incluyen indexación a gran escala y cargas de consulta.
    + Empiece con Almacenamiento optimizado, en L1 o L2, si va a indexar una gran cantidad de datos y la carga de consultas es relativamente baja, como con una aplicación empresarial interna.

1. [Genere un índice inicial](search-what-is-an-index.md) para determinar cómo se traducen los datos de origen a un índice. Esta es la única manera de calcular el tamaño del índice. 

1. [Supervise el almacenamiento, los límites del servicio, el volumen de consultas y la latencia](search-monitor-usage.md) en el portal. El portal muestra las consultas por segundo, las consultas limitadas y la latencia de búsqueda. Todos estos valores pueden ayudarle a decidir si ha seleccionado el nivel correcto.

1. Agregue réplicas si necesita alta disponibilidad o si experimenta un rendimiento de consultas lento.

   No hay instrucciones sobre cuántas réplicas se necesitan para acomodar las cargas de consulta. El rendimiento de consulta depende de la complejidad de la consulta y de las cargas de trabajo competitivas. Si bien la adición de réplicas genera claramente un mejor rendimiento, el resultado final no será estrictamente lineal: la adición de tres réplicas no garantiza el triple rendimiento. Para obtener una guía sobre cómo estimar las QPS de la solución, consulte [Escalado para mejorar el rendimiento](search-performance-optimization.md) y [Supervisión de consultas](search-monitor-queries.md).

> [!NOTE]
> Los requisitos de almacenamiento pueden parecer excesivos si incluye datos que nunca se buscarán. Lo ideal es que los documentos contengan solo los datos que necesita para la experiencia de búsqueda. No es posible realizar búsquedas en datos binarios, así que deben almacenarse por separado (quizás en Azure Blob Storage o en una tabla de Azure). Después hay que agregar un campo en el índice para mantener una referencia de URL a los datos externos. El tamaño máximo de un documento de búsqueda individual es 16 MB (o menos si va a cargar de forma masiva varios documentos en una sola solicitud). Para más información, vea [Límites de servicio en Azure Cognitive Search](search-limits-quotas-capacity.md).
>

**Consideraciones del volumen de consultas**

Las consultas por segundo (QPS) son una medida importante durante la optimización del rendimiento, pero solo suele ser una consideración de nivel si espera un volumen elevado de consultas desde el principio.

El nivel Estándar puede proporcionar un equilibrio entre réplicas y particiones. Puede aumentar el tiempo de respuesta de las consultas mediante la adición de réplicas para el equilibrio de carga o agregar particiones para el procesamiento paralelo. Luego puede ajustar el rendimiento después de aprovisionar el servicio.

Si prevé grandes volúmenes de consultas sostenidos desde el principio, considere la opción de los niveles Estándar superiores, respaldados por un hardware más potente. Luego puede desconectar particiones y réplicas, o incluso cambiar a un servicio de nivel inferior, si no llega a tener los volúmenes de consultas previstos. Para más información sobre cómo calcular el rendimiento de las consultas, consulte [Rendimiento y optimización de Azure Cognitive Search](search-performance-optimization.md).

El nivel Almacenamiento optimizado es útil para cargas de trabajo de datos de gran tamaño, ya que admite más almacenamiento de índice general disponible para cuando los requisitos de latencia de consulta sean menos importantes. Tendrá que seguir usando réplicas adicionales para el equilibrio de carga y particiones adicionales para el procesamiento paralelo. Luego puede ajustar el rendimiento después de aprovisionar el servicio.

**Contratos de nivel de servicio**

El nivel Gratis y las características en versión preliminar no están cubiertos por [Acuerdos de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/search/v1_0/). Para todos los niveles facturables, los SLA tomarán efecto cuando se aprovisione suficiente redundancia para el servicio. Necesitará dos o más réplicas para los SLA de consulta (lectura). Necesitará tres o más réplicas para los SLA de consulta e indización (lectura-escritura). El número de particiones no afecta a los SLA.

## <a name="tips-for-capacity-planning"></a>Sugerencias para el planeamiento de capacidad

+ Cree métricas sobre las consultas y recopile datos sobre los patrones de uso (consultas durante el horario laboral, indexación durante las horas de menor uso). Use estos datos para respaldar las decisiones de aprovisionamiento del servicio. Aunque no resulta práctico en una cadencia horaria o diaria, puede ajustar dinámicamente las particiones y los recursos para adaptar los cambios planeados en el volumen de consultas. También puede adaptar los cambios no planeados pero sostenidos si los niveles se mantienen lo suficiente para garantizar que se puedan realizar acciones.

+ Recuerde que la única desventaja de un aprovisionamiento inferior es que es posible que deba anular un servicio si los requisitos reales son mayores que los que había previsto. Para evitar la interrupción del servicio, se crea un nuevo servicio en un nivel superior y se ejecuta en paralelo hasta que todas las solicitudes y las aplicaciones se dirijan al nuevo punto de conexión.

## <a name="when-to-add-capacity"></a>Cuándo ampliar la capacidad

Inicialmente, se asigna un servicio a un nivel mínimo de recursos que consta de una partición y una réplica. El [plan que elija](search-sku-tier.md) determina el tamaño y la velocidad de la partición y cada plan se optimiza alrededor de un conjunto de características que se adapta a varios escenarios. Si elige un plan de gama superior, puede que necesite menos particiones que si elige S1. Una de las preguntas que tendrá que responder mediante pruebas autodirigidas es si una partición más grande y más cara aumentará más el rendimiento que dos particiones más baratas en un servicio aprovisionado con un plan inferior.

Un único servicio debe tener recursos suficientes para controlar todas las cargas de trabajo (indexación y consultas). Ninguna carga de trabajo se ejecuta en segundo plano. Puede programar la indexación en horas en las que las solicitudes de consulta son menos frecuentes por naturaleza, pero el servicio no dará prioridad a una tarea sobre otra. Además, una determinada cantidad de redundancia suaviza el rendimiento de la consulta cuando los servicios o nodos se están actualizando internamente.

Algunas directrices para determinar si se debe ampliar la capacidad incluyen:

+ Cumplimiento de los criterios de alta disponibilidad para el Acuerdo de Nivel de Servicio
+ La frecuencia de los errores HTTP 503 va en aumento
+ Se esperan grandes volúmenes de consultas

Como norma general, las aplicaciones de búsqueda tienden a necesitar más réplicas que particiones, sobre todo cuando las operaciones de servicio están orientadas a las cargas de trabajo de consulta. Cada réplica es una copia del índice, lo que permite que el servicio equilibre la carga de las solicitudes en varias copias. Azure Cognitive Search administra todo el equilibrio de carga y la replicación de un índice, y puede modificar el número de réplicas asignado a su servicio en cualquier momento. Puede asignar hasta 12 réplicas en un servicio de búsqueda estándar y 3 réplicas en un servicio de búsqueda básico. La asignación de réplicas se puede realizar desde [Azure Portal](search-create-service-portal.md) o una de las opciones de programación.

Las aplicaciones de búsqueda que requieren una actualización de datos casi en tiempo real necesitan una proporción mayor de particiones que de réplicas. Al agregarse particiones, se distribuyen las operaciones de lectura y escritura entre un número mayor de recursos de proceso. Además, se cuenta con más espacio en disco para almacenar documentos e índices adicionales.

Por último, las consultas en índices de mayor tamaño tardan más tiempo en realizarse. Por lo tanto, es posible que con cada aumento incremental de las particiones sea necesario también un aumento menor, pero proporcional, de las réplicas. La complejidad y el volumen de las consultas afectarán a la rapidez con que se ejecuta la consulta.

> [!NOTE]
> La adición de más réplicas o particiones aumenta el costo de ejecución del servicio y puede generar pequeñas variaciones en cómo se ordenan los resultados. Asegúrese de activar la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para comprender las implicaciones que tiene en la facturación el agregar más nodos. El [gráfico siguiente](#chart) puede ayudarle a establecer una referencia cruzada con el número de unidades de búsqueda necesarias para una configuración específica. Para obtener más información sobre cómo las réplicas adicionales afectan al procesamiento de las consultas, visite [Organización de los resultados](search-pagination-page-layout.md#ordering-results).

<a name="adjust-capacity"></a>

## <a name="add-or-reduce-replicas-and-partitions"></a>Adición o reducción de réplicas y particiones

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y seleccione su servicio de búsqueda.

1. En **Configuración**, abra la página **Escala** para modificar réplicas y particiones. 

   En la siguiente captura de pantalla se muestra un estándar básico aprovisionado con una réplica y una partición. La fórmula de la parte inferior indica cuántas unidades de búsqueda se usan (1). Si el precio por unidad era de 100 USD (no un precio real), el costo de ejecución de este servicio sería, de media, de 100 USD.

   :::image type="content" source="media/search-capacity-planning/1-initial-values.png" alt-text="Página de escala que muestra los valores actuales" border="true":::

1. Use el control deslizante para aumentar o reducir el número de particiones. La fórmula de la parte inferior indica cuántas unidades de búsqueda se usan. Seleccione **Guardar**.

   En este ejemplo se agrega una segunda réplica y también otra partición. Observe el recuento de unidades de búsqueda; ahora es cuatro porque la fórmula de facturación son las réplicas multiplicadas por las particiones (2 x 2). Cuanto más se duplica la capacidad, más se duplica el costo de ejecución del servicio. Si el costo de la unidad de búsqueda era de 100 USD, la nueva factura mensual sería ahora de 400 USD.

   Para ver los costos por unidad actuales de cada nivel, visite la [Página de precios](https://azure.microsoft.com/pricing/details/search/).

   :::image type="content" source="media/search-capacity-planning/2-add-2-each.png" alt-text="Adición de réplicas y particiones" border="true":::

1. Después de guardar, puede comprobar las notificaciones para confirmar que la acción se ha realizado correctamente.

   :::image type="content" source="media/search-capacity-planning/3-save-confirm.png" alt-text="Guardar cambios" border="true":::

   Los cambios en la capacidad pueden tardar entre 15 minutos y varias horas en completarse. No se puede cancelar una vez que se ha iniciado el proceso y no hay ningún tipo de supervisión en tiempo real de los ajustes de réplica y partición. Sin embargo, el siguiente mensaje se puede seguir viendo mientras se están realizando los cambios.

   :::image type="content" source="media/search-capacity-planning/4-updating.png" alt-text="Mensaje de estado del portal" border="true":::

> [!NOTE]
> Una vez que se aprovisiona un servicio, no se puede actualizar a un plan superior. Debe crear un servicio de búsqueda en el nivel nuevo y volver a cargar los índices. Consulte [Creación de un servicio Azure Cognitive Search mediante Azure Portal](search-create-service-portal.md) para obtener ayuda con el proceso de aprovisionamiento de servicios.
>
> Además, las particiones y las réplicas las administra el servicio de manera exclusiva e interna. No hay concepto de afinidad de procesador ni de asignación de una carga de trabajo a un nodo específico.
>

<a id="chart"></a>

## <a name="partition-and-replica-combinations"></a>Combinaciones de particiones y réplicas

Un servicio del nivel Básico puede tener exactamente 1 partición y hasta 3 réplicas para un límite máximo de 3 SU. El único recurso que puede ajustarse son las réplicas. Se necesita un mínimo de 2 réplicas para lograr una alta disponibilidad en las consultas.

Todos los servicios de búsqueda de los niveles Estándar y Almacenamiento optimizado pueden suponer las siguientes combinaciones de réplicas y particiones, con el límite de 36 unidades de búsqueda (SU) permitido para estos niveles. 

|   | **1 partición** | **2 particiones** | **3 particiones** | **4 particiones** | **6 particiones** | **12 particiones** |
| --- | --- | --- | --- | --- | --- | --- |
| **1 réplica** |1 unidad de búsqueda |2 unidades de búsqueda |3 unidades de búsqueda |4 unidades de búsqueda |6 unidades de búsqueda |12 unidades de búsqueda |
| **2 réplicas** |2 unidades de búsqueda |4 unidades de búsqueda |6 unidades de búsqueda |8 unidades de búsqueda |12 unidades de búsqueda |24 unidades de búsqueda |
| **3 réplicas** |3 unidades de búsqueda |6 unidades de búsqueda |9 unidades de búsqueda |12 unidades de búsqueda |18 unidades de búsqueda |36 unidades de búsqueda |
| **4 réplicas** |4 unidades de búsqueda |8 unidades de búsqueda |12 unidades de búsqueda |16 unidades de búsqueda |24 unidades de búsqueda |N/D |
| **5 réplicas** |5 unidades de búsqueda |10 unidades de búsqueda |15 unidades de búsqueda |20 unidades de búsqueda |30 unidades de búsqueda |N/D |
| **6 réplicas** |6 unidades de búsqueda |12 unidades de búsqueda |18 unidades de búsqueda |24 unidades de búsqueda |36 unidades de búsqueda |N/D |
| **12 réplicas** |12 unidades de búsqueda |24 unidades de búsqueda |36 unidades de búsqueda |N/D |N/D |N/D |

En el sitio web de Azure se explican con detalle la capacidad, los precios y las unidades de búsqueda. Para obtener más información, consulte [Detalles de precios](https://azure.microsoft.com/pricing/details/search/).

> [!NOTE]
> El número de réplicas y particiones se dividirse equitativamente en 12 (en concreto, 1, 2, 3, 4, 6, 12). Esto se debe a que Azure Cognitive Search divide previamente cada índice en 12 particiones para que se pueda repartir en porciones iguales entre todas las particiones. Por ejemplo, si su servicio tiene tres particiones y crea un nuevo índice, cada partición contendrá 4 particiones del índice. La manera en que Azure Cognitive Search particiona un índice es un detalle de implementación, sujeto a cambios en la futura versión. Aunque el número es 12 hoy, no debe esperar que ese número se siempre 12 en el futuro.
>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administrar costos](search-sku-manage-costs.md)