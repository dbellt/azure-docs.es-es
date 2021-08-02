---
title: 'Guía para la migración de MySQL en el entorno local a Azure Database for MySQL: Evaluación'
description: Antes de pasar directamente a la migración de una carga de trabajo de MySQL, existe un cierto número de tareas que se deben realizar con la debida diligencia.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: 9d7dc8626e86e7ab93c7a6e76cc426c3904147c2
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082990"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-assessment"></a>Guía para la migración de MySQL en el entorno local a Azure Database for MySQL: Evaluación

## <a name="prerequisites"></a>Prerrequisitos

[Caso de uso representativo](02-representative-use-case.md)

## <a name="overview"></a>Información general

Antes de pasar directamente a la migración de una carga de trabajo de MySQL, existe un cierto número de tareas que se deben realizar con la debida diligencia. Entre estas se incluye el análisis de los datos, el entorno de hospedaje y las cargas de trabajo de la aplicación para validar que la zona de aterrizaje de Azure esté configurada correctamente y preparada para hospedar las cargas de trabajo que se migrarán próximamente.

## <a name="limitations"></a>Limitaciones

Azure Database for MySQL es una versión totalmente compatible de MySQL Community Edition que se ejecuta como plataforma como servicio. Sin embargo, hay [algunas limitaciones](../../concepts-limits.md) con las que familiarizarse al realizar una evaluación inicial.

Entre las más importantes se incluyen:

  - El motor de almacenamiento es compatible solo con `InnoDB` y `Memory`

  - Compatibilidad limitada con `Privilege` (`DBA`, `SUPER`, `DEFINER`)

  - Instrucciones de manipulación de datos deshabilitadas (`SELECT ... INTO OUTFILE`, `LOAD DATA INFILE`)

  - Migración de base de datos significativa automática (5.6 a 5.7, 5.7 a 8.0)

  - Cuando se usan las [funciones definidas por el usuario de MySQL Server](https://dev.mysql.com/doc/refman/5.7/en/server-udfs.html), la única opción de hospedaje viable son las máquinas virtuales hospedadas en Azure, ya que no hay ninguna capacidad para cargar el componente `so` o `dll` en Azure Database for MySQL.

Muchos de los demás elementos son aspectos operativos con los que los administradores deben familiarizarse como parte de la administración del ciclo de vida de la carga de trabajo de los datos operativos. En esta guía se exploran muchos de estos aspectos operativos en la sección Administración posterior a la migración.

## <a name="mysql-versions"></a>Versiones de MySQL

MySQL tiene una rica historia que se remonta a 1995. Desde entonces, ha evolucionado hasta convertirse en un sistema de administración de bases de datos ampliamente utilizado. Azure Database for MySQL empezó siendo compatible con la versión 5.6 de MySQL y ha continuado hasta la versión 5.7 y, más recientemente, la versión 8.0. Para obtener la versión más reciente de Azure Database for MySQL, consulte [Versiones admitidas de servidores de Azure Database for MySQL](../../concepts-supported-versions.md). En la sección Administración posterior a la migración, se repasa cómo se aplican las actualizaciones (por ejemplo, de la versión 5.7.20 a la 5.7.21) en las instancias de MySQL en Azure.

> [!NOTE]
> El salto de la versión 5.x a la 8.0 se debe en gran medida a la adquisición de MySQL por parte de Oracle. Para más información sobre la historia de MySQL, vaya a la [página wiki sobre MySQL](https://en.wikipedia.org/wiki/MySQL).

Es fundamental conocer la versión de MySQL de origen. Las aplicaciones que usan el sistema pueden usar objetos de base de datos y características que son específicas de esa versión. La migración de una base de datos a una versión inferior podría provocar problemas de compatibilidad y la pérdida de funcionalidad. También se recomienda que los datos y la instancia de la aplicación se prueben exhaustivamente antes de migrar a una versión más reciente, ya que las nuevas características podrían interrumpir la aplicación.

Ejemplos que pueden influir en la ruta de migración y la versión:

  - 5.6: columna TIMESTAMP para un almacenamiento correcto de milisegundos y búsqueda de texto completo

  - 5.7: Compatibilidad con el tipo de datos JSON nativo

  - 8.0: Compatibilidad con el almacén de documentos NoSQL y con funciones de tabla JSON y DDL atómicas y seguras contra bloqueos

    > [!NOTE]
    > MySQL 5.6 deja de tener soporte técnico general en febrero de 2021. Las cargas de trabajo de MySQL se deben migrar a la versión 5.7 o posterior de MySQL.

Para comprobar la versión del servidor MySQL:

```
SHOW VARIABLES LIKE "%version%";
```

### <a name="database-storage-engines"></a>Motores de almacenamiento de bases de datos

Azure Database for MySQL solo admite los motores de almacenamiento de bases de datos [InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html) y [Memory](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html). Otros motores de almacenamiento, como [MyISAM](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html), se deben migrar a un motor de almacenamiento compatible. Las diferencias entre MyISAM e InnoDB son las características operativas y la salida de rendimiento. Las tablas de nivel superior y la estructura de esquemas normalmente no cambian entre los motores, pero los tipos de columna de índice y tabla pueden cambiar por motivos de almacenamiento y rendimiento. Aunque se sabe que InnoDB tiene tamaños de archivo de datos grandes, estos detalles de almacenamiento se administran mediante el servicio Azure Database for MySQL.

#### <a name="migrating-from-myisam-to-innodb"></a>Migración de MyISAM a InnoDB

La base de datos y las tablas de MyISAM deben convertirse en tablas de InnoDB. Después de la conversión, se debe probar la compatibilidad y el rendimiento de las aplicaciones. En la mayoría de los casos, las pruebas requieren volver a crear la tabla y cambiar el motor de almacenamiento de destino mediante instrucciones DDL. Es poco probable que este cambio tenga que realizarse durante la migración, ya que se produce durante la creación del esquema en el destino de Azure. Para más información, repase la [documentación para desarrolladores sobre conversión de tablas](https://dev.mysql.com/doc/refman/5.6/en/converting-tables-to-innodb.html) en el sitio web de MySQL.

Para encontrar información útil sobre las tablas, use esta consulta:

```dotnetcli
    SELECT 
        tab.table_schema,   
        tab.table_name,   
        tab.engine as engine_type,   
        tab.auto_increment,   
        tab.table_rows,   
        tab.create_time,   
        tab.update_time,   
        tco.constraint_type 
    FROM information_schema.tables tab   
    LEFT JOIN information_schema.table_constraints tco   
        ON (tab.table_schema = tco.table_schema   
            AND tab.table_name = tco.table_name   
            )   
    WHERE  
        tab.table_schema NOT IN ('mysql', 'information_schema', 'performance_
schema', 'sys')  
        AND tab.table_type = 'BASE TABLE';
```

> [!NOTE]
> La ejecución de una consulta en INFORMATION\_SCHEMA en varias bases de datos puede afectar al rendimiento. Ejecútela durante períodos de uso bajo.

Para convertir una tabla de MyISAM a InnoDB, ejecute lo siguiente:

```
ALTER TABLE {table\_name} ENGINE=InnoDB;
```

> [!NOTE]
> Este enfoque de conversión hace que la tabla se bloquee y todas las aplicaciones pueden esperar hasta que se complete la operación. El bloqueo de tabla hace que la base de datos aparezca sin conexión durante un breve período.

En su lugar, la tabla se puede crear con la misma estructura pero con un motor de almacenamiento diferente. Una vez creadas, copie las filas en la nueva tabla:

```
INSERT INTO {table\_name} SELECT * FROM {myisam\_table} ORDER BY {primary\_key\_columns}
```

> [!NOTE]
> Para que este enfoque sea correcto, la tabla original tendría que eliminarse y, a continuación, cambiar el nombre de la nueva tabla. Esta tarea provoca un breve período de tiempo de inactividad.

### <a name="database-data-and-objects"></a>Datos y objetos de la base de datos

Los datos son un componente de la migración de bases de datos. Los objetos que sustentan la base de datos se deben migrar y validar para asegurarse de que las aplicaciones siguen funcionando de forma confiable.

Esta es una lista de los elementos que debe inventariar antes y después de la migración:

  - Tablas (esquema)

  - Claves principales, claves externas

  - Índices

  - Funciones

  - Procedimientos

  - Desencadenadores

  - Vistas

### <a name="user-defined-functions"></a>Funciones definidas por el usuario

MySQL permite el uso de funciones que llaman a código externo. Desafortunadamente, las cargas de trabajo que utilizan funciones definidas por el usuario con código externo no se pueden migrar a Azure Database for MySQL. Esto se debe a que no se puede cargar el código dll o el respaldo de la función de MySQL necesario en el servidor de Azure.

Ejecute la consulta siguiente para buscar las funciones definidas por el usuario que se puedan instalar:

```
SELECT * FROM mysql.func;
```

## <a name="source-systems"></a>Sistemas de origen

El tiempo de preparación de la migración puede variar según el sistema de origen y su ubicación. Además de los objetos de la base de datos, considere cómo trasladar los datos del sistema de origen al de destino. La migración de datos puede resultar difícil cuando hay firewalls y otros componentes de red entre el origen y el destino.

Además, trasladar datos a través de Internet puede ser más lento que usar circuitos dedicados en Azure. Por lo tanto, si va a trasladar muchos gigabytes, terabytes y petabytes de datos, considere la posibilidad de configurar una conexión de [ExpressRoute](../../../expressroute/expressroute-introduction.md) entre la red de origen y la red de Azure.

Si ExpressRoute ya está presente, es probable que otras aplicaciones utilicen la conexión. La realización de una migración mediante una ruta ya existente puede provocar una presión en el rendimiento de la red y provocar un impacto significativo en el rendimiento de la migración y de las otras aplicaciones que usan la red.

Por último, se debe evaluar el espacio en disco. Al exportar una base de datos grande, tenga en cuenta el tamaño de los datos. Asegúrese de que el sistema donde se ejecuta la herramienta y, en última instancia, la ubicación de exportación, cuente con suficiente espacio en disco para realizar la operación de exportación.

### <a name="cloud-providers"></a>Proveedores de nube

La migración de bases de datos de proveedores de servicios en la nube como Amazon Web Services (AWS) puede requerir un paso de configuración de red adicional para acceder a las instancias de MySQL hospedadas en la nube. Las herramientas de migración, como los servicios de migración de datos, requieren acceso desde intervalos IP externos ya que, de lo contrario, podrían bloquearse.

### <a name="on-premises"></a>Local

Al igual que los proveedores de nube, si la carga de trabajo de datos de MySQL está detrás de firewalls u otras capas de seguridad de red, será necesario crear una ruta de acceso entre la instancia local y Azure Database for MySQL.

## <a name="tools"></a>Herramientas

Se pueden usar muchas herramientas y métodos para evaluar las cargas de trabajo y los entornos de datos de MySQL. Cada herramienta proporciona un conjunto diferente de características y funcionalidades de evaluación y migración. Como parte de esta guía, se revisan las herramientas más usadas a la hora de evaluar las cargas de trabajo de datos de MySQL.

### <a name="azure-migrate"></a>Azure Migrate

Aunque [Azure Migrate](/azure/migrate/migrate-services-overview) no admite la migración directa de cargas de trabajo de bases de datos de MySQL, se puede usar cuando los administradores no están seguros de qué usuarios y aplicaciones consumen los datos, ya sea hospedados en una máquina virtual o en una máquina basada en hardware. El [análisis de dependencias](/azure/migrate/concepts-dependency-visualization) se puede realizar instalando y ejecutando el agente de supervisión en la máquina que hospeda la carga de trabajo de MySQL. El agente recopila la información durante un período establecido como, por ejemplo, un mes. Los datos de dependencia se pueden analizar para buscar conexiones desconocidas que se realizan en la base de datos. Los datos de conexión pueden ayudar a identificar a los propietarios de aplicaciones que deben recibir una notificación de la migración pendiente.

Además del análisis de dependencias de las aplicaciones y los datos de conectividad de usuario, Azure Migrate también se puede usar para analizar [Hyper-V, VMware o los servidores físicos](../../../migrate/migrate-appliance-architecture.md) para proporcionar patrones de uso de las cargas de trabajo de bases de datos que ayuden a sugerir el entorno de destino adecuado.

### <a name="telgraf-for-linux"></a>Telegraf para Linux

Las cargas de trabajo de Linux pueden usar [Microsoft Monitoring Agent (MMA)](../../../azure-monitor/agents/agent-linux.md) para recopilar datos en las máquinas virtuales y en las físicas. Además, considere la posibilidad de usar el [agente de Telegraf](../../../azure-monitor/essentials/collect-custom-metrics-linux-telegraf.md) y su amplia gama de complementos para recopilar las métricas de rendimiento.

### <a name="service-tiers"></a>Niveles de servicio

Una vez que cuenta con la información de evaluación (CPU, memoria, almacenamiento, etc.), la siguiente opción del usuario de la migración es decidir qué [plan de tarifa](../../concepts-pricing-tiers.md) de Azure Database for MySQL empezar a usar.

En la actualidad hay tres planes:

  - **Básico**: cargas de trabajo que requieren proceso y rendimiento de E/S ligeros.

  - **De uso general**: para la mayoría de las cargas de trabajo empresariales que requieren un equilibrio entre proceso y memoria con rendimiento de E/S escalable.

  - **Optimizado para memoria**: para cargas de trabajo de bases de datos de alto rendimiento que requieren rendimiento en memoria para un procesamiento más rápido de las transacciones y mayor simultaneidad.

Los requisitos de RTO y RPO de la carga de trabajo de datos pueden influir en la decisión sobre qué plan elegir. Si la carga de trabajo de datos requiere más de 4 TB de almacenamiento, se requiere un paso adicional. Revise y seleccione [una región que admita](../../concepts-pricing-tiers.md#storage) hasta 16 TB de almacenamiento.

> [!NOTE]
> Póngase en contacto con el equipo de MySQL (AskAzureDBforMySQL@service.microsoft.com) para las regiones que no admitan sus requisitos de almacenamiento.

Normalmente, la toma de decisiones se centra en las necesidades de almacenamiento e IOPS, o en las operaciones de entrada/salida por segundo. Por ello, el sistema de destino siempre necesita, como mínimo, el mismo almacenamiento que en el sistema de origen. Además, dado que a las IOPS se les asignan 3/GB, es importante hacer coincidir las E/S por segundo con el tamaño de almacenamiento final.

| Factores | Nivel |
|---------|------|
| **Basic** | Máquina de desarrollo, no necesita alto rendimiento, con menos de 1 TB de almacenamiento. |
| **Uso general** | Necesita más operaciones de entrada/salida por segundo de lo que puede proporcionar el plan Básico, pero para almacenamiento necesita menos de 16 TB y menos de 4 GB de memoria. |
| **Memoria optimizada** | Cargas de trabajo de datos que usan una memoria alta o una caché alta, y una configuración de servidor relacionada con el búfer, como alta simultaneidad innodb_buffer_pool_instances, tamaños de BLOB grandes y sistemas con muchos servidores subordinados para la replicación. |

### <a name="costs"></a>Costos

Después de evaluar todas las cargas de trabajo de datos de MySQL en WWI, este determinó que necesitarían como mínimo 4 núcleos virtuales y 20 GB de memoria, y al menos 100 GB de espacio de almacenamiento con una capacidad de 450 IOPS. Debido al requisito de 450 IOPS, deben asignar al menos 150 GB de almacenamiento debido al [método de asignación de IOPS de Azure Database for MySQL](../../concepts-pricing-tiers.md#storage). Además, requieren al menos el 100 % del almacenamiento del servidor aprovisionado como almacenamiento de copia de seguridad y una réplica de lectura. No prevén una salida de más de 5 GB.

Mediante la [calculadora de precios de Azure Database for MySQL](https://azure.microsoft.com/pricing/details/mysql/), WWI pudo determinar los costos de la instancia de Azure Database for MySQL. Desde septiembre de 2020, el costo total de propiedad (TCO) se muestra en la tabla siguiente de la base de datos de WWI Conference:

| Recurso | Descripción | Cantidad | Coste |
|----------|-------------|----------|------|
| **Proceso (De uso general)** | 4 núcleos virtuales, 20 GB                   | 1 a 0,351 USD/hora                                               | 3074,76 USD/año |
| **Storage**                   | 5 GB                              | 12 x 150 a 0,115 USD                                           | 207 USD/año     |
| **Backup**                    | Hasta el 100 % del almacenamiento aprovisionado | Sin costo adicional hasta el 100 % del almacenamiento de servidor aprovisionado      | 0,00 USD/año    |
| **Réplica de lectura**              | Réplica regional de 1 segundo           | Proceso y almacenamiento                                           | 3281,76 USD/año |
| **Network**                   | Salida < 5 GB/mes                | Gratuito                                                        |               |
| **Total**                     |                                   |                                                             | 6563,52 USD/año |

Después de revisar los costos iniciales, el CIO de WWI confirmó que llevan en Azure mucho más de 3 años. Han decidido usar [instancias reservadas](../../concept-reserved-pricing.md) de 3 años para ahorrar unos 4000 USD por año adicionales:

| Recurso | Descripción | Cantidad | Coste |
|----------|-------------|----------|------|
| **Proceso (De uso general)** | 4 núcleos virtuales                          | 1 a 0,1375 USD/hora                                               | 1204,5 USD/año |
| **Storage**                   | 5 GB                              | 12 x 150 a 0,115 USD                                            | 207 USD/año    |
| **Backup**                    | Hasta el 100 % del almacenamiento aprovisionado | Sin costo adicional hasta el 100 % del almacenamiento de servidor aprovisionado       | 0,00 USD/año   |
| **Network**                   | Salida < 5 GB/mes                | Gratuito                                                         |              |
| **Réplica de lectura**              | Réplica regional de 1 segundo           | Proceso y almacenamiento                                            | 1411,5 USD/año |
| **Total**                     |                                   |                                                              | 2823 USD/año   |

Como se muestra en la tabla anterior, las copias de seguridad, la salida de red y las réplicas de lectura deben tenerse en cuenta en el costo total de propiedad (TCO). A medida que se agregan más bases de datos, el almacenamiento y el tráfico de red generados serían el único factor adicional basado en costos que se debería tener en cuenta.

> [!NOTE]
> Las estimaciones anteriores no incluyen ningún costo de [ExpressRoute](/azure/expressroute/expressroute-introduction), [Azure Application Gateway](/azure/application-gateway/overview), [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) o [App Service](/azure/app-service/overview) para las capas de aplicaciones.
>
> Los precios anteriores pueden cambiar en cualquier momento y varían en función de la región.

### <a name="application-implications"></a>Implicaciones de la aplicación

Al pasar a Azure Database for MySQL, es probable que la conversión a la comunicación basada en capa de sockets seguros (SSL) sea uno de los cambios más significativos para las aplicaciones. SSL está habilitado de forma predeterminada en Azure Database for MySQL y es probable que la aplicación local y la carga de trabajo de datos no estén configuradas para conectarse a MySQL mediante SSL. Cuando se habilita, el uso de SSL agrega cierta sobrecarga de procesamiento adicional y se debe supervisar.

> [!NOTE]
> Aunque la capa de sockets seguros (SSL) está habilitada de forma predeterminada, tiene la opción de deshabilitarla.

Siga las actividades de [Configuración de la conectividad SSL en la aplicación para conectarse de forma segura a Azure Database for MySQL](../../howto-configure-ssl.md) para volver a configurar la aplicación para que admita esta ruta de autenticación sólida.

Por último, modifique el nombre del servidor en las cadenas de conexión de la aplicación o cambie el DNS para que apunte al nuevo servidor de Azure Database for MySQL.

## <a name="wwi-scenario"></a>Escenario de WWI

WWI inició la evaluación mediante la recopilación de información sobre su patrimonio de datos de MySQL. Pudieron compilar lo siguiente:

| Nombre | Source | Motor de base de datos | Size | E/S | Versión | Propietario | Tiempo de inactividad |
|------|--------|-----------|------|------|---------|-------|----------|
| **WwwDB**        | AWS (PaaS)  | InnoDB | 1 GB  | 150 | 5.7 | Departamento de marketing | 1 hora  |
| **BlogDB**       | AWS (PaaS)  | InnoDB | 1 GB  | 100 | 5.7 | Departamento de marketing | 4 h |
| **ConferenceDB** | Local | InnoDB | 5 GB  | 50  | 5.5 | Departamento de ventas     | 4 h |
| **CustomerDB**   | Local | InnoDB | 10 GB | 75  | 5.5 | Departamento de ventas     | 2 horas |
| **SalesDB**      | Local | InnoDB | 20 GB | 75  | 5.5 | Departamento de ventas     | 1 hora  |

Se ha contactado con cada propietario de una base de datos para determinar el período de tiempo de inactividad aceptable. El método de planeamiento y migración seleccionado se basaba en el tiempo de inactividad aceptable de la base de datos.

En la primera fase, WWI se centró únicamente en la base de datos ConferenceDB. El equipo necesitó la experiencia de migración para ayudar a las migraciones de cargas de trabajo de datos en curso. Se seleccionó la base de datos ConferenceDB debido a la estructura simple de la base de datos y al gran tiempo de inactividad aceptable. Una vez migrada la base de datos, el equipo se ha centrado en migrar la aplicación a la zona de aterrizaje segura de Azure.

## <a name="assessment-checklist"></a>Lista de comprobación de evaluación

  - Pruebe que la carga de trabajo se ejecuta correctamente en el sistema de destino.

  - Asegúrese de que se han puesto en marcha los componentes de red adecuados para la migración.

  - Comprenda los requisitos de recursos de la carga de trabajo de datos.

  - Estime los costos totales.

  - Comprenda los requisitos de tiempo de inactividad.

  - Prepárese para realizar cambios en la aplicación.

> [!div class="nextstepaction"]
> [Planeamiento](./04-planning.md)