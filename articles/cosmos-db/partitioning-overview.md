---
title: Creación de particiones y escalado horizontal en Azure Cosmos DB
description: Obtenga información sobre la creación de particiones y las particiones lógicas y físicas en Azure Cosmos DB, los procedimientos recomendados a la hora de elegir una clave de partición y cómo administrar particiones lógicas
author: deborahc
ms.author: dech
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 099c65143f29f4fdf341b52e5d80731f1bdb0808
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107031008"
---
# <a name="partitioning-and-horizontal-scaling-in-azure-cosmos-db"></a>Creación de particiones y escalado horizontal en Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB usa la creación de particiones con el fin de escalar contenedores individuales en una base de datos para satisfacer las necesidades de rendimiento de la aplicación. En la creación de particiones, los elementos de un contenedor se dividen en distintos subconjuntos, que se llaman *particiones lógicas*. Las particiones lógicas se crean en función del valor de una *clave de partición* que está asociada con cada elemento de un contenedor. Todos los elementos de una partición lógica tienen el mismo valor de clave de partición.

Por ejemplo, un contenedor contiene elementos. Cada elemento tiene un valor único para la propiedad `UserID`. Si `UserID` actúa como la partición de clave para los elementos de un contenedor y hay 1000 valores `UserID` exclusivos, se crearán 1000 particiones lógicas del contenedor.

Además de una clave de partición que determina la partición lógica del elemento, cada elemento de un contenedor tiene también un *id. de elemento* (que es único dentro de una partición lógica). Al combinar la clave de partición y el *id. del elemento* se crea el *índice* del elemento, que los identifica de forma única. [Elegir una clave de partición](#choose-partitionkey) es una decisión importante que afecta al rendimiento de la aplicación.

En este artículo, se explica la relación entre las particiones lógicas y las físicas. También se describen los procedimientos recomendados para crear particiones y se proporciona una perspectiva detallada sobre el funcionamiento del escalado horizontal en Azure Cosmos DB. No es necesario comprender estos detalles internos para seleccionar la clave de partición, pero se tratan aquí para que sepa cómo funciona Azure Cosmos DB.

## <a name="logical-partitions"></a>Particiones lógicas

Una partición lógica consta de un conjunto de elementos con la misma clave de partición. Por ejemplo, en un contenedor con datos sobre nutrición, todos los elementos contienen la propiedad `foodGroup`. Puede utilizar `foodGroup` como clave de partición del contenedor. Los grupos de elementos que tienen valores específicos para `foodGroup`, tales como `Beef Products`, `Baked Products` y `Sausages and Luncheon Meats`, forman distintas particiones lógicas.

Una partición lógica también define el ámbito de las transacciones de base de datos. Puede actualizar los elementos dentro de una partición lógica mediante el uso de una [transacción con aislamiento de instantánea](database-transactions-optimistic-concurrency.md). Cuando se agregan nuevos elementos al contenedor, se crean nuevas particiones lógicas de forma transparente por el sistema. No tiene que preocuparse de quitar una partición lógica una vez eliminados los datos subyacentes.

No existen límites en el número de particiones lógicas que puede tener el contenedor. Cada partición lógica puede almacenar un máximo de 20 GB de datos. Las claves de partición correctas son aquellas que tienen una amplia gama de valores posibles. Por ejemplo, en un contenedor donde todos los elementos contienen una propiedad `foodGroup`, los datos de la partición lógica `Beef Products` podrían crecer hasta los 20 GB. [Seleccionar una clave de partición](#choose-partitionkey) con una amplia gama de valores posibles garantiza que el contenedor se puede escalar.

## <a name="physical-partitions"></a>Particiones físicas

Un contenedor se escala mediante la distribución de los datos y el rendimiento entre particiones físicas. De forma interna, a cada partición física se asignan una o varias particiones lógicas. Normalmente, los contenedores más pequeños tienen muchas particiones lógicas, pero solo necesitan una partición física. A diferencia de las particiones lógicas, las particiones físicas son una implementación interna del sistema y es Azure Cosmos DB quien se encarga en exclusiva de su administración.

El número de particiones físicas del contenedor depende de los siguientes factores:

* La cantidad de rendimiento aprovisionado (cada partición física individual puede proporcionar un rendimiento de hasta 10 000 unidades de solicitud por segundo). El límite de 10 000 RU/s para las particiones físicas implica que las particiones lógicas también tienen un límite de 10 000 RU/s, ya que cada partición lógica solo se asigna a una partición física.

* El almacenamiento de datos total (cada partición física individual puede almacenar hasta 50 GB de datos).

> [!NOTE]
> Las particiones físicas son una implementación interna del sistema y es Azure Cosmos DB quien se encarga en exclusiva de su administración. Al desarrollar las soluciones, no se centre en las particiones físicas porque no las puede controlar. En su lugar, céntrese en las claves de las particiones. Si elige una clave de partición que distribuya uniformemente el consumo del rendimiento por las particiones lógicas, tendrá la seguridad de que el consumo del rendimiento está equilibrado entre las particiones físicas.

No existen límites en el número de particiones físicas que puede tener el contenedor. A medida que aumente el tamaño de los datos o el rendimiento aprovisionado, Azure Cosmos DB creará nuevas particiones físicas automáticamente dividiendo las particiones existentes. Las divisiones de las particiones físicas no afectan a la disponibilidad de la aplicación. Cuando una partición física se divide, todos los datos que estén en una partición lógica específica se guardarán en la misma partición física. Las divisiones de las particiones físicas simplemente crean una nueva asignación entre las particiones lógicas y las particiones físicas.

El rendimiento aprovisionado para un contenedor se divide uniformemente entre las particiones físicas. Un diseño de claves de partición que no distribuye las solicitudes de manera uniforme puede generar demasiadas solicitudes dirigidas a un pequeño subconjunto de particiones que pasarán a ser "frecuentes". Las particiones frecuentes pueden conllevar un uso ineficaz del rendimiento aprovisionado, lo que podría limitar la velocidad y incrementar los costos.

Puede ver las particiones físicas de un contenedor en la sección **Almacenamiento** de la **hoja de métricas** de Azure Portal:

:::image type="content" source="./media/partitioning-overview/view-partitions-zoomed-out.png" alt-text="Consulta del número de particiones físicas" lightbox="./media/partitioning-overview/view-partitions-zoomed-in.png" ::: 

En la captura de pantalla anterior, un contenedor tiene `/foodGroup` como clave de partición. Cada una de las tres barras del grafo representa una partición física. En la imagen, el **intervalo de claves de partición** es el mismo que el de una partición física. La partición física seleccionada contiene las tres particiones lógicas de mayor tamaño: `Beef Products`, `Vegetable and Vegetable Products` y `Soups, Sauces, and Gravies`.

Si se aprovisiona un rendimiento de 18 000 unidades de solicitud por segundo (RU/s), cada una de las tres particiones físicas puede usar un tercio del rendimiento aprovisionado total. Dentro de la partición física seleccionada, las claves de partición lógicas `Beef Products`, `Vegetable and Vegetable Products` y `Soups, Sauces, and Gravies` podrán, en conjunto, utilizar 6 000 RU/s aprovisionadas de la partición física. Como el rendimiento aprovisionado se reparte uniformemente entre las particiones físicas del contenedor, es importante distribuir uniformemente el consumo del rendimiento [eligiendo la clave de partición lógica adecuada](#choose-partitionkey). 

## <a name="managing-logical-partitions"></a>Administración de particiones lógicas

Azure Cosmos DB administra de forma transparente y automática la ubicación de las particiones lógicas en particiones físicas para satisfacer de manera eficaz las necesidades de escalabilidad y rendimiento del contenedor. A medida que aumentan los requisitos de rendimiento y almacenamiento de la aplicación, Azure Cosmos DB mueve las particiones lógicas para distribuir automáticamente la carga entre un número más elevado de particiones físicas. Puede obtener más información sobre las [particiones físicas](partitioning-overview.md#physical-partitions).

Azure Cosmos DB usa la creación de particiones por hash para distribuir las particiones lógicas entre las particiones físicas. Azure Cosmos DB aplica un algoritmo hash al valor de clave de partición de un elemento. El resultado con hash determina la partición física. A continuación, Azure Cosmos DB asigna el espacio de claves de los hash de claves de partición uniformemente entre las particiones físicas.

Las transacciones (en procedimientos almacenados o desencadenadores) solo se permiten con elementos dentro de una única partición lógica.

## <a name="replica-sets"></a>Conjuntos de réplicas

Cada partición física se compone de un [*conjunto de réplicas*](global-dist-under-the-hood.md). Cada conjunto de réplicas hospeda una instancia del motor de base de datos. Un conjunto de réplicas hace que los datos almacenados en la partición física sean duraderos, coherentes y tengan una alta disponibilidad. Cada réplica que compone la partición física hereda la cuota de almacenamiento. Y todas las réplicas de una partición física admiten colectivamente el rendimiento asignado a la partición física. Azure Cosmos DB administra automáticamente los conjuntos de réplicas.

Normalmente, los contenedores pequeños solo necesitan una partición física, pero siguen teniendo al menos cuatro réplicas.

La siguiente imagen muestra cómo se asignan particiones lógicas a particiones físicas distribuidas globalmente:

:::image type="content" source="./media/partitioning-overview/logical-partitions.png" alt-text="Una imagen que muestra las particiones de Azure Cosmos DB" border="false":::

## <a name="choosing-a-partition-key"></a><a id="choose-partitionkey"></a>Elegir una clave de partición

Una clave de partición tiene dos componentes: **ruta de acceso de la clave de partición** y **valor de la clave de partición**. Por ejemplo, considere un elemento { "userId" : "Andrew", "worksFor": "Microsoft" }, si elige "userId" como clave de partición, los siguientes serán los dos componentes de la clave de partición:

* La ruta de acceso de la clave de partición (por ejemplo: "/userId"). La ruta de acceso de la clave de partición acepta caracteres alfanuméricos y guiones bajos (_). También puede usar objetos anidados mediante la notación de ruta de acceso estándar (/).

* El valor de la clave de partición (por ejemplo: "Andrew"). El valor de la clave de partición puede ser de tipo cadena o numérico.

Para obtener información sobre los límites de rendimiento, almacenamiento y longitud de la clave de partición, consulte el artículo [Cuotas de servicio de Azure Cosmos DB](concepts-limits.md).

La selección de la clave de partición es una decisión de diseño sencilla pero importante en Azure Cosmos DB. Una vez que haya seleccionado la clave de partición, no es posible cambiarla en contexto. Si necesita cambiarla, debe trasladar los datos a un nuevo contenedor con la nueva clave de partición deseada.

Para **todos** los contenedores, la clave de partición debe:

* Ser una propiedad con un valor que no cambie. Si una propiedad es la clave de partición, no se puede actualizar su valor.

* Tener una cardinalidad alta. En otras palabras, la propiedad debe tener una amplia gama de valores posibles.

* Distribuir el consumo de unidades de solicitud (RU) y el almacenamiento de datos uniformemente en todas las particiones lógicas. Esto garantiza una distribución uniforme del consumo de RU y el almacenamiento en las particiones físicas.

Si necesita [transacciones ACID de varios elementos](database-transactions-optimistic-concurrency.md#multi-item-transactions) en Azure Cosmos DB, deberá usar [procedimientos almacenados o desencadenadores](how-to-write-stored-procedures-triggers-udfs.md#stored-procedures). Todos los procedimientos almacenados y desencadenadores basados en JavaScript están limitados a una única partición lógica.

## <a name="partition-keys-for-read-heavy-containers"></a>Claves de partición para contenedores con lecturas frecuentes

Para la mayoría de los contenedores, los criterios anteriores se deben tener en cuenta al elegir una clave de partición. No obstante, en el caso de los contenedores con lecturas frecuentes, es posible que quiera elegir una clave de partición que aparece con frecuencia como filtro en las consultas. Las consultas se pueden [enrutar de manera eficaz solo a las particiones físicas pertinentes](how-to-query-container.md#in-partition-query) si se incluye la clave de partición en el predicado de filtro.

Si la mayoría de las solicitudes de la carga de trabajo son consultas, y la mayoría de las consultas tiene un filtro de igualdad en la misma propiedad, esta puede ser una buena opción de clave de partición. Por ejemplo, si ejecuta frecuentemente una consulta que filtra por `UserID`, al seleccionar `UserID` como clave de partición se reduciría el número de [consultas entre particiones](how-to-query-container.md#avoiding-cross-partition-queries).

Sin embargo, si el contenedor es pequeño, probablemente no tenga suficientes particiones físicas como para que deba preocuparle el impacto de las consultas entre particiones en el rendimiento. La mayoría de los contenedores pequeños en Azure Cosmos DB requieren solo una o dos particiones físicas.

Si el contenedor puede aumentar a unas cuantas particiones físicas, debe asegurarse de elegir una clave de partición que minimice las consultas entre particiones. El contenedor necesitará más de unas cuantas particiones físicas cuando se cumpla alguna de las siguientes condiciones:

* El contenedor tiene aprovisionadas más de 30 000 RU.

* El contenedor almacenará más de 100 GB de datos.

## <a name="using-item-id-as-the-partition-key"></a>Uso del id. de elemento como clave de partición

Si el contenedor tiene una propiedad con una amplia gama de posibles valores, probablemente sea una buena elección de clave de partición. Un posible ejemplo de una propiedad de este tipo, es el *id. de elemento*. En el caso de contenedores pequeños de lectura o escritura frecuente de cualquier tamaño, el *id. de elemento* es naturalmente una excelente opción de clave de partición.

La propiedad del sistema *id. de elemento* existe en todos los elementos del contenedor. Es posible que tenga otras propiedades que representen un identificador lógico para el elemento. En muchos casos, también son excelentes claves de partición por los mismos motivos que el *id. de elemento*.

El *id. de elemento* es una excelente opción de clave de partición por los siguientes motivos:

* Hay una amplia gama de valores posibles (un *id. de elemento* único por elemento).
* Dado que cada elemento tiene un *id.* único, este *id. de elemento* realiza un trabajo excelente para equilibrar de manera uniforme el consumo de RU y el almacenamiento de datos.
* Puede realizar fácilmente lecturas puntuales, ya que siempre conocerá la clave de partición de un elemento si conoce su *identificador*.

Algunos aspectos que se deben tener en cuenta al seleccionar el *id. de elemento* como clave de partición son:

* Si el *id. de elemento* es la clave de partición, se convertirá en un identificador único en todo el contenedor. No podrá tener elementos con *id. de elemento* duplicados.
* Si tiene un contenedor con lecturas frecuentes que tiene muchas [particiones físicas](partitioning-overview.md#physical-partitions), las consultas serán más eficaces si tienen un filtro de igualdad con el *id. de elemento*.
* No se pueden ejecutar procedimientos almacenados ni desencadenadores en varias particiones lógicas.

## <a name="next-steps"></a>Pasos siguientes

* Información sobre el [procesamiento aprovisionado en Azure Cosmos DB](request-units.md).
* Información sobre la [distribución global en Azure Cosmos DB](distribute-data-globally.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).
