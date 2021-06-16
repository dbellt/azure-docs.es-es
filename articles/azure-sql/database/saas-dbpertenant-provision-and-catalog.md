---
title: Aprovisionamiento de nuevos inquilinos en una aplicación multiinquilino
description: Obtenga información sobre cómo aprovisionar y catalogar nuevos inquilinos en una aplicación SaaS de multiinquilino de Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 09/24/2018
ms.openlocfilehash: 9097ce88ac141bf8c825fb4c720d2a01e0c0029c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699213"
---
# <a name="learn-how-to-provision-new-tenants-and-register-them-in-the-catalog"></a>Aprenda a aprovisionar y registrar nuevos inquilinos en el catálogo
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, aprenderá a aprovisionar y a catalogar patrones de SaaS. También aprenderá cómo se implementan en la aplicación Wingtip Tickets SaaS Database-Per-Tenant. Va a crear e inicializar nuevas bases de datos de inquilinos y las registrará en el catálogo de inquilinos de la aplicación. El catálogo es una base de datos que mantiene la asignación entre los diversos inquilinos de aplicaciones SaaS y sus datos. El catálogo desempeña un papel importante a la hora de dirigir las solicitudes de aplicación y administración a la base de datos correcta.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Aprovisionar un nuevo inquilino único
> * Aprovisionar un lote de inquilinos adicionales


Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* Aplicación Wingtip Tickets SaaS Database Per Tenant implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell está instalado. Para más información, consulte el artículo de [introducción a Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-saas-catalog-pattern"></a>Introducción al patrón de catálogo de SaaS

En una aplicación SaaS multiinquilino con copia de seguridad en base de datos, es importante saber dónde se almacena la información de cada inquilino. En el patrón de catálogo de SaaS, una base de datos de catálogos se usa para mantener la asignación entre cada inquilino y la base de datos en la que se almacenan sus datos. Este patrón se aplica cada vez que los datos de inquilino se distribuyen entre varias bases de datos.

Cada inquilino se identifica como una clave en el catálogo que se asigna a la ubicación de su base de datos. En la aplicación de Wingtip Tickets, la clave se forma a partir de un hash del nombre del inquilino. Este esquema permite que la aplicación construya la clave a partir del nombre de inquilino que se incluye en la dirección URL de la aplicación. También se pueden usar otros esquemas de claves de inquilino.

El catálogo permite cambiar el nombre o la ubicación de la base de datos con un mínimo impacto en la aplicación. En un modelo de base de datos multiinquilino, esta funcionalidad también permite mover a un inquilino entre bases de datos. El catálogo también puede usarse para indicar si un inquilino o base de datos está sin conexión debido a operaciones de mantenimiento u otras acciones. Esta funcionalidad se trata en el [tutorial Restauración de un solo inquilino](saas-dbpertenant-restore-single-tenant.md).

El catálogo también puede almacenar un inquilino adicional o metadatos de la base de datos, como la versión del esquema, el plan de servicio o los SLA que se ofrecen a los inquilinos. El catálogo puede almacenar información adicional que permite la administración de aplicaciones, soporte técnico al cliente o DevOps.

Además de la aplicación de SaaS, el catálogo puede habilitar herramientas de base de datos. En el ejemplo de Wingtip Tickets SaaS Database Per Tenant, el catálogo se usa para habilitar consultas entre inquilinos, que se detalla en el [tutorial de notificaciones ad hoc](saas-tenancy-cross-tenant-reporting.md). La administración de trabajos entre bases de datos se describe en los tutoriales sobre [administración de esquemas](saas-tenancy-schema-management.md) y [análisis de inquilinos](saas-tenancy-tenant-analytics.md).

En los ejemplos de SaaS de Wingtip Tickets, el catálogo se implementa mediante las características de administración de particiones de la [biblioteca de cliente de Elastic Database (EDCL)](elastic-database-client-library.md). La EDCL está disponible en Java y .NET Framework. La EDCL permite que una aplicación pueda crear, administrar y usar un mapa de particiones con copia de seguridad en base de datos.

Un mapa de particiones contiene una lista de particiones (bases de datos) y la asignación entre las claves (inquilinos) y las particiones. Las funciones de EDCL se usan durante el aprovisionamiento de inquilinos para crear las entradas en el mapa de particiones. Las aplicaciones las utilizan en tiempo de ejecución para conectarse a la base de datos correcta. La EDCL almacena en caché la información de conexión para minimizar el tráfico en la base de datos de catálogo y acelerar la aplicación.

> [!IMPORTANT]
> Los datos de asignación son accesibles en la base de datos de catálogo, pero *no los edite*. Para editar datos de asignación, utilice únicamente las API de la biblioteca de cliente de Elastic Database. No se admite la manipulación directa de los datos de asignación, que entraña el riesgo de dañar el catálogo.


## <a name="introduction-to-the-saas-provisioning-pattern"></a>Introducción al patrón de aprovisionamiento de SaaS

Cuando agrega un nuevo inquilino a una aplicación SaaS que usa el modelo de base de datos de un solo inquilino, debe aprovisionar una nueva base de datos de inquilino. La base de datos debe crearse en la ubicación adecuada y el nivel de servicio. También se debe inicializar con el esquema apropiado y los datos de referencia. Y debe registrarse en el catálogo en la clave de inquilino correspondiente.

Se pueden usar diferentes métodos de aprovisionamiento de la base de datos. Puede ejecutar scripts de SQL, implementar un bacpac o copiar una base de datos de la plantilla.

El aprovisionamiento de la base de datos debe formar parte de su estrategia de administración del esquema. Debe asegurarse de que las nuevas bases de datos se aprovisionan con el esquema más reciente. Este requisito se explora en el [tutorial de administración de esquemas](saas-tenancy-schema-management.md).

La aplicación Wingtip Tickets de base de datos por inquilino aprovisiona nuevos inquilinos mediante la copia de una base de datos de plantilla denominada _basetenantdb_, que se implementa en el servidor de catálogo. El aprovisionamiento se puede integrar en la aplicación como parte de la experiencia de registro. También puede admitirse sin conexión mediante el uso de scripts. En este tutorial se describe el aprovisionamiento con PowerShell.

Los scripts de aprovisionamiento copian la base de datos _basetenantdb_  para crear una nueva base de datos de inquilinos en un grupo elástico. La base de datos de inquilinos se crea en el servidor de inquilinos asignado al alias DNS _newtenant_. Este alias conserva una referencia al servidor que se usa para aprovisionar nuevos inquilinos y se actualiza para señalar un servidor de recuperación de inquilinos en los tutoriales de recuperación ante desastres ([recuperación ante desastres mediante restauración geográfica](./saas-dbpertenant-dr-geo-restore.md), [recuperación ante desastres mediante replicación geográfica](./saas-dbpertenant-dr-geo-replication.md)). Los scripts inicializan entonces la base de datos con información específica del inquilino y la registran en el mapa de particiones del catálogo. Las bases de datos de inquilinos reciben nombres basados en el nombre del inquilino. Este esquema de nomenclatura no es una parte fundamental del patrón. El catálogo asigna la clave de inquilino al nombre de la base de datos, de modo que se puede utilizar cualquier convención de nomenclatura.


## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtención de los scripts de la aplicación Wingtip Tickets SaaS Database Per Tenant

Los scripts de SaaS de Wingtip Tickets y el código fuente de la aplicación están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.


## <a name="provision-and-catalog-detailed-walkthrough"></a>Tutorial detallado de aprovisionamiento y catalogación

Para entender cómo la aplicación Wingtip Tickets implementa el aprovisionamiento de un nuevo inquilino, agregue un punto de interrupción y siga el flujo de trabajo mientras aprovisiona un inquilino.

1. En PowerShell ISE, abra ...\\Learning Modules\\ProvisionAndCatalog\\_Demo-ProvisionAndCatalog.ps1_ y establezca los parámetros siguientes:

   * **$TenantName** = el nombre de la nueva ubicación (por ejemplo, *Bushwillow Blues*).
   * **$VenueType** = uno de los tipos predefinidos de ubicación: _blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer_.
   * **$DemoScenario** = **1**, *aprovisionar un único inquilino*.

2. Para agregar un punto de interrupción, coloque el cursor en cualquier lugar de la línea, que dice: *New-Tenant `* . Después, presione F9.

   ![Captura de pantalla que muestra un script con "New-Tenant" resaltado para agregar un punto de interrupción.](./media/saas-dbpertenant-provision-and-catalog/breakpoint.png)

3. Presione F5 para ejecutar el script.

4. Después de que la ejecución del script detiene el punto de interrupción, presione F11 para depurar paso a paso el código por instrucciones.

   ![Depuración](./media/saas-dbpertenant-provision-and-catalog/debug.png)



Seguimiento de la ejecución del script mediante el uso de las opciones de menú **Depurar**. Presione F10 y F11 para omitir o acceder a las funciones llamadas. Para obtener más información sobre cómo depurar scripts de PowerShell, consulte [Sugerencias para trabajar con scripts de PowerShell y depurarlos](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).


No es necesario seguir explícitamente este flujo de trabajo. En él se explica cómo depurar el script.

* **Importación del módulo CatalogAndDatabaseManagement.psm1.** Proporciona un catálogo y una abstracción de nivel de inquilino sobre las funciones de [administración de particiones](elastic-scale-shard-map-management.md). En este módulo se encapsula gran parte del patrón del catálogo y merece la pena explorarlo.
* **Importación del módulo SubscriptionManagement.psm1.** Contiene funciones para iniciar sesión en Azure y seleccionar la suscripción a Azure con la que desea trabajar.
* **Obtención de los detalles de configuración.** Acceda a Get-Configuration (con F11) y vea cómo se especifica la configuración de la aplicación. Aquí se definen los nombres de recursos y otros valores específicos de la aplicación. No cambie estos valores hasta que esté familiarizado con los scripts.
* **Obtención del objeto de catálogo.** Vaya a Get-Catalog que crea y devuelve un objeto de catálogo que se usa en el script de nivel superior. Esta función utiliza las funciones de administración de particiones que se importan desde **AzureShardManagement.psm1**. El objeto de catálogo se compone de los elementos siguientes:

   * $catalogServerFullyQualifiedName se construye con la raíz estándar junto con su nombre de usuario: _catalog-\<user\>.database.windows.net_.
   * $catalogDatabaseName se obtiene de la configuración: *tenantcatalog*.
   * El objeto $shardMapManager se inicializa desde la base de datos del catálogo.
   * El objeto $shardMap se inicializa desde el mapa de particiones _tenantcatalog_ de la base de datos del catálogo. Un objeto de catálogo se compone y se devuelve. Se usa en el script de nivel superior.
* **Cálculo de la nueva clave de inquilino.** Para crear la clave de inquilino a partir del nombre del inquilino se utiliza una función hash.
* **Comprobación de si ya existe la clave de inquilino.** Se comprueba el catálogo para asegurarse de que la clave está disponible.
* **La base de datos de inquilino se aprovisiona con New-TenantDatabase**. Use F11 para acceder y ver cómo la base de datos se aprovisiona mediante una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

    El nombre de la base de datos se construye a partir del nombre del inquilino para dejar claro qué partición pertenece a cada inquilino También puede usar otras convenciones de nomenclatura de la base de datos. Una plantilla de Resource Manager crea una base de datos de inquilino mediante la copia de una base de datos de plantilla (_basetenantdb_) en el servidor de catálogo. Como alternativa, puede crear una base de datos e inicializarla mediante la importación de un bacpac. O bien, puede ejecutar un script de inicialización desde una ubicación conocida.

    La plantilla de Resource Manager se encuentra en la carpeta ...\Learning Modules\Common: *tenantdatabasecopytemplate.json*.

* **La base de datos de inquilinos se inicializa después.** Se agregan el nombre de ubicación (inquilino) y el tipo de ubicación. También puede hacer otra inicialización aquí.

* **La base de datos de inquilino se registra en el catálogo.** Se registra con *Add-TenantDatabaseToCatalog*  mediante el uso de la clave de inquilino. Use F11 para depurar paso a paso los detalles por instrucciones:

    * La base de datos de catálogo se agrega a la asignación de particiones (la lista de bases de datos conocidas).
    * Se crea la asignación que vincula el valor de clave a la partición.
    * Se agregan metadatos adicionales sobre el inquilino (nombre de la ubicación) a la tabla Tenants del catálogo. Esta tabla no forma parte del esquema de administración de particiones y la EDCL no la instala. Esta tabla muestra cómo la base de datos de catálogo puede ampliarse para admitir los datos adicionales específicos de la aplicación.


Después de que se complete el aprovisionamiento, la ejecución vuelve al script *Demo-ProvisionAndCatalog* original. Se abre la página **Eventos** para el nuevo inquilino en el explorador.

   ![Página Eventos](./media/saas-dbpertenant-provision-and-catalog/new-tenant.png)


## <a name="provision-a-batch-of-tenants"></a>Aprovisionamiento de un lote de inquilinos

Este ejercicio aprovisiona un lote de 17 inquilinos. Se recomienda que aprovisione este lote de inquilinos antes de completar otros tutoriales de Wingtip Tickets SaaS Database per Tenant. Hay un mayor número de bases de datos con las que trabajar.

1. En PowerShell ISE, abra ...\\Learning Modules\\ProvisionAndCatalog\\*Demo-ProvisionAndCatalog.ps1*. Cambie el parámetro *$DemoScenario* a 3:

   * **$DemoScenario** = **3**, *aprovisionar un lote de inquilinos*.
2. Presione F5 para ejecutar el script.

El script implementa un lote de inquilinos adicionales Este script usa una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) que controla el lote y delega el aprovisionamiento de cada base de datos a una plantilla vinculada. Esto permite que Azure Resource Manager negocie el proceso de aprovisionamiento para su script. Las plantillas aprovisionan las bases de datos en paralelo y controlan los reintentos, en caso de ser necesario. El script es idempotente, por lo que si se produce un error o se detiene por alguna razón, vuelva a ejecutarlo.

### <a name="verify-the-batch-of-tenants-that-successfully-deployed"></a>Comprobación de que el lote de inquilinos se haya implementado correctamente

* En [Azure Portal](https://portal.azure.com), vaya a la lista de servidores y abra el servidor *tenants1*. Seleccione **Bases de datos SQL** y compruebe que el lote de 17 bases de datos adicionales aparece ahora en la lista.

   ![Lista de base de datos](./media/saas-dbpertenant-provision-and-catalog/database-list.png)



## <a name="other-provisioning-patterns"></a>Otros patrones de aprovisionamiento

Otros patrones de aprovisionamiento que no se incluyen en este tutorial:

**Aprovisionamiento previo de bases de datos.** : El patrón del aprovisionamiento previo aprovecha el hecho de que las bases de datos de un grupo elástico no suponen costos adicionales. La facturación es para el grupo elástico, no para las bases de datos. Las bases de datos inactivas no consumen recursos. Al aprovisionar previamente bases de datos en un grupo y asignarlas posteriormente cuando sea necesario, puede reducirse el tiempo de agregar inquilinos. El número de bases de datos previamente aprovisionadas puede ajustarse según sea necesario para mantener un búfer adecuado para la velocidad de aprovisionamiento prevista.

**Aprovisionamiento automático**: en este patrón se usa un servicio de aprovisionamiento dedicado para aprovisionar servidores, grupos y bases de datos automáticamente según sea necesario. Si lo desea, puede incluir bases de datos de aprovisionamiento previo en grupos elásticos. Si se retiran y eliminan bases de datos, los huecos que dejan en los grupos elásticos pueden ocuparse aprovisionando el servicio. Este servicio puede ser simple o complejo, como controlar el aprovisionamiento en varias regiones geográficas y configurarse automáticamente la replicación geográfica para la recuperación ante desastres.

Con el patrón de aprovisionamiento automático, un script o aplicación cliente envía una solicitud de aprovisionamiento a una cola para que un servicio de aprovisionamiento la procese. A continuación, sondea el servicio para determinar la finalización. Si se utiliza el aprovisionamiento previo, las solicitudes se administran rápidamente. El servicio proporciona una base de datos de sustitución en segundo plano.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
>
> * Aprovisionar un nuevo inquilino único
> * Aprovisionar un lote de inquilinos adicionales
> * Acceder a los detalles de cómo aprovisionar inquilinos y registrarlos en el catálogo.

Pruebe el [Tutorial de supervisión del rendimiento](./saas-dbpertenant-performance-monitoring.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Tutoriales adicionales que se basan en el patrón de base de datos por inquilino de la aplicación SaaS Wingtip Tickets](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Biblioteca de cliente de base de datos elástica](elastic-database-client-library.md)
* [Depuración de scripts en Windows PowerShell ISE](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise)