---
title: Apache Spark trabaja lento cuando el almacenamiento para Azure HDInsight tiene muchos archivos
description: El trabajo de Apache Spark se ejecuta lentamente cuando el contenedor de almacenamiento de Azure contiene muchos archivos en Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/21/2019
ms.openlocfilehash: d256292956b42c65722ea5f9c87d6835297d1d40
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106219100"
---
# <a name="apache-spark-job-run-slowly-when-the-azure-storage-container-contains-many-files-in-azure-hdinsight"></a>El trabajo de Apache Spark se ejecuta lentamente cuando el contenedor de almacenamiento de Azure contiene muchos archivos en Azure HDInsight

En este artículo se describen los pasos de solución de problemas y las posibles soluciones para los problemas que se producen al usar componentes de Apache Spark en clústeres de Azure HDInsight.

## <a name="issue"></a>Problema

Cuando se ejecuta un clúster de HDInsight, el trabajo de Apache Spark que escribe en el contenedor de almacenamiento de Azure se vuelve lento si hay muchos archivos o subcarpetas. Por ejemplo, se tardan 20 segundos en escribir en un nuevo contenedor, pero aproximadamente 2 minutos cuando se escribe en un contenedor que tiene 200 000 archivos.

## <a name="cause"></a>Causa

Se trata de un problema conocido de Spark. La lentitud procede de las operaciones `ListBlob` y `GetBlobProperties` durante la ejecución del trabajo de Spark.

Para realizar el seguimiento de las particiones, Spark tiene que mantener una caché de tipo `FileStatusCache` que contiene información sobre la estructura de directorios. Con esta memoria caché, Spark puede analizar las rutas de acceso y tener en cuenta las particiones disponibles. La ventaja de realizar el seguimiento de las particiones es que Spark solo toca los archivos necesarios al leer los datos. Para mantener esta información actualizada, al escribir nuevos datos Spark tiene que enumerar todos los archivos del directorio y actualizar esta caché.

En Spark 2.1, aunque no es necesario actualizar la caché después de cada escritura, Spark comprobará si una columna de partición existente coincide con la propuesta en la solicitud de escritura actual, por lo que también realizará operaciones de enumeración al principio de cada escritura.

En Spark 2.2 deberá corregirse este problema de rendimiento al escribir datos en modo de anexión.

En Spark 2.3, se espera el mismo comportamiento que en Spark 2.2.

## <a name="resolution"></a>Resolución

Cuando se crea un conjunto de datos con particiones, es importante usar un esquema de partición que limite el número de archivos que Spark tiene que enumerar para actualizar la caché `FileStatusCache`.

Por cada N microlote en el que N % 100 = = 0 (100 es solo un ejemplo), mueva los datos existentes a otro directorio, que puede cargar Spark.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]
