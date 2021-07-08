---
title: 'Tutorial: Migración de SQL Server a Azure SQL Database sin conexión'
titleSuffix: Azure Database Migration Service
description: Aprenda a migrar de SQL Server a Azure SQL Database sin conexión mediante Azure Database Migration Service.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/03/2021
ms.openlocfilehash: 0383b76a08c5c56c27bfbc7f83bf786b4b452849
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110796713"
---
# <a name="tutorial-migrate-sql-server-to-azure-sql-database-using-dms"></a>Migración de SQL Server a Azure SQL Database mediante DMS

Puede usar Azure Database Migration Service para migrar las bases de datos de una instancia de SQL Server a [Azure SQL Database](/azure/sql-database/). En este tutorial migrará la base de datos [Adventureworks2016](/sql/samples/adventureworks-install-configure#download-backup-files) restaurada en una instancia local de SQL Server 2016 (o posterior) a una base de datos única o agrupada de Azure SQL Database mediante Azure Database Migration Service.

Aprenderá a:
> [!div class="checklist"]
>
> - Valorar y evaluar una base de datos local para detectar problemas de bloqueo mediante Data Migration Assistant.
> - Usar Data Migration Assistant para migrar un esquema de ejemplo de la base de datos. 
> - Registrar el proveedor de recursos de Azure DataMigration.
> - Crear una instancia de Azure Database Migration Service.
> - Crear un proyecto de migración mediante Azure Database Migration Service.
> - Ejecutar la migración.
> - Supervisar la migración


## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- Descargar e instalar [SQL Server 2016 o una versión posterior](https://www.microsoft.com/sql-server/sql-server-downloads).
- Habilitar el protocolo TCP/IP, que se deshabilita de forma predeterminada durante la instalación de SQL Server Express, siguiendo las instrucciones del artículo [Habilitar o deshabilitar un protocolo de red de servidor](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
- Crear una base de datos en Azure SQL Database, para lo que debe seguir la información del artículo [Creación de una base de datos única de Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md). Para los fines de este tutorial, se da por hecho que el nombre de Azure SQL Database es **AdventureWorksAzure**, pero puede asignarle otro si lo desea.

    > [!NOTE]
    > Si usa SQL Server Integration Services (SSIS) y desea migrar la base de datos de catálogo para los proyectos y paquetes SSIS (SSISDB) de SQL Server a Azure SQL Database, la SSISDB de destino se creará y administrará automáticamente en su nombre cuando aprovisione SSIS en Azure Data Factory (ADF). Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](./how-to-migrate-ssis-packages.md).
  
- Descargue e instale la versión más reciente de [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595).
- Cree una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los siguientes [puntos de conexión](../virtual-network/virtual-network-service-endpoints-overview.md) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > - Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > - Punto de conexión de Storage
    > - Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.
    >
    >Si no dispone de conectividad de sitio a sitio entre la red local y Azure, o si el ancho de banda de conectividad de sitio a sitio es limitado, considere la posibilidad de usar Azure Database Migration Service en modo híbrido (versión preliminar). El modo híbrido hace uso de un trabajo de migración local junto con una instancia de Azure Database Migration Service que se ejecuta en la nube. Para crear una instancia de Azure Database Migration Service en el modo híbrido, consulte el artículo [Creación de una instancia de Azure Database Migration Service en modo híbrido mediante Azure Portal](./quickstart-create-data-migration-service-hybrid-portal.md).

- Asegúrese de que las reglas de seguridad de salida del grupo de seguridad de red de la red virtual no bloquean el puerto de salida 443 de ServiceTag para ServiceBus, Storage y AzureMonitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
- Configurar su [Firewall de Windows para acceder al motor de base de datos](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
- Abra el firewall de Windows para que Azure Database Migration Service pueda acceder al servidor SQL Server de origen; de forma predeterminada, es el puerto TCP 1433. Si la instancia predeterminada está escuchando en otro puerto, agréguelo al firewall.
- Si se ejecutan varias instancias con nombre de SQL Server con puertos dinámicos, puede ser conveniente habilitar el servicio SQL Browser y permitir el acceso al puerto UDP 1434 mediante los firewalls para que Azure Database Migration Service pueda conectarse a una instancia con nombre en el servidor de origen.
- Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
- Crear una [regla de firewall](../azure-sql/database/firewall-configure.md) de dirección IP en el nivel de servidor para que Azure SQL Database permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.
- Asegúrese de que las credenciales usadas para conectarse a la instancia de SQL Server de origen tenga permisos [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
- Asegúrese de que las credenciales que se usan para conectarse a la instancia de Azure SQL Database de destino tienen el permiso [CONTROL DATABASE](/sql/t-sql/statements/grant-database-permissions-transact-sql) en las bases de datos de destino.

    > [!IMPORTANT]
    > La creación de una instancia de Azure Database Migration Service requiere acceso a la configuración de la red virtual que normalmente no está dentro del mismo grupo de recursos. Como resultado, el usuario que crea una instancia de DMS requiere el permiso en el nivel de suscripción. Para crear los roles necesarios, que puede asignar según sea necesario, ejecute el siguiente script:
    >
    > ```
    >
    > $readerActions = `
    > "Microsoft.Network/networkInterfaces/ipConfigurations/read", `
    > "Microsoft.DataMigration/*/read", `
    > "Microsoft.Resources/subscriptions/resourceGroups/read"
    >
    > $writerActions = `
    > "Microsoft.DataMigration/services/*/write", `
    > "Microsoft.DataMigration/services/*/delete", `
    > "Microsoft.DataMigration/services/*/action", `
    > "Microsoft.Network/virtualNetworks/subnets/join/action", `
    > "Microsoft.Network/virtualNetworks/write", `
    > "Microsoft.Network/virtualNetworks/read", `
    > "Microsoft.Resources/deployments/validate/action", `
    > "Microsoft.Resources/deployments/*/read", `
    > "Microsoft.Resources/deployments/*/write"
    >
    > $writerActions += $readerActions
    >
    > # TODO: replace with actual subscription IDs
    > $subScopes = ,"/subscriptions/00000000-0000-0000-0000-000000000000/","/subscriptions/11111111-1111-1111-1111-111111111111/"
    >
    > function New-DmsReaderRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Reader"
    > $aRole.Description = "Lets you perform read only actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    >
    > $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    >
    > function New-DmsContributorRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Contributor"
    > $aRole.Description = "Lets you perform CRUD actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    >
    >   $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    > 
    > function Update-DmsReaderRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Reader"
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > function Update-DmsConributorRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Contributor"
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > # Invoke above functions
    > New-DmsReaderRole
    > New-DmsContributorRole
    > Update-DmsReaderRole
    > Update-DmsConributorRole
    > ```

## <a name="assess-your-on-premises-database"></a>Evaluación de una base de datos local

Antes de poder migrar datos de una instancia de SQL Server a una base de datos única o agrupada de Azure SQL Database, debe evaluar la base de datos de SQL Server por si hay cualquier error de bloqueo que impida que se realice la migración. Con Data Migration Assistant, siga los pasos descritos en el artículo sobre cómo [realizar una evaluación de migración de SQL Server](/sql/dma/dma-assesssqlonprem) para completar la evaluación de la base de datos local. A continuación, se muestra un resumen de los pasos necesarios:

1. En Data Migration Assistant, seleccione el icono de Nuevo (+) y, luego, seleccione el tipo de proyecto **Evaluación**.
2. Especifique un nombre de proyecto. En la lista desplegable **Tipo de evaluación**, seleccione **Motor de base de datos**, en el cuadro de texto **Tipo de servidor de origen**, seleccione **SQL Server**; en el cuadro de texto **Tipo de servidor de destino**, seleccione **Azure SQL Database** y, después, seleccione **Crear** para crear el proyecto.

    Cuando esté evaluando la base de datos de SQL Server de origen que se migrará a una base de datos única o agrupada de Azure SQL Database, puede elegir uno o ambos de los siguientes tipos de informes de evaluación:

   - Check database compatibility (Comprobar compatibilidad de bases de datos)
   - Check feature parity (Comprobar paridad de características)

    De forma predeterminada, se seleccionan los dos tipos de informes.

3. En Data Migration Assistant, en la pantalla **Opciones**, seleccione **Siguiente**.
4. En la pantalla **Seleccionar orígenes**, cuadro de diálogo **Conectar a un servidor**, proporcione los detalles de conexión en su instancia de SQL Server y, luego, seleccione **Conectar**.
5. En el cuadro de diálogo **Agregar orígenes**, seleccione **Adventureworks2016**, luego **Agregar** y, por último, **Iniciar evaluación**.

    > [!NOTE]
    > Si usa SSIS, DMA no admite actualmente la valoración de la SSISDB de origen. De todas formas, los proyectos y paquetes SSIS se evaluarán/validarán a medida que se vuelven a implementar en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](./how-to-migrate-ssis-packages.md).

    Una vez completada la evaluación, los resultados se muestran tal y como se muestra en el siguiente gráfico:

    ![Evaluación de la migración de datos](media/tutorial-sql-server-to-azure-sql/dma-assessments.png)

    Para bases de datos de Azure SQL Database, las evaluaciones identifican problemas de paridad de características y problemas de bloqueo de migración para la implementación en una base de datos única o agrupada.

    - La categoría de **paridad de características de SQL Server** proporciona un conjunto completo de recomendaciones, alternativas disponibles en Azure y pasos de mitigación para ayudarlo a planear el trabajo en los proyectos de migración.
    - La categoría **Incidencias de compatibilidad** proporciona características no compatibles o parcialmente compatibles que reflejan los problemas de compatibilidad que podrían bloquear la migración de bases de datos de SQL Server a bases de datos de Azure SQL Database. También se proporcionan recomendaciones para ayudarlo a resolver esos problemas.

6. Revise los resultados de evaluación correspondientes a los problemas de bloqueo de migración y los de paridad de características seleccionando las opciones correspondientes.

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Cuando se haya familiarizado con la evaluación y piense que la base de datos seleccionada es una candidata viable para la migración a una base de datos única o agrupada de Azure SQL Database, use Data Migration Assistant para migrar el esquema a Azure SQL Database.

> [!NOTE]
> Antes de crear un proyecto de migración en Data Migration Assistant, asegúrese de que ya ha aprovisionado una base de datos en Azure, tal y como se mencionó en los requisitos previos. 

> [!IMPORTANT]
> Si usa SSIS, DMA no admite actualmente la migración de la SSISDB de origen, pero puede volver a implementar sus proyectos y paquetes SSIS en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](./how-to-migrate-ssis-packages.md).

Para migrar el esquema de **Adventureworks2016** a una base de datos única o agrupada de Azure SQL Database, siga estos pasos:

1. En Data Migration Assistant, seleccione el icono de Nuevo (+) y, en **Tipo de proyecto**, seleccione el tipo de proyecto **Migración**.
2. Especifique un nombre de proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server** y, luego, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **Azure SQL Database**.
3. En **Migration Scope** (Ámbito de la migración), seleccione **Solo esquema**.

    Después de realizar los pasos anteriores, la interfaz de Data Migration Assistant debe aparecer como se muestra en el siguiente gráfico:

    ![Creación de proyecto en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql/dma-create-project.png)

4. Seleccione **Crear** para crear el proyecto.
5. En Data Migration Assistant, especifique los detalles de conexión de origen de SQL Server, seleccione **Conectar** y, luego, elija la base de datos **Adventureworks2016**.

    ![Detalles de conexión de origen en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql/dma-source-connect.png)

6. Seleccione **Siguiente** en **Connect to target server** (Conectar a servidor de destino), especifique los detalles de conexión de destino de la base de datos de Azure SQL Database, seleccione **Conectar** y, luego, elija la base de datos **AdventureWorksAzure** que había aprovisionado previamente en Azure SQL Database.

    ![Detalles de conexión de destino en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql/dma-target-connect.png)

7. Seleccione **Siguiente** para avanzar a la pantalla **Seleccionar objetos**, en la que puede especificar los objetos de esquema de la base de datos **Adventureworks2016** que deben implementarse en Azure SQL Database.

    De forma predeterminada, se seleccionan todos los objetos.

    ![Generación de scripts SQL](media/tutorial-sql-server-to-azure-sql/dma-assessment-source.png)

8. Seleccione **Generar script SQL** para crear los scripts SQL y, luego, revíselos para ver si hay errores.

    ![Script de esquema](media/tutorial-sql-server-to-azure-sql/dma-schema-script.png)

9. Seleccione **Deploy schema** (Implementar esquema) para implementar el esquema en Azure SQL Database. Luego, una vez implementado, compruebe el servidor de destino por si hay cualquier anomalía.

    ![Implementación del esquema](media/tutorial-sql-server-to-azure-sql/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal. Busque y seleccione **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-sql-server-to-azure-sql/portal-select-subscription-1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-sql-server-to-azure-sql/portal-select-resource-provider.png)

3. Busque la migración y, después, seleccione **Registrar** para **Microsoft.DataMigration**.

    ![Registro del proveedor de recursos](media/tutorial-sql-server-to-azure-sql/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Creación de una instancia

1. En el menú de Azure Portal o en la página **principal**, seleccione **Crear un recurso**. Busque y seleccione **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-sql-server-to-azure-sql/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-create-1.png)
  
3. En la pantalla de aspectos básicos **Crear el servicio de migración**:

     - Seleccione la suscripción.
     - Cree un grupo de recursos o seleccione uno existente.
     - Especifique un nombre para la instancia de Azure Database Migration Service.
     - Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.
     - Elija **Azure** como modo de servicio.
     - Seleccione un plan de tarifa. Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de los valores básicos de la instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-settings-2.png)

     - Seleccione **Siguiente: Redes**.

4. En la pantalla de red **Crear el servicio de migración**:

    - Seleccione una red virtual existente o cree una nueva. La red virtual proporciona a Azure Database Migration Service acceso al servidor SQL Server de origen y a la instancia de Azure SQL Database de destino. Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](../virtual-network/quick-create-portal.md).

    ![Configuración de la red de la instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-settings-3.png)

    - Seleccione **Revisar y crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En el menú de Azure Portal, seleccione **Todos los servicios**. Busque y seleccione **Azure Database Migration Service**.

     ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, seleccione el nombre de la instancia de Azure Database Migration Service que creó.

3. Seleccione **Nuevo proyecto de migración**.

     ![Buscar una instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-instance-search.png)

4. En la pantalla **Nuevo proyecto de migración**, especifique el nombre del proyecto; en el cuadro de texto **Tipo de servidor de origen**, seleccione **SQL Server**; en el cuadro de texto **Tipo de servidor de destino**, seleccione **Azure SQL Database** y en **Elegir tipo de actividad de migración**, seleccione **Migración de datos**.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-sql-server-to-azure-sql/dms-create-project-2.png)

5. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Seleccionar origen**, especifique los detalles de conexión de la instancia de SQL Server de origen.

    Asegúrese de usar un nombre de dominio completo (FQDN) para el nombre de la instancia de SQL Server de origen. También puede usar la dirección IP en los casos en que no sea posible la resolución de nombres de DNS.

2. Si no ha instalado ningún certificado de confianza en el servidor de origen, seleccione la casilla de verificación **Certificado de servidor de confianza**.

    Si no hay ningún certificado de confianza instalado, SQL Server genera un certificado autofirmado cuando se inicia la instancia. Este certificado se usa para cifrar las credenciales de las conexiones del cliente.

    > [!CAUTION]
    > Las conexiones TLS cifradas con un certificado autofirmado no proporcionan una gran seguridad. Son susceptibles de sufrir ataques de tipo "Man in the middle". No debe confiar en TLS con certificados autofirmados en un entorno de producción, ni en servidores conectados a Internet.

    > [!IMPORTANT]
    > Si usa SSIS, DMS no admite actualmente la migración de la SSISDB de origen, pero puede volver a implementar sus proyectos y paquetes SSIS en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](./how-to-migrate-ssis-packages.md).

   ![Detalles del origen](media/tutorial-sql-server-to-azure-sql/dms-source-details-2.png)
   
3. Seleccione **Next: Select databases** (Siguiente: Seleccionar bases de datos).

## <a name="select-databases-for-migration"></a>Selección de bases de datos para la migración

Seleccione todas las bases de datos o las bases de datos específicas que desee migrar a Azure SQL Database. DMS proporciona el tiempo de migración esperado para las bases de datos seleccionadas. Si los tiempos de inactividad de la migración son aceptables, continúe con la migración. Si el tiempo de inactividad de la migración no es aceptable, considere la posibilidad de migrar a [SQL Managed Instance con tiempo de inactividad casi cero](tutorial-sql-server-managed-instance-online.md) o póngase en contacto con el [equipo de DMS](mailto:DMSFeedback@microsoft.com) para otras opciones. 

1. Elija las bases de datos que desea migrar de la lista de bases de datos disponibles. 
1. Revise el tiempo de inactividad esperado. Si es aceptable, seleccione **Next: Select target >>** (Siguiente: Seleccionar destino).

   ![Bases de datos de origen](media/tutorial-sql-server-to-azure-sql/select-database.png)



## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Select target** (Seleccionar destino), proporcione la configuración de autenticación a la instancia de Azure SQL Database. 

   ![Seleccionar destino](media/tutorial-sql-server-to-azure-sql/select-target.png)
   
   > [!NOTE]
   > Actualmente, la autenticación de SQL es el único sistema de autenticación compatible.

1. Seleccione **Siguiente: Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-sql-server-to-azure-sql/dms-map-targets-activity-2.png)

1. Seleccione **Siguiente: Configuración de valores de migración**, expanda la lista de tablas y revise la lista de campos afectados.

    Azure Database Migration Service selecciona automáticamente todas las tablas de origen vacías que existen en la base de datos de Azure SQL de destino. Si quiere volver a migrar tablas que ya contienen datos, deberá seleccionar de forma explícita las tablas de esta hoja.

    ![Selección de tablas](media/tutorial-sql-server-to-azure-sql/dms-configure-setting-activity-2.png)

1. Seleccione **Siguiente: Resumen**, examine la configuración de la migración y en el cuadro de texto **Nombre de actividad**, especifique el nombre de la actividad de migración.

    ![Selección de la opción de validación](media/tutorial-sql-server-to-azure-sql/dms-configuration-2.png)

## <a name="run-the-migration"></a>Ejecución de la migración

- Seleccione **Iniciar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Pendiente**.

    ![Estado de la actividad](media/tutorial-sql-server-to-azure-sql/dms-activity-status-1.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Completado**.

    ![Estado de actividad Completado](media/tutorial-sql-server-to-azure-sql/dms-completed-activity-1.png)

2. Compruebe las bases de datos de destino de la instancia de **Azure SQL Database** de destino.

### <a name="additional-resources"></a>Recursos adicionales

- Para más información acerca de Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
- Para más información sobre Azure SQL Database, consulte el artículo [¿Qué es el servicio Azure SQL Database?](../azure-sql/database/sql-database-paas-overview.md)
