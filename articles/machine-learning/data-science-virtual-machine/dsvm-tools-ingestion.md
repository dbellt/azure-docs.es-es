---
title: Herramientas de ingesta de datos
titleSuffix: Azure Data Science Virtual Machine
description: Obtenga información sobre las utilidades y herramientas de ingesta de datos instaladas previamente en Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: data-science-vm
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: 77968c7a611cdfade3d2f1eb2cb6e9b0a3768728
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110070955"
---
# <a name="data-science-virtual-machine-data-ingestion-tools"></a>Herramientas de ingesta de datos de Data Science Virtual Machine

Como uno de los primeros pasos técnicos en un proyecto de inteligencia artificial o ciencia de datos, debe identificar los conjuntos de datos que se van a usar y colocarlos en el entorno de análisis. Data Science Virtual Machine (DSVM) proporciona herramientas y bibliotecas para traer datos de distintos orígenes a un almacenamiento de datos de análisis local en DSVM o en una plataforma de datos local o en la nube.

Estas son algunas herramientas de movimiento de datos disponibles en DSVM.

## <a name="azure-cli"></a>Azure CLI

| Category | Value |
|--|--|
| ¿Qué es? | Una herramienta de administración para Azure. También contiene verbos de comando para mover datos de plataformas de datos de Azure como Azure Blob Storage o Azure Data Lake Store. |
| Versiones de DSVM compatibles | Windows, Linux |
| Usos típicos | Importación de datos a Azure Storage o Azure Data Lake Store o exportación. |
| ¿Cómo se usa o ejecuta? | Abra un símbolo del sistema y escriba `az` para obtener ayuda. |
| Vínculos a ejemplos | [Uso de la CLI de Azure](/cli/azure) |


## <a name="azcopy"></a>AzCopy

| Category | Value |
|--|--|
| ¿Qué es? | Una herramienta para copiar datos a y desde archivos locales, Azure Blob Storage, archivos y tablas. |
| Versiones de DSVM compatibles | Windows |
| Usos típicos | Copia de archivos a Azure Blob Storage y copia de blobs entre cuentas. |
| ¿Cómo se usa o ejecuta? | Abra un símbolo del sistema y escriba `azcopy` para obtener ayuda. |
| Vínculos a ejemplos | [AzCopy en Windows](../../storage/common/storage-use-azcopy-v10.md) |


## <a name="azure-cosmos-db-data-migration-tool"></a>Herramienta de migración de datos de Azure Cosmos DB

|--|--|
| ------------- | ------------- |
| ¿Qué es? | Herramienta para importar datos de varios orígenes en Azure Cosmos DB; una base de datos NoSQL en la nube. Estos orígenes incluyen: archivos JSON, archivos CSV, SQL, MongoDB, Azure Table Storage, Amazon DynamoDB y colecciones de SQL API de Azure Cosmos DB. |
| Versiones de DSVM compatibles | Windows |
| Usos típicos | Importación de archivos de una máquina virtual a CosmosDB, importación de datos desde Azure Table Storage a CosmosDB e importación de datos desde una base de datos de SQL Server a CosmosDB. |
| ¿Cómo se usa o ejecuta? | Para usar la versión de línea de comandos, abra un símbolo del sistema y escriba `dt`. Para usar la herramienta de la interfaz gráfica de usuario, abra un símbolo del sistema y escriba `dtui`. |
| Vínculos a ejemplos | [Importación de datos en CosmosDB](../../cosmos-db/import-data.md) |

## <a name="azure-storage-explorer"></a>Explorador de Azure Storage

| Category | Value |
|--|--|
| ¿Qué es? | Interfaz gráfica de usuario para interactuar con los archivos almacenados en la nube de Azure. |
| Versiones de DSVM compatibles | Windows |
| Usos típicos | Importación y exportación de datos en DSVM |
| ¿Cómo se usa o ejecuta? | Busque "Explorador de Azure Storage" en el menú Inicio. |
| Vínculos a ejemplos | [Explorador de Azure Storage](vm-do-ten-things.md#access-azure-data-and-analytics-services) |

## <a name="bcp"></a>BCP

| Category | Value |
|--|--|
| ¿Qué es? | Herramienta de SQL Server para copiar datos entre SQL Server y un archivo de datos. |
| Versiones de DSVM compatibles | Windows |
| Usos típicos | Importación de un archivo CSV a una tabla de SQL Server y exportación de una tabla de SQL Server a un archivo. |
| ¿Cómo se usa o ejecuta? | Abra un símbolo del sistema y escriba `bcp` para obtener ayuda. |
| Vínculos a ejemplos | [Utilidad bcp](/sql/tools/bcp-utility) |

## <a name="blobfuse"></a>blobfuse

| Category | Value |
|--|--|
| ¿Qué es? | Una herramienta para montar un contenedor de Azure Blob Storage en el sistema de archivos de Linux. |
| Versiones de DSVM compatibles | Linux |
| Usos típicos | Leer y escribir en los blobs de un contenedor. |
| ¿Cómo se usa y cómo se ejecuta? | Ejecute _blobfuse_ en un terminal. |
| Vínculos a ejemplos | [blobfuse en GitHub](https://github.com/Azure/azure-storage-fuse) |
