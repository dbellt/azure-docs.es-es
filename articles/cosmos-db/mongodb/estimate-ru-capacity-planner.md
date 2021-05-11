---
title: 'Estimación de los costos mediante la herramienta Capacity Planner de Azure Cosmos DB: API de MongoDB'
description: Capacity Planner de Azure Cosmos DB permite estimar el rendimiento (RU/s) necesario y el costo de la carga de trabajo. En este artículo se describe cómo usar la herramienta Capacity Planner para calcular el rendimiento y el costo necesarios al usar la API de Azure Cosmos DB para MongoDB.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 04/28/2021
ms.author: dech
ms.openlocfilehash: 10f03ce5d2171b28fa195f7f5c2b0f34a939eef0
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108202810"
---
# <a name="estimate-rus-using-the-azure-cosmos-db-capacity-planner---azure-cosmos-db-api-for-mongodb"></a>Estimación del número de RU/s mediante la herramienta Capacity Planner de Azure Cosmos DB: API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-sql-api](../includes/appliesto-mongodb-api.md)]

Para optimizar el costo y el rendimiento, es esencial configurar las bases de datos y las colecciones con la cantidad adecuada de rendimiento aprovisionado, o [unidades de solicitud (RU/s)](../request-units.md). En este artículo se describe cómo usar la herramienta [Capacity Planner](https://cosmos.azure.com/capacitycalculator/) de Azure Cosmos DB para obtener una estimación del número de RU/s necesarias y el costo de la carga de trabajo al usar la API de Azure Cosmos DB para MongoDB. Si va a usar SQL API, consulte el artículo sobre cómo [usar la calculadora de capacidad con SQL API](../estimate-ru-with-capacity-planner.md).

[!INCLUDE [capacity planner modes](../includes/capacity-planner-modes.md)]

## <a name="estimate-provisioned-throughput-and-cost-using-basic-mode"></a><a id="basic-mode"></a>Estimación del rendimiento aprovisionado y el costo mediante el modo básico
Para obtener una estimación rápida de la carga de trabajo mediante el modo básico, vaya a [Capacity Planner](https://cosmos.azure.com/capacitycalculator/). Escriba los siguientes parámetros en función de la carga de trabajo:

|**Entrada**  |**Descripción**  |
|---------|---------|
| API |Elección de MongoDB API |
|Número de regiones|La API de Azure Cosmos DB para MongoDB está disponible en todas las regiones de Azure. Seleccione el número de regiones necesarias para la carga de trabajo. Puede asociar cualquier número de regiones con su cuenta. Para más información, consulte [Distribución global](../distribute-data-globally.md).|
|Escrituras de varias regiones|Si habilita [Escrituras en varias regiones](../distribute-data-globally.md#key-benefits-of-global-distribution), la aplicación puede realizar operaciones de lectura y escritura en cualquier región de Azure. Si deshabilita las escrituras en varias regiones, la aplicación solo puede escribir en una región. <br/><br/> Habilite las escrituras en varias regiones si espera tener una carga de trabajo activa-activa que requiera escrituras de baja latencia en diferentes regiones. Por ejemplo, una carga de trabajo de IOT que escribe datos en la base de datos en grandes volúmenes en regiones diferentes. <br/><br/> Las escrituras en varias regiones garantizan un 99,999 % de disponibilidad de lectura y escritura. Las escrituras en varias regiones requieren más rendimiento en comparación con las regiones de escritura únicas. Para más información, consulte el artículo sobre [cómo se diferencian las RU para regiones de una sola escritura y de varias escrituras](../optimize-cost-regions.md).|
|Datos totales almacenados en el almacén transaccional |Datos totales estimados almacenados (GB) en el almacén transaccional en una sola región.|
|Uso del almacén analítico| Elija **Activado** si quiere usar el [almacén analítico de Synapse](../synapse-link.md). Escriba los **datos totales almacenados en el almacén analítico**, que representan los datos estimados almacenados (GB) en el almacén analítico en una sola región.  |
|Tamaño del elemento|Tamaño estimado de los documentos, que oscila entre 1 KB y 2 MB. |
|Búsquedas/s |Número de operaciones de búsqueda esperadas por segundo por región. |
|Inserciones/s |Número de operaciones de inserción esperadas por segundo por región. |
|Actualizaciones/s |Número de operaciones de actualización esperadas por segundo. Al seleccionar la indexación automática, las RU/s estimadas de la operación de actualización se calculan como una propiedad que se modifica en base a actualización. |
|Eliminaciones/s |Número de operaciones de eliminación esperadas por segundo. |

Después de rellenar los detalles necesarios, seleccione **Calcular**. La pestaña **Cost Estimate** (Estimación del costo) muestra el costo total del almacenamiento y el rendimiento aprovisionado. Puede expandir el vínculo **Mostrar detalles** de esta pestaña para obtener el desglose del rendimiento necesario para distintas solicitudes de CRUD y consulta. **Cada vez que cambie el valor de cualquier campo, seleccione Calcular para volver a calcular el costo estimado.**

:::image type="content" source="./media/estimate-ru-capacity-planner/basic-mode-mongodb-api.png" alt-text="Modo básico de Capacity Planner" border="true":::

## <a name="estimate-provisioned-throughput-and-cost-using-advanced-mode"></a><a id="advanced-mode"></a>Estimación del rendimiento aprovisionado y del costo mediante el modo avanzado

El modo avanzado permite proporcionar más configuraciones que afectan a la estimación de RU/s. Para usar esta opción, vaya a [Capacity Planner](https://cosmos.azure.com/capacitycalculator/) e **inicie sesión** en la herramienta con una cuenta que use para Azure. La opción de inicio de sesión está disponible en la esquina derecha.

Después de iniciar sesión, puede ver campos adicionales en comparación con los campos del modo básico. Escriba los parámetros adicionales en función de la carga de trabajo.

|**Entrada**  |**Descripción**  |
|---------|---------|
|API|Azure Cosmos DB es un servicio de varios modelos y varias API. Elija MongoDB API. |
|Número de regiones|La API de Azure Cosmos DB para MongoDB está disponible en todas las regiones de Azure. Seleccione el número de regiones necesarias para la carga de trabajo. Puede asociar cualquier número de regiones con su cuenta de Cosmos. Para más información, consulte [Distribución global](../distribute-data-globally.md).|
|Escrituras de varias regiones|Si habilita [Escrituras en varias regiones](../distribute-data-globally.md#key-benefits-of-global-distribution), la aplicación puede realizar operaciones de lectura y escritura en cualquier región de Azure. Si deshabilita las escrituras en varias regiones, la aplicación solo puede escribir en una región. <br/><br/> Habilite las escrituras en varias regiones si espera tener una carga de trabajo activa-activa que requiera escrituras de baja latencia en diferentes regiones. Por ejemplo, una carga de trabajo de IOT que escribe datos en la base de datos en grandes volúmenes en regiones diferentes. <br/><br/> Las escrituras en varias regiones garantizan un 99,999 % de disponibilidad de lectura y escritura. Las escrituras en varias regiones requieren más rendimiento en comparación con las regiones de escritura únicas. Para más información, consulte el artículo sobre [cómo se diferencian las RU para regiones de una sola escritura y de varias escrituras](../optimize-cost-regions.md).|
|Coherencia predeterminada|La API de Azure Cosmos DB para MongoDB admite cinco niveles de coherencia, que permiten a los desarrolladores encontrar un punto de equilibrio entre la coherencia, la disponibilidad y la latencia. Para más información, consulte el artículo [Niveles de coherencia](../consistency-levels.md). <br/><br/> De forma predeterminada, la API para MongoDB usa la coherencia de sesión, lo que garantiza la posibilidad de leer sus propias escrituras en una sesión. <br/><br/> La elección de la obsolescencia limitada o fuerte requerirá el doble de RU/s necesarias para las lecturas, en comparación con la sesión, el prefijo coherente y la coherencia final. La coherencia alta con escrituras en varias regiones no se admite y se establecerá automáticamente de forma predeterminada en las escrituras en una sola región con una coherencia alta. |
|Directiva de indexación| Si elige la opción **Desactivado**, ninguna de las propiedades se indexará. El resultado de todo esto es el cargo más bajo de RU para las escrituras. Desactive la directiva de indexación si solo tiene previsto realizar consultas con el campo _id y la clave de partición en cada consulta (ambas por consulta).<br/><br/> Si elige la opción **Automático**, la versión 3.6 y versiones posteriores de la API para MongoDB indexan automáticamente el _id archivado. Cuando elige la indexación automática, equivale a establecer un índice con caracteres comodín (donde cada propiedad se indexa automáticamente). Use índices comodín para todos los campos para consultas flexibles y eficaces.<br/><br/>Si elige la opción **Personalizado**, puede establecer cuántas propiedades se indexan con índices de varias claves o índices compuestos. Puede especificar el número de propiedades indexadas más adelante en el formulario. Para más información, consulte [Administración de índices](../mongodb-indexing.md) en la API para MongoDB.|
|Datos totales almacenados en el almacén transaccional |Datos totales estimados almacenados (en GB) en el almacén transaccional en una sola región.|
|Uso del almacén analítico| Elija **Activado** si quiere usar el [almacén analítico de Synapse](../synapse-link.md). Escriba los **datos totales almacenados en el almacén analítico**, que representan los datos estimados almacenados (GB) en el almacén analítico en una sola región.  |
|Modo de carga de trabajo|Seleccione la opción **Estable** si el volumen de la carga de trabajo es constante. <br/><br/> Seleccione la opción **Variable** si el volumen de la carga de trabajo cambia con el tiempo.  Por ejemplo, durante un día o un mes concreto. Las siguientes configuraciones están disponibles si elige la opción de carga de trabajo variable:<ul><li>Porcentaje de tiempo máximo: porcentaje de tiempo en un mes en el que la carga de trabajo requiere un rendimiento máximo (el más alto). </li></ul> <br/><br/> Por ejemplo, si tiene una carga de trabajo que tiene una actividad elevada de 9 a.m. a 6 p.m. en el horario comercial entre semana, el porcentaje de tiempo máximo es: 45 horas en período máximo / 730 horas / mes = ~ 6 %.<br/><br/>Con intervalos en el período máximo y fuera del período máximo, puede optimizar el costo [escalando y reduciendo verticalmente mediante programación el rendimiento](../set-throughput.md#update-throughput-on-a-database-or-a-container) en consecuencia.|
|Tamaño del elemento|Tamaño de los documentos, que van de 1 KB a 2 MB. Puede agregar estimaciones para varios elementos de ejemplo. <br/><br/>También puede **cargar el documento de ejemplo (JSON)** para una estimación más precisa.<br/><br/>Si la carga de trabajo tiene varios tipos de elementos (con contenido JSON diferente) en el mismo contenedor, puede cargar varios documentos JSON y obtener la estimación. Use el botón **Agregar nuevo elemento** para agregar varios documentos JSON de ejemplo.|
| Tipo de operación | Tipo de operación, como **Buscar**, **Agregar**, **Modificar**, etc.  |
| Cargo de unidad de solicitud (RU) por llamada | Cargo estimado de RU/s para ejecutar el tipo de operación seleccionado. |
| Lecturas/s por región | Número de tipos de operaciones seleccionadas ejecutadas por segundo por región. |

También puede usar el botón **Guardar estimación** para descargar un archivo CSV que contenga la estimación actual.

:::image type="content" source="./media/estimate-ru-capacity-planner/advanced-mode-mongodb-api.png" alt-text="Modo avanzado de Capacity Planner" border="true":::

Los precios que se muestran en la herramienta Capacity Planner de Azure Cosmos DB son estimaciones basadas en las tarifas públicas de rendimiento y almacenamiento. Todos los precios se muestran en dólares estadounidenses. Consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para ver todas las tarifas por región.  

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [modelo de precios de Azure Cosmos DB](../how-pricing-works.md).
* Cree una nueva [cuenta, base de datos y contenedor de Cosmos](../create-cosmosdb-resources-portal.md).
* Obtenga información para [optimizar el costo del rendimiento aprovisionado](../optimize-cost-throughput.md).
* Obtenga información para [optimizar el costo con capacidad reservada](../cosmos-db-reserved-capacity.md).
