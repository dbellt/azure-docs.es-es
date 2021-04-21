---
title: 'De Access a Azure SQL Database: guía de migración'
description: En esta guía va a aprender a migrar bases de datos de Microsoft Access a una base de datos de Azure SQL mediante SQL Server Migration Assistant para Access (SSMA para Access).
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.date: 03/19/2021
ms.openlocfilehash: 73ee4d4be16284880b10df4a52b422a08c04c6a0
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107284194"
---
# <a name="migration-guide-access-to-azure-sql-database"></a>Guía de migración: de Access a Azure SQL Database

En esta guía va a aprender a [migrar](https://azure.microsoft.com/migration/migration-journey) la base de datos de Microsoft Access a una base de datos de Azure SQL mediante [SQL Server Migration](https://azure.microsoft.com/en-us/migration/sql-server/) Assistant para Access (SSMA para Access).

Para ver otras guías de migración, consulte [Guía de Azure Database Migration](https://docs.microsoft.com/data-migration). 

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a migrar la base de datos de Access a una base de datos SQL, haga lo siguiente:

- Compruebe que el entorno de origen es compatible. 
- Descargue e instale [SQL Server Migration Assistant para Access](https://www.microsoft.com/download/details.aspx?id=54255).
- Asegúrese de que tiene conectividad y permisos suficientes para acceder tanto al origen como al destino.

## <a name="pre-migration"></a>Antes de la migración

Una vez cumplidos los requisitos previos, estará listo para detectar la topología del entorno y evaluar la viabilidad de la [migración a la nube de Azure](https://azure.microsoft.com/migration).


### <a name="assess"></a>Evaluar 

Use SSMA para Access para revisar objetos de base de datos y datos, y evaluar las bases de datos para la migración. 

Para crear una valoración, siga estos pasos: 

1. Abra [SSMA para Access](https://www.microsoft.com/download/details.aspx?id=54255). 
1. Seleccione **Archivo** y, a continuación, seleccione **Nuevo proyecto**. 
1. Facilite un nombre de proyecto y una ubicación para el proyecto y, luego, en la lista desplegable, seleccione **Azure SQL Database** como destino de la migración. 
1. Seleccione **Aceptar**. 

   ![Captura de pantalla del panel "Nuevo proyecto" para escribir el nombre y la ubicación del proyecto de migración.](./media/access-to-sql-database-guide/new-project.png)

1. Seleccione **Agregar bases de datos** y luego las bases de datos que se van a agregar al nuevo proyecto. 

   ![Captura de pantalla de la pestaña "Agregar bases de datos" de SSMA para Access.](./media/access-to-sql-database-guide/add-databases.png)

1. En el panel **Explorador de metadatos de Access**, haga clic con el botón derecho en una base de datos y seleccione **Crear informe**. También puede seleccionar la pestaña **Crear informe** en la parte superior derecha.

   ![Captura de pantalla del comando "Crear informe" del Explorador de metadatos de Access.](./media/access-to-sql-database-guide/create-report.png)

1. Revise el informe HTML para comprender las estadísticas de conversión, y los errores o advertencias. También puede abrir el informe en Excel para obtener un inventario de objetos de Access y entender el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes dentro de SSMAProjects. Por ejemplo:

   `drive:\<username>\Documents\SSMAProjects\MyAccessMigration\report\report_<date>`

   ![Captura de pantalla de una evaluación de informe de base de datos de ejemplo en SSMA.](./media/access-to-sql-database-guide/sample-assessment.png)

### <a name="validate-the-data-types"></a>Validación de los tipos de datos

Valide las asignaciones predeterminadas de los tipos de datos y cámbielas según los requisitos, si fuera necesario. Para ello:

1. En SSMA para Access, seleccione **Herramientas** y luego **Configuración del proyecto**. 
1. Seleccione la pestaña **Asignación de tipo**. 

   ![Captura de pantalla del panel "Asignación de tipo" de SSMA para Access.](./media/access-to-sql-database-guide/type-mappings.png)

1. Puede cambiar la asignación de tipo de cada tabla si selecciona el nombre de tabla en el panel **Explorador de metadatos de Access**.


### <a name="convert-the-schema"></a>Conversión del esquema

Para convertir objetos de base de datos, haga lo siguiente: 

1. Seleccione la pestaña **Conectarse a Azure SQL Database** y haga lo siguiente:

   a. Escriba los detalles para conectarse a la base de datos SQL.  
   b. Seleccione la base de datos SQL de destino en la lista desplegable. También puede escribir un nombre nuevo, en cuyo caso se crea una base de datos en el servidor de destino.  
   c. Proporcione los detalles de la autenticación.   
   d. Seleccione **Conectar**.

   ![Captura de pantalla del panel "Conectarse a Azure SQL Database" para escribir los detalles de la conexión.](./media/access-to-sql-database-guide/connect-to-sqldb.png)

1. En el panel **Explorador de metadatos de Access**, haga clic con el botón derecho en la base de datos y seleccione **Convertir esquema**. También puede seleccionar la base de datos y luego la pestaña **Convertir esquema**.

   ![Captura de pantalla del comando "Convertir esquema" en el panel "Explorador de metadatos de Access".](./media/access-to-sql-database-guide/convert-schema.png)

1. Una vez finalizada la conversión, compare los objetos convertidos con los originales para identificar posibles problemas y solucionarlos en función de las recomendaciones.

   ![Captura de pantalla que muestra una comparación de los objetos convertidos con los de origen.](./media/access-to-sql-database-guide/table-comparison.png)

    Compare el texto de Transact-SQL convertido con el código original y revise las recomendaciones.

   ![Captura de pantalla que muestra una comparación de las consultas convertidas con el código fuente.](./media/access-to-sql-database-guide/query-comparison.png) 

1. (Opcional) Para convertir un objeto individual, haga clic con el botón derecho en el objeto y seleccione **Convertir esquema**. Los objetos convertidos aparecen en negrita en **Explorador de metadatos de Access**: 

   ![Captura de pantalla que muestra que los objetos de Explorador de metadatos de Access se han convertido.](./media/access-to-sql-database-guide/converted-items.png)
 
1. En el panel **Salida**, seleccione el icono **Revisar resultados** y revise los errores del panel **Lista de errores**. 
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. Para ello, seleccione **Archivo** > **Guardar proyecto**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y de hacer correcciones antes de publicarlos en la base de datos SQL.

## <a name="migrate-the-databases"></a>Migración de las bases de datos

Después de evaluar las bases de datos y de solucionar las discrepancias, puede llevar a cabo el proceso de migración. La migración de datos es una operación de carga masiva que mueve filas de datos a una base de datos de Azure SQL en transacciones. El número de filas que se van a cargar en la base de dato SQL en cada transacción se define en la configuración del proyecto.

Para publicar el esquema y migrar los datos con SSMA para Access, haga lo siguiente: 

1. Si aún no lo ha hecho, seleccione **Conectarse a Azure SQL Database** y proporcione los detalles de la conexión. 

1. Publique el esquema. En el panel **Explorador de metadatos de Azure SQL Database**, haga clic con el botón derecho en la base de datos con la que está trabajando y seleccione **Sincronizar con la base de datos**. Esta acción publica el esquema de MySQL en la base de datos SQL.

1. En el panel **Sincronizar con la base de datos**, revise la asignación entre el proyecto de origen y el destino:

   ![Captura de pantalla del panel "Sincronizar con la base de datos" para revisar la sincronización con la base de datos.](./media/access-to-sql-database-guide/synchronize-with-database-review.png)

1. En el panel **Explorador de metadatos de Access**, active las casillas situadas junto a los elementos que quiere migrar. Para migrar toda la base de datos, active la casilla situada junto a ella. 

1. Migre los datos. Haga clic con el botón derecho en la base de datos o el objeto que quiere migrar y seleccione **Migrar datos**. Como alternativa, puede seleccionar la pestaña **Migrar datos** en la esquina superior derecha.  

   Para migrar datos de toda una base de datos, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active la casilla que hay junto a la tabla. Para omitir datos de tablas concretas, desactive la casilla.

    ![Captura de pantalla del comando "Migrar datos" en el panel "Explorador de metadatos de Access".](./media/access-to-sql-database-guide/migrate-data.png)

1. Una vez completada la migración, vea el **Informe de migración de datos**.  

    ![Captura de pantalla del panel "Informe de migración de datos" que muestra un informe de ejemplo para su revisión.](./media/access-to-sql-database-guide/migrate-data-review.png)

1. Conéctese a la base de datos de Azure SQL mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) y revise los datos y el esquema para validar la migración.

   ![Captura de pantalla del Explorador de objetos de SQL Server Management Studio para validar la migración en SSMA.](./media/access-to-sql-database-guide/validate-data.png)

## <a name="post-migration"></a>Etapa posterior a la migración 

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

Para obtener más ayuda a fin de completar este escenario de migración, vea el siguiente recurso. Se ha desarrollado para ayudar a integrar el proyecto de migración real.

| Título | Descripción |
| --- | --- |
| [Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | Proporciona sugerencias de plataformas de destino ideales, preparación para la nube, y niveles de corrección de aplicación o base de datos para cargas de trabajo especificadas. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado. |

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md).

- Para más información acerca de Azure SQL Database, consulte:
   - [Información general sobre SQL Database](../../database/sql-database-paas-overview.md)
   - [Calculadora del costo total de propiedad (TCO)](https://azure.microsoft.com/pricing/tco/calculator/) 


- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 
   -  [Recursos de Migración a la nube](https://azure.microsoft.com/migration/resources)


- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Para obtener información sobre cómo realizar pruebas A/B de capa de acceso a datos, vea [Información general de Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).
