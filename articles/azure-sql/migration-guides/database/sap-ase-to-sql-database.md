---
title: 'De SAP ASE a Azure SQL Database: guía de migración'
description: En esta guía obtendrá información sobre cómo migrar las bases de datos de SAP ASE a Azure SQL Database mediante SQL Server Migration Assistant para SAP Adaptive Server Enterprise.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: MashaMSFT
ms.author: mathoma
ms.date: 03/19/2021
ms.openlocfilehash: 0538071ffb9d244fb8b3493d6b63b27c6b56a726
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388540"
---
# <a name="migration-guide-sap-ase-to-azure-sql-database"></a>Guía de migración: de SAP ASE a Azure SQL Database

[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqldb.md)]

En esta guía obtendrá información sobre [cómo migrar](https://azure.microsoft.com/migration/migration-journey) las bases de datos de SAP Adaptive Server Enterprise (ASE) a Azure SQL Database mediante [SQL Server Migration](https://azure.microsoft.com/en-us/migration/sql-server/) Assistant para SAP Adaptive Server Enterprise.

Para ver otras guías de migración, consulte [Guía de Azure Database Migration](https://docs.microsoft.com/data-migration). 

## <a name="prerequisites"></a>Prerrequisitos 

Antes de empezar a migrar la base de datos de SAP SE a SQL Database, haga lo siguiente:

- Compruebe que el entorno de origen es compatible. 
- Descargue e instale [SQL Server Migration Assistant para SAP Adaptive Server Enterprise (anteriormente, SAP Sybase ASE)](https://www.microsoft.com/en-us/download/details.aspx?id=54256).
- Asegúrese de que tiene conectividad y permisos suficientes para acceder tanto al origen como al destino.

## <a name="pre-migration"></a>Antes de la migración

Una vez cumplidos los requisitos previos, estará listo para detectar la topología del entorno y evaluar la viabilidad de la [migración a la nube de Azure](https://azure.microsoft.com/migration).

### <a name="assess"></a>Evaluar

Con el uso de [SQL Server Migration Assistant (SSMA) para SAP Adaptive Server Enterprise (anteriormente, SAP Sybase ASE)](https://www.microsoft.com/en-us/download/details.aspx?id=54256), puede revisar los datos y objetos de base de datos, evaluar las bases de datos para la migración, migrar objetos de base de datos de Sybase a Azure SQL Database y, después, migrar datos a SQL Database. Para obtener más información, consulte [SQL Server Migration Assistant para Sybase (SybaseToSQL)](/sql/ssma/sybase/sql-server-migration-assistant-for-sybase-sybasetosql).

Para crear una valoración, siga estos pasos: 

1. Abra SSMA para Sybase. 
1. Seleccione **Archivo** y, a continuación, seleccione **Nuevo proyecto**. 
1. En el panel **Nuevo proyecto**, escriba un nombre y una ubicación para el proyecto y, a continuación, en la lista desplegable **Migrar a**, seleccione **Azure SQL Database**. 
1. Seleccione **Aceptar**.
1. En el panel **Conectar a Sybase**, escriba los detalles de conexión de SAP. 
1. Haga clic con el botón derecho en la base de datos de SAP que quiera migrar y, luego, seleccione **Crear informe**. Se generará un informe HTML. Como alternativa, puede seleccionar la pestaña **Crear informe** en la esquina superior derecha.
1. Revise el informe HTML para comprender las estadísticas de conversión, y los errores o advertencias. También puede abrir el informe en Excel para obtener un inventario de objetos de SAP ASE y conocer el esfuerzo necesario para realizar las conversiones de esquema. La ubicación predeterminada del informe es la carpeta de informes dentro de SSMAProjects. Por ejemplo:

   `drive:\<username>\Documents\SSMAProjects\MySAPMigration\report\report_<date>` 

### <a name="validate-the-type-mappings"></a>Validación de las asignaciones de tipos

Antes de llevar a cabo la conversión de esquemas, valide las asignaciones de tipos de datos predeterminadas o cámbielas en función de los requisitos. Para ello, seleccione **Herramientas** > **Configuración del proyecto**, o bien cambie la asignación de tipos de cada tabla seleccionando la tabla en el **Explorador de metadatos de SAP ASE**.

### <a name="convert-the-schema"></a>Conversión del esquema

Para convertir el esquema, haga lo siguiente:

1. (Opcional) Para convertir consultas dinámicas o especializadas, haga clic con el botón derecho en el nodo y seleccione **Agregar instrucción**. 
1. Seleccione la pestaña **Conectarse a Azure SQL Database** y, a continuación, escriba los detalles de la base de datos SQL. Puede optar por conectarse a una base de datos existente o proporcionar un nombre nuevo, en cuyo caso se creará una base de datos en el servidor de destino.
1. En el panel **Explorador de metadatos de Sybase**, haga clic con el botón derecho en el esquema de SAP ASE con el que está trabajando y seleccione **Convertir esquema**. 
1. Una vez convertido el esquema, compare y revise la estructura convertida con la estructura original para identificar posibles problemas. 

   Después de convertir el esquema, puede guardar este proyecto localmente para un ejercicio de corrección de esquema sin conexión. Para ello, seleccione **Archivo** > **Guardar proyecto**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y hacer correcciones para poder publicar el esquema en la base de datos SQL.

1. En el panel **Resultados**, seleccione **Revisar resultados** y revise los errores en el panel **Lista de errores**. 
1. Guarde el proyecto localmente para un ejercicio de corrección de esquema sin conexión. Para ello, seleccione **Archivo** > **Guardar proyecto**. Esto le ofrece la oportunidad de evaluar los esquemas de origen y de destino sin conexión, y hacer correcciones para poder publicar el esquema en la base de datos SQL.

## <a name="migrate-the-databases"></a>Migración de las bases de datos 

Una vez que haya cumplido los requisitos previos necesarios y completado las tareas asociadas a la fase *Antes de la migración*, estará a punto para ejecutar el esquema y la migración de datos.

Para publicar el esquema y migrar los datos, haga lo siguiente: 

1. Publique el esquema. En el panel del **Explorador de metadatos de Azure SQL Database**, haga clic con el botón derecho en la base de datos y, luego, seleccione **Sincronizar con base de datos**. Esta acción publica el esquema de SAP ASE en la base de datos SQL.

1. Migre los datos. En el panel del **Explorador de metadatos de SAP ASE**, haga clic con el botón derecho en la base de datos o el objeto de SAP ASE que quiera migrar y, a continuación, seleccione **Migrar datos**. Como alternativa, puede seleccionar la pestaña **Migrar datos** en la esquina superior derecha. 

   Para migrar datos de toda una base de datos, active la casilla situada junto al nombre de la base de datos. Para migrar datos de tablas concretas, expanda la base de datos, expanda **Tablas** y, a continuación, active la casilla que hay junto a la tabla. Para omitir datos de tablas concretas, desactive la casilla. 
1. Una vez completada la migración, vea el **Informe de migración de datos**. 
1. Para validar la migración, revise los datos y el esquema. Para ello, conéctese a la base de datos SQL mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

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


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md).

- Para más información acerca de Azure SQL Database, consulte:
   - [Información general sobre SQL Database](../../database/sql-database-paas-overview.md)
   - [Calculadora del costo total de propiedad (TCO)](https://azure.microsoft.com/pricing/tco/calculator/)  

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 
   -  [Recursos de migración a la nube](https://azure.microsoft.com/migration/resources)

- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Para más información sobre cómo realizar pruebas A/B en la capa de acceso a datos, consulte [Información general del Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).
