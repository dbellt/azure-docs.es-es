---
title: Aprovisionamiento en aplicaciones SaaS multiinquilino
description: Obtenga información sobre cómo aprovisionar y catalogar nuevos inquilinos en una aplicación SaaS multiinquilino de Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 09/24/2018
ms.openlocfilehash: b7fb97bbab252b5a11a1e66e89c717108781b0e2
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699081"
---
# <a name="provision-and-catalog-new-tenants-in-a-saas-application-using-a-sharded-multi-tenant-azure-sql-database"></a>Aprovisionamiento y catalogación de nuevos inquilinos en una aplicación SaaS con una instancia de Azure SQL Database multiinquilino con particiones
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describe el aprovisionamiento y la catalogación de nuevos inquilinos en un modelo o patrón de *base de datos multiinquilino con particiones*.

Este artículo se divide en dos partes principales:

- [Explicación conceptual](#goto_2_conceptual) del aprovisionamiento y la catalogación de nuevos inquilinos.

- [Tutorial](#goto_1_tutorial) que destaca el código de script de PowerShell que lleva a cabo el aprovisionamiento y la catalogación.
  - En el tutorial se usa la aplicación SaaS Wingtip Tickets, adaptada al patrón de base de datos multiinquilino con particiones.

<a name="goto_2_conceptual"></a>

## <a name="database-pattern"></a>Patrón de base de datos

En esta sección, y en otras tantas que vienen después, se analizarán los conceptos del patrón de base de datos multiinquilino con particiones.

En este modelo multiinquilino con particiones, los esquemas de tabla dentro de cada base de datos incluyen una clave de inquilino en la clave principal de las tablas que almacenan datos de inquilino. La clave de inquilino permite que cada base de datos individual almacene 0, 1 o varios inquilinos. El uso de bases de datos con particiones permite que el sistema admita fácilmente gran cantidad de inquilinos. Todos los datos de un inquilino se almacenan en una base de datos. La gran cantidad de inquilinos se distribuye entre las muchas bases de datos con particiones. Una base de datos de catálogo almacena la asignación de cada inquilino a su base de datos.

#### <a name="isolation-versus-lower-cost"></a>Aislamiento frente a bajo costo

Un inquilino que tiene una base de datos toda para él solo disfruta de las ventajas del aislamiento. La base de datos del inquilino puede restaurarse a una fecha anterior sin las limitaciones de que otros inquilinos resulten afectados. El rendimiento de la base de datos se puede ajustar para optimizar ese único inquilino, de nuevo, sin comprometer otros inquilinos. El problema es que el aislamiento cuesta más de lo que cuesta compartir una base de datos con otros inquilinos.

Cuando se aprovisiona un nuevo inquilino, este puede compartir una base de datos con otros inquilinos, o bien se le puede colocar en su propia base de datos nueva. Más adelante puede cambiar de opinión y mover la base de datos a la otra situación.

Las bases de datos con varios inquilinos e inquilinos únicos se pueden combinar en la misma aplicación SaaS para optimizar el costo o el aislamiento de cada inquilino.

   ![Aplicación de base de datos multiinquilino con particiones con catálogo de inquilinos](./media/saas-multitenantdb-provision-and-catalog/MultiTenantCatalog.png)

## <a name="tenant-catalog-pattern"></a>Patrón de un catálogo de inquilino

Si tiene dos o más bases de datos que contiene cada una al menos un inquilino, la aplicación debe tener una forma de detectar qué base de datos almacena el inquilino que en ese momento interesa. Una base de datos de catálogo almacena esta asignación.

#### <a name="tenant-key"></a>Clave de inquilino

Para cada inquilino, la aplicación Wingtip puede derivar una clave única, que es la clave de inquilino. La aplicación extrae el nombre del inquilino de la dirección URL de la página web. La aplicación aplica un algoritmo hash al nombre para obtener la clave. La aplicación usa la clave para acceder al catálogo. El catálogo crea referencias cruzadas a la información sobre la base de datos en la que está almacenado el inquilino. La aplicación usa la información de la base de datos para conectarse. También se pueden usar otros esquemas de claves de inquilino.

Usar un catálogo permite cambiar el nombre o la ubicación de una base de datos de inquilino después del aprovisionamiento sin interrumpir la aplicación. En un modelo de base de datos multiinquilino, el catálogo permite mover un inquilino entre bases de datos.

#### <a name="tenant-metadata-beyond-location"></a>Metadatos de inquilino más allá de ubicación

El catálogo también puede indicar si un inquilino está sin conexión debido a operaciones de mantenimiento u otras acciones. Y el catálogo se puede ampliar para almacenar metadatos de inquilino o base de datos adicionales, como los siguientes elementos:
- El nivel de servicio o la edición de una base de datos.
- La versión del esquema de la base de datos.
- El nombre del inquilino y su SLA (Acuerdo de Nivel de Servicio).
- Información para permitir la administración de aplicaciones, el soporte técnico al cliente o los procesos de DevOps.

El catálogo también se puede usar para habilitar la generación de informes entre inquilinos, la administración de esquemas y la extracción de datos con fines de análisis.

### <a name="elastic-database-client-library"></a>Biblioteca de cliente de Elastic Database

En Wingtip, el catálogo se implementa en la base de datos *tenantcatalog*. La base de datos *tenantcatalog* se crea mediante las características de administración de particiones de la [biblioteca de cliente de Elastic Database (EDCL)](elastic-database-client-library.md). La biblioteca permite que una aplicación pueda crear, administrar y usar un *mapa de particiones* que se almacena en una base de datos. Un mapa de particiones crea referencias cruzadas de la clave de inquilino con su partición, es decir, su base de datos con particiones.

Durante el aprovisionamiento del inquilino, se pueden usar las funciones de EDCL desde aplicaciones o scripts de PowerShell para crear las entradas en el mapa de particiones. Posteriormente, estas funciones se pueden usar para conectarse a la base de datos correcta. La biblioteca EDCL almacena en caché la información de conexión para reducir el tráfico en la base de datos de catálogo y acelerar el proceso de conexión.

> [!IMPORTANT]
> *No* edite los datos de la base de datos de catálogo mediante acceso directo. No se permiten actualizaciones directas debido al alto riesgo de daños en los datos. En su lugar, edite los datos de asignación solo mediante las API de EDCL.

## <a name="tenant-provisioning-pattern"></a>Patrón de aprovisionamiento de inquilinos

#### <a name="checklist"></a>Lista de comprobación

Cuando quiera aprovisionar un nuevo inquilino en una base de datos con particiones existente, debe hacerse las siguientes preguntas respecto a la base de datos:
- ¿Tiene suficiente espacio para el nuevo inquilino?
- ¿Tiene tablas con los datos de referencia necesarios para el nuevo inquilino, o se pueden agregar los datos?
- ¿Tiene la variación adecuada del esquema base para el nuevo inquilino?
- ¿Está en la ubicación geográfica adecuada cerca del nuevo inquilino?
- ¿Es en el nivel de servicio adecuado para el nuevo inquilino?

Si quiere que el nuevo inquilino esté aislado en su propia base de datos, puede crearlo para que cumpla las especificaciones del inquilino.

Tras completar el aprovisionamiento, debe registrar el inquilino en el catálogo. Por último, se puede agregar la asignación de inquilino para hacer referencia a la partición correspondiente.

#### <a name="template-database"></a>Base de datos de plantilla

Para aprovisionar la base de datos, ejecute los scripts SQL, implemente un elemento bacpac o copie una base de datos de plantilla. Las aplicaciones de Wingtip copian una base de datos de plantilla para crear nuevas bases de datos de inquilino.

Al igual que cualquier aplicación, Wingtip evoluciona con el tiempo. En ocasiones, Wingtip deberá realizar cambios en la base de datos. Los cambios pueden incluir los siguientes elementos:
- Esquema nuevo o modificado.
- Datos de referencia nuevos o modificados.
- Tareas rutinarias de mantenimiento de la base de datos para garantizar un rendimiento óptimo de la aplicación.

Con una aplicación SaaS, estos cambios deben implementarse de manera coordinada a lo largo de una línea potencialmente masiva de bases de datos de inquilino. Para que estos cambios estén presentes en futuras bases de datos de inquilino, deben incorporarse en el proceso de aprovisionamiento. Este reto se explora con más detalle en el [tutorial de administración de esquemas](saas-tenancy-schema-management.md).

#### <a name="scripts"></a>Scripts

Los scripts de aprovisionamiento de inquilinos de este tutorial admiten los siguientes escenarios:
- Aprovisionamiento de un inquilino en una base de datos existente compartida con otros inquilinos.
- Aprovisionamiento de un inquilino en su propia base de datos.

Luego, los datos de inquilino se inicializan y registran en el mapa de particiones del catálogo. En la aplicación de ejemplo, las bases de datos que contienen varios inquilinos reciben un nombre genérico, como *tenants1* o *tenants2*. Las bases de datos que contienen un único inquilino reciben el nombre del inquilino. Las convenciones de nomenclatura específicas que se usan en el ejemplo no son una parte fundamental del patrón, porque el uso de un catálogo permite asignar cualquier nombre a la base de datos.

<a name="goto_1_tutorial"></a>

## <a name="tutorial-begins"></a>Comienzo del tutorial

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Aprovisionar un inquilino en una base de datos multiinquilino
> * Aprovisionar un inquilino en una base de datos de un único inquilino
> * Aprovisionar un lote de inquilinos en bases de datos multiinquilino y en bases de datos de un único inquilino
> * Registrar una asignación de inquilino y base de datos en un catálogo

#### <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

- Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)

- La aplicación SaaS de base de datos multiinquilino Wingtip Tickets está implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets](./saas-multitenantdb-get-started-deploy.md)

- Obtenga los scripts y el código fuente de Wingtip:
    - Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB).
    - Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts de Wingtip.

## <a name="provision-a-tenant-into-a-database-shared-with-other-tenants"></a>Aprovisionamiento de un inquilino en una base de datos *compartida* con otros inquilinos

En esta sección, verá una lista de las acciones principales de aprovisionamiento que se realizan mediante los scripts de PowerShell. A continuación, usará al depurador de PowerShell ISE para recorrer los scripts y ver las acciones del código.

#### <a name="major-actions-of-provisioning"></a>Acciones principales de aprovisionamiento

A continuación se muestran los elementos clave del flujo de trabajo de aprovisionamiento que recorre:

- **Cálculo de la nueva clave de inquilino**: Para crear la clave de inquilino a partir del nombre del inquilino se utiliza una función hash.
- **Comprobación de si ya existe la clave de inquilino**: El catálogo se comprueba para garantizar que todavía no se registró la clave.
- **Inicialización del inquilino en la base de datos de inquilino predeterminada**: La base de datos de inquilino se actualiza para agregar la información del inquilino nuevo.
- **Registro del inquilino en el catálogo**: la asignación entre la clave de inquilino nueva y la base de datos tenants1 existente se agrega al catálogo.
- **Incorporación del nombre del inquilino a una tabla de extensión de catálogo**: El nombre de la ubicación se agrega a la tabla Inquilinos del catálogo.  Esta adición muestra cómo la base de datos de catálogo puede ampliarse para admitir datos adicionales específicos de la aplicación.
- **Apertura de la página Eventos para el inquilino nuevo**: La página de eventos de *Bushwillow Blues* se abre en el explorador.

   ![Captura de pantalla que muestra la página de eventos de un nuevo inquilino.](./media/saas-multitenantdb-provision-and-catalog/bushwillow.png)

#### <a name="debugger-steps"></a>Pasos del depurador

Para comprender cómo la aplicación Wingtip implementa el aprovisionamiento de un inquilino nuevo en una base de datos compartida, agregue un punto de interrupción y recorra el flujo de trabajo:

1. En *PowerShell ISE*, abra ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* y establezca los parámetros siguientes:
   - **$TenantName** = **Bushwillow Blues**, el nombre de la ubicación nueva.
   - **$VenueType** = **blues**, uno de los tipos de ubicación predefinidos: blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer (en minúsculas, sin espacios).
   - **$DemoScenario** = **1**, para aprovisionar un inquilino en una base de datos compartida con otros inquilinos.

2. Agregue un punto de interrupción colocando el cursor en cualquier lugar de la línea 38, que dice: *Nuevo inquilino '* y, a continuación, presione **F9**.

   ![Captura de pantalla que resalta la línea que incluye el nuevo inquilino.](./media/saas-multitenantdb-provision-and-catalog/breakpoint.png)

3. Para ejecutar los scripts, presione **F5**.

4. Después de que la ejecución del script se detiene en el punto de interrupción, presione **F11** para depurar paso a paso el código por instrucciones.

   ![Captura de pantalla que muestra Windows PowerShell ISE con el menú Depurar abierto y la opción Depurar paso a paso por instrucciones seleccionada.](./media/saas-multitenantdb-provision-and-catalog/debug.png)

5. Siga paso a paso la ejecución del script mediante las opciones del menú **Depurar**, **F10** y **F11**, para omitir las funciones llamadas o acceder a ellas.

Para obtener más información sobre cómo depurar scripts de PowerShell, consulte [Sugerencias para trabajar con scripts de PowerShell y depurarlos](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).

## <a name="provision-a-tenant-in-its-own-database"></a>Aprovisionamiento de un inquilino en su *propia* base de datos

#### <a name="major-actions-of-provisioning"></a>Acciones principales de aprovisionamiento

A continuación se muestran los elementos clave del flujo de trabajo que recorre mientras se hace seguimiento del script:

- **Cálculo de la nueva clave de inquilino**: Para crear la clave de inquilino a partir del nombre del inquilino se utiliza una función hash.
- **Comprobación de si ya existe la clave de inquilino**: El catálogo se comprueba para garantizar que todavía no se registró la clave.
- **Creación de una base de datos de inquilino nueva**: La base de datos se crea al copiar la base de datos *basetenantdb* con una plantilla de Resource Manager.  El nombre de la base de datos nueva se basa en el nombre del inquilino.
- **Incorporación de la base de datos al catálogo**: La base de datos de inquilino nueva se registra como una partición en el catálogo.
- **Inicialización del inquilino en la base de datos de inquilino predeterminada**: La base de datos de inquilino se actualiza para agregar la información del inquilino nuevo.
- **Registro del inquilino en el catálogo**: la asignación entre la clave de inquilino nueva y la base de datos *sequoiasoccer* existente se agrega al catálogo.
- **El nombre del inquilino se agrega al catálogo**: El nombre de la ubicación se agrega a la tabla de extensión Inquilinos del catálogo.
- **Apertura de la página Eventos para el inquilino nuevo**: La página de eventos de *Sequoia Soccer* se abre en el explorador.

   ![events](./media/saas-multitenantdb-provision-and-catalog/sequoiasoccer.png)

#### <a name="debugger-steps"></a>Pasos del depurador

Ahora, recorra el proceso de script al crear un inquilino en su propia base de datos:

1. Todavía en ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1*, establezca los parámetros siguientes:
   - **$TenantName** = **Sequoia Soccer**, el nombre de la ubicación nueva.
   - **$VenueType** = **soccer**, uno de los tipos de ubicación predefinidos: blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer (en minúsculas, sin espacio).
   - **$DemoScenario** = **2** para aprovisionar un inquilino en su propia base de datos.

2. Agregue un punto de interrupción nuevo colocando el cursor en cualquier lugar de la línea 57, que dice: *&&nbsp;$PSScriptRoot\New-TenantAndDatabase `* , y presione **F9**.

   ![punto de interrupción](./media/saas-multitenantdb-provision-and-catalog/breakpoint2.png)

3. Para ejecutar los scripts, presione **F5**.

4. Después de que la ejecución del script se detiene el punto de interrupción, presione **F11** para depurar paso a paso el código por instrucciones.  Use **F10** y **F11** para omitir las funciones para hacer seguimiento de la ejecución o acceder a ellas.

## <a name="provision-a-batch-of-tenants"></a>Aprovisionamiento de un lote de inquilinos

Este ejercicio aprovisiona un lote de 17 inquilinos. Se recomienda que aprovisione este lote de inquilinos antes de comenzar con otros tutoriales de Wingtip Tickets, de manera que haya más bases de datos con las cuales trabajar.

1. En *PowerShell ISE*, abra ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* y cambie el parámetro *$DemoScenario* a 4:
   - **$DemoScenario** = **4**, para aprovisionar un lote de inquilinos en una base de datos compartida.

2. Presione **F5** y ejecute el script.

### <a name="verify-the-deployed-set-of-tenants"></a>Comprobación del conjunto de inquilinos implementado

En esta fase, se tiene una combinación de inquilinos implementados en una base de datos compartida y los inquilinos implementados en sus propias bases de datos. Se puede usar Azure Portal para inspeccionar las bases de datos creadas. En [Azure Portal](https://portal.azure.com), abra el servidor **tenants1-mt-\<USER\>** mediante la navegación a la lista de servidores SQL Server.  La lista de **bases de datos SQL** debe incluir la base **tenants1** compartida y las bases de datos de los inquilinos que están en su propia base de datos:

   ![lista de base de datos](./media/saas-multitenantdb-provision-and-catalog/Databases.png)

Mientras Azure Portal muestra las bases de datos de inquilino, no le permite ver los inquilinos *dentro* de la base de datos compartida. La lista completa de inquilinos se puede ver en la página web del **centro de eventos** de Wingtip, y examinando el catálogo.

#### <a name="using-wingtip-tickets-events-hub-page"></a>Uso de la página del centro de eventos de Wingtip Tickets

Abra la página del centro de eventos en el explorador (http:events.wingtip-mt.\<USER\>.trafficmanager.net)

#### <a name="using-catalog-database"></a>Uso de la base de datos de catálogo

La lista completa de inquilinos y su base de datos correspondiente están disponibles en el catálogo. Se proporciona una vista SQL que combina el nombre del inquilino con el nombre de la base de datos. La vista muestra perfectamente el valor de extender los metadatos almacenados en el catálogo.
- La vista SQL está disponible en la base de datos tenantcatalog.
- El nombre del inquilino se almacena en la tabla de inquilinos.
- El nombre de la base de datos se almacena en las tablas de administración de particiones.

1. En SQL Server Management Studio (SSMS), conéctese al servidor de inquilinos en **catalog-mt.\<USER\>.database.windows.net**, con el inicio de sesión = **developer**, y la contraseña = **P\@ssword1**.

    ![Cuadro de diálogo de conexión SSMS](./media/saas-multitenantdb-provision-and-catalog/SSMSConnection.png)

2. En el Explorador de objetos de SSMS, examine las vistas de la base de datos *tenantcatalog*.

3. Haga clic con el botón derecho en la vista *TenantsExtended* y elija **Select Top 1000 Rows** (Seleccionar las 1000 primeras filas). Observe la asignación entre el nombre de inquilino y la base de datos para los distintos inquilinos.

    ![Vista ExtendedTenants en SSMS](./media/saas-multitenantdb-provision-and-catalog/extendedtenantsview.png)

## <a name="other-provisioning-patterns"></a>Otros patrones de aprovisionamiento

En esta sección se describen otros patrones interesantes de aprovisionamiento.

#### <a name="pre-provisioning-databases-in-elastic-pools"></a>Aprovisionamiento previo de las bases de datos en grupos elásticos

El patrón de aprovisionamiento previo aprovecha el hecho de que, cuando se usan los grupos elásticos, la facturación se hace en función del grupo y no de las bases de datos. Por lo tanto, las bases de datos se pueden agregar a un grupo elástico antes de que sea necesario incurrir en costos adicionales. Este aprovisionamiento previo reduce considerablemente el tiempo que se tarda en aprovisionar un inquilino en una base de datos. El número de bases de datos previamente aprovisionadas puede ajustarse según sea necesario para mantener un búfer adecuado para la velocidad de aprovisionamiento prevista.

#### <a name="auto-provisioning"></a>Aprovisionamiento automático

En el patrón de aprovisionamiento automático se usa un servicio de aprovisionamiento dedicado para aprovisionar servidores, grupos y bases de datos automáticamente según sea necesario. Esta automatización incluye el aprovisionamiento previo de bases de datos en grupos elásticos. Además, si se retiran y eliminan bases de datos, los espacios que esto crea en los grupos elásticos pueden ocuparse aprovisionando el servicio como se prefiera.

Este tipo de servicio automatizado puede ser simple o complejo. Por ejemplo, la automatización podría administrar el aprovisionamiento en varias regiones geográficas y podría establecer la replicación geográfica para la recuperación ante desastres. Con el patrón de aprovisionamiento automático, un script o aplicación cliente enviaría una solicitud de aprovisionamiento a una cola para que un servicio de aprovisionamiento la procesara. A continuación, el script realizaría un sondeo para detectar la finalización. Si se usa el aprovisionamiento previo, las solicitudes se administrarían rápidamente, mientras que otro servicio en segundo plano administraría el aprovisionamiento de una base de datos de reemplazo.

## <a name="additional-resources"></a>Recursos adicionales

<!-- - Additional [tutorials that build upon the Wingtip SaaS application](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
- [Biblioteca de cliente de base de datos elástica](elastic-database-client-library.md)
- [Cómo depurar scripts en ISE de Windows PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> * Aprovisionar un único inquilino nuevo en una base de datos multiinquilino compartida y su propia base de datos
> * Aprovisionar un lote de inquilinos adicionales
> * Recorrer los detalles sobre cómo aprovisionar inquilinos y registrarlos en el catálogo

Pruebe el [Tutorial de supervisión del rendimiento](./saas-multitenantdb-performance-monitoring.md).