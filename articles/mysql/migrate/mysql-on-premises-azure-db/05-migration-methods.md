---
title: Métodos de migración de la guía para la migración de una instancia local de MySQL a Azure Database for MySQL
description: Para mover los datos del origen al destino y llevar a cabo la migración, será necesario el uso de herramientas o características de MySQL.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: be44dd31b5a038414a10a7dac47e53a12573449e
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082966"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-migration-methods"></a>Métodos de migración de la guía para la migración de una instancia local de MySQL a Azure Database for MySQL

## <a name="prerequisites"></a>Prerrequisitos

[Planeamiento](04-planning.md)

## <a name="overview"></a>Información general

Para mover los datos del origen al destino y llevar a cabo la migración, es necesario el uso de herramientas o características de MySQL.

Es importante completar todas las fases de evaluación y planeamiento antes de iniciar las siguientes fases. La ruta de migración y la herramientas seleccionadas dependen de las decisiones que se tomen y los datos que se recopilen.

En esta sección, analizaremos las siguientes herramientas que se usan habitualmente:

  - MySQL Workbench

  - mysqldump

  - mydumper y myloader

  - Replicación de datos de entrada (binlog)

### <a name="mysql-workbench"></a>MySQL Workbench

[MySQL Workbench](https://www.mysql.com/products/workbench/) proporciona una experiencia de GUI enriquecida que permite a los desarrolladores y administradores diseñar, desarrollar y administrar las instancias de MySQL.

La versión más reciente de MySQL Workbench proporciona una sofisticada [funcionalidad de migración de objetos](https://www.mysql.com/products/workbench/migrate/) para mover una base de datos de un origen a un destino.

#### <a name="data-import-and-export"></a>Importación y exportación de datos

MySQL Workbench proporciona una interfaz de usuario basada en un asistente que permite exportar e importar total o parcialmente tablas y objetos de base de datos. Para ver un ejemplo de cómo usar MySQL Workbench, visite [Migración de la base de datos de MySQL mediante importación y exportación](../../concepts-migrate-import-export.md).

### <a name="dump-and-restore-mysqldump"></a>Volcado y restauración (mysqldump)

Normalmente, se proporciona `mysqldump` como parte de la instalación de MySQL. Es una [utilidad cliente](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) que se puede ejecutar para crear copias de seguridad lógicas que equivalen a un conjunto de instrucciones SQL que se pueden reproducir para restaurar la base de datos a un momento dado. `mysqldump` no se ha diseñado como una solución rápida o escalable para realizar una copia de seguridad o migrar grandes cantidades de datos. La ejecución de un gran conjunto de instrucciones SQL de inserción puede tener un rendimiento deficiente debido al número de operaciones de E/S de disco necesarias para actualizar los índices. Sin embargo, cuando se combina con otras herramientas que requieren el esquema original, `mysqldump` es una excelente herramienta para generar los esquemas de la base de datos y de las tablas. Los esquemas pueden crear el entorno de la zona de aterrizaje de destino.

La utilidad `mysqldump` proporciona características muy prácticas para la fase de migración de los datos. Antes de ejecutar la utilidad, debe considerarse el rendimiento. Vea [Consideraciones sobre el rendimiento](../../concepts-migrate-dump-restore.md#performance-considerations).

### <a name="mydumper-and-myloader"></a>mydumper y myloader

Para los entornos con bases de datos de gran tamaño que requieren una migración rápida, deben usarse [mydumper y myloader](https://github.com/maxbube/mydumper). Estas herramientas están escritas en C++ y usan técnicas multiproceso para enviar los datos con la mayor rapidez posible a la instancia de MySQL de destino. `mydumper` y `myloader` aprovechan el paralelismo y pueden acelerar la migración unas 10 veces o más.

Las versiones binarias de las herramientas que están disponibles para la descarga pública se han compilado para Linux. Para ejecutar estas herramientas en Windows, es necesario recompilar los proyectos de código abierto. Para la mayoría de los usuarios, la compilación de código fuente y la creación de versiones no es una tarea sencilla.

### <a name="data-in-replication-binlog"></a>Replicación de datos de entrada (binlog)

Al Igual que otros sistemas de administración de bases de datos, MySQL proporciona una característica de replicación de registros denominada [replicación binlog](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html). La característica replicación `binlog` facilita la migración de los datos y la creación de réplicas de lectura.

Use la replicación binlog para [migrar datos a Azure Database for MySQL](../../concepts-data-in-replication.md) en un escenario en línea. La replicación de datos ayuda a reducir el tiempo de inactividad necesario para realizar los últimos cambios en los datos de destino.

Para usar la característica replicación `binlog`, hay algunos [requisitos](../../howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication) de configuración:

  - Se recomienda que el servidor maestro utilice el motor InnoDB de MySQL. Si usa un motor de almacenamiento distinto de InnoDB, debe migrar las tablas a InnoDB.

  - Los usuarios de la migración deben tener permisos para configurar el registro binario y crear nuevos usuarios en el servidor maestro.

  - Si el servidor maestro tiene SSL habilitado, asegúrese de que el certificado de entidad de certificación SSL proporcionado para el dominio se haya incluido en el procedimiento almacenado mysql.az\_replication\_change\_master. Consulte los siguientes [ejemplos](../../howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication) y el parámetro master\_ssl\_ca.

  - Asegúrese de que la dirección IP del servidor maestro se haya agregado a las reglas de firewall del servidor de réplica de Azure Database for MySQL. Actualice las reglas de firewall mediante Azure Portal o la CLI de Azure.

  - Asegúrese de que el equipo que hospeda el servidor maestro permite el tráfico entrante y saliente en el puerto 3306.

  - Asegúrese de que el servidor maestro tenga una dirección IP accesible (pública o privada) desde el origen hasta los destinos.

Para llevar a cabo una migración mediante replicación, consulte [Configuración de la replicación de datos de entrada en Azure Database for MySQL](../../howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication) para obtener más información.

El método de replicación `binlog` tiene requisitos elevados de CPU y almacenamiento adicional. Los usuarios de la migración deben probar la carga ubicada en el sistema de origen durante las migraciones en línea y determinar si es aceptable.

### <a name="azure-database-migration-service-dms"></a>Azure Database Migration Service (DMS)

[Azure Database Migration Services (DMS)](https://azure.microsoft.com/services/database-migration/) es una herramienta de Azure basada en la nube que permite a los administradores realizar un seguimiento de las distintas configuraciones de migración y reutilizarlas si es necesario. DMS funciona mediante la creación de proyectos de migración con configuraciones que apuntan a varios orígenes y destinos. Admite [migraciones sin conexión](../../../dms/tutorial-mysql-azure-mysql-offline-portal.md). También admite cargas de trabajo de datos del entorno local y basadas en la nube, como MySQL de Amazon Relational Database Service (RDS).

Aunque el servicio DMS es una herramienta en línea, se basa en la característica de replicación `binlog` de MySQL para realizar las tareas. Actualmente, DMS automatiza parcialmente el proceso de migración sin conexión. DMS requiere que se genere y aplique el esquema correspondiente en la instancia de destino de Azure Database for MySQL. Los esquemas se pueden exportar con la utilidad cliente `mysqldump`.

### <a name="fastestminimum-downtime-migration"></a>Migración más rápida o con un tiempo de inactividad mínimo

Hay numerosas opciones para migrar los datos. Decidir qué opción usar es una tarea del equipo de migración, así como decidir la cantidad de tiempo de inactividad que los propietarios de la base de datos y las aplicaciones están dispuestos a aceptar. Algunas herramientas admiten estrategias de migración de datos multiproceso en paralelo, mientras que otras se han diseñado para realizar migraciones sencillas, únicamente de datos de tablas.

La opción más rápida y completa es usar la replicación `binlog` (ya sea directamente con MySQL o a través de DMS), pero supone el costo de agregar claves principales.

### <a name="decision-table"></a>Tabla de decisión

Hay muchas opciones que WWI puede usar para migrar sus cargas de trabajo de MySQL. Hemos proporcionado una tabla con las posibles opciones y las ventajas e inconvenientes de cada una de ellas:

| Objetivos | Descripción | Herramienta | Prerrequisitos | Ventajas | Inconvenientes |
|-----------|-------------|------|---------------|------------|---------------|
| **Migración más rápida posible** | Estrategia en paralelo | mydumper y myloader | Linux | Muy paralelizada | Limitación en el destino |
| **Migración en línea** | Mantiene el origen activo el máximo tiempo posible | binlog | Ninguno | Fluidez | Almacenamiento y procesamiento extra  |
| **Migración sin conexión** | Mantiene el origen activo el máximo tiempo posible | Database Migration Service (DMS) | Ninguno | Proceso repetible  | Limitada solo a datos; admite todas las versiones de MySQL |
| **Migración sin conexión muy personalizada** | Exportación selectiva de objetos | mysqldump | Ninguno | Muy personalizable | Manual |
| **Migración sin conexión semiautomatizada** | Exportación e importación basadas en la interfaz de usuario | MySQL Workbench | Descarga e instalación | Semiautomatizada | Solo se admiten los conjuntos de modificadores más comunes |

### <a name="wwi-scenario"></a>Escenario de WWI

WWI eligió su base de datos de conferencias como la primera carga de trabajo para migrar. El motivo para esta elección fue que tenía menos riesgo y el mayor tiempo de inactividad disponible debido al hueco en la programación anual de conferencias. Además, tras la evaluación del equipo de migración, se determinó intentar una migración sin conexión con MySQL Workbench.

### <a name="migration-methods-checklist"></a>Lista de comprobación de los métodos de migración

  - Asegurarse de que se selecciona el método correcto en función de los entornos de origen y destino.

  - Asegurarse de que el método puede cumplir los requisitos empresariales.

  - Comprobar siempre si la carga de trabajo de datos admite el método en cuestión.  


> [!div class="nextstepaction"]
> [Test Plans](./06-test-plans.md)