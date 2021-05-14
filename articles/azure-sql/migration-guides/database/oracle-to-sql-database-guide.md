---
title: 'De Oracle a Azure SQL Database: guía de migración'
description: Con esta guía aprenderá a migrar su esquema de Oracle a Azure SQL Database mediante SQL Server Migration Assistant para Oracle.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: MashaMSFT
ms.author: mathoma
ms.date: 08/25/2020
ms.openlocfilehash: bf602cabaef92bfa9096ab3225f0f6d726617776
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136686"
---
# <a name="migration-guide-oracle-to-azure-sql-database"></a>Guía de migración: de Oracle a Azure SQL Database

[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqldb.md)]

Con esta guía [aprenderá a migrar](https://azure.microsoft.com/migration/migration-journey) sus esquemas de Oracle a Azure SQL Database mediante [SQL Server Migration](https://azure.microsoft.com/en-us/migration/sql-server/) Assistant para Oracle (SSMA para Oracle).

Para ver otras guías de migración, consulte la [Guía de Azure Database Migration](/data-migration).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a migrar el esquema de Oracle a SQL Database:

- Compruebe que el entorno de origen sea compatible.
- Descargue [SSMA para Oracle](https://www.microsoft.com/download/details.aspx?id=54258).
- Debe tener una instancia de [SQL Database](../../database/single-database-create-quickstart.md) de destino.
- Obtenga los [permisos necesarios de SSMA para Oracle](/sql/ssma/oracle/connecting-to-oracle-database-oracletosql) y el [proveedor](/sql/ssma/oracle/connect-to-oracle-oracletosql).
 
## <a name="pre-migration"></a>Antes de la migración

Una vez cumplidos los requisitos previos, estará listo para detectar la topología del entorno y evaluar la viabilidad de la [migración a la nube de Azure](https://azure.microsoft.com/migration). Esta parte del proceso implica la elaboración de un inventario de las bases de datos que se deben migrar, la evaluación de las bases de datos en busca de posibles problemas de migración o bloqueadores y la resolución de cualquier aspecto que pueda haber descubierto.

### <a name="assess"></a>Evaluar

Al usar SSMA para Oracle, puede revisar los objetos y datos de base de datos, evaluar las bases de datos para la migración, migrar los objetos de base de datos a SQL Database y, por último, migrar los datos a la base de datos.

Para crear una evaluación:

1. Abra [SSMA para Oracle](https://www.microsoft.com/download/details.aspx?id=54258).
1. Seleccione **Archivo** y, a continuación, seleccione **Nuevo proyecto**.
1. Escriba el nombre del proyecto y elija una ubicación para guardar el proyecto. Después, seleccione **Azure SQL Database** como destino de la migración en la lista desplegable y seleccione **Aceptar**.

   ![Captura de pantalla que muestra el cuadro de diálogo Conectarse a Oracle.](./media/oracle-to-sql-database-guide/connect-to-oracle.png)

1. Seleccione **Connect to Oracle** (Conectarse a Oracle). Escriba los valores de los detalles de la conexión de Oracle en el cuadro de diálogo **Conectarse a Oracle**.

1. Seleccione los esquemas de Oracle que quiera migrar.

   ![Captura de pantalla que muestra la selección de esquema de Oracle.](./media/oracle-to-sql-database-guide/select-schema.png)

1. En **Explorador de metadatos de Oracle**, haga clic con el botón derecho en el esquema de Oracle que quiere migrar y seleccione **Crear informe** para generar un informe HTML. También puede seleccionar una base de datos y luego la pestaña **Crear informe**.

   ![Captura de pantalla que muestra la opción Crear informe.](./media/oracle-to-sql-database-guide/create-report.png)

1. Revise el informe en HTML para comprender las estadísticas de conversión, y los errores o advertencias. También puede abrir el informe en Excel para obtener un inventario de objetos de Oracle y conocer el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes dentro de SSMAProjects.

   Por ejemplo, vea `drive:\<username>\Documents\SSMAProjects\MyOracleMigration\report\report_2020_11_12T02_47_55\`.

   ![Captura de pantalla que muestra un informe de valoración.](./media/oracle-to-sql-database-guide/assessment-report.png)

### <a name="validate-the-data-types"></a>Validación de los tipos de datos

Valide las asignaciones predeterminadas de los tipos de datos y cámbielas según los requisitos, si es necesario. Para hacerlo, siga estos pasos:

1. En SSMA para Oracle, seleccione **Herramientas** y luego **Configuración del proyecto**.
1. Seleccione la pestaña **Asignación de tipo**.

   ![Captura de pantalla que muestra la pestaña Asignación de tipo.](./media/oracle-to-sql-database-guide/type-mappings.png)

1. Puede cambiar la asignación de tipo de cada tabla si selecciona la tabla en **Explorador de metadatos de Oracle**.

### <a name="convert-the-schema"></a>Conversión del esquema

Para convertir el esquema:

1. (Opcional) Agregue consultas dinámicas o ad hoc a las instrucciones. Haga clic con el botón derecho en el nodo y seleccione **Agregar instrucciones**.
1. Seleccione la pestaña **Conectar a Azure SQL Database**.
    1. En **SQL Database**, escriba los detalles de la conexión para conectar la base de datos.
    1. Seleccione la instancia de destino de SQL Database en la lista desplegable o escriba un nombre nuevo, en cuyo caso se creará una base de datos en el servidor de destino.
    1. Escriba los detalles de autenticación y seleccione **Conectar**.

    ![Captura de pantalla que muestra la pestaña Conectarse a Azure SQL Database.](./media/oracle-to-sql-database-guide/connect-to-sql-database.png)

1. En **Explorador de metadatos de Oracle**, haga clic con el botón derecho en el esquema de Oracle y seleccione **Convertir esquema**. También puede seleccionar el esquema y luego la pestaña **Convertir esquema**.

   ![Captura de pantalla que muestra la pestaña Convertir esquema.](./media/oracle-to-sql-database-guide/convert-schema.png)

1. Una vez finalizada la conversión, compare y revise los objetos convertidos con los objetos originales para identificar posibles problemas y solucionarlos en función de las recomendaciones:

   ![Captura de pantalla que muestra el esquema de revisión de las recomendaciones.](./media/oracle-to-sql-database-guide/table-mapping.png)

1. Compare el texto de Transact-SQL convertido con el procedimiento almacenado original y revise las recomendaciones.

   ![Captura de pantalla que muestra la opción Revisar recomendaciones.](./media/oracle-to-sql-database-guide/procedure-comparison.png)

1. En el panel de resultados, seleccione **Revisar resultados** y revise los errores en el panel **Lista de errores**.
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. En el menú **Archivo**, seleccione **Guardar proyecto**. Este paso le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y realizar correcciones antes de publicar el esquema en SQL Database.

## <a name="migrate"></a>Migrar

Después de evaluar las bases de datos y de solucionar las discrepancias, el siguiente paso consiste en ejecutar el proceso de migración. La migración conlleva dos pasos: publicar el esquema y migrar los datos.

Para publicar el esquema y migrar los datos, haga lo siguiente:

1. Para publicar el esquema, haga clic con el botón derecho en la base de datos desde el nodo **Bases de datos** del **Explorador de metadatos de Azure SQL Database** y seleccione **Sincronizar con la base de datos**.

   ![Captura de pantalla que muestra la pestaña Sincronizar con la base de datos.](./media/oracle-to-sql-database-guide/synchronize-with-database.png)

1. Revise la asignación entre el proyecto de origen y el de destino.

   ![Captura de pantalla que muestra la revisión de Sincronizar con base de datos.](./media/oracle-to-sql-database-guide/synchronize-with-database-review.png)

1. Para migrar los datos, haga clic con el botón derecho en la base de datos o el objeto que quiera migrar en el **Explorador de metadatos de Oracle** y seleccione **Migrar datos**. También puede seleccionar la pestaña **Migrar datos**. Para migrar datos de una base de datos completa, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active las casillas que hay junto a las tablas. Para omitir datos de tablas concretas, desactive las casillas.

   ![Captura de pantalla que muestra la pestaña Migrar datos.](./media/oracle-to-sql-database-guide/migrate-data.png)

1. Especifique los detalles de la conexión para las instancias de Oracle y SQL Database.
1. Una vez completada la migración, vea el **Informe de migración de datos**.

   ![Captura de pantalla que muestra el Informe de migración de datos.](./media/oracle-to-sql-database-guide/data-migration-report.png)

1. Conéctese a la instancia de SQL Database mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) y revise los datos y el esquema para validar la migración.

   ![Captura de pantalla en la que se muestra SQL Server Management Studio.](./media/oracle-to-sql-database-guide/validate-data.png)

También puede usar SQL Server Integration Services para realizar la migración. Para obtener más información, consulte:

- [Introducción a SQL Server Integration Services](/sql/integration-services/sql-server-integration-services)
- [SQL Server Integration Services para Azure y movimiento de datos híbridos](https://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/SSIS%20Hybrid%20and%20Azure.docx)

## <a name="post-migration"></a>Etapa posterior a la migración

Cuando haya completado correctamente la fase de *migración*, deberá realizar una serie de tareas posteriores para asegurarse de que todo funcione de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. En algunos casos, completar esta tarea requerirá la realización de cambios en las aplicaciones.

[Data Access Migration Toolkit](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit) es una extensión para Visual Studio Code que permite analizar el código fuente de Java y detectar llamadas y consultas de la API de acceso a datos. El kit de herramientas proporciona una vista de un solo panel de lo que se debe solucionar para admitir el nuevo back-end de base de datos. Para obtener más información, consulte la entrada de blog [Migración de la aplicación Java desde Oracle](https://techcommunity.microsoft.com/t5/microsoft-data-migration/migrate-your-java-applications-from-oracle-to-sql-server-with/ba-p/368727).

### <a name="perform-tests"></a>Realización de pruebas

El enfoque de prueba para la migración de bases de datos consta de las siguientes actividades:

1. **Desarrollar pruebas de validación**: para probar la migración de bases de datos, debe utilizar consultas SQL. Debe crear las consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.
1. **Configurar un entorno de prueba**: el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.
1. **Ejecutar pruebas de validación**: ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.
1. **Ejecutar pruebas de rendimiento**: ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.

### <a name="optimize"></a>Optimización

La fase posterior a la migración es fundamental para reconciliar cualquier problema de precisión de datos y comprobar su integridad, así como para solucionar problemas de rendimiento con la carga de trabajo.

> [!NOTE]
> Para obtener más información sobre estos problemas y los pasos para mitigarlos, consulte la [Guía de optimización y validación posterior a la migración](/sql/relational-databases/post-migration-validation-and-optimization-guide).

## <a name="migration-assets"></a>Recursos de migración

Para obtener más ayuda para completar este escenario de migración, consulte los siguientes recursos. Se han desarrollado para ayudar en la interacción con un proyecto de migración real.

| **Título/vínculo**                                                                                                                                          | **Descripción**                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | Esta herramienta proporciona sugerencias de plataformas de destino "ideales", preparación para la nube y el nivel de corrección de la aplicación o base de datos para una carga de trabajo determinada. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado.                                                          |
| [Artefactos de script de inventario de Oracle](https://github.com/Microsoft/DataMigrationTeam/tree/master/Oracle%20Inventory%20Script%20Artifacts)                 | Este recurso incluye una consulta PL/SQL que visita las tablas del sistema de Oracle y proporciona un recuento de objetos por tipo de esquema, tipo de objeto y estado. También proporciona una estimación aproximada de datos sin procesar en cada esquema y del tamaño de las tablas de cada esquema; almacena los resultados en formato CSV.                                                                                                               |
| [Automatización de la recopilación y consolidación de evaluaciones de SSMA para Oracle](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Automate%20SSMA%20Oracle%20Assessment%20Collection%20%26%20Consolidation)                                             | Este conjunto de recursos usa un archivo .csv como entrada (sources.csv en las carpetas del proyecto) para generar los archivos XML necesarios para ejecutar una evaluación de SSMA en modo de consola. El cliente proporciona el archivo source.csv basado en un inventario de las instancias de Oracle existentes. Los archivos de salida son AssessmentReportGeneration_source_1.xml, ServersConnectionFile.xml y VariableValueFile.xml.|
| [Errores comunes de SSMA para Oracle y procedimiento para corregirlos](https://aka.ms/dmj-wp-ssma-oracle-errors)                                                           | Con Oracle, puede asignar una condición no escalar en la cláusula WHERE. Sin embargo, SQL Server no admite este tipo de condición. Como resultado, SSMA para Oracle no convierte las consultas que tienen una condición no escalar en la cláusula WHERE, sino que genera el error O2SS0001. Estas notas del producto proporcionan más detalles sobre el problema y las maneras de resolverlo.          |
| [Manual de migración de Oracle a SQL Server](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20SQL%20Server%20Migration%20Handbook.pdf)                | Este documento se centra en las tareas asociadas a la migración de un esquema de Oracle a la versión más reciente de SQL Server Database. Si la migración requiere cambios en las características o funcionalidades, se debe tener muy en cuenta el posible impacto de cada cambio en las aplicaciones que usan la base de datos.                                                     |

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md).

- Para más información sobre SQL Database, consulte:
  - [¿Qué es Azure SQL Database?](../../database/sql-database-paas-overview.md)
  - [Calculadora del costo total de propiedad (TCO) de Azure](https://azure.microsoft.com/pricing/tco/calculator/)

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs)
   -  [Recursos de migración a la nube](https://azure.microsoft.com/migration/resources)

- En cuanto al contenido de vídeo, consulte:
    - [Información general sobre el recorrido de la migración y las herramientas y servicios recomendados para llevar a cabo la evaluación y la migración](https://azure.microsoft.com/resources/videos/overview-of-migration-and-recommended-tools-services/).