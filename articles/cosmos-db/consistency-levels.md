---
title: Niveles de coherencia en Azure Cosmos DB
description: Azure Cosmos DB tiene cinco niveles de coherencia para ayudar a equilibrar la coherencia, la disponibilidad y la latencia finales.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 03/22/2021
ms.openlocfilehash: 31c5be9ce48ffea8ebd23e893e2d77e6365d2327
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467673"
---
# <a name="consistency-levels-in-azure-cosmos-db"></a>Niveles de coherencia en Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Las bases de datos distribuidas que dependen de la replicación para alta disponibilidad, baja latencia o ambas, deben equilibrar la coherencia de lectura, la disponibilidad, la latencia y el rendimiento, tal como se define en el [teorema de PACLC](https://en.wikipedia.org/wiki/PACELC_theorem). La linearización del modelo de coherencia fuerte es el estándar de oro de la programación de datos, pero agrega un precio excesivo a las latencias de escritura mayores debido a que los datos tienen que replicarse y confirmarse entre grandes distancias. Una coherencia fuerte también puede verse afectada por una disponibilidad reducida (durante errores) porque los datos no se puedan replicar y confirmar en cada región. La coherencia posible ofrece una mayor disponibilidad y un mejor rendimiento, pero es más difícil programar aplicaciones porque es posible que los datos no sean totalmente coherentes en todas las regiones.

La mayoría de las bases de datos NoSQL distribuidas disponibles comercialmente en el mercado hoy en día solo ofrecen coherencia fuerte y posible. Azure Cosmos DB ofrece cinco niveles bien definidos. De más fuerte a más débil, los niveles son:

- *Fuerte*
- *Obsolescencia limitada*
- *De sesión*
- *De prefijo coherente*
- *Posible*

Cada nivel proporciona equilibrio entre la disponibilidad y el rendimiento. En la imagen siguiente se muestran los distintos niveles de coherencia como un espectro.

:::image type="content" source="./media/consistency-levels/five-consistency-levels.png" alt-text="Coherencia como espectro" border="false" :::

Los niveles de coherencia son independientes de la región y están garantizados para todas las operaciones, independientemente de la región desde la que se realizan las operaciones de lectura y escritura, del número de regiones asociadas con la cuenta de Azure Cosmos o de si la cuenta está configurada para una o varias regiones de escritura.

## <a name="consistency-levels-and-azure-cosmos-db-apis"></a>Niveles de coherencia y API de Azure Cosmos DB

Azure Cosmos DB también proporciona compatibilidad nativa con las API compatibles con el protocolo de conexión para bases de datos populares. Entre ellas se incluyen MongoDB, Apache Cassandra, Gremlin y Azure Table Storage. Al usar Gremlin API y Table API, se usa el nivel de coherencia predeterminado configurado en la cuenta de Azure Cosmos. Para más información sobre la asignación del nivel de coherencia entre Cassandra API o la API para MongoDB y los niveles de coherencia de Azure Cosmos DB, consulte [Asignación de coherencia de Cassandra API](cassandra-consistency.md) y [Asignación de coherencia de API para MongoDB](mongodb-consistency.md).

## <a name="scope-of-the-read-consistency"></a>Ámbito de la coherencia de lectura

La coherencia de lectura se aplica a una sola operación de lectura limitada a una partición lógica. Un cliente remoto o un procedimiento almacenado pueden emitir la operación de lectura.

## <a name="configure-the-default-consistency-level"></a>Configuración del nivel de coherencia predeterminado

Puede configurar el nivel de coherencia predeterminado de su cuenta de Azure Cosmos en cualquier momento. El nivel de coherencia predeterminado configurado en su cuenta se aplica a todas las bases de datos y contenedores de bases de datos de Azure Cosmos de esa cuenta. Todas las operaciones de lectura y consulta que se emitan con arreglo a un contenedor o una base de datos usan el nivel de coherencia especificado de forma predeterminada. Para más información, consulte cómo [configurar el nivel de coherencia predeterminado](how-to-manage-consistency.md#configure-the-default-consistency-level). También puede invalidar el nivel de coherencia predeterminado para una solicitud específica. Para obtener más información, consulte el artículo [Invalidación del nivel de coherencia predeterminado](how-to-manage-consistency.md?#override-the-default-consistency-level).

> [!IMPORTANT]
> Es necesario volver a crear cualquier instancia de SDK después de cambiar el nivel de coherencia predeterminado. Para ello, reinicie la aplicación. Esto garantiza que el SDK usa el nuevo nivel de coherencia predeterminado.

## <a name="guarantees-associated-with-consistency-levels"></a>Garantías asociadas a los niveles de coherencia

Azure Cosmos DB garantiza que el 100 por ciento de las solicitudes de lectura cumplan con la garantía de coherencia del nivel de coherencia que elija. Las definiciones precisas de los cinco niveles de coherencia de Azure Cosmos DB que usan el lenguaje de especificación TLA+ se proporcionan en el repositorio [azure-cosmos-tla](https://github.com/Azure/azure-cosmos-tla) de GitHub.

En las secciones siguientes se describe la semántica de los cinco niveles de coherencia.

### <a name="strong-consistency"></a>Coherencia fuerte

La coherencia fuerte ofrece una garantía de linearización. La linearización hace referencia a la capacidad de servir solicitudes simultáneamente. Se garantiza que las lecturas devuelven la versión más reciente de un elemento. Un cliente nunca ve una escritura no confirmada ni parcial. Se garantiza que los usuarios siempre leerán la escritura confirmada más reciente.

  En el gráfico siguiente se ilustra la coherencia fuerte con notas musicales. Una vez que los datos se han escrito en la región "Oeste de EE. UU. 2", al leer los datos de otras regiones, obtiene el valor más reciente:

  :::image type="content" source="media/consistency-levels/strong-consistency.gif" alt-text="Ilustración del nivel de coherencia fuerte":::

### <a name="bounded-staleness-consistency"></a>Coherencia de obsolescencia limitada

En la obsolescencia limitada se garantiza que las lecturas respetan la garantía de prefijo coherente. Las lecturas pueden ir con retraso respecto a las escrituras en un máximo de versiones *"K"* (es decir, "actualizaciones") de un elemento o en el intervalo de tiempo *"T"* , lo que se lea primero. En otras palabras, cuando elige la obsolescencia limitada, la "obsolescencia" se puede configurar de dos maneras:

- El número de versiones (*K*) del elemento
- El intervalo de tiempo (*T*) que las lecturas pueden retrasarse con respecto a las escrituras

En el caso de una cuenta de una sola región, el valor mínimo de *K* y *T* es de 10 operaciones de escritura o de 5 segundos. En el caso de cuentas de varias regiones, el valor mínimo de *K* y *T* es de 100 000 operaciones de escritura o de 300 segundos.

La obsolescencia limitada ofrece un orden global total fuera de la "ventana de obsolescencia". Cuando un cliente realiza operaciones de lectura en una región que acepta las escrituras, las garantías que proporciona la obsolescencia limitada son idénticas a las garantías por la coherencia fuerte. A medida que la ventana de obsolescencia se aproxime al tiempo o la actualización, el más cercano, el servicio limitará las nuevas escrituras para permitir que la replicación se ponga al día y respete la garantía de coherencia.

Dentro de la ventana de obsolescencia, Obsolescencia limitada proporciona las siguientes garantías de coherencia:

- Coherencia para clientes de la misma región para una cuenta con región de escritura única = Alta
- Coherencia para clientes de distintas regiones para una cuenta con región de escritura única = De prefijo coherente
- Coherencia para clientes que escriben en una sola región para una cuenta con varias regiones de escritura = De prefijo coherente
- Coherencia para clientes que escriben en diferentes regiones para una cuenta con varias regiones de escritura = Final

  Las aplicaciones distribuidas globalmente que esperan bajas latencias de escritura pero que necesitan una garantía de orden global, suelen elegir la obsolescencia limitada. La obsolescencia limitada es ideal para las aplicaciones que se caracterizan por la colaboración en grupo y el uso compartido, el tablero de cotizaciones, la publicación o suscripción, la puesta en cola, etc. En el gráfico siguiente se ilustra la coherencia de obsolescencia limitada con notas musicales. Una vez que los datos se han escrito en la región "Oeste de EE. UU. 2", las regiones "Este de EE. UU. 2" y "Este de Australia" leen el valor escrito según el tiempo de retardo máximo configurado o las operaciones máximas:

  :::image type="content" source="media/consistency-levels/bounded-staleness-consistency.gif" alt-text="Ilustración del nivel de coherencia de obsolescencia limitada":::

### <a name="session-consistency"></a>Coherencia de sesión

En la coherencia de sesión, en una sesión de cliente individual, se garantiza que las lecturas respetan las garantías de prefijo coherente, lecturas monotónicas, escrituras monotónicas, lectura de la escritura y escritura tras las lecturas. Esto da por hecho una sesión de "escritor" individual o el uso compartido del token de sesión para varios escritores.

Los clientes que están fuera de la sesión que realiza escrituras verán las garantías siguientes:

- Coherencia para clientes de la misma región para una cuenta con región de escritura única = De prefijo coherente
- Coherencia para clientes de distintas regiones para una cuenta con región de escritura única = De prefijo coherente
- Coherencia para clientes que escriben en una sola región para una cuenta con varias regiones de escritura = De prefijo coherente
- Coherencia para clientes que escriben en varias regiones para una cuenta con varias regiones de escritura = Posible

  La coherencia de la sesión es el nivel de coherencia más usado para regiones únicas, así como para aplicaciones distribuidas globalmente. Proporciona latencias de escritura, disponibilidad y rendimiento de lectura equiparables a los de la coherencia final, pero también proporciona las garantías de coherencia que satisfacen las necesidades de aplicaciones escritas para funcionar en el contexto de un usuario. En el gráfico siguiente se ilustra la coherencia de la sesión con notas musicales. El "escritor de Oeste de EE. UU. 2" y el "lector de Oeste de EE. UU. 2" usan la misma sesión (sesión A), por lo que ambos leen los mismos datos al mismo tiempo. Sin embargo, la región "Este de Australia" usa la "sesión B", por lo que recibe los datos más tarde pero en el mismo orden en que se escriben.

  :::image type="content" source="media/consistency-levels/session-consistency.gif" alt-text="Ilustración del nivel de coherencia de sesión":::

### <a name="consistent-prefix-consistency"></a>Coherencia de prefijo coherente

En la opción de prefijo coherente, las actualizaciones que se devuelven contienen prefijos para todas las actualizaciones, sin espacios. El nivel de coherencia del prefijo coherente garantiza que las lecturas nunca vean escrituras desordenadas.

Si se realizan escrituras en el orden `A, B, C`, un cliente ve `A`, `A,B` o `A,B,C`, pero nunca permutaciones fuera de servicio como `A,C` o `B,A,C`. El prefijo coherente proporciona latencias de escritura, disponibilidad y rendimiento de lectura equiparables a los de la coherencia final, pero también proporciona las garantías de orden que se ajustan a las necesidades de escenarios en los que el orden es importante.

A continuación se muestran las garantías de coherencia para Prefijo coherente:

- Coherencia para clientes de la misma región para una cuenta con región de escritura única = De prefijo coherente
- Coherencia para clientes de distintas regiones para una cuenta con región de escritura única = De prefijo coherente
- Coherencia para clientes que escriben en una sola región para una cuenta con varias regiones de escritura = De prefijo coherente
- Coherencia para clientes que escriben en varias regiones para una cuenta con varias regiones de escritura = Final

En el gráfico siguiente se ilustra la coherencia del prefijo de coherencia con notas musicales. En todas las regiones, las lecturas no ven nunca las escrituras desordenadas:

  :::image type="content" source="media/consistency-levels/consistent-prefix.gif" alt-text="Ilustración de prefijo coherente":::

### <a name="eventual-consistency"></a>Coherencia final

En la coherencia final, no hay ninguna garantía de ordenación para las lecturas. En ausencia de escrituras adicionales, las réplicas terminarán por converger.  
La coherencia final es la forma más débil de coherencia, ya que un cliente puede leer los valores que son más antiguos que los que había leído antes. La coherencia final es adecuada cuando la aplicación no requiere ninguna garantía de ordenación. Entre los ejemplos se incluye el recuento de retweets, Me gusta o comentarios no encadenados. En el gráfico siguiente se ilustra la coherencia final con notas musicales.

  :::image type="content" source="media/consistency-levels/eventual-consistency.gif" alt-text="Ilustración de coherencia final":::

## <a name="consistency-guarantees-in-practice"></a>Garantías de coherencia en la práctica

En la práctica, es posible que obtenga mayores garantías de coherencia. Las garantías de coherencia para una operación de lectura se corresponden con la actualización y el orden del estado de la base de datos que solicita. La coherencia de lectura está asociada con la ordenación y la propagación de las operaciones de escritura/actualización.  

Si no hay ninguna operación de escritura en la base de datos, una operación de lectura con los niveles de coherencia **posible**, **sesión** o **prefijo coherente** es probable que produzca los mismos resultados que una operación de lectura con el nivel de coherencia alto.

Si la cuenta de Azure Cosmos está configurada con un nivel de coherencia que no es el de coherencia alta, puede averiguar la probabilidad de que los clientes obtengan lecturas de coherencia alta para sus cargas de trabajo consultando la métrica de *obsolescencia limitada de probabilidad* (PBS). Esta métrica se expone en Azure Portal; para obtener más información, consulte [Supervisión de la métrica de obsolescencia limitada de manera probabilística (PBS)](how-to-manage-consistency.md#monitor-probabilistically-bounded-staleness-pbs-metric).

La obsolescencia limitada de probabilidad muestra cómo de posible es la coherencia final. Esta métrica proporciona una visión general de la frecuencia con la que puede obtener una coherencia mayor que el nivel de coherencia que tiene configurado actualmente en su cuenta de Azure Cosmos. En otras palabras, puede ver la probabilidad (en milisegundos) de obtener lecturas con coherencia alta para una combinación de regiones de escritura y lectura.

## <a name="consistency-levels-and-latency"></a>Latencia y niveles de coherencia

Se garantiza que la latencia de lectura de todos los niveles de coherencia siempre es inferior a 10 milisegundos en el percentil 99. La latencia media de lectura (en el percentil 50) es normalmente de cuatro milisegundos o menos.

Se garantiza que la latencia de escritura de todos los niveles de coherencia siempre sea inferior a 10 milisegundos en el percentil 99. La latencia media de escritura (en el percentil 50) es normalmente de cinco milisegundos o menos. Las cuentas de Azure Cosmos que abarcan varias regiones y están configuradas con una coherencia alta son una excepción a esta garantía.

### <a name="write-latency-and-strong-consistency"></a>Latencia de escritura y coherencia fuerte

Para las cuentas de Azure Cosmos configuradas con coherencia sólida con más de una región, la latencia de escritura es igual al doble del tiempo de ida y vuelta (RTT) entre cualquiera de las dos regiones más alejadas, más de diez milisegundos en el percentil 99. El RTT de red alto entre las regiones se traducirá a una latencia mayor para solicitudes de Cosmos DB, ya que la coherencia fuerte completa una operación solo después de asegurarse de que se ha confirmado en todas las regiones de una cuenta.

La latencia de RTT exacta depende de la distancia a la velocidad de la luz y la topología de red de Azure. La red de Azure no proporciona ningún SLA de latencia para el RTT entre dos regiones de Azure, pero publica las [estadísticas de latencia de recorrido de ida y vuelta de red de Azure](../networking/azure-network-latency.md). Para la cuenta de Azure Cosmos, las latencias de replicación se muestran en Azure Portal. Puede usar Azure Portal (vaya a la hoja Métrica, seleccione la pestaña Coherencia) para supervisar las latencias de replicación entre diversas regiones asociadas con su cuenta de Azure Cosmos.

> [!IMPORTANT]
> La coherencia fuerte para las cuentas con regiones que abarcan más de 5000 millas (8000 kilómetros) se bloquea de forma predeterminada debido a una elevada latencia de escritura. Para habilitar esta funcionalidad, póngase en contacto con el soporte técnico.

## <a name="consistency-levels-and-throughput"></a>Rendimiento y niveles de coherencia

- En el caso de la obsolescencia fuerte y limitada, las lecturas se realizan en dos réplicas en un conjunto de cuatro réplicas (cuórum minoritario) para proporcionar garantías de coherencia. Los niveles de sesión, prefijo coherente y posible realizan lecturas de réplica única. El resultado es que, para el mismo número de unidades de solicitud, el rendimiento de lectura para la obsolescencia fuerte y limitada es la mitad de los demás niveles de coherencia.

- Para un tipo determinado de operación de escritura (por ejemplo, Insert, Replace, Upsert o Delete), el rendimiento de escritura de las unidades de solicitud es idéntico para todos los niveles de coherencia.

|**Nivel de coherencia**|**Lecturas de cuórum**|**Escrituras de cuórum**|
|--|--|--|
|**Fuerte**|Minoría local|Mayoría global|
|**Obsolescencia limitada**|Minoría local|Mayoría local|
|**De sesión**|Réplica única (con token de sesión)|Mayoría local|
|**De prefijo coherente**|Réplica única|Mayoría local|
|**Posible**|Réplica única|Mayoría local|

> [!NOTE]
> El costo de RU/s para lecturas minoritarias locales es el doble que el de los niveles de coherencia más débiles porque las lecturas se realizan en dos réplicas para proporcionar garantías de coherencia para la obsolescencia alta y limitada.

## <a name="consistency-levels-and-data-durability"></a><a id="rto"></a>Durabilidad de los datos y niveles de coherencia

En un entorno de base de datos distribuida de forma global, existe una relación directa entre el nivel de coherencia y la durabilidad de los datos se produce una interrupción en toda la región. A medida que desarrolle el plan de continuidad empresarial, tendrá que saber el tiempo máximo aceptable para que la aplicación se recupere por completo tras un evento de interrupción. El tiempo necesario para que una aplicación se recupere totalmente se conoce como **objetivo de tiempo de recuperación** (**RTO**). También debe conocer el período máximo de actualizaciones de datos recientes que la aplicación puede tolerar perder al recuperarse después de un evento de interrupción. El período de tiempo de las actualizaciones que se puede permitir perder se conoce como **objetivo de punto de recuperación** (**RPO**).

En la tabla siguiente se define la relación entre el modelo de coherencia y la durabilidad de los datos si se produce una interrupción en toda la región. Es importante tener en cuenta que, en un sistema distribuido, aunque la coherencia sea alta, no es posible tener una base de datos distribuida con un RPO y un RTO de cero debido al [teorema de CAP](https://en.wikipedia.org/wiki/CAP_theorem).

|**Regiones**|**Modo de replicación**|**Nivel de coherencia**|**RPO**|**RTO**|
|---------|---------|---------|---------|---------|
|1|Una o varias regiones de escritura|Cualquier nivel de coherencia|< 240 minutos|<1 semana|
|>1|Región de escritura única|Sesión, prefijo coherente, eventual|< 15 minutos|< 15 minutos|
|>1|Una región de escritura|De obsolescencia entrelazada|*K* & *T*|< 15 minutos|
|>1|Una región de escritura|Alta|0|< 15 minutos|
|>1|Varias regiones de escritura|Sesión, prefijo coherente, eventual|< 15 minutos|0|
|>1|Varias regiones de escritura|De obsolescencia entrelazada|*K* & *T*|0|

*K* = número de versiones *"K"* (es decir, actualizaciones) de un elemento.

*T* = intervalo de tiempo *"T"* desde la última actualización.

En el caso de una cuenta de una sola región, el valor mínimo de *K* y *T* es de 10 operaciones de escritura o de 5 segundos. En el caso de cuentas de varias regiones, el valor mínimo de *K* y *T* es de 100 000 operaciones de escritura o de 300 segundos. Esto define el RPO mínimo para los datos cuando se usa la obsolescencia limitada.

## <a name="strong-consistency-and-multiple-write-regions"></a>Coherencia fuerte y varias regiones de escritura

Las cuentas de Cosmos configuradas con varias regiones de escritura no se pueden configurar para lograr una coherencia fuerte, ya que no es posible que un sistema distribuido proporcione un RPO de cero y un RTO de cero. Además, no existe ninguna ventaja en la latencia de escritura por usar la coherencia fuerte con varias regiones de escritura porque una escritura en cualquier región debe replicarse y confirmarse en todas las regiones configuradas dentro de la cuenta. Esto produce la misma latencia de escritura que una cuenta con una sola región de escritura.

## <a name="additional-reading"></a>Lecturas adicionales

Para más información sobre los conceptos de coherencia, lea los artículos siguientes:

- [High-level TLA+ specifications for the five consistency levels offered by Azure Cosmos DB](https://github.com/Azure/azure-cosmos-tla) (Especificaciones TLA+ de alto nivel de los cinco niveles de coherencia que ofrece Azure Cosmos DB)
- [Replicated Data Consistency Explained Through Baseball (video)](https://www.youtube.com/watch?v=gluIh8zd26I) (Vídeo sobre Coherencia de datos replicados explicada mediante el béisbol), por Doug Terry
- [Replicated Data Consistency Explained Through Baseball (whitepaper)](https://www.microsoft.com/research/publication/replicated-data-consistency-explained-through-baseball/) (Coherencia de datos replicados explicada mediante el béisbol [notas del producto]), por Doug Terry
- [Session guarantees for weakly consistent replicated data](https://dl.acm.org/citation.cfm?id=383631) (Garantías de sesión para datos replicados con coherencia débil)
- [Consistency Tradeoffs in Modern Distributed Database Systems Design: CAP is Only Part of the Story](https://www.computer.org/csdl/magazine/co/2012/02/mco2012020037/13rRUxjyX7k) (Compromisos de coherencia en el diseño de sistemas modernos de bases de datos distribuidas: CAP es solo parte de la historia)
- [Probabilistic Bounded Staleness (PBS) for Practical Partial Quorums](https://vldb.org/pvldb/vol5/p776_peterbailis_vldb2012.pdf) [Obsolescencia limitada probabilística (PBS) para cuórums parciales prácticos].
- [Eventual Consistent - Revisited (Coherencia posible: revisión)](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los niveles de coherencia de Azure Cosmos DB, lea los artículos siguientes:

- [Configuración del nivel de coherencia predeterminado](how-to-manage-consistency.md#configure-the-default-consistency-level)
- [Invalidación del nivel de coherencia predeterminado](how-to-manage-consistency.md#override-the-default-consistency-level)
- [Acuerdo de Nivel de Servicio de Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/)
