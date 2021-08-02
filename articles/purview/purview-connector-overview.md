---
title: Introducción a los conectores de Purview
description: En este artículo se describen los diferentes almacenes de datos y funcionalidades admitidos en Purview
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 05/17/2021
ms.openlocfilehash: 09ed5297bb6432640d6cce7674c2246088823d37
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111750612"
---
# <a name="supported-data-stores"></a>Almacenes de datos compatibles

Purview admite los siguientes almacenes de datos. Haga clic en cada almacén de datos para obtener información detallada sobre las funcionalidades admitidas y las configuraciones correspondientes.

## <a name="purview-data-sources"></a>Orígenes de datos de Purview

|**Categoría**|  **Almacén de datos**  |**Extracción de metadatos**|**Examen completo**|**Examen incremental**|**Examen con ámbito**|**Clasificación**|**Lineage**|
|---|---|---|---|---|---|---|---|
| Azure | [Azure Blob Storage](register-scan-azure-blob-storage-source.md)| Sí| Sí| Sí| Sí| Sí| Sí|
||[Azure Cosmos DB](register-scan-azure-cosmos-database.md)|Sí| Sí| Sí| Sí| Sí| Sí|
||[Azure Data Explorer](register-scan-azure-data-explorer.md)|Sí| Sí| Sí| Sí| Sí| Sí|
||[Azure Data Lake Storage Gen1](register-scan-adls-gen1.md)|Sí| Sí| Sí| Sí| Sí| Sí|
||[Azure Data Lake Storage Gen2](register-scan-adls-gen2.md)|Sí| Sí| Sí| Sí| Sí| Sí|
||[Azure SQL Database](register-scan-azure-sql-database.md)|Sí| Sí| No| Sí| Sí| Sí|
||[Instancia administrada de Azure SQL Database](register-scan-azure-sql-database-managed-instance.md)|Sí| Sí| No| Sí| Sí| Sí|
||[Grupo de SQL dedicado de Azure (antes denominado SQL DW)](register-scan-azure-synapse-analytics.md)|Sí| Sí| No| Sí| Sí| Sí|
||[Azure Synapse Analytics (área de trabajo)](register-scan-synapse-workspace.md)|Sí| Sí| No| Sí| Sí| Sí|
|Base de datos|[Base de datos de metastore de Hive](register-scan-oracle-source.md)|Sí| Sí| No| No| No| Sí|
||[Oracle DB](register-scan-oracle-source.md)|Sí| Sí| No| No| No| Sí|
||[SQL Server](register-scan-on-premises-sql-server.md)|Sí| Sí| No| Sí| Sí| Sí|
||[Teradata](register-scan-teradata-source.md)|Sí| Sí| No| No| No| Sí|
|Power BI|[Power BI](register-scan-power-bi-tenant.md)|Sí| Sí| No| No| No| Sí|
|Servicios y aplicaciones|[SAP ECC](register-scan-sapecc-source.md)|Sí| Sí| No| Sí| Sí| Sí|
||[SAP S4HANA](register-scan-saps4hana-source.md)|Sí| Sí| No| Sí| Sí| Sí|
|Nubes múltiples|[Amazon S3](register-scan-amazon-s3.md)|Sí| Sí| No| No| No| Sí|

## <a name="scan-regions"></a>Regiones de la exploración
A continuación se muestra una lista de todas las regiones de origen de datos de Azure (centro de datos) donde se ejecuta el explorador de Purview. Si el origen de datos de Azure se encuentra en una región que no está en esta lista, el explorador se ejecutará en la región de la instancia de Purview.
 
### <a name="purview-scanner-regions"></a>Regiones del explorador de Purview

- EastUs
- EastUs2 
- SouthCentralUS
- WestUs
- WestUs2
- SoutheastAsia
- Oeste de Europa
- Norte de Europa
- UkSouth
- AustraliaEast
- CanadaCentral
- BrazilSouth
- CentralIndia
- JapanEast
- SouthAfricaNorth
- FranceCentral
- KoreaCentral
- CentralUS
- NorthCentralUS
- EastAsia
- WestCentralUS
- AustraliaSoutheast

## <a name="next-steps"></a>Pasos siguientes

- [Registro y examen del origen de Azure Blob Storage](register-scan-azure-blob-storage-source.md)
