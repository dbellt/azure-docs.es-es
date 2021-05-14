---
title: Estimación de los costos mediante la herramienta de planeamiento de capacidad de Azure Cosmos DB - API de SQL
description: Capacity Planner de Azure Cosmos DB permite estimar el rendimiento (RU/s) necesario y el costo de la carga de trabajo. En este artículo se describe cómo usar la herramienta de planeamiento de capacidad para calcular el rendimiento y los costos necesarios al usar la API de SQL.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/20/2021
ms.author: dech
ms.openlocfilehash: 6329935c780494722a78e00e82446834f72aa37b
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209308"
---
# <a name="estimate-rus-using-the-azure-cosmos-db-capacity-planner---sql-api"></a>Estimación de RU/s mediante la herramienta de planeamiento de capacidad de Azure Cosmos DB - API de SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La configuración de las bases de datos y los contenedores de Azure Cosmos con la cantidad adecuada de rendimiento aprovisionado, o [unidades de solicitud (RU/s)](request-units.md), para la carga de trabajo es esencial para optimizar el costo y el rendimiento. En este artículo se describe cómo usar la [herramienta de planeamiento de capacidad](https://cosmos.azure.com/capacitycalculator/) de Azure Cosmos DB para obtener una estimación de las RU/s necesarias y el costo de la carga de trabajo al usar la API de SQL. Si usa la API para MongoDB, consulte el artículo sobre cómo [usar la calculadora de capacidad con MongoDB](mongodb/estimate-ru-capacity-planner.md).

[!INCLUDE [capacity planner modes](includes/capacity-planner-modes.md)]

## <a name="estimate-provisioned-throughput-and-cost-using-basic-mode"></a><a id="basic-mode"></a>Estimación del rendimiento aprovisionado y el costo mediante el modo básico
Para obtener una estimación rápida de la carga de trabajo mediante el modo básico, vaya a [Capacity Planner](https://cosmos.azure.com/capacitycalculator/). Escriba los siguientes parámetros en función de la carga de trabajo:

|**Entrada**  |**Descripción**  |
|---------|---------|
| API |Selección de la API de SQL (Core) |
|Número de regiones|Azure Cosmos DB está disponible en todas las regiones de Azure. Seleccione el número de regiones necesarias para la carga de trabajo. Puede asociar cualquier número de regiones con su cuenta de Cosmos. Consulte la información sobre la [distribución global](distribute-data-globally.md) en Azure Cosmos DB para más detalles.|
|Escrituras de varias regiones|Si habilita [Escrituras en varias regiones](distribute-data-globally.md#key-benefits-of-global-distribution), la aplicación puede realizar operaciones de lectura y escritura en cualquier región de Azure. Si deshabilita las escrituras en varias regiones, la aplicación solo puede escribir en una región. <br/><br/> Habilite las escrituras en varias regiones si espera tener una carga de trabajo activa-activa que requiera escrituras de baja latencia en diferentes regiones. Por ejemplo, una carga de trabajo de IOT que escribe datos en la base de datos en grandes volúmenes en regiones diferentes. <br/><br/> Las escrituras en varias regiones garantizan un 99,999 % de disponibilidad de lectura y escritura. Las escrituras en varias regiones requieren más rendimiento en comparación con las regiones de escritura únicas. Para más información, consulte el artículo sobre [cómo se diferencian las RU para regiones de una sola escritura y de varias escrituras](optimize-cost-regions.md).|
|Datos totales almacenados en el almacén transaccional |Datos totales estimados almacenados (GB) en el almacén transaccional en una sola región.|
|Uso del almacén analítico| Elija **Activado** si quiere usar el almacén analítico. Escriba los **datos totales almacenados en el almacén analítico**, que representan los datos estimados almacenados (GB) en el almacén analítico en una sola región.  |
|Tamaño del elemento|Tamaño estimado del elemento de datos (por ejemplo, un documento), que va de 1 KB a 2 MB. |
|Consultas/s |Número de consultas esperadas cada segundo por región. La carga de RU promedio para ejecutar una consulta se calcula en 10 RU. |
|Lecturas puntuales/s |Número de operaciones de lectura puntuales esperadas cada segundo por región. Las lecturas puntuales son la búsqueda de clave y valor en un único id. de elemento y clave de partición. Para más información acerca de las lecturas puntuales, consulte el artículo [Opciones de lectura de datos](optimize-cost-reads-writes.md#reading-data-point-reads-and-queries). |
|Creaciones/s |Número de operaciones de creación esperadas cada segundo por región. |
|Actualizaciones/s |Número de operaciones de actualización esperadas cada segundo por región. Al seleccionar la indexación automática, las RU/s estimadas de la operación de actualización se calculan como una propiedad que se modifica en base a una actualización. |
|Eliminaciones/s |Número de operaciones de eliminación esperadas cada segundo por región. |

Después de rellenar los detalles necesarios, seleccione **Calcular**. La pestaña **Cost Estimate** (Estimación del costo) muestra el costo total del almacenamiento y el rendimiento aprovisionado. Puede expandir el vínculo **Mostrar detalles** de esta pestaña para obtener el desglose del rendimiento necesario para distintas solicitudes de CRUD y consulta. Cada vez que cambie el valor de cualquier campo, seleccione **Calcular** para volver a calcular el costo estimado.

:::image type="content" source="./media/estimate-ru-with-capacity-planner/basic-mode-sql-api.png" alt-text="Modo básico de Capacity Planner" border="true":::

## <a name="estimate-provisioned-throughput-and-cost-using-advanced-mode"></a><a id="advanced-mode"></a>Estimación del rendimiento aprovisionado y del costo mediante el modo avanzado

El modo avanzado permite proporcionar más configuraciones que afectan a la estimación de RU/s. Para usar esta opción, vaya a [Capacity Planner](https://cosmos.azure.com/capacitycalculator/) e **inicie sesión** en la herramienta con una cuenta que use para Azure. La opción de inicio de sesión está disponible en la esquina derecha.

Después de iniciar sesión, se mostrarán campos adicionales en comparación con los campos del modo básico. Escriba los parámetros adicionales en función de la carga de trabajo.

|**Entrada**  |**Descripción**  |
|---------|---------|
|API|Azure Cosmos DB es un servicio de varios modelos y varias API. Elija la API de SQL (Core). |
|Número de regiones|Azure Cosmos DB está disponible en todas las regiones de Azure. Seleccione el número de regiones necesarias para la carga de trabajo. Puede asociar cualquier número de regiones con su cuenta de Cosmos. Consulte la información sobre la [distribución global](distribute-data-globally.md) en Azure Cosmos DB para más detalles.|
|Escrituras de varias regiones|Si habilita [Escrituras en varias regiones](distribute-data-globally.md#key-benefits-of-global-distribution), la aplicación puede realizar operaciones de lectura y escritura en cualquier región de Azure. Si deshabilita las escrituras en varias regiones, la aplicación solo puede escribir en una región. <br/><br/> Habilite las escrituras en varias regiones si espera tener una carga de trabajo activa-activa que requiera escrituras de baja latencia en diferentes regiones. Por ejemplo, una carga de trabajo de IOT que escribe datos en la base de datos en grandes volúmenes en regiones diferentes. <br/><br/> Las escrituras en varias regiones garantizan un 99,999 % de disponibilidad de lectura y escritura. Las escrituras en varias regiones requieren más rendimiento en comparación con las regiones de escritura únicas. Para más información, consulte el artículo sobre [cómo se diferencian las RU para regiones de una sola escritura y de varias escrituras](optimize-cost-regions.md).|
|Coherencia predeterminada|Azure Cosmos DB admite cinco niveles de coherencia, para permitir a los desarrolladores encontrar un punto de equilibrio entre la coherencia, la disponibilidad y los ajustes de la latencia. Para más información, consulte el artículo [Niveles de coherencia](consistency-levels.md). <br/><br/> De forma predeterminada, Azure Cosmos DB usa la coherencia de la sesión, lo que garantiza la capacidad de leer sus propias escrituras en una sesión. <br/><br/> La elección de la obsolescencia limitada o fuerte requerirá el doble de RU/s necesarias para las lecturas, en comparación con la sesión, el prefijo coherente y la coherencia final. La coherencia alta con escrituras en varias regiones no se admite y se establecerá automáticamente de forma predeterminada en las escrituras en una sola región con una coherencia alta. |
|Directiva de indexación|De forma predeterminada, Azure Cosmos DB [indexa todas las propiedades](index-policy.md) de todos los elementos para consultas flexibles y eficientes (se asigna a la directiva de indexación **Automática**). <br/><br/> Si elige **Desactivado**, ninguna de las propiedades se indexará. El resultado de todo esto es el cargo más bajo de RU para las escrituras. Seleccione **Desactivado** para la directiva si espera realizar solamente [lecturas de punto](/dotnet/api/microsoft.azure.cosmos.container.readitemasync) (búsquedas de valores de clave) o escrituras, y ninguna consulta. <br/><br/> Si elige **Automático**, Azure Cosmos DB indexará automáticamente todos los elementos a medida que se escriban. <br/><br/> La directiva de indexación **personalizada** permite incluir o excluir propiedades específicas del índice para un menor rendimiento y almacenamiento de escritura. Para más información, consulte los artículos sobre la [directiva de indexación](index-overview.md) y las [directivas de indexación de ejemplo](how-to-manage-indexing-policy.md#indexing-policy-examples).|
|Datos totales almacenados en el almacén transaccional |Datos totales estimados almacenados (GB) en el almacén transaccional en una sola región.|
|Uso del almacén analítico| Elija **Activado** si quiere usar el almacén analítico. Escriba los **Datos totales almacenados en el almacén analítico**, que representan los datos estimados almacenados (GB) en el almacén analítico de una sola región.  |
|Modo de carga de trabajo|Seleccione la opción **Estable** si el volumen de la carga de trabajo es constante. <br/><br/> Seleccione la opción **Variable** si el volumen de la carga de trabajo cambia con el tiempo.  Por ejemplo, durante un día o un mes concreto. Las siguientes configuraciones están disponibles si elige la opción de carga de trabajo variable:<ul><li>Porcentaje de tiempo máximo: porcentaje de tiempo en un mes en el que la carga de trabajo requiere un rendimiento máximo (el más alto). </li></ul> <br/><br/> Por ejemplo, si tiene una carga de trabajo que tiene una actividad elevada de 9 a.m. a 6 p.m. en el horario comercial entre semana, el porcentaje de tiempo máximo es: 45 horas en período máximo / 730 horas / mes = ~ 6 %.<br/><br/>Con intervalos en el período máximo y fuera del período máximo, puede optimizar el costo [escalando y reduciendo verticalmente mediante programación el rendimiento](set-throughput.md#update-throughput-on-a-database-or-a-container) en consecuencia.|
|Tamaño del elemento|Tamaño del elemento de datos (por ejemplo, un documento), que va de 1 KB a 2 MB. Puede agregar estimaciones para varios elementos de ejemplo. <br/><br/>También puede **cargar el documento de ejemplo (JSON)** para una estimación más precisa.<br/><br/>Si la carga de trabajo tiene varios tipos de elementos (con contenido JSON diferente) en el mismo contenedor, puede cargar varios documentos JSON y obtener la estimación. Use el botón **Agregar nuevo elemento** para agregar varios documentos JSON de ejemplo.|
| Número de propiedades | Número promedio de propiedades por elemento. |
|Lecturas puntuales/s |Número de operaciones de lectura puntuales esperadas cada segundo por región. Las lecturas puntuales son la búsqueda de clave y valor en un único id. de elemento y clave de partición. Las operaciones de lectura puntuales son diferentes de las operaciones de lectura de consultas. Para más información acerca de las lecturas puntuales, consulte el artículo [Opciones de lectura de datos](optimize-cost-reads-writes.md#reading-data-point-reads-and-queries). Si el modo de carga de trabajo es **Variable**, puede proporcionar el número esperado de operaciones de lectura puntuales en las horas de mucha actividad y poca actividad. |
|Creaciones/s |Número de operaciones de creación esperadas cada segundo por región. |
|Actualizaciones/s |Número de operaciones de actualización esperadas cada segundo por región. |
|Eliminaciones/s |Número de operaciones de eliminación esperadas cada segundo por región. |
|Consultas/s |Número de consultas esperadas cada segundo por región. Para una estimación precisa, use el costo promedio de las consultas o escriba las RU/s que usan las consultas a partir de las estadísticas de consulta en Azure Portal. |
| Carga de RU/s promedio por consulta | De manera predeterminada, el costo promedio de las consultas por segundo por región se calcula en 10 RU/s. Puede aumentarlo o reducirlo en función de las cargas de RU/s en función de la carga estimada de la consulta.|

También puede usar el botón **Guardar estimación** para descargar un archivo CSV que contenga la estimación actual.

:::image type="content" source="./media/estimate-ru-with-capacity-planner/advanced-mode-sql-api.png" alt-text="Modo avanzado de Capacity Planner" border="true":::

Los precios que se muestran en Capacity Planner de Azure Cosmos DB son estimaciones en función de las tarifas de precios públicas para el rendimiento y el almacenamiento. Todos los precios se muestran en dólares estadounidenses. Consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para ver todas las tarifas por región.  

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [modelo de precios de Azure Cosmos DB](how-pricing-works.md).
* Cree una nueva [cuenta, base de datos y contenedor de Cosmos](create-cosmosdb-resources-portal.md).
* Obtenga información para [optimizar el costo del rendimiento aprovisionado](optimize-cost-throughput.md).
* Obtenga información para [optimizar el costo con capacidad reservada](cosmos-db-reserved-capacity.md).
