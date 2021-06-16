---
title: Administración de esquemas en una aplicación multiinquilino
description: Administración del esquema para varios inquilinos en una aplicación multiinquilino que usa Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: f4d7ae6849249fec874d0d5af2df042323c81886
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708195"
---
# <a name="manage-schema-in-a-saas-application-that-uses-sharded-multi-tenant-databases"></a>Administración del esquema en una aplicación SaaS que usa bases de datos multiinquilino particionadas
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial se examinan los desafíos de mantenimiento de una gran cantidad de bases de datos en una aplicación de software como servicio (SaaS). Se muestran soluciones para distribuir de manera ramificada los cambios de esquema en todas las bases de datos.

Como cualquier otra aplicación, la aplicación SaaS de Wingtip Tickets evolucionará con el tiempo y requerirá que se hagan cambios en la base de datos. Los cambios pueden afectar los datos de referencia o el esquema, o bien aplicar tareas de mantenimiento de la base de datos. Con una aplicación SaaS con un patrón de base de datos por inquilino, se deben coordinar los cambios en toda una cantidad potencialmente enorme de bases de datos de inquilino. Además, debe incorporar estos cambios en el proceso de aprovisionamiento de base de datos para garantizar que se incluyen en las bases de datos nuevas a medida que se crean.

#### <a name="two-scenarios"></a>Dos escenarios

En este tutorial se exploran estos dos escenarios:
- La implementación de actualizaciones de los datos de referencia para todos los inquilinos.
- La recompilación de un índice de la tabla que contiene los datos de referencia.

La característica [Trabajos elásticos](./elastic-jobs-overview.md) de Azure SQL Database se usa para ejecutar estas operaciones entre distintas bases de datos de inquilino. Los trabajos también funcionan en la base de datos de inquilino de "plantilla". En la aplicación Wingtip Tickets de ejemplo, esta base de datos de plantilla se copia para aprovisionar una base de datos de inquilino nueva.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un agente de trabajo.
> * Ejecutar una consulta T-SQL en bases de datos multiinquilino.
> * Actualizar los datos de referencia en todas las bases de datos de inquilino.
> * Crear un índice en una tabla en todas las bases de datos de inquilino.

## <a name="prerequisites"></a>Requisitos previos

- La aplicación de base de datos multiinquilino de Wingtip Tickets ya debe estar implementada:
    - Si desea ver las instrucciones, consulte el primer tutorial, en el que se presenta la aplicación de base de datos multiinquilino Wingtip Tickets de SaaS:<br />[Implementación y exploración de una aplicación multiinquilino con particiones que usa Azure SQL Database](./saas-multitenantdb-get-started-deploy.md).
        - El proceso de implementación se ejecuta durante menos de cinco minutos.
    - Debe tener instalada la versión *multiinquilino con particiones* de Wingtip. Las versiones para *Independiente* y *Base de datos por inquilino* no admiten este tutorial.

- Debe estar instalada la versión más reciente de SQL Server Management Studio (SSMS). [Descarga e instalación de SSMS](/sql/ssms/download-sql-server-management-studio-ssms).

- Azure PowerShell debe estar instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps).

> [!NOTE]
> En este tutorial se usan características del servicio Azure SQL Database que se encuentran en una versión preliminar limitada ([trabajos de Elastic Database](elastic-database-client-library.md)). Si desea seguir este tutorial, envíe su identificador de suscripción a *SaaSFeedback\@microsoft.com* con el asunto Elastic Jobs Preview. Después de recibir la confirmación de que se ha habilitado su suscripción, [descargue e instale los cmdlets de trabajos de versión preliminar más recientes](https://github.com/jaredmoo/azure-powershell/releases). Esta versión preliminar es limitada, por lo que debe ponerse en contacto con *SaaSFeedback\@microsoft.com* para realizar preguntas u obtener soporte técnico relacionados.

## <a name="introduction-to-saas-schema-management-patterns"></a>Introducción a los patrones de administración de esquema de SaaS

El modelo de base de datos multiinquilino con particiones utilizado en este ejemplo habilita una base de datos de inquilino para que contenga uno o más inquilinos. En este ejemplo se analiza la posibilidad de usar una combinación de bases de datos multiinquilino o de un solo inquilino, lo cual permitiría habilitar un modelo *híbrido* de administración de inquilinos. La administración de cambios en estas bases de datos puede ser complicada. La característica [Trabajos elásticos](./elastic-jobs-overview.md) facilita la administración de grandes cantidades de bases de datos. Los trabajos le permiten ejecutar de manera segura y confiable los scripts de Transact-SQL como tareas en un grupo de bases de datos de inquilino. Las tareas son independientes de la interacción o la intervención del usuario. Este método se puede usar para implementar cambios en el esquema o en los datos de referencia comunes en todos los inquilinos de una aplicación. También puede usar Trabajos elásticos para mantener una copia de la base de datos de la plantilla golden. La plantilla se usa para crear inquilinos nuevos, asegurándose siempre de que se usen los datos de referencia y el esquema más recientes.

![pantalla](./media/saas-multitenantdb-schema-management/schema-management.png)

## <a name="elastic-jobs-limited-preview"></a>Versión preliminar limitada de Trabajos elásticos

Hay una nueva versión de Trabajos elásticos que ahora es una característica integrada de Azure SQL Database. Esta nueva versión de Trabajos elásticos ofrece actualmente una versión preliminar limitada. La versión preliminar limitada actualmente admite usar PowerShell para crear un agente de trabajo y T-SQL para crear y administrar trabajos.
> [!NOTE]
> En este tutorial se usan características del servicio SQL Database que se encuentran en una versión preliminar limitada (trabajos de Elastic Database). Si desea seguir este tutorial, envíe su identificador de suscripción a SaaSFeedback@microsoft.com con el asunto Elastic Jobs Preview. Después de recibir la confirmación de que se ha habilitado su suscripción, descargue e instale los cmdlets de trabajos de versión preliminar más recientes. Esta versión preliminar es limitada, por lo que debe ponerse en contacto con SaaSFeedback@microsoft.com para realizar preguntas u obtener soporte técnico relacionados.

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtención del código fuente y los scripts de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) de GitHub. Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="create-a-job-agent-database-and-new-job-agent"></a>Creación de una base de datos de agente de trabajo y un agente de trabajo nuevo

Para este tutorial, es necesario usar PowerShell para crear la base de datos de agente de trabajo y el agente de trabajo. Al igual que la base de datos MSDB que usa el Agente SQL, un agente de trabajo usa una base de datos de Azure SQL Database para almacenar definiciones de trabajos, estados de trabajo y el historial. Una vez que se crea el agente de trabajo, se pueden crear y supervisar trabajos inmediatamente.

1. En **PowerShell ISE**, abra *...\\Learning Modules\\Schema Management\\Demo-SchemaManagement.ps1*.
2. Presione **F5** para ejecutar el script.

El script *Demo-SchemaManagement.ps1* llama al script *Deploy-SchemaManagement.ps1* para crear una base de datos denominada _jobagent_ en el servidor de catálogo. El script luego crea el agente de trabajo, con la base de datos _jobagent_ como parámetro.

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>Creación de un trabajo para implementar nuevos datos de referencia en todos los inquilinos

#### <a name="prepare"></a>Preparación

Cada base de datos de inquilinos incluye un conjunto de los tipos de ubicación en la tabla **VenueTypes**. Cada tipo de ubicación define la clase de eventos que se organizan en una ubicación. Estos tipos de ubicación se corresponden con las imágenes de fondo que se ven en la aplicación de eventos de inquilino.  En este ejercicio, se implementa una actualización en todas las bases de datos para agregar dos tipos de ubicación adicionales: *Motorcycle Racing* (motociclismo) y *Swimming Club* (club de natación).

En primer lugar, revise los tipos de ubicación que se incluyen en cada base de datos de inquilino. Conéctese a una de las bases de datos de inquilino en SQL Server Management Studio (SSMS) y revise la tabla VenueTypes.  También puede consultar esta tabla en el Editor de consultas de Azure Portal, al que se accede desde la página de la base de datos.

1. Abra SSMS y conéctese al servidor de inquilino: *tenants1-dpt-&lt;usuario&gt;.database.windows.net*
1. Para confirmar que *Motorcycle Racing* y *Swimming Club***no están** incluidos actualmente, vaya a la base de datos *contosoconcerthall* en el servidor *tenants1-dpt-&lt;usuario&gt;* y consulte la tabla *VenueTypes*.



#### <a name="steps"></a>Pasos

Ahora cree un trabajo para actualizar la tabla **VenueTypes** en cada base de datos de inquilinos. Para ello, agregue los dos tipos de ubicación nuevos.

Para crear un trabajo nuevo, puede usar el conjunto de procedimientos almacenamientos en el sistema de trabajos que se crearon en la base de datos _jobagent_. Los procedimientos almacenados se crearon cuando se creó el agente de trabajo.

1. En SSMS, conéctese también al servidor de inquilino: tenants1-mt-&lt;user&gt;.database.windows.net

2. Vaya a la base de datos *tenants1*.

3. Consulte la tabla *VenueTypes* para confirmar que ni *Motorcycle Racing* ni *Swimming Club* están todavía en la lista de resultados.

4. Conéctese al servidor de catálogo, que es *catalog-mt-&lt;user&gt;.database.windows.net*.

5. Conéctese a la base de datos _jobagent_ en el servidor de catálogo.

6. En SSMS, abra el archivo *...\\Learning Modules\\Schema Management\\DeployReferenceData.sql*.

7. Modifique la instrucción: set @User = &lt;user&gt; y sustituya el valor de usuario utilizado al implementar la aplicación SaaS de base de datos multiinquilino Wingtip Tickets.

8. Presione **F5** para ejecutar el script.

#### <a name="observe"></a>Observación

Observe los elementos siguientes en el script *DeployReferenceData.sql*:

- **sp\_add\_target\_group** crea el nombre del grupo de destino *DemoServerGroup* y agrega miembros de destino al grupo.

- **sp\_add\_target\_group\_member** agrega los elementos siguientes:
    - Un tipo de miembro de destino de *servidor*.
        - Se trata del servidor *tenants1-mt-&lt;user&gt;* que contiene las bases de datos de inquilinos.
        - Al incluir el servidor, se incluyen las bases de datos de inquilino que existen en el momento en que se ejecuta el trabajo.
    - Un tipo de miembro de destino de *base de datos* para la base de datos de plantilla (*basetenantdb*) que reside en el servidor on *catalog-mt-&lt;usuario&gt;* .
    - Un tipo de miembro de destino de *base de datos* para incluir la base de datos *adhocreporting* que se usa en un tutorial posterior.

- **sp\_add\_job** crea un trabajo denominado *Reference Data Deployment*.

- **sp\_add\_jobstep** crea el paso de trabajo que contiene el texto del comando T-SQL para actualizar la tabla de referencia VenueTypes.

- En las demás vistas del script se muestran los objetos existentes y se supervisa la ejecución de los trabajos. Use estas consultas para revisar el valor de estado en la columna **ciclo de vida** a fin de determinar si el trabajo finalizó. El trabajo actualiza la base de datos de inquilinos y actualiza las dos bases de datos adicionales que contienen la tabla de referencia.

En SSMS, vaya a la base de datos de inquilino del servidor *tenants1-mt-&lt;user&gt;* . Consulte la tabla *VenueTypes* para confirmar que ni *Motorcycle Racing* ni *Swimming Club* ahora se agregaron a la tabla. El número total de tipos de ubicación debe haber aumentado en dos.

## <a name="create-a-job-to-manage-the-reference-table-index"></a>Creación de un trabajo para administrar el índice de tabla de referencia

Este ejercicio crea un trabajo para recompilar el índice en la clave principal de la tabla de referencia en todas las bases de datos de inquilino. La recompilación de un índice es una operación de administración de bases de datos típica que un administrador puede ejecutar después de cargar una gran cantidad de datos con el fin de mejorar el rendimiento.

1. En SSMS, conéctese a la base de datos _jobagent_ en el servidor *catalog-mt-&lt;usuario&gt;.database.windows.net*.

2. En SSMS, abra *...\\Learning Modules\\Schema Management\\OnlineReindex.sql*.

3. Presione **F5** para ejecutar el script.

#### <a name="observe"></a>Observación

Observe los elementos siguientes en el script *OnlineReindex.sql*:

* **sp\_add\_job** crea un trabajo denominado *Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885*.

* **sp\_add\_jobstep** crea el paso de trabajo que contiene el texto del comando T-SQL para actualizar el índice.

* Las demás vistas del script permiten supervisar la ejecución del trabajo. Use estas consultas para revisar el valor de estado en la columna **ciclo de vida** a fin de determinar si el trabajo ha finalizado correctamente en todos los miembros del grupo de destino.

## <a name="additional-resources"></a>Recursos adicionales

<!-- TODO: Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment (*Tutorial link to come*)
(saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->
* [Administración de bases de datos escaladas horizontalmente en la nube](./elastic-jobs-overview.md)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> * Crear un agente de trabajo para ejecutar trabajos T-SQL en varias bases de datos
> * Actualizar los datos de referencia en todas las bases de datos de inquilino
> * Crear un índice en una tabla en todas las bases de datos de inquilino

A continuación, pruebe el [tutorial de notificaciones ad hoc](./saas-multitenantdb-adhoc-reporting.md) para explorar la ejecución de consultas distribuidas en las bases de datos de inquilino.