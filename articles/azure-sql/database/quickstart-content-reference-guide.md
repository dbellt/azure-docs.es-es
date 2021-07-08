---
title: Referencia de contenido del inicio rápido de una base de datos única
description: Busque una referencia del contenido de todos los inicios rápidos que le ayuden a empezar a trabajar rápidamente con Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: service-overview
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 07/29/2019
ms.openlocfilehash: e3bbba9245ea7aa998a1adecb60e05e9ff7d70c0
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690009"
---
# <a name="getting-started-with-single-databases-in-azure-sql-database"></a>Introducción a las bases de datos únicas en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Una base de datos única](../index.yml) es una base de datos de plataforma como servicio (PaaS) que está totalmente administrada (DbaaS) y que resulta el motor de almacenamiento ideal para las aplicaciones modernas creadas en la nube. En esta sección, aprenderá a crear y configurar rápidamente una base de datos única en Azure SQL Database.

## <a name="quickstart-overview"></a>Introducción al artículo de inicio rápido

En esta sección, verá información general de los artículos disponibles que le ayudarán a empezar a trabajar rápidamente con las bases de datos únicas. Los siguientes inicios rápidos le permiten crear rápidamente una base de datos única, configurar una regla de firewall de nivel de servidor y, después, importar una base de datos en la nueva base de datos única mediante un archivo `.bacpac`:

- [Creación de una base de datos única mediante Azure Portal](single-database-create-quickstart.md).
- Después de crear la base de datos, necesitaría [proteger la base de datos mediante la configuración de reglas de firewall](firewall-create-server-level-portal-quickstart.md).
- Si tiene una base de datos en SQL Server que desee migrar a Azure SQL Database, debe instalar [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595), que analizará las bases de datos de SQL Server y buscará cualquier problema que pudiera bloquear la migración. Si no se encuentra ninguno, puede exportar la base de datos como archivo `.bacpac` e [importarlo mediante Azure Portal o SqlPackage](database-import.md).


## <a name="automating-management-operations"></a>Automatización de las operaciones de administración

Puede usar PowerShell o la CLI de Azure para crear, configurar y escalar la base de datos.

- [Creación y configuración de una base de datos única con PowerShell](scripts/create-and-configure-database-powershell.md) o la [CLI de Azure](scripts/create-and-configure-database-cli.md)
- [Actualización de una base de datos única y escalado de recursos mediante PowerShell](scripts/monitor-and-scale-database-powershell.md) o la [CLI de Azure](scripts/monitor-and-scale-database-cli.md)

## <a name="migrating-to-a-single-database-with-minimal-downtime"></a>Migración a una base de datos única con tiempo de inactividad mínimo

Estos inicios rápidos le permiten crear o importar rápidamente la base de datos a Azure mediante un archivo `.bacpac`. Sin embargo, los archivos `.bacpac` y `.dacpac` están diseñados para mover rápidamente bases de datos entre las distintas versiones de SQL Server y dentro de Azure SQL, o para implementar la integración continua en la canalización de DevOps. No obstante, este método no está diseñado para la migración de las bases de datos de producción con un tiempo de inactividad mínimo, ya que tendría que dejar de agregar nuevos datos, esperar a que se completara la exportación de la base de datos de origen a un archivo `.bacpac` y, finalmente, esperar a que terminara la importación en Azure SQL Database. Todo este tiempo de espera genera un tiempo de inactividad de la aplicación, especialmente en el caso de grandes bases de datos. Para mover la base de datos de producción, probablemente necesite una manera mejor de realizar la migración que garantice un tiempo de inactividad mínimo. Para ello, use [Data Migration Service (DMS)](../../dms/tutorial-sql-server-to-azure-sql.md?toc=%2fazure%2fsql-database%2ftoc.json) para migrar la base de datos con el menor tiempo de inactividad posible. DMS lo consigue mediante la inserción de forma incremental de los cambios realizados en la base de datos de origen en la base de datos única que se restaura. De este modo, puede cambiar rápidamente la aplicación de la base de datos de origen a la de destino con tiempo de inactividad mínimo.

## <a name="hands-on-learning-modules"></a>Módulos de aprendizaje práctico

El siguiente módulo de Microsoft Learn le ayuda a aprender de forma gratuita sobre Azure SQL Database.

- [Aprovisionamiento de una base de datos en SQL Database para almacenar datos de aplicación](/learn/modules/provision-azure-sql-db/)
- [Desarrollo y configuración de una aplicación de ASP.NET que consulta una base de datos en Azure SQL Database](/learn/modules/develop-app-that-queries-azure-sql/)
- [Protección de la base de datos en Azure SQL Database](/learn/modules/secure-your-azure-sql-database/)

## <a name="next-steps"></a>Pasos siguientes

- Buscar una [lista detallada de las características admitidas en Azure SQL Database](features-comparison.md).
- Aprender a [proteger mejor la base de datos](secure-database-tutorial.md).
- Para conocer procedimientos más avanzados, consulte [Cómo usar una base de datos única en Azure SQL Database](how-to-content-reference-guide.md).
- Buscar más scripts de ejemplo escritos en [PowerShell](powershell-script-content-guide.md) y la [CLI de Azure](az-cli-script-samples-content-guide.md).
- Conozca más información sobre la [API de administración](single-database-manage.md) que puede usar para configurar las bases de datos.
- [Identificación de la SKU correcta de Azure SQL Database o de Instancia administrada de Azure SQL para una base de datos local](/sql/dma/dma-sku-recommend-sql-db/).