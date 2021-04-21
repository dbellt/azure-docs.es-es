---
title: 'De MySQL a Azure SQL Database: guía de migración'
description: En esta guía obtendrá información sobre cómo migrar las bases de datos de MySQL a Azure SQL Database mediante SQL Server Migration Assistant para MySQL (SSMA para MySQL).
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: MashaMSFT
ms.author: mathoma
ms.date: 03/19/2021
ms.openlocfilehash: d4510aa5cda61dac88102c89b3e03da231380bd6
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107389458"
---
# <a name="migration-guide-mysql-to-azure-sql-database"></a>Guía de migración: de MySQL a Azure SQL Database
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqldb.md)]

En esta guía obtendrá información sobre [cómo migrar](https://azure.microsoft.com/migration/migration-journey) una base de datos de MySQL a otra de Azure SQL mediante [SQL Server Migration Assistant](https://azure.microsoft.com/en-us/migration/sql-server/) para MySQL (SSMA para MySQL). 

Para ver otras guías de migración, consulte [Guía de Azure Database Migration](https://docs.microsoft.com/data-migration). 

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a migrar la base de datos de MySQL a una base de datos SQL, haga lo siguiente:

- Compruebe que el entorno de origen es compatible. Actualmente, se admiten MySQL 5.6 y 5.7. 
- Descargue e instale [SQL Server Migration Assistant para MySQL](https://www.microsoft.com/download/details.aspx?id=54257).
- Asegúrese de que tiene conectividad y permisos suficientes para acceder tanto al origen como al destino.

## <a name="pre-migration"></a>Antes de la migración 

Una vez cumplidos los requisitos previos, estará listo para detectar la topología del entorno y evaluar la viabilidad de la [migración a la nube de Azure](https://azure.microsoft.com/migration).

### <a name="assess"></a>Evaluar 

Use SQL Server Migration Assistant (SSMA) para MySQL con el fin de revisar datos y objetos de bases de datos, y evaluar estas últimas para la migración. 

Para crear una evaluación, siga estos pasos:

1. Abra [SSMA para MySQL](https://www.microsoft.com/download/details.aspx?id=54257). 
1. Seleccione **File** (Archivo) y, a continuación, seleccione **New Project** (Nuevo proyecto). 
1. En el panel **Nuevo proyecto**, escriba un nombre y una ubicación para el proyecto y, a continuación, en la lista desplegable **Migrar a**, seleccione **Azure SQL Database**. 
1. Seleccione **Aceptar**.

   ![Captura de pantalla del panel "New Project" (Nuevo proyecto) para escribir el nombre del proyecto, la ubicación y el destino de la migración.](./media/mysql-to-sql-database-guide/new-project.png)

1. Seleccione la pestaña **Connect to MySQL** (Conectar a MySQL) y proporcione los detalles para conectarse al servidor MySQL. 

   ![Captura de pantalla del panel "Connect to MySQL" (Conectar a MySQL) para especificar las conexiones con el origen.](./media/mysql-to-sql-database-guide/connect-to-mysql.png)

1. En el panel **MySQL Metadata Explorer** (Explorador de metadatos de MySQL), haga clic con el botón derecho en el esquema de MySQL y seleccione **Create Report** (Crear informe). Como alternativa, puede seleccionar la pestaña **Create Report** (Crear informe) en la esquina superior derecha.

   ![Captura de pantalla de los vínculos "Create Report" (Crear informe) en SSMA para MySQL.](./media/mysql-to-sql-database-guide/create-report.png)

1. Revise el informe HTML para comprender las estadísticas de conversión, los errores y las advertencias. Analícelo para comprender los problemas de conversión y las soluciones. 
   También puede abrir el informe en Excel para obtener un inventario de objetos de MySQL y conocer el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes dentro de SSMAProjects. Por ejemplo: 
   
   `drive:\Users\<username>\Documents\SSMAProjects\MySQLMigration\report\report_2016_11_12T02_47_55\`
 
   ![Captura de pantalla de un informe de conversión de ejemplo en SSMA.](./media/mysql-to-sql-database-guide/conversion-report.png)

### <a name="validate-the-data-types"></a>Validación de los tipos de datos

Valide las asignaciones de tipos de datos predeterminadas y cámbielas según los requisitos, si es necesario. Para ello: 

1. Seleccione **Tools** (Herramientas) y, luego, **Project Settings** (Configuración del proyecto).  
1. Seleccione la pestaña **Type mappings** (Asignaciones de tipos). 

   ![Captura de pantalla del panel "Type Mappings" (Asignaciones de tipos) de SSMA para MySQL.](./media/mysql-to-sql-database-guide/type-mappings.png)

1. Puede cambiar la asignación de tipos de cada tabla; para ello, seleccione el nombre de la tabla en el panel **MySQL Metadata Explorer** (Explorador de metadatos de MySQL). 

### <a name="convert-the-schema"></a>Conversión del esquema 

Para convertir el esquema, haga lo siguiente: 

1. (Opcional) Para convertir consultas dinámicas o especializadas, haga clic con el botón derecho en el nodo y seleccione **Add statement** (Agregar instrucción). 

1. Seleccione la pestaña **Connect to Azure SQL Database** (Conectar a Azure SQL Database) y haga lo siguiente:

   a. Escriba los detalles para conectarse a la base de datos SQL.  
   b. Seleccione la base de datos SQL de destino en la lista desplegable. También puede proporcionar un nombre nuevo, en cuyo caso se crea una base de datos en el servidor de destino.  
   c. Proporcione los detalles de la autenticación.  
   d. Seleccione **Conectar**.

   ![Captura de pantalla del panel "Connect to Azure SQL Database" (Conectar a Azure SQL Database) en SSMA para MySQL.](./media/mysql-to-sql-database-guide/connect-to-sqldb.png)
 
1. Haga clic con el botón derecho en el esquema con el que está trabajando y, a continuación, seleccione **Convert Schema** (Convertir esquema). Como alternativa, puede seleccionar la pestaña **Convert Schema** (Convertir esquema) en la esquina superior derecha.

   ![Captura de pantalla del comando "Convert Schema" (Convertir Esquema) en el panel "MySQL Metadata Explorer" (Explorador de metadatos de MySQL).](./media/mysql-to-sql-database-guide/convert-schema.png)

1. Una vez finalizada la conversión, revise y compare los objetos convertidos con los objetos originales para identificar posibles problemas y solucionarlos en función de las recomendaciones. 

   ![Captura de pantalla que muestra una comparación de los objetos convertidos con los objetos originales.](./media/mysql-to-sql-database-guide/table-comparison.png)

   Compare el texto de Transact-SQL convertido con el código original y revise las recomendaciones.

   ![Captura de pantalla que muestra una comparación de las consultas convertidas con el código fuente.](./media/mysql-to-sql-database-guide/procedure-comparison.png)

1. En el panel **Output** (Salida), seleccione **Review results** (Revisar resultados) y revise los errores en el panel **Error list** (Lista de errores). 
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. Para ello, seleccione **Archivo** > **Guardar proyecto**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y realizar correcciones antes de publicar el esquema en la base de datos SQL.

   Compare los procedimientos convertidos con los procedimientos originales, como se muestra aquí: 

   ![Captura de pantalla que muestra una comparación de los procedimientos convertidos con los procedimientos originales.](./media/mysql-to-sql-database-guide/procedure-comparison.png)


## <a name="migrate-the-databases"></a>Migración de las bases de datos 

Después de evaluar las bases de datos y de solucionar las discrepancias, puede ejecutar el proceso de migración. La migración implica dos pasos: publicar el esquema y migrar los datos. 

Para publicar el esquema y migrar los datos, haga lo siguiente: 

1. Publique el esquema. En el panel **Azure SQL Database Metadata Explorer** (Explorador de metadatos de Azure SQL Database), haga clic con el botón derecho en la base de datos y, luego, seleccione **Synchronize with Database** (Sincronizar con base de datos). Esta acción publica el esquema de MySQL en la base de datos SQL.

   ![Captura de pantalla del panel "Synchronize with the Database" (Sincronizar con base de datos) para revisar la asignación de base de datos.](./media/mysql-to-sql-database-guide/synchronize-database-review.png)

1. Migre los datos. En el panel **MySQL Metadata Explorer** (Explorador de metadatos de MySQL), haga clic con el botón derecho en el esquema de MySQL que desea migrar y, a continuación, seleccione **Migrate Data** (Migrar datos). Como alternativa, puede seleccionar la pestaña **Migrate Data** (Migrar datos) en la esquina superior derecha.

   Para migrar datos de toda una base de datos, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active la casilla que hay junto a la tabla. Para omitir datos de tablas concretas, desactive la casilla.

   ![Captura de pantalla del comando "Migrate Data" (Migrar datos) en el panel "MySQL Metadata Explorer" (Explorador de metadatos de MySQL).](./media/mysql-to-sql-database-guide/migrate-data.png)

1. Una vez completada la migración, consulte **Data Migration Report** (Informe de migración de datos).
   
   ![Captura de pantalla del informe de migración de datos.](./media/mysql-to-sql-database-guide/data-migration-report.png)

1. Conéctese a la base de datos SQL mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) y revise los datos y el esquema para validar la migración.

   ![Captura de pantalla de SQL Server Management Studio.](./media/mysql-to-sql-database-guide/validate-in-ssms.png)

## <a name="post-migration"></a>Después de la migración 

Cuando haya completado correctamente la fase de *migración*, deberá realizar una serie de tareas posteriores para asegurarse de que todo funcione de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. Lograr esto requerirá en algunos casos realizar cambios en las aplicaciones.

### <a name="perform-tests"></a>Realización de pruebas

El enfoque de prueba para la migración de bases de datos consta de las siguientes actividades:

1. **Desarrollar pruebas de validación**: para probar la migración de bases de datos, debe utilizar consultas SQL. Debe crear las consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.

1. **Configurar un entorno de prueba**: el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.

1. **Ejecutar pruebas de validación**: ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.

1. **Ejecutar pruebas de rendimiento**: ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.

### <a name="optimize"></a>Optimización

La fase posterior a la migración es fundamental para reconciliar cualquier problema de precisión de datos y comprobar su integridad, así como para solucionar problemas de rendimiento con la carga de trabajo.

Para obtener más información sobre estos problemas y los pasos para mitigarlos, consulte la [Guía de optimización y validación posterior a la migración](/sql/relational-databases/post-migration-validation-and-optimization-guide).

## <a name="migration-assets"></a>Recursos de migración

Para obtener más ayuda a fin de completar este escenario de migración, vea el recurso siguiente. Se ha desarrollado para ofrecer compatibilidad con la involucración de un proyecto de migración del mundo real.

| Título | Descripción |
| --- | --- |
| [Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | Proporciona sugerencias de plataformas de destino "ideales", preparación para la nube y los niveles de corrección de la aplicación y la base de datos para las cargas de trabajo especificadas. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado. |

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.

## <a name="next-steps"></a>Pasos siguientes 

- Consulte [Calculadora del costo total de propiedad (TCO)](https://aka.ms/azure-tco) para ayudar a calcular el ahorro de costos que puede obtener mediante la migración de sus cargas de trabajo a Azure.

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md).

- Para ver otras guías de migración, consulte [Guía de Azure Database Migration](https://datamigration.microsoft.com/). 

- Para ver vídeos sobre migración, consulte [Información general sobre el recorrido de la migración y las herramientas y los servicios de migración y evaluación recomendados](https://azure.microsoft.com/resources/videos/overview-of-migration-and-recommended-tools-services/).

- Para obtener más [recursos sobre la migración a la nube](https://azure.microsoft.com/migration/resources/), consulte las [soluciones de migración](https://azure.microsoft.com/migration).

