---
title: 'Oracle a SQL Server en Azure Virtual Machines: guía de migración'
description: En esta guía se explica cómo migrar los esquemas de Oracle a SQL Server en Azure Virtual Machines mediante SQL Server Migration Assistant para Oracle.
ms.service: virtual-machines-sql
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 2133f3da926dfd1e92d911004393893f58d27db6
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136460"
---
# <a name="migration-guide-oracle-to-sql-server-on-azure-virtual-machines"></a>Guía de migración: Oracle a SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqldb.md)]

En esta guía se explica cómo migrar los esquemas de Oracle a SQL Server en Azure Virtual Machines mediante SQL Server Migration Assistant para Oracle. 

Para ver otras guías de migración, consulte [Migración de bases de datos](/data-migration). 

## <a name="prerequisites"></a>Requisitos previos 

Para migrar el esquema de Oracle a SQL Server en Azure Virtual Machines, necesita:

- Un entorno de origen compatible.
- [SQL Server Migration Assistant (SSMA) para Oracle](https://www.microsoft.com/en-us/download/details.aspx?id=54258).
- Una [VM con SQL Server](../../virtual-machines/windows/sql-vm-create-portal-quickstart.md) de destino.
- Los [permisos necesarios para SSMA para Oracle](/sql/ssma/oracle/connecting-to-oracle-database-oracletosql) y el [proveedor](/sql/ssma/oracle/connect-to-oracle-oracletosql).
- Conectividad y permisos suficientes para acceder al origen y al destino. 


## <a name="pre-migration"></a>Antes de la migración

Para preparar la migración a la nube, compruebe que el entorno de origen es compatible y que cumple los requisitos previos. Esto le va a ayudar a garantizar una migración correcta y eficaz.

Esta parte del proceso conlleva: 
- Realizar un inventario de las bases de datos que se necesita migrar.
- Evaluar esas bases de datos para detectar posibles problemas u obstáculos de migración. 
- Resolver los problemas detectados. 

### <a name="discover"></a>Descubra

Use [MAP Toolkit](https://go.microsoft.com/fwlink/?LinkID=316883) para identificar orígenes de datos existentes y detalles sobre las características que usa la empresa. Esto le va a ayudar a entender mejor la migración y a planearla. Este proceso conlleva examinar la red para identificar las instancias de Oracle de la organización y las versiones y las características que se usan.

Para usar MAP Toolkit para hacer un examen de inventario, siga estos pasos: 


1. Abra [MAP Toolkit](https://go.microsoft.com/fwlink/?LinkID=316883).


1. Seleccione **Create/Select database** (Crear o seleccionar base de datos):

   ![Captura de pantalla que muestra la opción para crear o seleccionar una base de datos.](./media/oracle-to-sql-on-azure-vm-guide/select-database.png)

1. Seleccione **Create an inventory database** (Crear una base de datos de inventario). Escriba un nombre para la nueva base de datos de inventario que está creando, proporcione una breve descripción y seleccione **OK** (Aceptar): 

   :::image type="content" source="media/oracle-to-sql-on-azure-vm-guide/create-inventory-database.png" alt-text="Captura de pantalla que muestra la interfaz para crear una base de datos de inventario.":::

1. Seleccione **Collect inventory data** (Recopilar datos de inventario) para abrir **Inventory and Assessment Wizard** (Asistente para el inventario y evaluación):

   :::image type="content" source="media/oracle-to-sql-on-azure-vm-guide/collect-inventory-data.png" alt-text="Captura de pantalla que muestra el vínculo para recopilar datos de inventario.":::


1. En **Inventory and Assessment Wizard** (Asistente para el inventario y la evaluación), seleccione **Oracle** y luego **Next** (Siguiente):

   ![Captura de pantalla que muestra la página de escenarios de inventario del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/choose-oracle.png)

1. Seleccione la opción de búsqueda de equipos que mejor se adapte a su entorno y a sus necesidades empresariales y luego seleccione **Next** (Siguiente): 

   ![Captura de pantalla que muestra la página de métodos de detección del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/choose-search-option.png)

1. Escriba las credenciales o cree unas nuevas para los sistemas que quiera explorar y, a continuación, seleccione **Next** (Siguiente):

    ![Captura de pantalla que muestra la página de credenciales de todos los equipos del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/choose-credentials.png)


1. Establezca el orden de las credenciales y, a continuación, seleccione **Next** (Siguiente): 

   ![Captura de pantalla que muestra la página de orden de credenciales del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/set-credential-order.png)  


1. Especifique las credenciales de cada equipo que quiera detectar. Puede usar unas credenciales únicas para cada equipo o máquina o bien la lista de credenciales de todos los equipos.  

   ![Captura de pantalla que muestra la página de especificación de equipos y credenciales del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/specify-credentials-for-each-computer.png)


1. Compruebe las selecciones y seleccione **Finish** (Finalizar):

   ![Captura de pantalla que muestra la página de resumen del asistente para el inventario y la evaluación.](./media/oracle-to-sql-on-azure-vm-guide/review-summary.png)


1. Una vez completado el examen, vea el resumen **Data Collection** (Recopilación de datos). El examen puede tardar unos minutos, en función del número de bases de datos. Seleccione **Close** (Cerrar) cuando haya terminado: 

   ![Captura de pantalla que muestra el resumen de recopilación de datos.](./media/oracle-to-sql-on-azure-vm-guide/collection-summary-report.png)


1. Seleccione **Options** (Opciones) para generar un informe sobre la evaluación de Oracle y los detalles de la base de datos. Seleccione ambas opciones, una a una, para generar el informe.


### <a name="assess"></a>Evaluar

Después de identificar los orígenes de datos, use [SQL Server Migration Assistant para Oracle](https://www.microsoft.com/en-us/download/details.aspx?id=54258) para evaluar las instancias de Oracle que se van a migrar a la VM con SQL Server. El asistente le ayuda a entender las diferencias entre las bases de datos de origen y de destino. Puede revisar los datos y los objetos de base de datos, evaluar las bases de datos para la migración, migrar objetos de base de datos a SQL Server y luego migrar datos a SQL Server.

Para crear una evaluación, siga estos pasos: 


1. Abra [SQL Server Migration Assistant para Oracle](https://www.microsoft.com/en-us/download/details.aspx?id=54258). 
1. En el menú **Archivo**, seleccione **Nuevo proyecto**. 
1. Facilite un nombre de proyecto y una ubicación para el proyecto y seleccione un destino de migración de SQL Server en la lista. Seleccione **OK** (Aceptar): 

   ![Captura de pantalla que muestra el cuadro de diálogo Nuevo proyecto.](./media/oracle-to-sql-on-azure-vm-guide/new-project.png)


1. Seleccione **Connect to Oracle** (Conectarse a Oracle). Escriba los valores de la conexión de Oracle en el cuadro de diálogo **Conectarse a Oracle**:

   ![Captura de pantalla que muestra el cuadro de diálogo Conectarse a Oracle.](./media/oracle-to-sql-on-azure-vm-guide/connect-to-oracle.png)

   Seleccione los esquemas de Oracle que quiere migrar: 

   ![Captura de pantalla que muestra la lista de esquemas de Oracle que se pueden migrar.](./media/oracle-to-sql-on-azure-vm-guide/select-schema.png)


1. En **Explorador de metadatos de Oracle**, haga clic con el botón derecho en el esquema de Oracle que quiere migrar y seleccione **Crear informe**. Se genera un informe HTML. También puede seleccionar la base de datos y luego **Crear informe** en el menú superior.

   ![Captura de pantalla que muestra cómo crear un informe.](./media/oracle-to-sql-on-azure-vm-guide/create-report.png)

1. Revise el informe HTML para ver las estadísticas de conversión, los errores y las advertencias. Analícelo para entender los problemas de conversión y las soluciones.

    También puede abrir el informe en Excel para obtener un inventario de objetos de Oracle y conocer el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes de SSMAProjects. 

   Por ejemplo: `drive:\<username>\Documents\SSMAProjects\MyOracleMigration\report\report_2016_11_12T02_47_55\`


   ![Captura de pantalla que muestra un informe de conversión.](./media/oracle-to-sql-on-azure-vm-guide/conversion-report.png)


### <a name="validate-data-types"></a>Validación de los tipos de datos

Valide las asignaciones predeterminadas de los tipos de datos y cámbielas según los requisitos, si fuera necesario. Para hacerlo, siga estos pasos: 


1. En el menú **Herramientas**, seleccione **Configuración del proyecto**. 
1. Seleccione la pestaña **Asignación de tipo**. 

   ![Captura de pantalla que muestra la pestaña Asignación de tipo.](./media/oracle-to-sql-on-azure-vm-guide/type-mappings.png)

1. Puede cambiar la asignación de tipo de cada tabla si selecciona la tabla en **Explorador de metadatos de Oracle**. 

### <a name="convert-the-schema"></a>Conversión del esquema

Para convertir el esquema, siga estos pasos: 

1. (Opcional) Para convertir consultas dinámicas o ad hoc, haga clic con el botón derecho en el nodo y seleccione **Agregar instrucción**.

1. Seleccione **Conectarse a SQL Server** en el menú superior. 
     1. Escriba los detalles de la conexión de SQL Server en Azure VM. 
     1. Seleccione la base de datos de destino en la lista o proporcione un nombre nuevo. Si facilita un nombre nuevo, se crea una base de datos en el servidor de destino. 
     1. Proporcione los detalles de la autenticación. 
     1. Seleccione **Conectar**. 


   ![Captura de pantalla que muestra cómo conectarse a SQL Server.](./media/oracle-to-sql-on-azure-vm-guide/connect-to-sql-vm.png)

1. Haga clic con el botón derecho en el esquema de Oracle en **Explorador de metadatos de Oracle** y seleccione **Convertir esquema**. También puede seleccionar **Convertir esquema** en el menú superior:

   ![Captura de pantalla que muestra cómo convertir el esquema.](./media/oracle-to-sql-on-azure-vm-guide/convert-schema.png)


1. Una vez completada la conversión del esquema, revise los objetos convertidos y compárelos con los originales para identificar posibles problemas. Use las recomendaciones para solucionar cualquier problema:

   ![Captura de pantalla que muestra una comparación de dos esquemas.](./media/oracle-to-sql-on-azure-vm-guide/table-mapping.png)

   Compare el texto de Transact-SQL convertido al código original y revise las recomendaciones: 

   ![Captura de pantalla que muestra Transact-SQL, procedimientos almacenados y una advertencia.](./media/oracle-to-sql-on-azure-vm-guide/procedure-comparison.png)

   Puede guardar el proyecto localmente para un ejercicio de corrección de esquema sin conexión. Para ello, seleccione **Guardar proyecto** en el menú **Archivo**. Guardar el proyecto en local le permite evaluar los esquemas de origen y de destino sin conexión y hacer correcciones antes de publicar el esquema en SQL Server.

1. Seleccione **Revisar resultados** en el panel **Salida** y revise los errores en el panel **Lista de errores**. 
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. Seleccione **Guardar proyecto** en el menú **Archivo**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión y de hacer correcciones antes de publicar el esquema en SQL Server en Azure Virtual Machines.


## <a name="migrate"></a>Migrar

Una vez aplicados los requisitos previos necesarios y completadas las tareas asociadas a la fase de premigración, está listo para iniciar la migración del esquema y los datos. La migración conlleva dos pasos: publicar el esquema y migrar los datos. 


Para publicar el esquema y migrar los datos, siga estos pasos: 

1. Publique el esquema: haga clic con el botón derecho en la base de datos en **Explorador de metadatos de SQL Server** y seleccione **Sincronizar con base de datos**. Al hacerlo, se publica el esquema de Oracle en SQL Server en Azure Virtual Machines. 

   ![Captura de pantalla que muestra el comando Sincronizar con base de datos.](./media/oracle-to-sql-on-azure-vm-guide/synchronize-database.png)

   Revise la asignación entre el proyecto de origen y el de destino:

   ![Captura de pantalla que muestra el estado de sincronización.](./media/oracle-to-sql-on-azure-vm-guide/synchronize-database-review.png)



1. Migre los datos: haga clic con el botón derecho en la base de datos o el objeto que quiere migrar en **Explorador de metadatos de Oracle** y seleccione **Migrar datos**. También puede seleccionar **Migrar datos** en el menú superior.

   Para migrar datos de toda una base de datos, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active la casilla situada junto a la tabla. Para omitir datos de tablas individuales, desactive las casillas en cuestión.

   ![Captura de pantalla que muestra el comando Migrar datos.](./media/oracle-to-sql-on-azure-vm-guide/migrate-data.png)

1. Proporcione los detalles de conexión de Oracle y SQL Server en Azure Virtual Machines en el cuadro de diálogo.
1. Una vez terminada la migración, vea el **Informe de migración de datos**:

    ![Captura de pantalla que muestra el Informe de migración de datos.](./media/oracle-to-sql-on-azure-vm-guide/data-migration-report.png)

1. Conéctese a la instancia de SQL Server en Azure Virtual Machines mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms). Para validar la migración, revise los datos y el esquema:


   ![Captura de pantalla que muestra una instancia de SQL Server en SSMA.](./media/oracle-to-sql-on-azure-vm-guide/validate-in-ssms.png)

En lugar de usar SSMA, podría utilizar SQL Server Integration Services (SSIS) para migrar los datos. Para obtener más información, consulte: 
- El artículo [SQL Server Integration Services](//sql/integration-services/sql-server-integration-services).
- Las notas del producto [SSIS para la migración de datos híbrida y de Azure](https://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/SSIS%20Hybrid%20and%20Azure.docx).


## <a name="post-migration"></a>Etapa posterior a la migración 

Cuando haya completado la fase de migración, debe realizar una serie de tareas postmigración para asegurarse de que todo funcione de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a usar el destino. Esos cambios pueden exigir modificaciones en las aplicaciones.

[Data Access Migration Toolkit](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit) es una extensión para Visual Studio Code. Permite analizar el código fuente de Java y detectar llamadas API y consultas de acceso a datos. El kit de herramientas proporciona una vista de un solo panel de lo que se debe solucionar para admitir el nuevo back-end de base de datos. Para obtener más información, vea [Migración de la aplicación Java desde Oracle](https://techcommunity.microsoft.com/t5/microsoft-data-migration/migrate-your-java-applications-from-oracle-to-sql-server-with/ba-p/368727). 

### <a name="perform-tests"></a>Realización de pruebas

Para probar la migración de la base de datos, realice estas actividades:

1. **Desarrollar pruebas de validación**. para probar la migración de bases de datos, debe utilizar consultas SQL. Cree las consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.

2. **Configure un entorno de prueba**. el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.

3. **Ejecutar pruebas de validación**. ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.

4. **Ejecutar pruebas de rendimiento**. ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.

### <a name="optimize"></a>Optimización

La fase de postmigración es vital para resolver problemas de precisión de los datos y para comprobar su integridad. También es fundamental para solucionar problemas de rendimiento de la carga de trabajo.

> [!Note]
> Para obtener más información sobre estos problemas y los pasos concretos para mitigarlos, vea la [Guía de optimización y validación posterior a la migración](/sql/relational-databases/post-migration-validation-and-optimization-guide).


## <a name="migration-resources"></a>Recursos de migración 

Para obtener más ayuda a fin de completar este escenario de migración, vea los siguientes recursos, desarrollados como apoyo en un proyecto de migración real.

| **Título o vínculo**                                                                                                                                          | **Descripción**                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | Esta herramienta proporciona sugerencias de plataformas de destino ideales, preparación para la nube y niveles de corrección de aplicación o base de datos para una carga de trabajo determinada. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado.                                                          |
| [Artefactos de script de inventario de Oracle](https://github.com/Microsoft/DataMigrationTeam/tree/master/Oracle%20Inventory%20Script%20Artifacts)                 | Este recurso incluye una consulta PL/SQL destinada a las tablas del sistema de Oracle y proporciona un recuento de objetos por tipo de esquema, tipo de objeto y estado. También proporciona una estimación aproximada de datos sin procesar en cada esquema y del tamaño de las tablas de cada esquema; almacena los resultados en formato CSV.                                                                                                               |
| [Automatización de la recopilación y consolidación de evaluaciones de SSMA para Oracle](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Automate%20SSMA%20Oracle%20Assessment%20Collection%20%26%20Consolidation)                                             | Este conjunto de recursos usa un archivo .csv como entrada (sources.csv en las carpetas del proyecto) para generar los archivos XML necesarios para ejecutar la evaluación de SSMA en modo de consola. El archivo source.csv se proporciona al realizar un inventario de las instancias de Oracle existentes. Los archivos de salida son AssessmentReportGeneration_source_1.xml, ServersConnectionFile.xml y VariableValueFile.xml.|
| [Problemas de SSMA y posibles soluciones al migrar bases de datos de Oracle](https://aka.ms/dmj-wp-ssma-oracle-errors)                                                           | Con Oracle, puede asignar una condición no escalar en una cláusula WHERE. SQL Server no admite este tipo de condición. Por tanto, SSMA para Oracle no convierte las consultas que tienen una condición no escalar en la cláusula WHERE, sino que genera un error: O2SS0001. Estas notas del producto proporcionan detalles sobre el problema y maneras de resolverlo.          |
| [Manual de migración de Oracle a SQL Server](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20SQL%20Server%20Migration%20Handbook.pdf)                | Este documento se centra en las tareas asociadas a la migración de un esquema de Oracle a la versión más reciente de SQL Server. Si la migración requiere cambios en las características o la funcionalidad, se debe considerar cuidadosamente el posible efecto de cada cambio en las aplicaciones que usan la base de datos.                                                     |


El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es desbloquear y acelerar la modernización compleja de proyectos de migración de una plataforma de datos a la plataforma de datos de Microsoft Azure.


## <a name="next-steps"></a>Pasos siguientes

- Para comprobar la disponibilidad de los servicios aplicables a SQL Server, visite el [centro de infraestructura global de Azure](https://azure.microsoft.com/global-infrastructure/services/?regions=all&amp;products=synapse-analytics,virtual-machines,sql-database).

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de terceros que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos y tareas especializadas, vea [Servicios y herramientas para la migración de datos](../../../dms/dms-tools-matrix.md).

- Para obtener más información sobre Azure SQL, vea:
   - [Opciones de implementación](../../azure-sql-iaas-vs-paas-what-is-overview.md)
   - [SQL Server en Azure Virtual Machines](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
   - [Calculadora del costo total de propiedad de Azure](https://azure.microsoft.com/pricing/tco/calculator/)


- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Para obtener información sobre licencias, vea:
   - [Enfoque "traiga su propia licencia" con la Ventaja híbrida de Azure](../../virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change.md)
   - [Obtención de soporte extendido gratuito de SQL Server 2008 y SQL Server 2008 R2](../../virtual-machines/windows/sql-server-2008-extend-end-of-support.md)

- Para evaluar la capa de acceso de la aplicación, use [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Para obtener detalles sobre cómo realizar pruebas A/B de capa de acceso a datos, vea [Información general de Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).