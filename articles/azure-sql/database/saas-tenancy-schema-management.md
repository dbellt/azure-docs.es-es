---
title: Administración de esquemas en una aplicación de un solo inquilino
description: Administración del esquema para varios inquilinos en una aplicación de un solo inquilino que utiliza Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 09/19/2018
ms.openlocfilehash: 86ddf829f44f18dd7e29ae99aa86a3cad6c2b224
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694222"
---
# <a name="manage-schema-in-a-saas-application-using-the-database-per-tenant-pattern-with-azure-sql-database"></a>Administración del esquema en una aplicación SaaS con el patrón base de datos por inquilino con Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]
 
A medida que una aplicación de base de datos evoluciona, resulta inevitable realizar cambios en los datos de referencia o el esquema de la base de datos.  También es necesario realizar las tareas de mantenimiento de la base de datos de manera periódica. Administrar una aplicación que usa el patrón de base de datos por inquilino requiere aplicar estos cambios o tareas de mantenimiento a todo un conjunto de bases de datos de inquilino.

Este tutorial explora dos escenarios: la implementación de actualizaciones de datos de referencia para todos los inquilinos y la recompilación de un índice en la tabla que contiene los datos de referencia. La característica [Trabajos elásticos](./elastic-jobs-overview.md) se usa para ejecutar estas acciones en todas las bases de datos de inquilino y en la base de datos de plantilla que se usa para crear bases de datos de inquilino nuevas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> 
> * Crear un agente de trabajo
> * Hacer que los trabajos T-SQL se ejecuten en todas las bases de datos de inquilino
> * Actualizar los datos de referencia en todas las bases de datos de inquilino
> * Crear un índice en una tabla en todas las bases de datos de inquilino


Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* Aplicación Wingtip Tickets SaaS Database Per Tenant implementada. Para implementarla en menos de cinco minutos, consulte el artículo de [implementación y exploración de la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)
* La versión más reciente de SQL Server Management Studio (SSMS) está instalada. [Descarga e instalación de SSMS](/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="introduction-to-saas-schema-management-patterns"></a>Introducción a los patrones de administración de esquema de SaaS

El patrón de base de datos por inquilino aísla de manera eficaz los datos de inquilino, pero aumenta la cantidad de bases de datos para administrar y mantener. La característica [Trabajos elásticos](./elastic-jobs-overview.md) facilita la administración de varias bases de datos. Los trabajos le permiten ejecutar de manera segura y confiable las tareas (scripts T-SQL) en un grupo de bases de datos. Los trabajos pueden implementar cambios en los datos de referencia comunes y el esquema en todas las bases de datos de inquilino de una aplicación. Trabajos elásticos también se puede usar para mantener una base de datos de *plantilla* que se usa para crear inquilinos nuevos, garantizando así que siempre tenga el esquema y los datos de referencia más recientes.

![pantalla](./media/saas-tenancy-schema-management/schema-management-dpt.png)


## <a name="elastic-jobs-public-preview"></a>Versión preliminar pública de trabajos elásticos

Hay una nueva versión de Trabajos elásticos que ahora es una característica integrada de Azure SQL Database. Esta nueva versión de Trabajos elásticos ofrece actualmente una versión preliminar pública. La versión preliminar pública actualmente admite usar PowerShell para crear un agente de trabajo y T-SQL para crear y administrar trabajos.
Consulte el artículo sobre [trabajos de bases de datos elásticas](./elastic-jobs-overview.md) para más información.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtención de los scripts de la aplicación Wingtip Tickets SaaS Database Per Tenant

El código fuente y los scripts de administración de la aplicación están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="create-a-job-agent-database-and-new-job-agent"></a>Creación de una base de datos de agente de trabajo y un agente de trabajo nuevo

En este tutorial es necesario usar PowerShell para crear un agente de trabajo y su base de datos de agente de trabajo de respaldo. La base de datos del agente de trabajo contiene las definiciones del trabajo, el estado del trabajo y el historial. Una vez que se crea el agente de trabajo y su base de datos, es posible crear y supervisar los trabajos de inmediato.

1. En **PowerShell ISE**, abra …\\Learning Modules\\Schema Management\\*Demo-SchemaManagement.ps1*.
1. Presione **F5** para ejecutar el script.

El script *Demo-SchemaManagement.ps1* llama al script *Deploy-SchemaManagement.ps1* para crear una base de datos denominada *osagent* en el servidor de catálogo. Luego crea el agente de trabajo con la base de datos como parámetro.

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>Creación de un trabajo para implementar nuevos datos de referencia en todos los inquilinos

En la aplicación Wingtip Tickets, cada base de datos de inquilino incluye un conjunto de tipos de ubicación admitidos. Cada ubicación corresponde a un tipo de ubicación específico, que define la clase de eventos que se puede organizar y determina la imagen de fondo que se usa en la aplicación. Para que la aplicación admita nuevas clases de eventos, estos datos de referencia se deben actualizar y se deben agregar nuevos tipos de ubicación.  En este ejercicio, se implementa una actualización en todas las bases de datos del inquilino para agregar dos tipos de ubicación adicionales: *Motorcycle Racing* (motociclismo) y *Swimming Club* (club de natación).

En primer lugar, revise los tipos de ubicación que se incluyen en cada base de datos de inquilino. Conéctese a una de las bases de datos de inquilino en SQL Server Management Studio (SSMS) y revise la tabla VenueTypes.  También puede consultar esta tabla en el Editor de consultas de Azure Portal, al que se accede desde la página de la base de datos. 

1. Abra SSMS y conéctese al servidor de inquilino: *tenants1-dpt-&lt;usuario&gt;.database.windows.net*
1. Para confirmar que *Motorcycle Racing* y *Swimming Club* **no están** incluidos actualmente, vaya a la base de datos _contosoconcerthall_ en el servidor *tenants1-dpt-&lt;usuario&gt;* y consulte la tabla *VenueTypes*.

Ahora vamos a crear un trabajo para actualizar la tabla *VenueTypes* en todas las bases de datos de inquilino para agregar los nuevos tipos de ubicación.

Para crear un trabajo nuevo, se usa un conjunto de procedimientos almacenados en el sistema de trabajos que se creó en la base de datos _jobagent_ cuando se creó el agente de trabajo.

1. En SSMS, conéctese al servidor del catálogo: servidor *catalog-dpt-&lt;usuario&gt;.database.windows.net* 
1. En SSMS, abra el archivo …\\Learning Modules\\Schema Management\\DeployReferenceData.sql.
1. Modifique la instrucción: SET @wtpUser = &lt;usuario&gt; y sustituya el valor de usuario utilizado al implementar la aplicación Wingtip Tickets SaaS Database Per Tenant.
1. Asegúrese de estar conectado a la base de datos _jobagent_ y presione **F5** para ejecutar el script

Observe los elementos siguientes en el script *DeployReferenceData.sql*:
* **sp\_add\_target\_group** crea el nombre del grupo de destino DemoServerGroup.
* **sp\_add\_target\_group\_member** se usa para definir el conjunto de bases de datos de destino.  En primer lugar, se agrega el servidor _tenants1-dpt-&lt;usuario&gt;_ .  Agregar el servidor como destino hace que las bases de datos de ese servidor en el momento de la ejecución del trabajo se incluyan en el trabajo. Luego, la base de datos _basetenantdb_ y la base de datos *adhocreporting* (que se usan en un tutorial posterior) se agregan como destinos.
* **sp\_add\_job** crea un trabajo denominado _Reference Data Deployment_ (Implementación de datos de referencia).
* **sp\_add\_jobstep** crea el paso de trabajo que contiene el texto del comando T-SQL para actualizar la tabla de referencia VenueTypes.
* En las demás vistas del script se muestran los objetos existentes y se supervisa la ejecución de los trabajos. Use estas consultas para revisar el valor de estado de la columna **lifecycle** (ciclo de vida) para determinar si el trabajo finalizó en todas las bases de datos de destino.

Una vez que se completa el script, puede comprobar si se actualizaron los datos de referencia.  En SSMS, vaya a la base de datos *contosoconcerthall* en el servidor *tenants1-dpt-&lt;usuario&gt;* y consulte la tabla *VenueTypes*.  Compruebe que ahora *Motorcycle Racing* y *Swimming Club* **están** presentes.


## <a name="create-a-job-to-manage-the-reference-table-index"></a>Creación de un trabajo para administrar el índice de tabla de referencia

Este ejercicio usa un trabajo para recompilar el índice en la clave principal de la tabla de referencia.  Esta es una operación de mantenimiento de base de datos típica que se puede realizar después de cargar grandes cantidades de datos.

Cree un trabajo con los mismos procedimientos almacenados del "sistema" de trabajos.

1. Abra SSMS y conéctese al servidor _catalog-dpt-&lt;usuario&gt;.database.windows.net_
1. Abra el archivo _…\\Learning Modules\\Schema Management\\OnlineReindex.sql_
1. Haga clic con el botón derecho, seleccione Conexión y conéctese al servidor _catalog-dpt-&lt;usuario&gt;.database.windows.net_ si aún no lo está
1. Asegúrese de estar conectado a la base de datos _jobagent_ y presione **F5** para ejecutar el script

Observe los elementos siguientes en el script _OnlineReindex.sql_:
* **sp\_add\_job** crea un trabajo denominado "Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885".
* **sp\_add\_jobstep** crea el paso de trabajo que contiene el texto del comando T-SQL para actualizar el índice.
* Las demás vistas del script permiten supervisar la ejecución del trabajo. Use estas consultas para revisar el valor de estado en la columna **ciclo de vida** a fin de determinar si el trabajo ha finalizado correctamente en todos los miembros del grupo de destino.



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> 
> * Crear un agente de trabajo para ejecutar en varias bases de datos de trabajos T-SQL
> * Actualizar los datos de referencia en todas las bases de datos de inquilino
> * Crear un índice en una tabla en todas las bases de datos de inquilino

A continuación, pruebe el [tutorial de notificaciones ad hoc](./saas-tenancy-cross-tenant-reporting.md) para explorar la ejecución de consultas distribuidas en las bases de datos de inquilino.


## <a name="additional-resources"></a>Recursos adicionales

* [Otros tutoriales basados en la implementación de la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Administración de bases de datos escaladas horizontalmente en la nube](./elastic-jobs-overview.md)