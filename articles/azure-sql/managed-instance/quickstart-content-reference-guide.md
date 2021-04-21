---
title: Referencia de contenido para los primeros pasos
titleSuffix: Azure SQL Managed Instance
description: 'Una referencia de contenido que sirve de ayuda para dar los primeros pasos con Instancia administrada de Azure SQL. '
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: quickstart
author: davidtrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 07/11/2019
ms.openlocfilehash: ec767e5f3a88c52f9686eec8f7e458ab517ee35f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784342"
---
# <a name="getting-started-with-azure-sql-managed-instance"></a>Introducción a Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Instancia administrada de Azure SQL](sql-managed-instance-paas-overview.md) crea una base de datos que tiene casi un 100 % de compatibilidad con el motor de base de datos más reciente de SQL Server (Enterprise Edition), el cual proporciona una implementación nativa de la [red virtual (VNet)](../../virtual-network/virtual-networks-overview.md), lo que permite solucionar problemas de seguridad habituales, y un [modelo de negocio](https://azure.microsoft.com/pricing/details/sql-database/) favorable para los clientes existentes de SQL Server.

En este artículo, encontrará referencias a contenido que le enseñarán a crear y configurar rápidamente una instancia administrada de SQL y a migrar las bases de datos.

## <a name="quickstart-overview"></a>Introducción al artículo de inicio rápido

Los siguientes inicios rápidos permiten crear rápidamente una instancia administrada de SQL, configurar una máquina virtual o una conexión VPN de punto a sitio para una aplicación cliente y restaurar una base de datos en la nueva instancia administrada de SQL mediante un archivo `.bak`.

### <a name="configure-environment"></a>Configuración del entorno

Como primer paso, tendría que crear su primera instancia administrada de SQL con el entorno de red en el que se va a colocar, y habilitar la conexión desde el equipo o máquina virtual en el que se ejecutan las consultas a Instancia administrada de SQL. Puede usar las guías a continuación:

- [Creación de una instancia administrada de SQL mediante Azure Portal](instance-create-quickstart.md). En Azure Portal puede configurar los parámetros necesarios (nombre de usuario y contraseña, número de núcleos y cantidad de almacenamiento máxima) y crear automáticamente el entorno de red de Azure sin necesidad de conocer los detalles de red ni los requisitos de infraestructura. Solo debe asegurarse de que tiene un [tipo de suscripción](resource-limits.md#supported-subscription-types) que tenga permiso para crear una instancia administrada de SQL. Si tiene su propia red y desea utilizarla, o bien desea personalizar la red, consulte el artículo en el que se indica cómo [configurar una red virtual para Instancia administrada de Azure SQL](vnet-existing-add-subnet.md) o cómo [crear una red virtual para Instancia administrada de Azure SQL](virtual-network-subnet-create-arm-template.md).
- Se crea una instancia administrada de SQL en su propia red virtual sin ningún punto de conexión público. Para acceder a la aplicación cliente, puede **crear una máquina virtual en la misma red virtual (en una subred diferente)** o **crear una conexión VPN de punto a sitio a la red virtual desde el equipo cliente** mediante uno de estos inicios rápidos:
  - Habilite el [punto de conexión público](public-endpoint-configure.md) en la instancia administrada de SQL para acceder a los datos directamente desde su entorno.
  - Cree una [máquina virtual de Azure en la red virtual de la instancia administrada de SQL](connect-vm-instance-configure.md) para obtener conectividad con la aplicación cliente, incluido SQL Server Management Studio.
  - Configure una [conexión de punto a sitio con la instancia administrada de SQL](point-to-site-p2s-configure.md) desde el equipo cliente en el que tiene SQL Server Management Studio y otras aplicaciones de conectividad de cliente. Esta es otra opción para conectarse a la instancia administrada de SQL y a su red virtual.

  > [!NOTE]
  > - También puede usar ExpressRoute o una conexión de sitio a sitio desde la red local, pero estos métodos están fuera del ámbito de estos inicios rápidos.
  > - Si cambia el período de retención de 0 (retención ilimitada) a cualquier otro valor, tenga en cuenta que la retención solo se aplicará a los registros escritos una vez cambiado el valor de retención (los registros escritos durante el período en el que la retención se estableció en ilimitada se conservan, incluso después de habilitarse la retención).

Como alternativa a la creación manual de una instancia administrada de SQL, puede usar [PowerShell](scripts/create-configure-managed-instance-powershell.md), [PowerShell con una plantilla de Resource Manager](./create-template-quickstart.md) o la [CLI de Azure](/cli/azure/sql/mi#az_sql_mi_create) para crear scripts y automatizar este proceso.

### <a name="migrate-your-databases"></a>Migración de las bases de datos

Después de crear una instancia administrada de SQL y configurar el acceso, es posible empezar a migrar las bases de datos de SQL Server. La migración puede generar un error si hay características no admitidas en la base de datos de origen que desee migrar. Para evitar errores y comprobar la compatibilidad, puede usar [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) para analizar las bases de datos en SQL Server y detectar cualquier problema que pudiera bloquear la migración a una instancia administrada de SQL, como la existencia de secuencias de [FileStream](/sql/relational-databases/blob/filestream-sql-server) o varios archivos de registro. Si resuelve estos problemas, las bases de datos estarán preparadas para su migración a Instancia administrada de SQL. El [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview) es otra herramienta útil para registrar una carga de trabajo en SQL Server y reproducirla en una instancia administrada de SQL, con el fin de que pueda determinar si va a haber problemas de rendimiento si se realiza la migración a una instancia administrada de SQL.

Una vez que esté seguro de que puede migrar la base de datos a una instancia administrada de SQL, puede usar las funcionalidades nativas de restauración de SQL Server para restaurar una base de datos en una instancia administrada de SQL desde un archivo `.bak`. Este método se puede usar para migrar bases de datos desde el motor de base de datos de SQL Server instalado localmente o Azure Virtual Machines. Para ver un inicio rápido, consulte el artículo sobre la [restauración desde una copia de seguridad en una instancia administrada de SQL](restore-sample-database-quickstart.md). En este inicio rápido, puede restaurar a partir de un archivo `.bak` almacenado en Azure Blob Storage mediante el comando `RESTORE` de Transact-SQL.

> [!TIP]
> Para usar el comando `BACKUP` de Transact-SQL para crear una copia de seguridad de la base de datos en Azure Blob Storage, consulte [Copia de seguridad en URL de SQL Server](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

Estos inicios rápidos permiten crear, configurar y restaurar una copia de seguridad de la base de datos en una instancia administrada de SQL. En algunos escenarios, puede que tenga que personalizar o automatizar la implementación de Instancia administrada de SQL y el entorno de red necesario. Estos escenarios se describen a continuación.

## <a name="customize-network-environment"></a>Personalización del entorno de red

Aunque la red virtual o subred se pueden configurar automáticamente cuando la instancia se [crea mediante Azure Portal](instance-create-quickstart.md), puede que sea conveniente crearla antes de empezar a crear instancias en Instancia administrada de SQL, ya que puede configurar los parámetros de la red virtual y la subred. La manera más fácil de crear y configurar el entorno de red es usar la plantilla de [implementación de recursos de Azure](virtual-network-subnet-create-arm-template.md) que crea y configura la red y la subred donde se colocará la instancia. Basta con presionar el botón de implementación de Azure Resource Manager y rellenar el formulario con los parámetros.

Como alternativa, puede usar este [script de PowerShell](https://www.powershellmagazine.com/2018/07/23/configuring-azure-environment-to-set-up-azure-sql-database-managed-instance-preview/) para automatizar la creación de la red.

Si ya tiene una red virtual y una subred donde le gustaría implementar la instancia administrada de SQL, deberá asegurarse de que ambas satisfacen los [requisitos de red](connectivity-architecture-overview.md#network-requirements). Use este [script de PowerShell para verificar que la subred está correctamente configurada](vnet-existing-add-subnet.md). Este script valida la red y notifica cualquier problema, le indica lo que se debe cambiar y, a continuación, le ofrece la posibilidad de realizar los cambios necesarios en la red virtual o en la subred. Ejecute este script si no desea configurar manualmente la red virtual o la subred. También puede ejecutarlo después de cualquier reconfiguración importante de la infraestructura de la red. Si quiere crear y configurar su propia red, lea acerca de la [arquitectura de conectividad](connectivity-architecture-overview.md) y esta [guía definitiva para crear y configurar un entorno de Instancia administrada de SQL](https://medium.com/azure-sqldb-managed-instance/the-ultimate-guide-for-creating-and-configuring-azure-sql-managed-instance-environment-91ff58c0be01).

## <a name="migrate-to-a-sql-managed-instance"></a>Migración a una instancia administrada de SQL

Los inicios rápidos mencionados permiten configurar rápidamente una instancia administrada de SQL y mover las bases de datos mediante la funcionalidad `RESTORE` nativa. Este es un buen punto de partida si quiere completar pruebas de concepto y comprobar que la solución puede funcionar en la instancia administrada.

Sin embargo, para migrar la base de datos de producción o incluso las bases de datos de desarrollo y pruebas que quiera usar en alguna prueba de rendimiento, deberá considerar la posibilidad de usar algunas técnicas adicionales, como:

- Pruebas de rendimiento: debe medir las métricas de rendimiento de línea de base en la instancia de SQL Server de origen y compararlas con las de rendimiento de la instancia administrada de destino a la que ha migrado la base de datos. Más información sobre los [procedimientos recomendados de comparación de rendimiento](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/The-best-practices-for-performance-comparison-between-Azure-SQL/ba-p/683210).
- Migración en línea: con la funcionalidad `RESTORE` nativa que se describen en este artículo, tendrá que esperar a que las bases de datos se restauren (y se copien en Azure Blob Storage si aún no están almacenadas allí). Esto produce un tiempo de inactividad de la aplicación especialmente en el caso de bases de datos grandes. Para mover la base de datos de producción, use [Data Migration Service (DMS)](../../dms/tutorial-sql-server-to-managed-instance.md?toc=%2fazure%2fsql-database%2ftoc.json) para migrar la base de datos con el menor tiempo de inactividad posible. Para ello, DMS inserta los cambios realizados en la base de datos de origen y en la base de datos de la instancia administrada de SQL que se está restaurando. De este modo, puede cambiar rápidamente la aplicación de la base de datos de origen a la de destino con un tiempo de inactividad mínimo.

Más información acerca del [proceso de migración recomendado](migrate-to-instance-from-sql-server.md).

## <a name="next-steps"></a>Pasos siguientes

- Busque una [lista de alto nivel de las características compatibles de Instancia administrada de SQL aquí](../database/features-comparison.md) y [detalles y problemas conocidos aquí](transact-sql-tsql-differences-sql-server.md).
- Información acerca de las [características técnicas de Instancia administrada de SQL](resource-limits.md#service-tier-characteristics).
- Para conocer procedimientos más avanzados, consulte el artículo en el que se indica [cómo usar una instancia administrada de SQL](how-to-content-reference-guide.md).
- [Identificación de la SKU de instancia administrada de Azure SQL adecuada para una base de datos local](/sql/dma/dma-sku-recommend-sql-db/).