---
title: Almacenes de datos admitidos en Azure Data Share
description: Obtenga información sobre los almacenes de datos que se pueden usar en Azure Data Share.
ms.service: data-share
author: jifems
ms.author: jife
ms.topic: conceptual
ms.date: 04/20/2021
ms.openlocfilehash: def73d137f3cc2c79ae8417995ec6bdf6c519b7d
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107812631"
---
# <a name="supported-data-stores-in-azure-data-share"></a>Almacenes de datos admitidos en Azure Data Share

Azure Data Share proporciona un uso compartido de datos abierto y flexible, lo que incluye la habilidad de compartir desde y hacia diferentes almacenes de datos. Los proveedores de datos pueden compartir datos de un tipo de almacén de datos y los consumidores pueden elegir en qué almacén van a recibir los datos. 

En este artículo obtendrá información sobre el amplio conjunto de almacenes de datos de Azure que se admiten en Azure Data Share. También aprenderá cómo los proveedores y los consumidores de datos pueden combinar distintos almacenes de datos. 

## <a name="supported-data-stores"></a>Almacenes de datos compatibles 

En la siguiente tabla se explican los almacenes de datos que admite Azure Data Share. 

| Almacén de datos | Uso compartido basado en instantáneas completas | Uso compartido basado en instantáneas incrementales | Uso compartido en contexto 
|:--- |:--- |:--- |:--- |:--- |:--- |:--- |
| Azure Blob Storage |✓ |✓ | |
| Azure Data Lake Storage Gen1 |✓ |✓ | |
| Azure Data Lake Storage Gen2 |✓ |✓ ||
| Azure SQL Database |✓ | | |
| Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse) |✓ | | |
| Grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo) |✓ | | |
| Explorador de datos de Azure | | |✓ |

## <a name="data-store-support-matrix"></a>Matriz de soporte de almacén de datos

Azure Data Share permite a los consumidores de datos elegir un almacén para aceptar los datos. Por ejemplo, los datos compartidos de Azure SQL Database se pueden recibir en Azure Data Lake Storage Gen2, Azure SQL Database o Azure Synapse Analytics. Cuando los clientes configuran un recurso compartido de datos recibidos, pueden elegir en qué formato los reciben. 

En la siguiente tabla se explican las combinaciones y opciones que los consumidores de datos pueden elegir cuando aceptan y configuran un recurso compartido de datos. Para obtener más información, consulte cómo [configurar una asignación de conjunto de datos](how-to-configure-mapping.md).

| Almacén de datos | Blob Storage | Data Lake Storage Gen1 | Data Lake Storage Gen2 | SQL Database | Synapse Analytics (anteriormente SQL Data Warehouse) | Grupo de SQL dedicado de Synapse Analytics (área de trabajo) | Data Explorer
|:--- |:--- |:--- |:--- |:--- |:--- |:--- | :--- |
| Blob Storage | ✓ || ✓ |||
| Data Lake Storage Gen1 | ✓ | | ✓ |||
| Data Lake Storage Gen2 | ✓ | | ✓ |||
| SQL Database | ✓ | | ✓ | ✓ | ✓ | ✓ ||
| Synapse Analytics (anteriormente SQL Data Warehouse) | ✓ | | ✓ | ✓ | ✓ | ✓ ||
| Grupo de SQL dedicado de Synapse Analytics (área de trabajo) | ✓ | | ✓ | ✓ | ✓ | ✓ ||
| Data Explorer ||||||| ✓ |

## <a name="share-from-a-storage-account"></a>Uso compartido desde una cuenta de almacenamiento
Azure Data Share admite el uso compartido de archivos, carpetas y sistemas de archivos desde Azure Data Lake Storage Gen1 y Azure Data Lake Storage Gen2. También admite el uso compartido de blobs, carpetas y contenedores desde Azure Blob Storage. Puede compartir blobs en bloques, en anexos o en páginas, y se reciben como blobs en bloques.

Cuando los sistemas de archivos, los contenedores o las carpetas se comparten en el uso compartido basado en instantáneas, los consumidores de datos pueden optar por hacer una copia completa de los datos compartidos. O bien, pueden usar la funcionalidad de instantáneas incrementales para copiar solo los archivos nuevos o actualizados. 

La instantánea incremental se basa en la hora de última modificación de los archivos. Los archivos existentes con el mismo nombre que los archivos de los datos recibidos se sobrescriben en la instantánea. Los archivos que se eliminan del origen no se eliminan en el destino. 

Para obtener más información, consulte[Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage](how-to-share-from-storage.md).

## <a name="share-from-a-sql-based-source"></a>Uso compartido desde un origen basado en SQL
Azure Data Share admite el uso compartido de tablas y vistas desde Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse). También admite el uso compartido de tablas desde el grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo). Actualmente, no se admite el uso compartido desde el grupo de SQL sin servidor de Azure Synapse Analytics (área de trabajo). 

El consumidor de datos puede optar por aceptar los datos en Azure Data Lake Storage Gen2 o Azure Blob Storage como un archivo CSV o Parquet. También pueden aceptar los datos como tablas en Azure SQL Database y Azure Synapse Analytics.

Cuando los consumidores aceptan los datos en Azure Data Lake Storage Gen2 o Azure Blob Storage, las instantáneas completas sobrescriben el contenido del archivo de destino si este ya existe. Cuando los datos se reciben en una tabla y la tabla de destino todavía no existe, Azure Data Share crea una tabla SQL con el esquema de origen. Si ya existe una tabla de destino con el mismo nombre, se quitará y se sobrescribirá con la instantánea completa más reciente. Actualmente no se admiten las instantáneas incrementales.

Para obtener más información, consulte [Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics](how-to-share-from-sql.md).

## <a name="share-from-data-explorer"></a>Uso compartido desde Data Explorer
Azure Data Share permite compartir bases de datos en contexto desde los clústeres de Azure Data Explorer. Un proveedor de datos puede compartir en el nivel de base de datos o de clúster. 

Cuando los datos se comparten en el nivel de base de datos, los consumidores de datos solo pueden acceder a las bases de datos que comparte el proveedor de datos. Si un proveedor comparte datos en el nivel de clúster, los consumidores de datos pueden acceder a todas las bases de datos del clúster del proveedor, incluidas las bases de datos futuras que cree el proveedor.

Para acceder a las bases de datos compartidas, los consumidores de datos necesitan su propio clúster de Azure Data Explorer. Este clúster debe estar en el mismo centro de datos de Azure que el clúster de Azure Data Explorer del proveedor de datos. 

Cuando se establece la relación de uso compartido, Azure Data Share crea un vínculo simbólico entre los clústeres del consumidor y el proveedor. Los datos que se ingieren en el clúster de origen mediante el modo por lotes aparecen en el clúster de destino en cuestión de minutos.

Para obtener más información, consulte el tema sobre [uso compartido y recepción de datos de Azure Data Explorer](/azure/data-explorer/data-share). 

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo empezar a compartir datos, vaya al tutorial [Uso compartido de datos](share-your-data.md).
