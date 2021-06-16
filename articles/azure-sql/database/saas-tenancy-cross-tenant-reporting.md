---
title: Notificación de consultas en varias bases de datos
description: Notificación entre inquilinos mediante consultas distribuidas.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewers: ''
ms.date: 01/25/2019
ms.openlocfilehash: e6e9a3e47dd35916a1869d72a2267defe63ec44b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697477"
---
# <a name="cross-tenant-reporting-using-distributed-queries"></a>Notificación entre inquilinos mediante consultas distribuidas.
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, se ejecutan consultas distribuidas en todo el conjunto de bases de datos de inquilino para la notificación. Estas consultas pueden extraer información incluida en los datos operativos diarios de los inquilinos SaaS Wingtip Tickets. Para ello, implemente una base de datos de notificación adicional en el servidor de catálogo y use una consulta elástica para habilitar las consultas distribuidas.


En este tutorial, obtendrá información:

> [!div class="checklist"]
> 
> * Implementación de una base de datos de notificaciones
> * Acerca de cómo ejecutar consultas distribuidas en todas las bases de datos de inquilinos
> * Cómo pueden habilitar las vistas globales en cada base de datos la realización eficiente de consultas en varios inquilinos


Para completar este tutorial, asegúrese de cumplir estos requisitos previos:


* Aplicación Wingtip Tickets SaaS Database Per Tenant implementada. Para implementarla en menos de cinco minutos, consulte el artículo de [implementación y exploración de la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)
* SQL Server Management Studio (SSMS) está instalado. Para descargar e instalar SSMS, consulte [Descarga de SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).


## <a name="cross-tenant-reporting-pattern"></a>Patrón de notificación entre inquilinos

![patrón de consulta distribuida entre inquilinos](./media/saas-tenancy-cross-tenant-reporting/cross-tenant-distributed-query.png)

Una oportunidad que brindan las aplicaciones SaaS es el uso de la gran cantidad de datos de inquilino almacenados en la nube para obtener información del funcionamiento y el uso de la aplicación. Esta información puede guiarle en el desarrollo de características, mejoras de facilidad de uso y otras inversiones en sus aplicaciones y servicios.

Es fácil obtener acceso a estos datos en una sola base de datos multiinquilino, pero no es tan fácil cuando se distribuyen a escala en miles de bases de datos. Un enfoque es usar la [consulta elástica](elastic-query-overview.md), que permite consultas en un conjunto distribuido de bases de datos con un esquema común. Estas bases de datos se pueden distribuir en distintos grupos de recursos y suscripciones, pero deben compartir el inicio de sesión. La consulta elástica usa una única base de datos *principal* en la que se definen tablas externas que reflejan tablas o vistas en las bases de datos (de inquilino) distribuidas. Las consultas que se envían a esta base de datos principal se compilan para producir un plan de consulta distribuido, con partes de la consulta aplicadas en las bases de datos de inquilino según sea necesario. La consulta elástica usa el mapa de particiones de la base de datos del catálogo para determinar la ubicación de todas las bases de datos de inquilino. La configuración y la consulta de la base de datos principal son sencillas y utilizan [Transact-SQL](/sql/t-sql/language-reference) estándar; además, admiten consultas desde herramientas como Power BI y Excel.

Mediante la distribución de las consultas en las bases de datos de inquilino, la consulta elástica proporciona una visión inmediata de los datos de producción activos. Como la consulta elástica puede extraer los datos de muchas bases de datos, la latencia de las consultas puede ser mayor que las consultas equivalentes enviadas a una sola base de datos multiinquilino. Diseñe las consultas para minimizar los datos que se devuelven a la base de datos principal. La consulta elástica a menudo resulta más apropiada para consultar pequeñas cantidades de datos en tiempo real, en lugar de generar informes o consultas de análisis complejos o usados con frecuencia. Si las consultas no funcionan correctamente, examine el [plan de ejecución](/sql/relational-databases/performance/display-an-actual-execution-plan) para ver qué parte de la consulta se ha transferido a la base de datos remota y cuántos datos se devuelven. Las consultas que requieren un procesamiento analítico o de agregación complejo pueden ser mejor atendidas si se extraen los datos de inquilino en una base de datos o un almacenamiento de datos optimizado para las consultas de análisis. Este enfoque se explica en el [tutorial de análisis de inquilino](saas-tenancy-tenant-analytics.md). 

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtención de los scripts de la aplicación Wingtip Tickets SaaS Database Per Tenant

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="create-ticket-sales-data"></a>Creación de datos de ventas de entradas

Para ejecutar consultas en un conjunto de datos más interesante, ejecute el generador de entradas para crear datos de ventas de entradas.

1. En *PowerShell ISE*, abra el script ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1* y establezca el valor siguiente:
   * **$DemoScenario** = 1, **Purchase tickets for events at all venues** (comprar entradas para eventos de todas las ubicaciones).
2. Presione **F5** para ejecutar el script y generar datos de ventas de entradas. Mientras se ejecuta el script, siga con los pasos de este tutorial. Los datos de entradas se consultan en la sección *Ejecución de consultas distribuidas ad hoc*, de modo que espere a que el generador de entradas finalice.

## <a name="explore-the-global-views"></a>Exploración de las vistas globales

En la aplicación Wingtip Tickets SaaS Database Per Tenant, cada inquilino tiene una base de datos. Por lo tanto, los datos de las tablas de base de datos se limita a la perspectiva de un solo inquilino. Sin embargo, al consultar en todas las bases de datos, es importante que la consulta elástica pueda tratar los datos como si formaran parte de una sola base de datos lógica compartida por el inquilino. 

Para simular este patrón, se agrega un conjunto de vistas "globales" a la base de datos de inquilino que proyectan un identificador de inquilino en cada una de las tablas que se consultan globalmente. Por ejemplo, la vista *VenueEvents* agrega un identificador *VenueId* calculado a las columnas que se proyectan desde la tabla *Events*. Análogamente, las vistas *VenueTicketPurchases* y *VenueTickets* agregan una columna *VenueId* calculada proyectada desde sus respectivas tablas. Estas vistas las utiliza la consulta elástica para usar consultas en paralelo e insertarlas en la base de datos remota de inquilinos cuando hay una columna *VenueId* presente. Así se reduce considerablemente la cantidad de datos que se devuelven, lo que da como resultado un aumento sustancial del rendimiento para muchas consultas. Estas vistas globales se han creado previamente en todas las bases de datos de inquilino.

1. Abra SSMS y [conéctese al servidor tenants1-&lt;USUARIO&gt;](saas-tenancy-wingtip-app-guidance-tips.md#explore-database-schema-and-execute-sql-queries-using-ssms).
1. Expanda **Bases de datos**, haga clic con el botón derecho en _contosoconcerthall_ y seleccione **Nueva consulta**.
1. Ejecute las siguientes consultas para explorar la diferencia entre las tablas de inquilino único y las vistas globales:

   ```T-SQL
   -- The base Venue table, that has no VenueId associated.
   SELECT * FROM Venue

   -- Notice the plural name 'Venues'. This view projects a VenueId column.
   SELECT * FROM Venues

   -- The base Events table, which has no VenueId column.
   SELECT * FROM Events

   -- This view projects the VenueId retrieved from the Venues table.
   SELECT * FROM VenueEvents
   ```

En estas vistas, el identificador *VenueId* se ha calculado como un valor hash del nombre de Venue, pero se puede usar cualquier estrategia para introducir un valor único. Esta estrategia es similar a cómo se calcula la clave de inquilino para usarse en el catálogo.

Para examinar la definición de la vista *Venues*:

1. En **Explorador de objetos**, expanda **contosoconcerthall** > **Vistas**:

   ![En la captura de pantalla se muestra el contenido del nodo Vistas, incluidos cuatro tipos de dbo Venue.](./media/saas-tenancy-cross-tenant-reporting/views.png)

2. Haga clic con el botón derecho en **dbo.Venues**.
3. Seleccione **Incluir vista como** > **CREATE To** > **Nueva ventana del Editor de consultas**.

Incluya en el script cualquiera de las otras vistas *Venue* para ver cómo agregan el identificador *VenueId*.

## <a name="deploy-the-database-used-for-distributed-queries"></a>Implementación de la base de datos usada para las consultas distribuidas

En este ejercicio se implementa la base de datos _adhocreporting_. Esta es la base de datos principal que contendrá el esquema utilizado para realizar consultas en todas las bases de datos de inquilino. La base de datos se implementa en el servidor de catálogo existente, que es el utilizado para todas las bases de datos relacionadas con la administración en la aplicación de ejemplo.

1. En *PowerShell ISE*, abra ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1*. 

1. Establezca **$DemoScenario = 2**, _Deploy Ad-hoc reporting database_.

1. Presione **F5** para ejecutar el script y crear la base de datos *adhocreporting*.

En la siguiente sección, se agrega el esquema a la base de datos de modo que se pueda usar para ejecutar consultas distribuidas.

## <a name="configure-the-head-database-for-running-distributed-queries"></a>Configuración de la base de datos "principal" para ejecutar consultas distribuidas

En este ejercicio se agrega el esquema (el origen de datos externo y las definiciones de tabla externas) a la base de datos _adhocreporting_ que permite realizar consultas en todas las bases de datos de inquilino.

1. Abra SQL Server Management Studio y conéctese a la base de datos de notificaciones ad hoc que creó en el paso anterior. El nombre de la base de datos será *adhocreporting*.
2. Abra ...\Learning Modules\Operational Analytics\Adhoc Reporting\ _Initialize-AdhocReportingDB.sql_ en SSMS.
3. Revise el script SQL y tenga en cuenta lo siguiente:

   La consulta elástica usa una credencial de ámbito de base de datos para acceder a cada una de las bases de datos de inquilino. Esta credencial debe estar disponible en todas las bases de datos y normalmente se le deberían conceder los derechos mínimos necesarios para permitir estas consultas.

    ![crear credencial](./media/saas-tenancy-cross-tenant-reporting/create-credential.png)

   Con la base de datos del catálogo como origen de datos externo, las consultas se distribuyen a todas las bases de datos registradas en el catálogo cuando se ejecuta la consulta. Dado que los nombres de servidor son diferentes para cada implementación, este script obtiene la ubicación de la base de datos de catálogo del servidor actual (@@servername) donde se ejecuta el script.

    ![crear origen de datos externos](./media/saas-tenancy-cross-tenant-reporting/create-external-data-source.png)

   Las tablas externas que hacen referencia a las vistas globales descritas en la sección anterior y definidas con **DISTRIBUTION = SHARDED(VenueId)** . Dado que cada identificador *VenueId* se asigna a una base de datos individual, se mejora el rendimiento de muchos escenarios, tal como se muestra en la sección siguiente.

    ![crear tablas externas](./media/saas-tenancy-cross-tenant-reporting/external-tables.png)

   La tabla local _VenueTypes_ que se crea y rellena. Esta tabla de datos de referencia es común en todas las bases de datos de inquilino, por lo que se puede representar como una tabla local y rellenarse con los datos comunes. Para algunas consultas, el hecho de contar con esta tabla definida en la base de datos principal puede reducir la cantidad de datos que debe mover a la base de datos principal.

    ![crear tabla](./media/saas-tenancy-cross-tenant-reporting/create-table.png)

   Si incluye tablas de referencia de esta manera, asegúrese de actualizar el esquema de tabla y los datos cada vez que actualice las bases de datos de inquilino.

4. Presione **F5** para ejecutar el script e inicializar la base de datos *adhocreporting*. 

Ahora puede ejecutar consultas distribuidas y recopilar información de todos los inquilinos.

## <a name="run-distributed-queries"></a>Ejecución de consultas distribuidas

Ahora que la base de datos *adhocreporting* está configurada, continúe y ejecute algunas consultas distribuidas. Incluya el plan de ejecución para comprender mejor dónde sucede el procesamiento de las consultas. 

Al inspeccionar el plan de ejecución, mantenga el mouse sobre los iconos de plan para obtener más información. 

Es importante tener en cuenta que, si configura **DISTRIBUTION = SHARDED(VenueId)** al definir el origen de datos externo, se mejora el rendimiento de muchos escenarios. Dado que cada identificador *VenueId* se asigna a una base de datos individual, el filtrado se lleva a cabo fácilmente de forma remota y devuelve solo los datos necesarios.

1. Abra ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReportingQueries.sql* en SSMS.
2. Asegúrese de que está conectado a la base de datos **adhocreporting**.
3. Seleccione el menú **Consulta** y haga clic en **Incluir plan de ejecución real**.
4. Resalte la consulta *Which venues are currently registered?* (¿Qué ubicaciones están registradas actualmente?) y presione **F5**.

   La consulta devuelve la lista de ubicaciones completa e ilustra lo rápido y fácil que es realizar consultas en todos los inquilinos y devolver los datos de cada uno.

   Inspeccione el plan y compruebe que el costo íntegro está en la consulta remota. Cada base de datos de inquilino ejecuta la consulta de manera remota y devuelve su información a la base de datos principal.

   ![SELECT * FROM dbo.Venues](./media/saas-tenancy-cross-tenant-reporting/query1-plan.png)

5. Seleccione la siguiente consulta y presione **F5**.

   Esta consulta combina datos de las bases de datos de inquilino y de la tabla local *VenueTypes* (local porque es una tabla de la base de datos *adhocreporting*).

   Inspeccione el plan y observe que la mayoría del costo se corresponde con la consulta remota. Cada base de datos de inquilino devuelve su información sobre la ubicación y realiza una combinación local con la tabla local *VenueTypes* para mostrar el nombre descriptivo.

   ![Combinación de datos remotos y locales](./media/saas-tenancy-cross-tenant-reporting/query2-plan.png)

6. Ahora seleccione la consulta *On which day were the most tickets sold?* (¿Qué día se vendieron más entradas?) y presione **F5**.

   Esta consulta hace una combinación y una agregación un poco más complejas. La mayor parte del procesamiento se produce de forma remota.  Solo se devuelven filas individuales, que contienen el recuento diario de la venta de entradas de cada lugar, a la base de datos principal.

   ![Query](./media/saas-tenancy-cross-tenant-reporting/query3-plan.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> 
> * Ejecutar consultas distribuidas en todas las bases de datos de inquilinos
> * Implemente una base de datos de notificaciones y defina el esquema necesario para ejecutar consultas distribuidas.


Ahora pruebe el [tutorial de análisis de inquilinos](saas-tenancy-tenant-analytics.md) para explorar la extracción de datos en una base de datos de análisis independiente para llevar a cabo un procesamiento de análisis más complejo.

## <a name="additional-resources"></a>Recursos adicionales

* Otros [tutoriales basados en la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Consulta elástica](elastic-query-overview.md)