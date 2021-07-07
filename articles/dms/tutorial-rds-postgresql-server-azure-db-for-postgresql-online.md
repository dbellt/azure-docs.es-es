---
title: 'Tutorial: Migración de RDS PostgreSQL en línea a Azure Database for PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Aprenda a realizar una migración en línea de PostgreSQL de RDS a Azure Database for PostgreSQL mediante Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 04/11/2020
ms.openlocfilehash: 534790f7a63f8fbf11e80c71c00cf1b98ec4b5da
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063950"
---
# <a name="tutorial-migrate-rds-postgresql-to-azure-db-for-postgresql-online-using-dms"></a>Tutorial: Migración de RDS PostgreSQL a Azure DB for PostgreSQL en línea mediante DMS

Puede utilizar Azure Database Migration Service para migrar bases de datos de una instancia de PostgreSQL de RDS a [Azure Database for PostgreSQL](../postgresql/index.yml) mientras la base de datos de origen permanece en línea durante la migración. En otras palabras, la migración se puede completar con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **DVD Rental** de una instancia de PostgreSQL 9.6 de RDS a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Migre el esquema de ejemplo mediante la utilidad pg_dump.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración
> * Realizar la migración total.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service. El disco se cifra para impedir el robo de datos durante el proceso de migración.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe cómo realizar una migración en línea de una instancia local de PostgreSQL a Azure Database for PostgreSQL.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Descargue e instale [PostgreSQL community edition](https://www.postgresql.org/download/) 9.5, 9.6 o 10. La versión del servidor PostgreSQL Server de origen debe ser la 9.5.11, 9.6.7, 10 o una posterior. Para más información, consulte el artículo acerca de las [versiones de base de datos admitidas de PostgreSQL](../postgresql/concepts-supported-versions.md).

   Tenga en cuenta también que la versión de Azure Database for PostgreSQL de destino debe ser igual o posterior a la versión de RDS PostgreSQL. Por ejemplo, RDS PostgreSQL 9.6 solo puede migrarse a Azure Database for PostgreSQL 9.6, 10 u 11, pero no a Azure Database for PostgreSQL 9.5.

* Cree una instancia de [Azure Database for PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) o [Hiperescala (Citus) de Azure Database for PostgreSQL](../postgresql/quickstart-create-hyperscale-portal.md). Consulte esta [sección](../postgresql/quickstart-create-server-database-portal.md#connect-to-the-server-with-psql) del documento para obtener detalles sobre cómo conectarse al servidor PostgreSQL mediante pgAdmin.
* Cree una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.
* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean el puerto de salida 443 de ServiceTag para ServiceBus, Storage y AzureMonitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service tenga acceso al servidor PostgreSQL de origen que, de manera predeterminada, es el puerto TCP 5432.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](../postgresql/concepts-firewall-rules.md) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service tener acceso a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.

### <a name="set-up-aws-rds-postgresql-for-replication"></a>Configuración de AWS RDS PostgreSQL para la replicación

1. Para crear un nuevo grupo de parámetros, siga las instrucciones proporcionadas por AWS en el artículo [Trabajo con los grupos de parámetros de base de datos](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html).
2. Use el nombre de usuario maestro para conectarse al origen de Azure Database Migration Service. Si usa una cuenta distinta de la cuenta de usuario maestro, la cuenta debe contar con los roles rds_superuser y rds_replication. El rol rds_replication concede permisos para administrar ranuras lógicas y transmitir datos mediante ranuras lógicas.
3. Cree un nuevo grupo de parámetros con la siguiente configuración:

    a. Establezca el parámetro rds.logical_replication de su grupo de parámetros de base de datos en 1.

    b. max_wal_senders = [número de tareas simultáneas]; el parámetro max_wal_senders establece el número de tareas simultáneas que puede ejecutar; se recomiendan 10 tareas.

    c. max_replication_slots = [número de ranuras]; se recomienda establecer en 5 ranuras.

4. Asocie el grupo de parámetros que creó a la instancia de PostgreSQL de RDS.

## <a name="migrate-the-schema"></a>Migración del esquema

1. Extraiga el esquema de la base de datos de origen y aplíquelo a la base de datos de destino para completar la migración de todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados.

    La forma más fácil de migrar solo el esquema es usar pg_dump con la opción -s. Para obtener más información, consulte los [ejemplos](https://www.postgresql.org/docs/9.6/app-pgdump.html#PG-DUMP-EXAMPLES) en el tutorial de Postgres pg_dump.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Por ejemplo, para volcar un archivo de esquema para la base de datos **dvdrental**, use el siguiente comando:

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

2. Cree una base de datos vacía en el servicio de destino, que es Azure Database for PostgreSQL. Para conectar y crear una base de datos, consulte uno de los siguientes artículos:

    * [Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL en Azure Portal](../postgresql/quickstart-create-server-database-portal.md)
    * [Creación de un servidor de Hiperescala (Citus) de Azure Database for PostgreSQL en Azure Portal](../postgresql/quickstart-create-hyperscale-portal.md)

3. Importe el esquema al servicio de destino, que es Azure Database for PostgreSQL. Para restaurar el archivo de volcado del esquema, ejecute el siguiente comando:

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Por ejemplo:

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

  > [!NOTE]
   > El servicio de migración controla internamente la habilitación o deshabilitación de claves externas y desencadenadores para garantizar una migración de datos confiable y sólida. Como resultado, no tiene que preocuparse por realizar modificaciones en el esquema de la base de datos de destino.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Creación de una instancia de Azure Database Migration Service

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de PostgreSQL de origen y a la instancia de Azure Database for PostgreSQL de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](../virtual-network/quick-create-portal.md).

6. Seleccione un plan de tarifa. Para la migración en línea, asegúrese de seleccionar el plan de tarifa Premium: 4vCores.

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-settings5.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que ha creado, selecciónela y, luego, elija **+ Nuevo proyecto de migración**.
3. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Tipo de servidor de origen**, seleccione **AWS RDS for PostgreSQL** y, en el cuadro de texto **Tipo de servidor de destino**, elija **Azure Database for PostgreSQL**.
4. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    > [!IMPORTANT]
    > Asegúrese de seleccionar **Migración de datos en línea**; las migraciones sin conexión no se admiten en este escenario.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-create-project5.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

5. Seleccione **Guardar**.

6. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

    > [!NOTE]
    > Tome nota de los requisitos previos necesarios para configurar la migración en línea en la hoja de creación de proyecto.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

* En la pantalla **Agregar detalles de origen**, especifique los detalles de conexión de la instancia de PostgreSQL de origen.

   ![Detalles del origen](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y, a continuación, en la pantalla **Detalles de destino**, especifique los detalles de conexión para el servidor de Azure Database for PostgreSQL de destino, que se ha aprovisionado previamente y tiene el esquema **DVD Rentals** implementado mediante pg_dump.

    ![Detalles de destino](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-target-details.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-map-target-databases.png)

3. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

    ![Estado de actividad: en ejecución](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-activity-status3.png)

2. En **NOMBRE DE LA BASE DE DATOS**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    **Carga completa de los datos** muestra el estado de migración de la carga inicial mientras que **Sincronización de datos incrementales** muestra el estado de la captura de datos modificados (CDC).

    ![Pantalla de inventario: carga completa de los datos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-full-load.png)

    ![Pantalla de inventario: sincronización de datos incrementales](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez finalizada la carga completa inicial, las bases de datos se marcan como **Lista para la transición**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

2. Espere a que el contador **Cambios pendientes** muestre **0** para garantizar que todas las transacciones entrantes a la base de datos de origen están detenidas, active la casilla **Confirmar** y, luego, seleccione **Aplicar**.

    ![Pantalla de migración total completa](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-complete-cutover.png)

3. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure Database for PostgreSQL de destino.

Su migración en línea de una instancia local de RDS PostgreSQL a Azure Database for PostgreSQL ya está completa.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
* Para más información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](../postgresql/overview.md)
* Para otras preguntas, envíe un correo electrónico al alias [Ask Azure Database Migrations](mailto:AskAzureDatabaseMigrations@service.microsoft.com).
