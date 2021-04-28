---
title: Información sobre los precios de Azure Data Factory a través de ejemplos
description: En este artículo se explica y muestra el modelo de precios de Azure Data Factory con ejemplos detallados
author: shirleywangmsft
ms.author: shwang
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/14/2020
ms.openlocfilehash: 48c0f47b3b8ca5eddef76c66ad220a18018dc321
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904900"
---
# <a name="understanding-data-factory-pricing-through-examples"></a>Descripción de los precios de Data Factory a través de ejemplos

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se explica y muestra el modelo de precios de Azure Data Factory con ejemplos detallados.

> [!NOTE]
> Los precios usados en los ejemplos siguientes son hipotéticos y no pretenden reflejar precios reales.

## <a name="copy-data-from-aws-s3-to-azure-blob-storage-hourly"></a>Copia de datos desde AWS S3 a Azure Blob Storage cada hora

En este escenario, va a copiar datos desde AWS S3 a Azure Blob Storage en una programación por hora.

Para lograr el escenario, es preciso crear una canalización con los siguientes elementos:

1. Una actividad de copia con un conjunto de datos de entrada para los datos que se van a copiar desde AWS S3.

2. Un conjunto de datos de salida para los datos de Azure Storage.

3. Un desencadenador de programación para ejecutar la canalización cada hora.

   ![Diagrama que muestra una canalización con un desencadenador de programación. En la canalización, la actividad de copia fluye hacia un conjunto de datos de entrada, que fluye hacia un servicio vinculado de AWS S3 y la actividad de copia también fluye hacia un conjunto de datos de salida, que fluye hacia un servicio vinculado de Azure Storage.](media/pricing-concepts/scenario1.png)

| **Operaciones** | **Tipos y unidades** |
| --- | --- |
| Creación de un servicio vinculado | 2 entidades de lectura y escritura  |
| Creación de conjuntos de datos | 4 entidades de lectura y escritura (2 para la creación del conjunto de datos y 2 para las referencias del servicio vinculado) |
| Creación de una canalización | 3 entidades de lectura y escritura (1 para la creación de la canalización y 2 para las referencias del conjunto de datos) |
| Obtención de la canalización | 1 entidad de lectura y escritura |
| Ejecución de la canalización | 2 ejecuciones de actividad (1 para la ejecución del desencadenador y 1 para ejecuciones de la actividad) |
| Asunción de la copia de datos: tiempo de ejecución = 10 min | 10 \* 4 Azure Integration Runtime (el valor predeterminado de la unidad de integración de datos es 4). Para más información sobre las unidades de integración de datos y la optimización del rendimiento de la copia, consulte [este artículo](copy-activity-performance.md) |
| Asunción de la supervisión de la canalización: solo se produjo 1 ejecución | 2 registros de ejecución de supervisión recuperados (1 para la ejecución de la canalización y 1 para la ejecución de la actividad) |

**Precio total del escenario: 0,16811 $**

- Operaciones de Data Factory = **0,0001 $**
  - Lectura y escritura = 10\*00001 = 0,0001 $ [1 L/E = 0,50 $/50000 = 0,00001]
  - Supervisión = 2\*000005 = 0,00001 $ [1 supervisión = 0,25 $/50000 = 0,000005]
- Orquestación y ejecución la canalización = **0,168 $**
  - Ejecuciones de actividad = 001\*2 = 0,002 [1 ejecución = 1 $/1000 = 0,001]
  - Actividades de movimiento de datos = 0,166 $ (prorrateo durante 10 minutos de tiempo de ejecución. 0,25 $/hora en Azure Integration Runtime)

## <a name="copy-data-and-transform-with-azure-databricks-hourly"></a>Copiar datos y transformarlos con Azure Databricks por hora

En este escenario, quiere copiar datos desde AWS S3 a Azure Blob Storage y transformarlos con Azure Databricks en una programación por hora.

Para lograr el escenario, es preciso crear una canalización con los siguientes elementos:

1. Una actividad de copia con un conjunto de datos de entrada para los datos que desea copiar desde AWS S3 y un conjunto de datos de salida para los datos del almacenamiento de Azure.
2. Una actividad de Azure Databricks para la transformación de datos.
3. Un desencadenador de programación para ejecutar la canalización cada hora.

![Diagrama que muestra una canalización con un desencadenador de programación. En la canalización, la actividad de copia fluye hacia un conjunto de datos de entrada, un conjunto de datos de salida y una actividad de DataBricks, que se ejecuta en Azure Databricks. El conjunto de datos de entrada fluye hacia un servicio vinculado de AWS S3. El conjunto de datos de salida fluye hacia un servicio vinculado de Azure Storage.](media/pricing-concepts/scenario2.png)

| **Operaciones** | **Tipos y unidades** |
| --- | --- |
| Creación de un servicio vinculado | 3 entidades de lectura y escritura  |
| Creación de conjuntos de datos | 4 entidades de lectura y escritura (2 para la creación del conjunto de datos y 2 para las referencias del servicio vinculado) |
| Creación de una canalización | 3 entidades de lectura y escritura (1 para la creación de la canalización y 2 para las referencias del conjunto de datos) |
| Obtención de la canalización | 1 entidad de lectura y escritura |
| Ejecución de la canalización | 3 ejecuciones de actividad (1 para la ejecución del desencadenador, 2 para ejecuciones de actividad) |
| Asunción de la copia de datos: tiempo de ejecución = 10 min | 10 \* 4 Azure Integration Runtime (el valor predeterminado de la unidad de integración de datos es 4). Para más información sobre las unidades de integración de datos y la optimización del rendimiento de la copia, consulte [este artículo](copy-activity-performance.md) |
| Asunción de la supervisión de la canalización: solo se produjo 1 ejecución | 3 registros de ejecución de supervisión recuperados (1 para la ejecución de la canalización y 2 para la ejecución de la actividad) |
| Asunción de la ejecución de la actividad de Databricks: tiempo de ejecución = 10 min | Ejecución de la actividad de canalización externa de 10 min |

**Precio total del escenario: 0,16916 $**

- Operaciones de Data Factory = **0,00012 $**
  - Lectura y escritura = 11\*00001 = 0,00011 $ [1 L/E = 0,50 $/50000 = 0,00001]
  - Supervisión = 3\*000005 = 0,00001 $ [1 supervisión = 0,25 $/50000 = 0,000005]
- Orquestación y ejecución de la canalización = **0,16904 $**
  - Ejecuciones de actividad = 001\*3 = 0,003 [1 ejecución = 1 $/1000 = 0,001]
  - Actividades de movimiento de datos = 0,166 $ (prorrateo durante 10 minutos de tiempo de ejecución. 0,25 $/hora en Azure Integration Runtime)
  - Actividad de canalización externa = 0,000041 $ (prorrateo durante 10 minutos de tiempo de ejecución. 0,00025 $/hora en Azure Integration Runtime)

## <a name="copy-data-and-transform-with-dynamic-parameters-hourly"></a>Copiar datos y transformarlos con parámetros dinámicos por hora

En este escenario, quiere copiar datos desde AWS S3 a Azure Blob Storage y transformarlos con Azure Databricks (con parámetros dinámicos en el script) en una programación por hora.

Para lograr el escenario, es preciso crear una canalización con los siguientes elementos:

1. Una actividad de copia con un conjunto de datos de entrada para los datos que desea copiar desde AWS S3 y un conjunto de datos de salida para los datos del almacenamiento de Azure.
2. Una actividad de búsqueda para pasar parámetros de forma dinámica al script de transformación.
3. Una actividad de Azure Databricks para la transformación de datos.
4. Un desencadenador de programación para ejecutar la canalización cada hora.

![Diagrama que muestra una canalización con un desencadenador de programación. En la canalización, la actividad de copia fluye hacia un conjunto de datos de entrada, un conjunto de datos de salida y una actividad de búsqueda que fluye a una actividad de DataBricks, que se ejecuta en Azure Databricks. El conjunto de datos de entrada fluye hacia un servicio vinculado de AWS S3. El conjunto de datos de salida fluye hacia un servicio vinculado de Azure Storage.](media/pricing-concepts/scenario3.png)

| **Operaciones** | **Tipos y unidades** |
| --- | --- |
| Creación de un servicio vinculado | 3 entidades de lectura y escritura  |
| Creación de conjuntos de datos | 4 entidades de lectura y escritura (2 para la creación del conjunto de datos y 2 para las referencias del servicio vinculado) |
| Creación de una canalización | 3 entidades de lectura y escritura (1 para la creación de la canalización y 2 para las referencias del conjunto de datos) |
| Obtención de la canalización | 1 entidad de lectura y escritura |
| Ejecución de la canalización | 4 ejecuciones de actividad (1 para la ejecución del desencadenador, 3 para ejecuciones de actividad) |
| Asunción de la copia de datos: tiempo de ejecución = 10 min | 10 \* 4 Azure Integration Runtime (el valor predeterminado de la unidad de integración de datos es 4). Para más información sobre las unidades de integración de datos y la optimización del rendimiento de la copia, consulte [este artículo](copy-activity-performance.md) |
| Asunción de la supervisión de la canalización: solo se produjo 1 ejecución | 4 registros de ejecución de supervisión recuperados (1 para la ejecución de la canalización y 3 para la ejecución de la actividad) |
| Asunción de la ejecución de actividad de búsqueda: tiempo de ejecución = 1 min | Ejecución de la actividad de canalización de 1 min |
| Asunción de la ejecución de la actividad de Databricks: tiempo de ejecución = 10 min | Ejecución de la actividad de canalización externa de 10 min |

**Precio total del escenario: 0,17020 $**

- Operaciones de Data Factory = **0,00013 $**
  - Lectura y escritura = 11\*00001 = 0,00011 $ [1 L/E = 0,50 $/50000 = 0,00001]
  - Supervisión = 4\*000005 = 0,00002 $ [1 supervisión = 0,25 $/50000 = 0,000005]
- Orquestación y ejecución de canalizaciones = **0,17007 $**
  - Ejecuciones de actividad = 001\*4 = 0,004 [1 ejecución = 1 $/1000 = 0,001]
  - Actividades de movimiento de datos = 0,166 $ (prorrateo durante 10 minutos de tiempo de ejecución. 0,25 $/hora en Azure Integration Runtime)
  - Actividad de canalización = 0,00003 $ (prorrateo durante 1 minuto de tiempo de ejecución. 0,002 $/hora en Azure Integration Runtime)
  - Actividad de canalización externa = 0,000041 $ (prorrateo durante 10 minutos de tiempo de ejecución. 0,00025 $/hora en Azure Integration Runtime)

## <a name="using-mapping-data-flow-debug-for-a-normal-workday"></a>Uso de la depuración del flujo de datos de asignación en un día laboral normal

Como ingeniero de datos, Sam es responsable de diseñar, compilar y probar flujos de datos de asignación cada día. Sam inicia sesión en la interfaz de usuario de ADF por la mañana y habilita el modo de depuración de Data Flow. El TTL predeterminado para las sesiones de depuración es de 60 minutos. Sam trabaja a lo largo del día durante 8 horas, por lo que la sesión de depuración no expira nunca. Por lo tanto, el precio del día de Sam será:

**8 (horas) x 8 (núcleos optimizados para proceso) x 0,193 USD = 12,35 USD**

Al mismo tiempo, Chris, otro ingeniero de datos, también inicia sesión en la interfaz de usuario del explorador ADF para generar perfiles de datos y trabajar en el diseño ETL. Chris no trabaja en ADF todo el día, como Sam. Chris solo necesita usar el depurador de flujo de datos durante 1 hora en el mismo período y el mismo día que Sam. Estos son los cargos en los que Chris incurre por el uso de la depuración:

**1 (hora) x 8 (núcleos de uso general) x $0,274 = $2,19**

## <a name="transform-data-in-blob-store-with-mapping-data-flows"></a>Transformación de datos del almacén de blobs con flujo de datos de asignación

En este escenario, le interesa transformar los datos del almacén de blobs visualmente en flujos de datos de asignación de ADF según una programación por hora.

Para lograr el escenario, es preciso crear una canalización con los siguientes elementos:

1. Una actividad de Data Flow con la lógica de transformación.

2. Un conjunto de datos de entrada para los datos de Azure Storage.

3. Un conjunto de datos de salida para los datos de Azure Storage.

4. Un desencadenador de programación para ejecutar la canalización cada hora.

| **Operaciones** | **Tipos y unidades** |
| --- | --- |
| Creación de un servicio vinculado | 2 entidades de lectura y escritura  |
| Creación de conjuntos de datos | 4 entidades de lectura y escritura (2 para la creación del conjunto de datos y 2 para las referencias del servicio vinculado) |
| Creación de una canalización | 3 entidades de lectura y escritura (1 para la creación de la canalización y 2 para las referencias del conjunto de datos) |
| Obtención de la canalización | 1 entidad de lectura y escritura |
| Ejecución de la canalización | 2 ejecuciones de actividad (1 para la ejecución del desencadenador y 1 para ejecuciones de la actividad) |
| Suposiciones de Data Flow: tiempo de ejecución = 10 min + TTL de 10 minutos | 10 \* 16 núcleos de proceso general con TTL de 10 |
| Asunción de la supervisión de la canalización: solo se produjo 1 ejecución | 2 registros de ejecución de supervisión recuperados (1 para la ejecución de la canalización y 1 para la ejecución de la actividad) |

**Precio total del escenario: 1,4631 USD**

- Operaciones de Data Factory = **0,0001 $**
  - Lectura y escritura = 10\*00001 = 0,0001 $ [1 L/E = 0,50 $/50000 = 0,00001]
  - Supervisión = 2\*000005 = 0,00001 $ [1 supervisión = 0,25 $/50000 = 0,000005]
- Orquestación y ejecución de canalizaciones = **1,463 USD**
  - Ejecuciones de actividad = 001\*2 = 0,002 [1 ejecución = 1 $/1000 = 0,001]
  - Actividades de Data Flow = 1,461 USD prorrateados por 20 minutos (tiempo de ejecución de 10 minutos + TTL de 10 minutos). 0,274 USD/hora en Azure Integration Runtime con un proceso general de 16 núcleos

## <a name="data-integration-in-azure-data-factory-managed-vnet"></a>Integración de datos en VNET administrada de Azure Data Factory
En este escenario, quiere eliminar archivos originales en Azure Blob Storage y copiar datos de Azure SQL Database en Azure Blob Storage. Va a realizar esta ejecución dos veces en diferentes canalizaciones. El tiempo de ejecución de estas dos canalizaciones se superpone.
![Escenario 4](media/pricing-concepts/scenario-4.png) Para realizar el escenario, es preciso crear dos canalizaciones con los siguientes elementos:
  - Una actividad de canalización: actividad de eliminación.
  - Una actividad de copia con un conjunto de datos de entrada para los datos que se van a copiar desde Azure Blob Storage.
  - Un conjunto de datos de salida para los datos en Azure SQL Database.
  - Un desencadenador de programación para ejecutar la canalización.


| **Operaciones** | **Tipos y unidades** |
| --- | --- |
| Creación de un servicio vinculado | 4 entidades de lectura y escritura |
| Creación de conjuntos de datos | 8 entidades de lectura y escritura (4 para la creación del conjunto de datos y 4 para las referencias del servicio vinculado) |
| Creación de una canalización | 6 entidades de lectura y escritura (2 para la creación de la canalización y 4 para las referencias del conjunto de datos) |
| Obtención de la canalización | 2 entidades de lectura y escritura |
| Ejecución de la canalización | 6 ejecuciones de actividad (2 para la ejecución del desencadenador y 4 para ejecuciones de actividad) |
| Ejecución de la actividad de eliminación: cada tiempo de ejecución = 5 minutos. La ejecución de la actividad de eliminación en la primera canalización es de 10:00 AM UTC a 10:05 AM UTC. La ejecución de la actividad de eliminación en la segunda canalización es de 10:02 AM UTC a 10:07 AM UTC.|Ejecución total de actividad de canalización de 7 minutos en VNET administrada. La actividad de canalización admite hasta 50 actividades simultáneas en una VNET administrada. |
| Asunción de copia de datos: cada tiempo de ejecución = 10 minutos. La ejecución de la copia en la primera canalización es de 10:06 AM UTC a 10:15 AM UTC. La ejecución de la actividad de eliminación en la segunda canalización es de 10:08 AM UTC a 10:17 AM UTC. | 10 * 4 Azure Integration Runtime (valor predeterminado de DIU = 4). Para obtener más información sobre las unidades de integración de datos y la optimización del rendimiento de la copia, vea [este artículo](copy-activity-performance.md) |
| Asunción de la supervisión de la canalización: Solo se han producido 2 ejecuciones | 6 registros de ejecución de supervisión recuperados (2 para la ejecución de la canalización y 4 para la ejecución de la actividad) |


**Precio total del escenario: 0,45523 USD**

- Operaciones de Data Factory = 0,00023 USD
  - Lectura y escritura = 20*00001 = 0,0002 USD [1 lectura y escritura = 0,50 USD/50000 = 0,00001]
  - Supervisión = 6*000005 = 0,00003 USD [1 supervisión = 0,25 USD/50000 = 0,000005]
- Orquestación y ejecución de canalizaciones = 0,455 USD
  - Ejecuciones de actividad = 0,001*6 = 0,006 [1 ejecución = 1 USD/1000 = 0,001]
  - Actividades de movimiento de datos = 0,333 USD (prorrateo durante 10 minutos de tiempo de ejecución. 0,25 $/hora en Azure Integration Runtime)
  - Actividad de canalización = 0,116 USD (prorrateo durante 7 minutos de tiempo de ejecución. 1 USD/hora en Azure Integration Runtime)

> [!NOTE]
> Estos precios son solo para los fines de este ejemplo.

**P+F**

P: Si quiero ejecutar más de 50 actividades de canalización, ¿estas actividades se pueden ejecutar simultáneamente?

A. Se permite un máximo de 50 actividades de canalización simultáneas.  La actividad de canalización 51ª se pone en cola hasta que hay un "hueco libre". Lo mismo ocurre con las actividades externas. Se permite un máximo de 800 actividades externas simultáneas.

## <a name="next-steps"></a>Pasos siguientes

¡Ahora que comprende los precios de Azure Data Factory, puede empezar a trabajar!

- [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](quickstart-create-data-factory-portal.md)

- [Introducción al servicio Azure Data Factory](introduction.md)

- [Creación visual de Azure Data Factory](author-visually.md)
