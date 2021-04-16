---
title: Espacio de nombres jerárquico de Azure Data Lake Storage Gen2
description: Describe el concepto de un espacio de nombres jerárquico de Azure Data Lake Storage Gen2.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: normesta
ms.reviewer: jamesbak
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: de7f62dd4d4205e819278e6dc90d905e825580f2
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220511"
---
# <a name="azure-data-lake-storage-gen2-hierarchical-namespace"></a>Espacio de nombres jerárquico de Azure Data Lake Storage Gen2

La adición de un **espacio de nombres jerárquico** es un mecanismo clave que permite a Azure Data Lake Storage Gen2 proporcionar rendimiento para el sistema de archivos a precios y escala de almacenamiento de objetos. Esto permite que la colección de objetos o archivos dentro de una cuenta esté organizada en una jerarquía de directorios y subdirectorios anidados de la misma manera en que se organiza el sistema de archivos en el equipo. Con un espacio de nombres jerárquico habilitado, una cuenta de almacenamiento es capaz de proporcionar la escalabilidad y la rentabilidad del almacenamiento de objetos, con una semántica de sistema de archivos con la que los marcos y motores de análisis están familiarizados.

## <a name="the-benefits-of-a-hierarchical-namespace"></a>Ventajas de un espacio de nombres jerárquico

Las siguientes ventajas están asociadas a los sistemas de archivos que implementan un espacio de nombres jerárquico sobre los datos de blob:

- **Manipulación atómica de directorios:** los almacenamientos de objetos se aproximan a una jerarquía de directorios mediante la adopción de una convención de inserción de barras diagonales (/) en el nombre del objeto para indicar los segmentos de ruta. Aunque esta convención funciona para la organización de objetos, no proporciona asistencia para acciones como mover o eliminar directorios, ni para cambiarles el nombre. Sin directorios reales, las aplicaciones podrían tener que procesar millones de blobs individuales para lograr realizar las tareas de nivel de directorio. Por el contrario, un espacio de nombres jerárquico procesa estas tareas mediante la actualización de una sola entrada (el directorio principal).

    Esta notable optimización es especialmente significativa para muchos marcos de análisis de macrodatos. Con frecuencia, herramientas como Hive, Spark, etc. escriben la salida en ubicaciones temporales y, a continuación, cambian el nombre de la ubicación al finalizar el trabajo. Sin un espacio de nombres jerárquico, muchas veces este cambio de nombre puede tardar más tiempo que el propio proceso de análisis. La disminución de la latencia de trabajo equivale a la reducción del costo total de propiedad (TCO) de las cargas de trabajo de análisis.

- **Estilo de interfaz familiar:** tanto los desarrolladores como los usuarios comprenden correctamente los sistemas de archivos. Al moverse a la nube, no es necesario aprender un nuevo paradigma de almacenamiento, ya que la interfaz del sistema de archivo que presenta Data Lake Storage Gen2 es el mismo paradigma que usan los equipos, grandes y pequeños.

Uno de los motivos por los que históricamente los almacenamientos de objetos no admiten espacios de nombres jerárquicos es porque estos limitan la escala. Sin embargo, el espacio de nombres jerárquico de Data Lake Storage Gen2 se escala linealmente y no degrada la capacidad de datos ni el rendimiento.

## <a name="deciding-whether-to-enable-a-hierarchical-namespace"></a>Decisión de si habilitar un espacio de nombres jerárquico

Una vez que haya habilitado un espacio de nombres jerárquico en su cuenta, no podrá revertirlo a un espacio de nombres plano. Por lo tanto, tenga en cuenta si tiene sentido habilitar un espacio de nombres jerárquico en función de la naturaleza de las cargas de trabajo del almacén de objetos.

Puede que algunas cargas de trabajo no obtengan ninguna ventaja al habilitar un espacio de nombres jerárquico. Algunos ejemplos incluyen las copias de seguridad, el almacenamiento de imágenes y otras aplicaciones donde la organización de objetos y los propios objetos se almacenan por separado (por ejemplo, en una base de datos independiente). 

Además, aunque la compatibilidad con las características de almacenamiento de blobs y el ecosistema de servicios de Azure sigue creciendo, todavía hay algunas características y servicios de Azure que aún no se admiten en cuentas que tienen un espacio de nombres jerárquico. Consulte [Problemas conocidos](data-lake-storage-known-issues.md). 

En general, se recomienda activar un espacio de nombres jerárquico para las cargas de trabajo de almacenamiento diseñadas para sistemas de archivos que manipulan directorios. Esto incluye todas las cargas de trabajo que son principalmente para el procesamiento de análisis. Los conjuntos de datos que requieren un alto grado de organización también se beneficiarán de la habilitación de un espacio de nombres jerárquico.

Las razones para habilitar un espacio de nombres jerárquico las determina un análisis de TCO. Por lo general, las mejoras en la latencia de carga de trabajo debido a la aceleración de almacenamiento requerirán recursos de proceso durante menos tiempo. La latencia de muchas cargas de trabajo se puede mejorar gracias a la manipulación atómica de directorios que habilita un espacio de nombres jerárquico. En muchas cargas de trabajo, el recurso de proceso representa más del 85 % del costo total e incluso una reducción modesta de latencia de carga de trabajo equivale a una cantidad significativa de los ahorros de TCO. Incluso en los casos en los que al habilitar un espacio de nombres jerárquico aumentan los costos de almacenamiento, el TCO aún disminuye debido a los costos de proceso reducidos.

Para analizar las diferencias en los precios del almacenamiento de datos, los precios de las transacciones y los precios de las reservas de capacidad de almacenamiento entre cuentas que tienen un espacio de nombres jerárquico plano en lugar de un espacio de nombres jerárquico, consulte [Precios de Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

## <a name="enabling-a-hierarchical-namespace"></a>Habilitación de un espacio de nombres jerárquico

No se puede habilitar un espacio de nombres jerárquico en una cuenta de almacenamiento existente. Solo puede habilitar un espacio de nombres jerárquico al crear una cuenta de almacenamiento. Para obtener instrucciones, consulte [Creación de una cuenta de almacenamiento para su uso con Azure Data Lake Storage Gen2 habilitado](create-data-lake-storage-account.md).

## <a name="next-steps"></a>Pasos siguientes

- [Crear una cuenta de almacenamiento](../common/storage-account-create.md)