---
title: Azure Data Explorer (Kusto)
description: En este artículo se proporciona información sobre cómo usar el conector para mover datos entre Azure Data Explorer (Kusto) y grupos de Apache Spark sin servidor.
services: synapse-analytics
ms.author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 05/19/2020
ms.reviewer: ManojRaheja
author: midesa
ms.openlocfilehash: 5b7707354d7bf63671e21d1adbaebc2ef8d71d0e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952251"
---
# <a name="azure-data-explorer-kusto-connector-for-apache-spark"></a>Conector de Azure Data Explorer (Kusto) para Apache Spark
El conector de Azure Data Explorer (Kusto) para Apache Spark está diseñado para transferir datos de forma eficaz entre clústeres de Kusto y Spark. Este conector está disponible en Python, Java y .NET. Está integrado en el entorno de ejecución de Apache Spark 2.4 para Azure Synapse.

## <a name="authentication"></a>Authentication
Al usar cuadernos de Azure Synapse o definiciones de trabajos de Apache Spark, la autenticación entre sistemas se realiza sin problemas con el servicio vinculado. El servicio de token se conecta con Azure Active Directory para obtener los tokens de seguridad que se van a usar al acceder al clúster de Kusto.

Para las canalizaciones de Azure Synapse, la autenticación usará el nombre de entidad de seguridad de servicio. Actualmente, las identidades administradas no se admiten con el conector de Azure Data Explorer.

## <a name="prerequisites"></a>Prerrequisitos 
  - [Conectarse a Azure Data Explorer](../../quickstart-connect-azure-data-explorer.md): deberá configurar un servicio vinculado para conectarse a un clúster de Kusto existente.

## <a name="limitations"></a>Limitaciones
  - El conector de Azure Data Explorer (Kusto) solo se admite actualmente en el entorno de ejecución de Apache Spark 2.4 para Azure Synapse.
  - El servicio vinculado de Azure Data Explorer solo se puede configurar con el nombre principal de servicio.
  - En los cuadernos de Azure Synapse o las definiciones de trabajos de Apache Spark, el conector de Azure Data Explorer usará el tránsito de AAD para conectarse al clúster de Kusto.


## <a name="use-the-azure-data-explorer-kusto-connector"></a>Uso del conector de Azure Data Explorer (Kusto)
En la sección siguiente se proporciona un ejemplo sencillo de cómo escribir datos en una tabla de Kusto y leer datos de ella. Consulte el [proyecto del conector de Azure Data Explorer (Kusto)](https://github.com/Azure/azure-kusto-spark) para obtener documentación detallada. 

### <a name="read-data"></a>Lectura de datos

```python
kustoDf  = spark.read \
            .format("com.microsoft.kusto.spark.synapse.datasource") \
            .option("spark.synapse.linkedService", "<link service name>") \
            .option("kustoDatabase", "<Database name>") \
            .option("kustoQuery", "<KQL Query>") \
            .load()

display(kustoDf)
```

También puede leer por lotes con el modo de distribución forzada y otras opciones avanzadas. Para más información, puede consultar la [referencia sobre las opciones de origen de Kusto](https://github.com/Azure/azure-kusto-spark/blob/master/connector/src/main/scala/com/microsoft/kusto/spark/datasource/KustoSourceOptions.scala).

```python
crp = sc._jvm.com.microsoft.azure.kusto.data.ClientRequestProperties()
crp.setOption("norequesttimeout",True)
crp.toString()

kustoDf  = spark.read \
            .format("com.microsoft.kusto.spark.synapse.datasource") \
            .option("spark.synapse.linkedService", "<link service name>") \
            .option("kustoDatabase", "<Database name>") \
            .option("kustoQuery", "<KQL Query>") \
            .option("clientRequestPropertiesJson", crp.toString()) \
            .option("readMode", 'ForceDistributedMode') \
            .load()

display(kustoDf) 
```
### <a name="write-data"></a>Escritura de datos

```python
df.write \
    .format("com.microsoft.kusto.spark.synapse.datasource") \
    .option("spark.synapse.linkedService", "<link service name>") \
    .option("kustoDatabase", "<Database name>") \
    .option("kustoTable", "<Table name>") \
    .mode("Append") \
    .save()
```
Además, también puede escribir datos por lotes proporcionando propiedades de ingesta adicionales. Para más información sobre las propiedades de ingesta admitidas, puede visitar el [material de referencia sobre las propiedades de ingesta de Kusto](/azure/data-explorer/ingestion-properties).


 ```python
extentsCreationTime = sc._jvm.org.joda.time.DateTime.now().plusDays(1)
csvMap = "[{\"Name\":\"ColA\",\"Ordinal\":0},{\"Name\":\"ColB\",\"Ordinal\":1}]"
# Alternatively use an existing csv mapping configured on the table and pass it as the last parameter of SparkIngestionProperties or use none


sp = sc._jvm.com.microsoft.kusto.spark.datasink.SparkIngestionProperties(
        False, ["dropByTags"], ["ingestByTags"], ["tags"], ["ingestIfNotExistsTags"], extentsCreationTime, csvMap, None)

df.write \
    .format("com.microsoft.kusto.spark.synapse.datasource") \
    .option("spark.synapse.linkedService", "<link service name>") \
    .option("kustoDatabase", "<Database name>") \
    .option("kustoTable", "<Table name>") \
    .option("sparkIngestionPropertiesJson", sp.toString()) \
    .option("tableCreateOptions","CreateIfNotExist") \
    .mode("Append") \
    .save()
```

## <a name="next-steps"></a>Pasos siguientes
- [Conectarse al Explorador de datos de Azure](../../quickstart-connect-azure-data-explorer.md)
- [Conector de Apache Spark para Azure Data Explorer (Kusto)](https://github.com/Azure/azure-kusto-spark)