---
title: Guía de escalabilidad y rendimiento de la actividad de copia
description: Conozca los factores más importantes que afectan al rendimiento del movimiento de datos en Azure Data Factory cuando se usa la actividad de copia.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/15/2020
ms.openlocfilehash: 75f9080b43333168802a72e60751eec2a765c6d4
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580822"
---
# <a name="copy-activity-performance-and-scalability-guide"></a>Guía de escalabilidad y rendimiento de la actividad de copia

> [!div class="op_single_selector" title1="Seleccione la versión de Azure Data Factory que usa:"]
> * [Versión 1](v1/data-factory-copy-activity-performance.md)
> * [Versión actual](copy-activity-performance.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

A veces, desea realizar una migración de datos a gran escala desde el lago de datos o el almacenamiento de datos empresariales (EDW) a Azure. Otras veces quiere ingerir grandes cantidades de datos, desde distintos orígenes en Azure, para el análisis de macrodatos. En cada caso, es fundamental lograr un rendimiento y una escalabilidad óptimos.

Azure Data Factory (ADF) proporciona un mecanismo para ingerir datos. ADF tiene las ventajas siguientes:

* Administra grandes cantidades de datos
* Tiene un alto rendimiento
* Es rentable

Estas ventajas hacen de ADF una buena opción para aquellos ingenieros de datos que desean crear canalizaciones de ingesta de datos escalables con un alto rendimiento.

Después de leer este artículo, podrá responder a las siguientes preguntas:

* ¿Qué nivel de rendimiento y escalabilidad puedo conseguir si uso la actividad de copia de ADF para escenarios de migración de datos y de ingesta de datos?
* ¿Qué pasos se deben seguir para optimizar el rendimiento de la actividad de copia de ADF?
* ¿Qué mecanismos de optimización del rendimiento de ADF puedo utilizar para optimizar el rendimiento de una única ejecución de la actividad de copia?
* ¿Qué otros factores ajenos a ADF se deben tener en cuenta al optimizar el rendimiento de la copia?

> [!NOTE]
> Si no está familiarizado con la actividad de copia en general, consulte la [información general de la actividad de copia](copy-activity-overview.md) antes de leer este artículo.

## <a name="copy-performance-and-scalability-achievable-using-adf"></a>Rendimiento y escalabilidad de copia que pueden lograrse mediante ADF

ADF ofrece una arquitectura sin servidor que permite el paralelismo en diferentes niveles.

Esta arquitectura hace posible el desarrollo de canalizaciones que maximizan el rendimiento del movimiento de datos para su entorno. Estas canalizaciones hacen un uso completo de los siguientes recursos:

* Ancho de banda de red entre los almacenes de datos de origen y destino
* Operaciones de entrada/salida por segundo (IOPS) y ancho de banda del almacén de datos de origen o destino

Este uso completo significa que puede calcular el rendimiento general midiendo el rendimiento mínimo disponible con los siguientes recursos:

* Almacén de datos de origen
* Almacén de datos de destino
* Ancho de banda de red entre los almacenes de datos de origen y de destino

En la siguiente tabla se muestra el cálculo de la duración del movimiento de datos. La duración de cada celda se calcula en función de un determinado ancho de banda de red y almacén de datos y de un determinado tamaño de carga de datos.

> [!NOTE]
> La duración proporcionada a continuación está pensada para representar un rendimiento factible en una solución de integración de datos de un extremo a otro que se haya implementado mediante ADF, usando una o varias de las técnicas de optimización del rendimiento descritas en [Características de optimización del rendimiento de la actividad de copia](#copy-performance-optimization-features). Entre estas técnicas se incluye el uso de ForEach para crear particiones y generar varias actividades de copia simultáneas. Se recomienda seguir el procedimiento descrito en [Pasos de optimización del rendimiento](#performance-tuning-steps) con el fin de optimizar el rendimiento de la actividad de copia para su conjunto de datos y su configuración del sistema en particular. Debe usar los números obtenidos en las pruebas de optimización del rendimiento con relación al planeamiento de la implementación de producción, el planeamiento de la capacidad y la proyección de facturación.

&nbsp;

| Tamaño de los datos / <br/> bandwidth | 50 Mbps    | 100 Mbps  | 500 Mbps  | 1 Gbps   | 5 Gbps   | 10 Gbps  | 50 Gbps   |
| --------------------------- | ---------- | --------- | --------- | -------- | -------- | -------- | --------- |
| **1 GB**                    | 2,7 min    | 1,4 min   | 0,3 min   | 0,1 min  | 0,03 min | 0,01 min | 0,0 min   |
| **10 GB**                   | 27,3 min   | 13,7 min  | 2,7 min   | 1,3 min  | 0,3 min  | 0,1 min  | 0,03 min  |
| **100 GB**                  | 4,6 h    | 2,3 h   | 0,5 h   | 0,2 h  | 0,05 h | 0,02 h | 0,0 h   |
| **1 TB**                    | 46,6 h   | 23,3 h  | 4,7 h   | 2,3 h  | 0,5 h  | 0,2 h  | 0,05 h  |
| **10 TB**                   | 19,4 días  | 9,7 días  | 1,9 días  | 0,9 días | 0,2 días | 0,1 días | 0,02 días |
| **100 TB**                  | 194,2 días | 97,1 días | 19,4 días | 9,7 días | 1,9 días | 1 día    | 0,2 días  |
| **1 PB**                    | 64,7 meses    | 32,4 meses   | 6,5 meses    | 3,2 meses   | 0,6 meses   | 0,3 meses   | 0,06 meses   |
| **10 PB**                   | 647,3 meses   | 323,6 meses  | 64,7 meses   | 31,6 meses  | 6,5 meses   | 3,2 meses   | 0,6 meses    |
| | |  | | |  | | |

La copia de ADF puede escalarse en diferentes niveles:

![Cómo se escala la copia de ADF](media/copy-activity-performance/adf-copy-scalability.png)

* El flujo de control de ADF puede iniciar varias actividades de copia en paralelo, por ejemplo, mediante un [bucle ForEach](control-flow-for-each-activity.md).

* Una sola actividad de copia puede aprovechar los recursos de proceso escalables.
  * Al usar el entorno de ejecución de integración de Azure, puede especificar [hasta 256 unidades de integración de datos (DIU)](#data-integration-units) para cada actividad de copia sin servidor.
  * Al usar IR autohospedado, puede utilizar uno de los métodos siguientes:
    * Escalar verticalmente la máquina de forma manual.
    * Escalar horizontalmente a varias máquinas ([hasta 4 nodos](create-self-hosted-integration-runtime.md#high-availability-and-scalability)), y una única actividad de copia dividirá su conjunto de archivos en todos los nodos.

* Una única actividad de copia lee y escribe en el almacén de datos mediante varios subprocesos [en paralelo](#parallel-copy).

## <a name="performance-tuning-steps"></a>Pasos de optimización del rendimiento

Para optimizar el rendimiento del servicio Azure Data Factory con la actividad de copia, siga estos pasos:

1. **Seleccione un conjunto de datos de prueba y establezca una línea de base**.

    Durante el desarrollo, pruebe la canalización, para lo que debe usar la actividad de copia con unos datos de ejemplo representativos. El conjunto de datos que elija debe representar los patrones de datos típicos a lo largo de los siguientes atributos:

    * Estructura de carpetas
    * Patrón de archivo
    * Esquema de datos

    Y el conjunto de datos debe ser lo suficientemente grande como para evaluar el rendimiento de la copia. Para un buen tamaño, la actividad de copia tarda al menos 10 minutos en completarse. Recopile los detalles de la ejecución y las características del rendimiento después de la [supervisión de la actividad de copia](copy-activity-monitoring.md).

2. **Cómo maximizar el rendimiento de una única actividad de copia**:

    Se recomienda maximizar primero el rendimiento mediante una única actividad de copia.

    * **Si la actividad de copia se va a ejecutar en un entorno de ejecución de integración de _Azure_:**

        Comience con los valores predeterminados de las [unidades de integración de datos (DIU)](#data-integration-units) y la configuración de [copia paralela](#parallel-copy).

    * **Si la actividad de copia se va a ejecutar en un entorno de ejecución de integración _autohospedado_:**

        Se recomienda usar una máquina dedicada para hospedar el entorno de ejecución de integración. La máquina debe ser independiente del servidor que hospeda el almacén de datos. Comience con los valores predeterminados para la configuración de [copia en paralelo](#parallel-copy) y use un solo nodo para el IR autohospedado.

    Realice una serie de pruebas de rendimiento. Tome nota del rendimiento conseguido. Incluya los valores reales usados, como las DIU y las copias paralelas. Consulte la [supervisión de la actividad de copia](copy-activity-monitoring.md) para obtener información sobre cómo recopilar los resultados de las pruebas y la configuración de rendimiento. Obtenga información acerca de cómo [solucionar problemas de rendimiento de la actividad de copia](copy-activity-performance-troubleshooting.md) para identificar y resolver el cuello de botella.

    Recorra en iteración para realizar series de pruebas de rendimiento adicionales siguiendo las instrucciones para la solución de problemas y la guía de optimización. Una vez que la ejecución de la actividad de copia única no pueda lograr un mejor rendimiento, considere la posibilidad de maximizar el rendimiento agregado mediante la ejecución de varias copias de forma simultánea. Esta opción se describe en la siguiente viñeta numerada.

3. **Cómo maximizar el rendimiento agregado mediante la ejecución de varias copias simultáneamente:**

    Ahora ha maximizado el rendimiento de una única actividad de copia. Si aún no ha llegado a los límites superiores de rendimiento de su entorno, puede ejecutar varias actividades de copia en paralelo. Puede ejecutar en paralelo mediante construcciones de flujo de control de ADF. Una de estas construcciones el [bucle For Each](control-flow-for-each-activity.md). Para obtener más información, consulte los siguientes artículos sobre las plantillas de solución:

    * [Copia de archivos de varios contenedores](solution-template-copy-files-multiple-containers.md)
    * [Migración de datos de Amazon S3 a ADLS Gen2](solution-template-migration-s3-azure.md)
    * [Copia masiva con una tabla de controles](solution-template-bulk-copy-with-control-table.md)

4. **Expanda la configuración a todo el conjunto de datos.**

    Cuando esté satisfecho con los resultados y el rendimiento de la ejecución, puede expandir la definición y la canalización para cubrir todo el conjunto de datos.

## <a name="troubleshoot-copy-activity-performance"></a>Solución de problemas de rendimiento de la actividad de copia

Siga los [pasos de optimización del rendimiento](#performance-tuning-steps) para planear y realizar la prueba de rendimiento de su escenario. Y aprenda a solucionar los problemas de rendimiento de la ejecución de la actividad de copia en Azure Data Factory desde [Solución de problemas de rendimiento de la actividad de copia](copy-activity-performance-troubleshooting.md).

## <a name="copy-performance-optimization-features"></a>Características de optimización del rendimiento

Azure Data Factory proporciona las siguientes características de optimización del rendimiento:

* [Unidades de integración de datos](#data-integration-units)
* [Escalabilidad del entorno de ejecución de integración autohospedado](#self-hosted-integration-runtime-scalability)
* [Copia paralela](#parallel-copy)
* [Copias almacenadas provisionalmente](#staged-copy)

### <a name="data-integration-units"></a>Unidades de integración de datos

Una unidad de integración de datos (DIU) es una medida que representa la potencia de una sola unidad en Azure Data Factory. La potencia es una combinación de CPU, memoria y asignación de recursos de red. La DIU solo se aplica al [entorno de ejecución de integración de Azure](concepts-integration-runtime.md#azure-integration-runtime). La DIU no se aplica al [entorno de ejecución de integración autohospedado](concepts-integration-runtime.md#self-hosted-integration-runtime). [Obtenga más información aquí](copy-activity-performance-features.md#data-integration-units).

### <a name="self-hosted-integration-runtime-scalability"></a>Escalabilidad del entorno de ejecución de integración autohospedado

Es posible que desee hospedar una carga de trabajo simultánea creciente. O bien, puede que desee conseguir un mayor rendimiento en el nivel de carga de trabajo actual. Puede mejorar la escala del procesamiento mediante los siguientes métodos:

* Puede escalar _verticalmente_ el entorno de ejecución de integración autohospedado aumentando el número de [trabajos simultáneos](create-self-hosted-integration-runtime.md#scale-up) que se pueden ejecutar en un nodo.  
El escalado vertical solo funciona si el procesador y la memoria del nodo son inferiores al uso completo.
* Puede escalar _horizontalmente_ el entorno de ejecución de integración autohospedado agregando más nodos (máquinas).

Para más información, consulte:

* [Características de optimización del rendimiento de la actividad de copia: escalabilidad del entorno de ejecución de integración autohospedado](copy-activity-performance-features.md#self-hosted-integration-runtime-scalability)
* [Creación y configuración de un entorno de ejecución de integración autohospedado: consideraciones de escala](create-self-hosted-integration-runtime.md#scale-considerations)

### <a name="parallel-copy"></a>Copia en paralelo

Puede establecer la propiedad `parallelCopies` para indicar el paralelismo que desea que utilice la actividad de copia. Considere esta propiedad como el número máximo de subprocesos dentro de la actividad de copia. Los subprocesos operan en paralelo. Los subprocesos leen desde el origen o escriben en los almacenes de datos receptores. [Obtenga más información](copy-activity-performance-features.md#parallel-copy).

### <a name="staged-copy"></a>copia almacenada provisionalmente

Una operación de copia de datos puede enviar los datos _directamente_ al almacén de datos receptor. Como alternativa, puede usar el almacenamiento de blobs como almacenamiento _provisional_. [Más información](copy-activity-performance-features.md#staged-copy).

## <a name="next-steps"></a>Pasos siguientes

Consulte los restantes artículos acerca de la actividad de copia:

* [Información general de la actividad de copia](copy-activity-overview.md)
* [Solución de problemas de rendimiento de la actividad de copia](copy-activity-performance-troubleshooting.md)
* [Características de optimización del rendimiento de la actividad de copia](copy-activity-performance-features.md)
* [Uso de Azure Data Factory para migrar datos del lago de datos y el almacenamiento de datos a Azure](data-migration-guidance-overview.md)
* [Migración de datos de AWS S3 a Azure Storage](data-migration-guidance-s3-azure-storage.md)
