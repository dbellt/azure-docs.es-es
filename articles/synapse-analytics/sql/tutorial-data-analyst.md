---
title: 'Tutorial: Uso de un grupo de SQL sin servidor para analizar Azure Open Datasets en Synapse Studio'
description: Este tutorial le mostrará cómo realizar con facilidad el análisis de datos exploratorios mediante la combinación de diferentes instancias de Azure Open Datasets con un grupo de SQL sin servidor y visualizar los resultados en Synapse Studio.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 11/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: eed38a5dfeafee6a951ea745f3fa12ba11a36699
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109813729"
---
# <a name="tutorial-explore-and-analyze-data-lakes-with-serverless-sql-pool"></a>Tutorial: Exploración y análisis de lagos de datos con un grupo de SQL sin servidor

En este tutorial, aprenderá a realizar análisis de datos exploratorios. Combinará diferentes conjuntos de datos de Azure Open Datasets utilizando un grupo de SQL sin servidor. A continuación, visualizará los resultados en la característica Synapse Studio de Azure Synapse Analytics.

La función OPENROWSET (BULK...) permite acceder a archivos en Azure Storage. La función [OPENROWSET](develop-openrowset.md) lee el contenido de un origen de datos remoto (por ejemplo, un archivo) y lo devuelve en forma de un conjunto de filas.

## <a name="automatic-schema-inference"></a>Inferencia automática del esquema

Puesto que los datos se almacenan en el formato de archivo parquet, está disponible la inferencia de esquemas automática. Puede consultar fácilmente los datos sin enumerar los tipos de datos de todas las columnas de los archivos. También puede utilizar el mecanismo de la columna virtual y la función FilePath para filtrar un determinado subconjunto de archivos.

> [!NOTE]
> Si usa una base de datos con una intercalación que no sea la predeterminada (la intercalación predeterminada es SQL_Latin1_General_CP1_CI_AS), debe tener en cuenta la distinción entre mayúsculas y minúsculas. 
> 
> Si crea una base de datos con intercalación que distingue mayúsculas de minúsculas, al especificar columnas, asegúrese de usar el nombre correcto de la columna.
> 
> Un nombre de columna de ejemplo, como "tpepPickupDateTime", sería correcto, mientras que "tpeppickupdatetime" no funcionaría en la intercalación no predeterminada.

En primer lugar, vamos a familiarizarnos con los datos de los taxis de Nueva York mediante la ejecución de la siguiente consulta:

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
```

El [conjunto de datos New York City (NYC) Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) contiene:

- Fechas y horas de recogida y llegada a destino.
- Ubicaciones de recogida y llegada a destino. 
- Distancias de la carrera.
- Tarifas desglosadas.
- Tipos de tarifa.
- Formas de pago. 
- Recuentos de pasajeros indicados por el conductor.

Del mismo modo, puede consultar el conjunto de datos de los días festivos locales y nacionales mediante la siguiente consulta:

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
        FORMAT='PARQUET'
    ) AS [holidays]
```

Por último, también puede consultar el conjunto de datos meteorológicos mediante la siguiente consulta:

```sql
SELECT
    TOP 100 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
```

Puede ver más detalles sobre el significado de cada una de las columnas individuales de las descripciones de los siguientes conjuntos de datos: 
- [NYC Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/)
- [Días festivos públicos](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/)
- [Weather Data](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/)

## <a name="time-series-seasonality-and-outlier-analysis"></a>Análisis de series temporales, estacionalidad y valores atípicos

Puede resumir fácilmente el número anual de carreras de taxi con la siguiente consulta:

```sql
SELECT
    YEAR(tpepPickupDateTime) AS current_year,
    COUNT(*) AS rides_per_year
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) >= '2009' AND nyc.filepath(1) <= '2019'
GROUP BY YEAR(tpepPickupDateTime)
ORDER BY 1 ASC
```

El fragmento a continuación muestra el resultado para el número anual de carreras de taxi:

![Fragmento del resultado del número anual de carreras de taxi](./media/tutorial-data-analyst/yearly-taxi-rides.png)

Los datos se pueden visualizar en Synapse Studio realizando un cambio de la vista de **Tabla** a la de **Gráfico**. Puede elegir entre diferentes tipos de gráficos: **de área**, **de barras**, **de columnas**, **de líneas**, **circular** y **de dispersión**. En este caso, vamos a trazar el gráfico **de columnas** con la columna de **Categoría** establecida en **current_year**:

![Gráfico de columnas que muestra las carreras por año](./media/tutorial-data-analyst/column-chart-rides-year.png)

En esta visualización, puede ver una tendencia descendente en el número de carreras con el paso de los años. Posiblemente, esta disminución se debe al aumento de la popularidad de las empresas de uso compartido de vehículos.

> [!NOTE]
> En el momento de escribir este tutorial, los datos de 2019 están incompletos. Como resultado, hay una gran caída en el número de carreras de ese año.

A continuación, vamos a centrar el análisis en un solo año, por ejemplo, 2016. La siguiente consulta devuelve el número diario de carreras durante ese año:

```sql
SELECT
    CAST([tpepPickupDateTime] AS DATE) AS [current_day],
    COUNT(*) as rides_per_day
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) = '2016'
GROUP BY CAST([tpepPickupDateTime] AS DATE)
ORDER BY 1 ASC
```

El fragmento siguiente muestra el resultado de esta consulta:

![Fragmento del resultado del número diario de carreras para 2016](./media/tutorial-data-analyst/daily-rides.png)

De nuevo, puede visualizar los datos fácilmente trazando el gráfico de **columnas** con la columna de **Categoría** establecida en **current_day** y la columna de **Leyenda (series)** establecida en **rides_per_day**.

![Gráfico de columnas que muestra el número diario de carreras para 2016](./media/tutorial-data-analyst/column-chart-daily-rides.png)

En el gráfico trazado, puede ver que hay un patrón semanal, con los sábados como día de máxima actividad. Durante los meses de verano, hay menos carreras de taxi debido al período de vacaciones. Observe igualmente que hay algunas reducciones significativas en el número de carreras de taxi sin un patrón claro de cuándo y por qué se producen.

A continuación, vamos a ver si este descenso de las carreras está relacionado con las días festivos. Para determinar si hay alguna relación, podemos combinar el conjunto de datos de las carreras, NYC Taxi, con el conjunto de datos Public Holidays:

```sql
WITH taxi_rides AS
(
    SELECT
        CAST([tpepPickupDateTime] AS DATE) AS [current_day],
        COUNT(*) as rides_per_day
    FROM  
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
            FORMAT='PARQUET'
        ) AS [nyc]
    WHERE nyc.filepath(1) = '2016'
    GROUP BY CAST([tpepPickupDateTime] AS DATE)
),
public_holidays AS
(
    SELECT
        holidayname as holiday,
        date
    FROM
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
            FORMAT='PARQUET'
        ) AS [holidays]
    WHERE countryorregion = 'United States' AND YEAR(date) = 2016
)
SELECT
*
FROM taxi_rides t
LEFT OUTER JOIN public_holidays p on t.current_day = p.date
ORDER BY current_day ASC
```

![Visualización de los resultados de los conjuntos de datos de carreras de taxi de Nueva York y días festivos locales y nacionales](./media/tutorial-data-analyst/rides-public-holidays.png)

Esta vez, queremos resaltar el número de carreras de taxi durante los días festivos locales y nacionales. Para ello, elegiremos **none** en la columna de **Categoría** y **rides_per_day** y **holiday** como columnas de **Leyenda (series)** .

![Gráfico trazado del número de carreras de taxi durante los días festivos locales y nacionales](./media/tutorial-data-analyst/plot-chart-public-holidays.png)

En el gráfico trazado, puede ver que durante los días festivos locales y nacionales, el número de carreras de taxi es inferior. Todavía hay una gran reducción no explicada el 23 de enero. Vamos a comprobar el tiempo en Nueva York en ese día consultando el conjunto de datos meteorológicos:

```sql
SELECT
    AVG(windspeed) AS avg_windspeed,
    MIN(windspeed) AS min_windspeed,
    MAX(windspeed) AS max_windspeed,
    AVG(temperature) AS avg_temperature,
    MIN(temperature) AS min_temperature,
    MAX(temperature) AS max_temperature,
    AVG(sealvlpressure) AS avg_sealvlpressure,
    MIN(sealvlpressure) AS min_sealvlpressure,
    MAX(sealvlpressure) AS max_sealvlpressure,
    AVG(precipdepth) AS avg_precipdepth,
    MIN(precipdepth) AS min_precipdepth,
    MAX(precipdepth) AS max_precipdepth,
    AVG(snowdepth) AS avg_snowdepth,
    MIN(snowdepth) AS min_snowdepth,
    MAX(snowdepth) AS max_snowdepth
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
WHERE countryorregion = 'US' AND CAST([datetime] AS DATE) = '2016-01-23' AND stationname = 'JOHN F KENNEDY INTERNATIONAL AIRPORT'
```

![Visualización del resultado del conjunto de datos meteorológico](./media/tutorial-data-analyst/weather-data-set-visualization.png)

Los resultados de la consulta indican que la reducción del número de viajes de taxi se debió a:

- Una tormenta de nieve ese día en Nueva York con gran acumulación de nieve (~30 cm).
- Hizo frío (una temperatura inferior a cero grados Celsius).
- Hizo viento (~ 10 m/s).

En este tutorial se ha mostrado la forma en que un analista de datos puede realizar rápidamente análisis de datos exploratorios, combinar de forma sencilla diferentes conjuntos de datos mediante un grupo de SQL sin servidor y visualizar los resultados mediante Azure Synapse Studio.

## <a name="next-steps"></a>Pasos siguientes

Para aprender a conectar un grupo de SQL sin servidor a Power BI Desktop y crear informes, consulte el artículo [Conexión de un grupo de SQL sin servidor a Power BI Desktop y creación de informes](tutorial-connect-power-bi-desktop.md).

Para más información sobre el uso de tablas externas en los grupos de SQL sin servidor, consulte [Uso de tablas externas con Synapse SQL](develop-tables-external-tables.md?tabs=sql-pool).
 
