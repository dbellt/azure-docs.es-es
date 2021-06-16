---
title: 'Tutorial: Migración de MySQL en línea a Azure Database for MySQL'
titleSuffix: Azure Database Migration Service
description: Aprenda a realizar una migración en línea de una instancia local de MySQL a Azure Database for MySQL mediante Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/08/2020
ms.openlocfilehash: 44d856637e3e3c999933669e60ee1df29b2c7c8f
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111966410"
---
# <a name="tutorial-migrate-mysql-to-azure-database-for-mysql-online-using-dms"></a>Tutorial: Migración de MySQL a Azure Database for MySQL en línea mediante DMS

Azure Database Migration Service se puede usar para migrar las bases de datos de una instancia local de MySQL a [Azure Database for MySQL](../mysql/index.yml) con un tiempo de inactividad mínimo. En otras palabras, se puede lograr la migración con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **Employees** de una instancia local de MySQL 5.7 a Azure Database for MySQL mediante una actividad de migración en línea de Azure Database Migration Service.

> [!IMPORTANT]
> El escenario de migración en línea "MySQL a Azure Database for MySQL" **no estará disponible a partir del 1 de junio de 2021**. Una [funcionalidad de migración sin conexión](./tutorial-mysql-azure-mysql-offline-portal.md) paralelizada y de alto rendimiento está **ahora disponible en versión preliminar** para admitir migraciones de "MySQL a Azure Database for MySQL". Para las migraciones en línea, puede usar herramientas de código abierto, como [MyDumper/MyLoader](https://centminmod.com/mydumper.html), con [replicación de datos de entrada](../mysql/concepts-data-in-replication.md).

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Migre el esquema de ejemplo mediante la utilidad mysqldump.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

> [!NOTE]
> Comunicación sin prejuicios
>
> Microsoft admite un entorno diverso e inclusivo. En este artículo se incluyen referencias a la palabra _esclavo_. En la [guía de estilo para la comunicación sin prejuicios](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) de Microsoft se reconoce que se trata de una palabra excluyente. Se usa en este artículo por coherencia, ya que actualmente es la palabra que aparece en el software. Cuando el software se actualice para quitarla, este artículo se actualizará en consecuencia.
>


## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Descargue e instale [MySQL community edition](https://dev.mysql.com/downloads/mysql/) 5.6 o 5.7. La versión de MySQL local debe coincidir con la versión de Azure Database for MySQL. Por ejemplo, MySQL 5.6 solo puede migrar a Azure Database for MySQL 5.6 y no a la versión actualizada 5.7. No se admiten migraciones a o desde MySQL 8.0.
* [Creación de una instancia en Azure Database for MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Consulte el artículo [Uso de MySQL Workbench para conectarse y consultar datos en Azure Database for MySQL](../mysql/connect-workbench.md) y obtenga más información sobre cómo conectarse y crear una base de datos mediante la aplicación Workbench.  
* Cree una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service con el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los [puntos de conexión](../virtual-network/virtual-network-service-endpoints-overview.md) de servicio siguientes a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean el puerto de salida 443 de ServiceTag para ServiceBus, Storage y AzureMonitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el firewall de Windows para que Azure Database Migration Service pueda acceder al servidor MySQL de origen que, de manera predeterminada, es el puerto TCP 3306.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](../azure-sql/database/firewall-configure.md) de nivel de servidor para que Azure Database for MySQL permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.
* La versión de la instancia de MySQL de origen debe ser compatible con MySQL community edition. Para determinar la versión de la instancia de MySQL, en la utilidad de MySQL o en MySQL Workbench, ejecute el siguiente comando:

    ```
    SELECT @@version;
    ```

* Azure Database for MySQL solo es compatible con tablas de InnoDB. Para convertir tablas de MyISAM a InnoDB, consulte el artículo [Converting Tables from MyISAM to InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html) (Cómo convertir tablas de MyISAM a InnoDB)

* Habilite el registro binario en el archivo my.ini (Windows) o en el archivo my.cnf (Unix) en la base de datos de origen con la configuración siguiente:

  * **server_id** = 1 o mayor (pertinente solo para MySQL 5.6)
  * **log-bin**=\<path> (pertinente solo para MySQL 5.6)    Por ejemplo, log-bin = E:\MySQL_logs\BinLog
  * **binlog_format** = row
  * **Expire_logs_days** = 5 (se recomienda no usar cero; pertinente solo para MySQL 5.6)
  * **Binlog_row_image** = full (pertinente solo para MySQL 5.6)
  * **log_slave_updates** = 1

* El usuario debe tener el rol ReplicationAdmin con los siguientes privilegios:

  * **REPLICATION CLIENT** (CLIENTE DE REPLICACIÓN): necesario solo para tareas de procesamiento de cambios. En otras palabras, las tareas solo de carga completa no requieren este privilegio.
  * **REPLICATION REPLICA** (RÉPLICA DE REPLICACIÓN): necesario solo para tareas de procesamiento de cambios. En otras palabras, las tareas solo de carga completa no requieren este privilegio.
  * **SUPER** (SÚPER): solo se requiere en las versiones anteriores a MySQL 5.6.6.

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Para completar todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados, se debe extraer el esquema de la base de datos de origen y aplicarlo a la base de datos. Para extraer el esquema, puede usar mysqldump con el parámetro `--no-data`.

Suponiendo que tiene la base de datos de ejemplo de **empleados** de MySQL en el sistema local, el comando para realizar la migración del esquema mediante mysqldump es:

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

Por ejemplo:

```
mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
```

Para importar el esquema en la instancia de Azure Database for MySQL de destino, ejecute el siguiente comando:

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

Por ejemplo:

```
mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
 ```

Si tiene claves externas en el esquema, se producirá un error en la carga inicial y la sincronización continua de la migración.  Ejecute el siguiente script en MySQL Workbench para extraer el script de colocación de clave externa y agregar el script de clave externa.

```sql
SET group_concat_max_len = 8192;
    SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
    FROM
    (SELECT
    KCU.REFERENCED_TABLE_SCHEMA as SchemaName,
    KCU.TABLE_NAME,
    KCU.COLUMN_NAME,
    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
    FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
    WHERE
      KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
      AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
  AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries
  GROUP BY SchemaName;
 ```

Ejecute el script de colocación de clave externa (que es el de la segunda columna) en el resultado de la consulta para colocar la clave externa.

> [!NOTE]
> Azure DMS no admite la acción referencial CASCADE, que ayuda a eliminar o actualizar automáticamente una fila coincidente en la tabla secundaria cuando se elimina o se actualiza una fila en la tabla primaria. Para obtener más información, consulte la sección sobre acciones referenciales del artículo [FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) en la documentación de MySQL.
> Azure DMS requiere que se quiten las restricciones de claves externas en el servidor de base de datos de destino durante la carga de datos inicial, y no se pueden usar acciones referenciales. Si la carga de trabajo depende de la actualización de una tabla secundaria relacionada a través de esta acción referencial, se recomienda realizar un [volcado y restauración](../mysql/concepts-migrate-dump-restore.md) en su lugar. 


> [!IMPORTANT]
> Si importa datos mediante una copia de seguridad, quite los comandos CREATE DEFINER manualmente o mediante el comando --skip-definer durante una operación mysqldump. DEFINER requiere privilegios de superusuarios para crear y está restringido en Azure Database for MySQL.

Si la base de datos utiliza desencadenadores, se aplicará la integridad de datos en el destino antes de llevar a cabo la migración completa de los datos desde el origen. Es recomendable deshabilitar los desencadenadores en todas las tablas del destino durante la migración y, a continuación, habilitar los desencadenadores una vez finalizada esta.

Ejecute el siguiente script en MySQL Workbench en la base de datos de destino para extraer el script del desencadenador de anulación y el script del desencadenador de adición.

```sql
SELECT
    SchemaName,
    GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery,
    Concat('DELIMITER $$ \n\n', GROUP_CONCAT(AddQuery SEPARATOR '$$\n'), '$$\n\nDELIMITER ;') as AddQuery
FROM
(
SELECT 
    TRIGGER_SCHEMA as SchemaName,
    Concat('DROP TRIGGER `', TRIGGER_NAME, "`") as DropQuery,
    Concat('CREATE TRIGGER `', TRIGGER_NAME, '` ', ACTION_TIMING, ' ', EVENT_MANIPULATION, 
            '\nON `', EVENT_OBJECT_TABLE, '`\n' , 'FOR EACH ', ACTION_ORIENTATION, ' ',
            ACTION_STATEMENT) as AddQuery
FROM  
    INFORMATION_SCHEMA.TRIGGERS
ORDER BY EVENT_OBJECT_SCHEMA, EVENT_OBJECT_TABLE, ACTION_TIMING, EVENT_MANIPULATION, ACTION_ORDER ASC
) AS Queries
GROUP BY SchemaName
```

Ejecute la consulta del desencadenador de anulación generada (columna DropQuery) en el resultado para quitar los desencadenadores de la base de datos de destino. La consulta para agregar desencadenadores se puede guardar y usar una vez finalizada la migración de datos.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-mysql-to-azure-mysql-online/01-portal-select-subscriptions.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-mysql-to-azure-mysql-online/02-01-portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-mysql-to-azure-mysql-online/02-02-portal-register-resource-provider.png)

## <a name="create-a-database-migration-service-instance"></a>Creación de una instancia de Azure Database Migration Service

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-mysql-to-azure-mysql-online/03-dms-portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/04-dms-portal-marketplace-create.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione un plan de tarifa y pase a la pantalla de redes. La funcionalidad de migración sin conexión está disponible en los planes de tarifa Estándar y Premium.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración básica de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/05-dms-portal-create-basic.png)

5. Seleccione una red virtual existente en la lista o proporcione el nombre de la nueva red virtual que se va a crear. Acceda a la pantalla de revisión y creación. Como alternativa, puede agregar etiquetas al servicio mediante la pantalla de etiquetas.

    La red virtual proporciona a Azure Database Migration Service acceso al servidor SQL Server de origen y a la instancia de Azure SQL Database de destino.

    ![Configuración de red de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/06-dms-portal-create-networking.png)

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](../virtual-network/quick-create-portal.md).

6. Revise las configuraciones y seleccione **Crear** para crear el servicio.
    
    ![Creación de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/07-dms-portal-create-submit.png)

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.  

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

    ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/08-01-dms-portal-search-service.png)

2. Seleccione la instancia del servicio de migración en los resultados de búsqueda y seleccione **Nuevo proyecto de migración**.
    
    ![Creación de un proyecto de migración](media/tutorial-mysql-to-azure-mysql-online/08-02-dms-portal-new-project.png)

3. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de selección **Tipo de servidor de origen**, seleccione **MySQL** y, en el cuadro selección **Tipo de servidor de destino**, seleccione **Azure Database for MySQL**. Por último, en el cuadro de selección **Tipo de actividad de migración**, seleccione **Migración de datos en línea**. Seleccione **Crear y ejecutar una actividad**.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/09-dms-portal-project-mysql-create.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

## <a name="configure-migration-project"></a>Configuración del proyecto de migración

1. En la pantalla **Seleccionar origen**, especifique los detalles de conexión de la instancia de origen de MySQL Server y seleccione **Siguiente: Seleccionar destino>>** .

    ![Pantalla Agregar detalles de origen](media/tutorial-mysql-to-azure-mysql-online/10-dms-portal-project-mysql-source.png)

2. En la pantalla **Seleccionar destino**, especifique los detalles de conexión para la instancia de destino de Azure Database for MySQL y seleccione **Siguiente: Seleccionar bases de datos>>** .

    ![Pantalla Agregar detalles de destino](media/tutorial-mysql-to-azure-mysql-online/11-dms-portal-project-mysql-target.png)

3. En la pantalla **Seleccionar bases de datos**, asigne el origen a la base de datos de destino para la migración y seleccione **Siguiente: Configurar los valores de migración>>** . Puede seleccionar la opción **Make Source Server Readonly** (Hacer que el servidor de origen sea de solo lectura) para que el origen sea de solo lectura, pero tenga en cuenta que esta es una configuración a nivel de servidor. Si la opción se selecciona, establecerá todo el servidor como de solo lectura, no únicamente las bases de datos seleccionadas.
    
    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.
    ![Pantalla Seleccionar detalles de la base de datos](media/tutorial-mysql-to-azure-mysql-online/12-dms-portal-project-mysql-select-db.png)
    
    > [!NOTE] 
   > Aunque en este paso se pueden seleccionar varias bases de datos, cada instancia de Azure Database Migration Service admite un máximo de cuatro para la migración simultánea. Además, hay un límite de diez instancias de Azure Database Migration Service por suscripción y región. Por ejemplo, si tiene 80 bases de datos para migrar, no puede migrar más de 40 simultáneamente, y solo si ha creado 10 instancias de Azure Database Migration Service.

4. En la pantalla **Configurar los valores de la migración**, seleccione las tablas que formarán parte de la migración. A continuación, seleccione **Siguiente: Resumen>>** . Si las tablas de destino contienen datos, no se seleccionarán de forma predeterminada, pero puede seleccionarlos explícitamente y se truncarán antes de iniciar la migración.

    ![Pantalla Seleccionar tablas](media/tutorial-mysql-to-azure-mysql-online/13-dms-portal-project-mysql-select-tbl.png)

5. En la pantalla **Resumen**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y de destino coincidan con los que especificó anteriormente.

    ![Resumen del proyecto de migración](media/tutorial-mysql-to-azure-mysql-online/14-dms-portal-project-mysql-activity-summary.png)

6. Seleccione **Iniciar migración**. Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**. El **estado** cambia a **En ejecución** cuando se inician las migraciones de tabla.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Completado**.

     ![Estado de actividad Completado](media/tutorial-mysql-to-azure-mysql-online/15-dms-activity-completed.png)

2. En **Nombre de la base de datos**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    La carga completa de los datos muestra el estado de migración de la carga inicial mientras que la sincronización de datos incrementales muestra el estado de la captura de datos modificados (CDC).

     ![Estado de actividad: carga completa efectuada](media/tutorial-mysql-to-azure-mysql-online/16-dms-activity-full-load-completed.png)

     ![Estado de actividad: sincronización de datos incrementales](media/tutorial-mysql-to-azure-mysql-online/17-dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-mysql-to-azure-mysql-online/18-dms-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure SQL de destino.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure Database for MySQL, consulte el artículo [Known issues and workarounds with Azure Database for MySQL online migrations](known-issues-azure-mysql-online.md) (Problemas conocidos y soluciones para las migraciones en línea de Azure Database for MySQL).
* Para más información acerca de Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
* Para información sobre Azure Database for MySQL, consulte el artículo [¿Qué es Azure Database for MySQL?](../mysql/overview.md)