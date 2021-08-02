---
title: Introducción al flujo de datos de limpieza y transformación en Azure Data Factory
description: Tutorial sobre cómo preparar los datos en Azure Data Factory mediante el flujo de datos de limpieza y transformación
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/08/2021
ms.openlocfilehash: 523a87c3adb77c55440392381ebe43ea24627c14
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111756659"
---
# <a name="prepare-data-with-data-wrangling"></a>Preparación de los datos con una limpieza y transformación de datos

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

La limpieza y transformación de datos en una factoría de datos permite crear mashups de Power Query interactivas de forma nativa en ADF y, después, ejecutarlas a escala dentro de una canalización de ADF.

> [!NOTE]
> La actividad de Power Query en ADF está actualmente disponible en versión preliminar pública.

## <a name="create-a-power-query-activity"></a>Creación de una actividad de Power Query

Hay dos maneras de crear una Power Query en Azure Data Factory. Una forma consiste en hacer clic en el icono del signo más y seleccionar **Flujo de datos** en el panel de recursos de Factory.

> [!NOTE]
> Anteriormente, la característica de limpieza y transformación de datos se encontraba en el flujo de trabajo del flujo de datos. Ahora, creará el mashup de limpieza y transformación de datos a partir de ```New > Power query```

![Captura de pantalla que muestra Power Query en el panel de recursos de fábrica.](media/data-flow/power-query-wrangling.png)

El otro método está en el panel de actividades del lienzo de la canalización. Abra el acordeón **Power Query** y arrastre la actividad **Power Query** al lienzo.

![Captura de pantalla que resalta la opción de limpieza y transformación de datos.](media/data-flow/power-query-activity.png)

## <a name="author-a-power-query-data-wrangling-activity"></a>Creación de una actividad de limpieza y transformación de datos de Power Query

Agregue un **Conjunto de datos de origen** para el mashup de Power Query. Puede elegir un conjunto de datos existente o crear uno. Después de guardar el mashup, puede agregar la actividad de limpieza y transformación de datos de Power Query a su canalización y seleccionar un conjunto de datos de receptor para indicarle a ADF dónde colocar los datos. Si bien puede elegir uno o más conjuntos de datos de origen, en este momento solo se permite un receptor. Elegir un conjunto de datos de receptor es opcional, pero se requiere al menos un conjunto de datos de origen.

![Limpieza y transformación](media/wrangling-data-flow/tutorial4.png)

Haga clic en **Crear** para abrir el editor de mashup Power Query Online.

En primer lugar, elegirá un origen de conjunto de datos para el editor de mashup.

![Origen de Power Query.](media/wrangling-data-flow/pq-new-source.png)

Una vez que haya completado la compilación de Power Query, se puede guardar y agregar el mashup como una actividad a la canalización. Es cuando establecerá las propiedades del conjunto de datos de receptor.

![Receptor de Power Query.](media/wrangling-data-flow/pq-new-sink.png)

Cree la Power Query de limpieza y transformación con la preparación de datos sin código. Para ver la lista de funciones disponibles, consulte las [funciones de transformación](wrangling-functions.md). ADF traduce el script M en un script de flujo de datos para que pueda ejecutar Power Query a escala mediante el entorno Spark de flujo de datos de Azure Data Factory.

![Captura de pantalla que muestra el proceso para creación de Power Query de limpieza y transformación de datos.](media/wrangling-data-flow/tutorial6.png)

## <a name="running-and-monitoring-a-power-query-data-wrangling-activity"></a>Ejecución y supervisión de una actividad de limpieza y transformación de datos de Power Query

Para una ejecución de depuración de canalización de una actividad de Power Query, haga clic en **Depurar** en el lienzo de la canalización. Una vez publicada la canalización, **Trigger now** (Desencadenar ahora) ejecuta a petición la última canalización publicada. Las canalizaciones de Power Query se pueden programar con todos los desencadenadores de Azure Data Factory existentes.

![Captura de pantalla que muestra cómo agregar una actividad de limpieza y transformación de datos de Power Query.](media/data-flow/pq-activity-001.png)

Vaya a la pestaña **Supervisión** para visualizar la salida de una ejecución de actividad de Power Query desencadenada.

![Captura de pantalla que muestra la salida de una ejecución de actividad de limpieza y transformación de datos de Power Query desencadenada.](media/wrangling-data-flow/tutorial2.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [crear un flujo de datos de asignación](tutorial-data-flow.md).
