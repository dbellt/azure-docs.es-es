---
title: 'Grupos de SQL sin servidor: autoayuda'
description: Esta sección contiene información que puede ayudarle a solucionar los problemas que surjan con grupos de SQL sin servidor.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: ae9a1a374724465148f922efe486c4680e1e36ab
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144284"
---
# <a name="self-help-for-serverless-sql-pool"></a>Autoayuda para grupos de SQL sin servidor

Este artículo contiene información sobre cómo solucionar los problemas más frecuentes con grupos de SQL sin servidor en Azure Synapse Analytics.

## <a name="serverless-sql-pool-is-grayed-out-in-synapse-studio"></a>El grupo de SQL sin servidor está atenuado en Synapse Studio

Si Synapse Studio no puede establecer la conexión con el grupo de SQL sin servidor, este aparecerá atenuado o con el estado "Sin conexión". Normalmente, este problema se da cuando se produce uno de los siguientes casos:

1) La red impide la comunicación con el back-end de Azure Synapse. El caso más frecuente es que el puerto 1443 está bloqueado. Para que el grupo de SQL sin servidor funcione, desbloquee este puerto. Hay otros problemas que pueden impedir el funcionamiento de un grupo de SQL sin servidor funcione. [Para más información, visite la guía para la solución de problemas](../troubleshoot/troubleshoot-synapse-studio.md).
2) No tiene permisos para iniciar sesión en un grupo de SQL sin servidor. Para obtener acceso, uno de los administradores del área de trabajo de Azure Synapse debe agregarle al administrador de áreas de trabajo o al rol de administrador de SQL. [Visite guía completa sobre el control de acceso para más información](../security/synapse-workspace-access-control-overview.md).

## <a name="query-fails-because-file-cannot-be-opened"></a>Se produce un error en la consulta porque no se puede abrir el archivo

Si aparece en la consulta un error que indica que no se puede abrir el archivo porque no existe o porque lo está usando otro proceso, y está seguro de que el archivo existe y que no lo usa ningún otro proceso, significa que el grupo de SQL sin servidor no puede acceder al archivo. Este problema suele ocurrir porque su identidad de Azure Active Directory no tiene derechos de acceso al archivo. De forma predeterminada, el grupo de SQL sin servidor intenta acceder al archivo mediante su identidad de Azure Active Directory. Para resolver este problema, debe tener los derechos adecuados para acceder al archivo. La manera más fácil es concederse el rol "Colaborador de datos de blobs de almacenamiento" en la cuenta de almacenamiento que está intentando consultar. 
- [Visite la guía completa sobre el control de acceso de Azure Active Directory para el almacenamiento para más información](../../storage/common/storage-auth-aad-rbac-portal.md). 
- [Visite Control del acceso a la cuenta de almacenamiento del grupo de SQL sin servidor en Azure Synapse Analytics](develop-storage-files-storage-access-control.md)

## <a name="query-fails-because-it-cannot-be-executed-due-to-current-resource-constraints"></a>Se produce un error en la consulta porque no se puede ejecutar debido a las restricciones de recursos actuales 

Si la consulta no se puede ejecutar y aparece un mensaje de error en el que se indica que la consulta no se puede ejecutar debido a las actuales restricciones de los recursos, significa que el grupo de SQL sin servidor no puede ejecutarla en este momento a causa de las restricciones de recursos: 

- Asegúrese de que se usan tipos de datos de tamaños razonables. Además, especifique el esquema para los archivos Parquet para las columnas de cadenas, ya que serán VARCHAR(8000) de forma predeterminada. 

- Si la consulta se dirige a archivos CSV, considere la posibilidad de [crear estadísticas](develop-tables-statistics.md#statistics-in-serverless-sql-pool). 

- Para optimizar la consulta, visite [los procedimientos recomendados para mejorar el rendimiento en los grupos de SQL sin servidor](./best-practices-serverless-sql-pool.md).  

## <a name="create-statement-is-not-supported-in-master-database"></a>CREATE 'STATEMENT' no se admite en la base de datos maestra

Si la consulta no se puede ejecutar y aparece el mensaje de error:

> "Failed to execute query. Error: CREATE EXTERNAL TABLE/DATA SOURCE/DATABASE SCOPED CREDENTIAL/FILE FORMAT is not supported in master database" (No se pudo ejecutar la consulta. Error: CREATE EXTERNAL TABLE/DATA SOURCE/DATABASE SCOPED CREDENTIAL/FILE FORMAT no se admiten en la base de datos maestra). 

Esto significa que la base de datos maestra del grupo de SQL sin servidor no admite la creación de:
  - Tablas externas
  - Orígenes de datos externos
  - Credenciales cuyo ámbito es la base de datos
  - Formatos de archivo externos

Solución:

  1. Cree una base de datos de usuarios:

```sql
CREATE DATABASE <DATABASE_NAME>
```

  2. Ejecute la instrucción de creación en el contexto de <NOMBRE_BASE DE DATOS>, que produjo un error antes en la base de datos maestra. 
  
  Ejemplo de creación de formato de archivo externo:
    
```sql
USE <DATABASE_NAME>
CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat] 
WITH ( FORMAT_TYPE = PARQUET)
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los artículos siguientes para aprender a usar un grupo de SQL sin servidor:

- [Consulta de archivos CSV](query-single-csv-file.md)

- [Consulta de carpetas y varios archivos .csv](query-folders-multiple-csv-files.md)

- [Consulta de archivos específicos](query-specific-files.md)

- [Consulta de archivos Parquet](query-parquet-files.md)

- [Consulta de tipos anidados de Parquet](query-parquet-nested-types.md)

- [Consulta de archivos JSON](query-json-files.md)

- [Creación y uso de vistas](create-use-views.md)

- [Creación y uso de tablas externas](create-use-external-tables.md)

- [Almacenamiento de los resultados de las consultas en Storage](create-external-table-as-select.md)