---
title: 'De Db2 a Azure SQL Managed Instance: guía de migración'
description: Con esta guía aprenderá a migrar sus bases de datos de IBM Db2 a Azure SQL Managed Instance mediante SQL Server Migration Assistant para Db2.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 4c3fd064eae2eaa918993b9de9789a25a27d05d7
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136506"
---
# <a name="migration-guide-ibm-db2-to-azure-sql-managed-instance"></a>Guía de migración: de IBM Db2 a Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

Con esta guía aprenderá a migrar sus bases de datos de IBM Db2 a Azure SQL Managed Instance mediante SQL Server Migration Assistant para Db2. 

Para ver otras guías de migración, consulte la [Guía de Azure Database Migration](/data-migration). 

## <a name="prerequisites"></a>Requisitos previos 

Para migrar la base de datos de Db2 a SQL Managed Instance, necesita lo siguiente:

- Comprobar que el [entorno de origen sea compatible](/sql/ssma/db2/installing-ssma-for-db2-client-db2tosql#prerequisites).
- Descargar [SQL Server Migration Assistant (SSMA) para Db2](https://www.microsoft.com/download/details.aspx?id=54254).
- Una instancia de destino de [Azure SQL Managed Instance](../../managed-instance/instance-create-quickstart.md).
- Conectividad y permisos suficientes para acceder tanto al origen como al destino. 

## <a name="pre-migration"></a>Antes de la migración

Una vez cumplidos los requisitos previos, estará a punto para detectar la topología de su entorno y evaluar la viabilidad de la migración. 

### <a name="assess-and-convert"></a>Evaluación y conversión

Cree una evaluación mediante SQL Server Migration Assistant. 

Para crear una evaluación, siga estos pasos:

1. Abra [SSMA para Db2](https://www.microsoft.com/download/details.aspx?id=54254). 
1. Seleccione **Archivo** > **Nuevo proyecto**. 
1. Proporcione un nombre de proyecto y una ubicación para guardar el proyecto. Después, seleccione Azure SQL Managed Instance como destino de la migración en la lista desplegable y seleccione **Aceptar**.

   :::image type="content" source="media/db2-to-managed-instance-guide/new-project.png" alt-text="Captura de pantalla que muestra los detalles del proyecto que se van a especificar.":::


1. En **Conectar a Db2**, escriba los valores de los detalles de conexión de Db2. 

   :::image type="content" source="media/db2-to-managed-instance-guide/connect-to-db2.png" alt-text="Captura de pantalla que muestra las opciones para conectarse a la instancia de Db2.":::


1. Haga clic con el botón derecho en el esquema de Db2 que quiera migrar y, luego, elija **Create Report** (Crear informe). Se generará un informe HTML. También puede elegir **Crear informe** en la barra de navegación después de seleccionar el esquema.

   :::image type="content" source="media/db2-to-managed-instance-guide/create-report.png" alt-text="Captura de pantalla que muestra cómo crear un informe.":::

1. Revise el informe en HTML para comprender las estadísticas de conversión, y los errores o advertencias. También puede abrir el informe en Excel para obtener un inventario de objetos Db2 y conocer el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes dentro de *SSMAProjects*.

   Por ejemplo: `drive:\<username>\Documents\SSMAProjects\MyDb2Migration\report\report_<date>`. 

   :::image type="content" source="media/db2-to-managed-instance-guide/report.png" alt-text="Captura de pantalla del informe que se debe revisar para identificar los errores o las advertencias":::


### <a name="validate-data-types"></a>Validación de los tipos de datos

Valide las asignaciones de tipos de datos predeterminadas y cámbielas según los requisitos, si es necesario. Para hacerlo, siga estos pasos: 

1. Seleccione **Herramientas** en el menú principal. 
1. Seleccione **Configuración del proyecto**. 
1. Seleccione la pestaña **Type mappings** (Asignaciones de tipos).

   :::image type="content" source="media/db2-to-managed-instance-guide/type-mapping.png" alt-text="Captura de pantalla que muestra la selección del esquema y la asignación de tipos.":::

1. Puede cambiar la asignación de tipos de cada tabla seleccionando la tabla en el **Explorador de metadatos de Db2**. 

### <a name="convert-schema"></a>Convertir esquema 

Para convertir el esquema, siga estos pasos:

1. (Opcional) Agregue consultas dinámicas o ad hoc a las instrucciones. Haga clic con el botón derecho en el nodo y elija **Add statements** (Agregar instrucciones). 
1. Seleccione **Conexión a Azure SQL Managed Instance**. 
    1. Escriba los detalles de la conexión para conectarse a Azure SQL Managed Instance. 
    1. Elija la base de datos de destino de la lista desplegable o indique un nombre nuevo, en cuyo caso se creará una base de datos en el servidor de destino. 
    1. Proporcione los detalles de la autenticación. 
    1. Seleccione **Conectar**.

   :::image type="content" source="media/db2-to-managed-instance-guide/connect-to-sql-managed-instance.png" alt-text="Captura de pantalla que muestra la información necesaria para conectarse a SQL Server.":::


1. Haga clic con el botón derecho en el esquema y elija **Convertir esquema**. También puede elegir **Convertir esquema** en la barra de navegación superior después de seleccionar el esquema.

   :::image type="content" source="media/db2-to-managed-instance-guide/convert-schema.png" alt-text="Captura de pantalla que muestra la selección del esquema y la opción para convertirlo.":::

1. Una vez finalizada la conversión, compare y revise la estructura del esquema para identificar posibles problemas. Solucione los problemas en función de las recomendaciones.

   :::image type="content" source="media/db2-to-managed-instance-guide/compare-review-schema-structure.png" alt-text="Captura de pantalla que muestra la comparación y revisión de la estructura del esquema para identificar posibles problemas.":::

1. En el panel **Resultados**, seleccione **Revisar resultados**. En el panel **Lista de errores**, revise los errores. 
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. En el menú **Archivo**, seleccione **Guardar proyecto**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y hacer correcciones para poder publicar el esquema en SQL Managed Instance.

## <a name="migrate"></a>Migrar

Cuando haya terminado de evaluar las bases de datos y de resolver las discrepancias, el siguiente paso consiste en ejecutar el proceso de migración.

Para publicar el esquema y migrar los datos, siga estos pasos:

1. Publique el esquema. En el **Explorador de metadatos de Azure SQL Managed Instance**, en el nodo **Bases de datos**, haga clic con el botón derecho en la base de datos. A continuación, seleccione **Sincronizar con la base de datos**.

   :::image type="content" source="media/db2-to-managed-instance-guide/synchronize-with-database.png" alt-text="Captura de pantalla que muestra la opción para sincronizar con la base de datos.":::

1. Migre los datos. Haga clic con el botón derecho en la base de datos o el objeto que quiera migrar en el **Explorador de metadatos de Db2** y elija **Migrar datos**. Como alternativa, puede seleccionar **Migrar datos** en la barra de navegación. Para migrar datos de toda una base de datos, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active la casilla que hay junto a la tabla. Para omitir datos de tablas concretas, desactive la casilla.

   :::image type="content" source="media/db2-to-managed-instance-guide/migrate-data.png" alt-text="Captura de pantalla que muestra la selección del esquema y la elección de la opción Migrar datos.":::

1. Especifique los detalles de la conexión para las instancias de Db2 y SQL Managed Instance. 
1. Una vez finalizada la migración, vea el **informe de migración de datos**.  

   :::image type="content" source="media/db2-to-managed-instance-guide/data-migration-report.png" alt-text="Captura de pantalla que muestra dónde revisar el informe de migración de datos.":::

1. Conéctese a la instancia de Azure SQL Managed Instance mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms). Para validar la migración, revise los datos y el esquema:

   :::image type="content" source="media/db2-to-managed-instance-guide/compare-schema-in-ssms.png" alt-text="Captura de pantalla que muestra la comparación del esquema en SQL Server Management Studio.":::

## <a name="post-migration"></a>Etapa posterior a la migración 

Cuando haya completado correctamente, deberá realizar una serie de tareas posteriores para asegurarse de que todo funcione de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones 

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. Lograr esto requerirá en algunos casos realizar cambios en las aplicaciones.


### <a name="perform-tests"></a>Realización de pruebas

Las pruebas constan de las siguientes actividades:

1. **Desarrollar pruebas de validación**: para probar la migración de bases de datos, debe utilizar consultas SQL. Debe crear las consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.
1. **Configurar el entorno de prueba**: el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.
1. **Ejecutar pruebas de validación**: ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.
1. **Ejecutar pruebas de rendimiento**: ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.

## <a name="advanced-features"></a>Características avanzadas 

Asegúrese de aprovechar las características avanzadas basadas en la nube que ofrece Azure SQL Managed Instance, como las de [alta disponibilidad integrada](../../database/high-availability-sla.md), [detección de amenazas](../../database/azure-defender-for-sql.md) y [supervisión y ajuste de la carga de trabajo](../../database/monitor-tune-overview.md). 

Algunas características de SQL Server solo están disponibles cuando el [nivel de compatibilidad de la base de datos](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) cambia al nivel de compatibilidad más reciente. 

## <a name="migration-assets"></a>Recursos de migración 

Para obtener más ayuda, consulte los siguientes recursos, que se desarrollaron como apoyo de la participación en un proyecto de migración en el mundo real:

|Recurso  |Descripción  |
|---------|---------|
|[Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool)| Esta herramienta proporciona las plataformas de destino de ajuste perfecto sugeridas, la preparación para la nube, y el nivel de corrección de la aplicación o base de datos para una carga de trabajo determinada. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado.|
|[Paquete de detección y evaluación de recursos de datos de Db2 zOS](https://github.com/microsoft/DataMigrationTeam/tree/master/DB2%20zOS%20Data%20Assets%20Discovery%20and%20Assessment%20Package)|Después de ejecutar el script de SQL en una base de datos, puede exportar los resultados a un archivo en el sistema de archivos. Se admiten varios formatos de archivo, incluido *.csv, para que se puedan capturar los resultados en herramientas externas, como hojas de cálculo. Este método puede resultar útil si quiere compartir fácilmente los resultados con equipos que no tengan el área de trabajo instalada.|
|[Artefactos y scripts de inventario de IBM Db2 LUW](https://github.com/microsoft/DataMigrationTeam/blob/master/IBM%20DB2%20LUW%20Inventory%20Scripts%20and%20Artifacts)|Este recurso incluye una consulta SQL que accede a las tablas del sistema IBM Db2 LUW, versión 11.1, y proporciona un recuento de objetos por esquema y tipo de objeto, una estimación aproximada de "datos sin procesar" en cada esquema y el tamaño de las tablas de cada esquema, cuyos resultados se almacenan en formato CSV.|
|[Guía de configuración del escalado puro de Db2 LUW en Azure](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/DB2%20PureScale%20on%20Azure.pdf)|Esta guía sirve como punto de partida para un plan de implementación de Db2. Aunque sus requisitos empresariales varíen, se aplica el mismo patrón básico. Este patrón de arquitectura también se puede utilizar para aplicaciones OLAP en Azure.|

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener los servicios y las herramientas de Microsoft y de otros fabricantes para ayudarle en diversos escenarios de migración de datos y bases de datos, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md).

- Para obtener más información acerca Azure SQL Managed Instance, consulte:
   - [Introducción a SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md)
   - [Calculadora del costo total de propiedad (TCO)](https://azure.microsoft.com/pricing/tco/calculator/) 


- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para la gestión de los costos y los ajustes de tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Para obtener más información sobre cómo realizar pruebas A/B de capa de acceso a datos, consulte [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).