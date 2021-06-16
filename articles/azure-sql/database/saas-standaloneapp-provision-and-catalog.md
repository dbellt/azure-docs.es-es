---
title: Tutorial de aplicaciones SaaS multiinquilino
description: Aprovisionar y catalogar nuevos inquilinos mediante el patrón de aplicación independiente
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
ms.openlocfilehash: f9f85621f81091698af622a40a0e31ee9db7ea9f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689007"
---
# <a name="provision-and-catalog-new-tenants-using-the--application-per-tenant-saas-pattern"></a>Aprovisionar y catalogar nuevos inquilinos mediante el patrón SaaS de aplicación por inquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describe el aprovisionamiento y catalogación de nuevos inquilinos mediante un patrón SaaS de aplicación por inquilino independiente.
Este artículo se divide en dos partes principales:
* Explicación conceptual del aprovisionamiento y la catalogación de nuevos inquilinos.
* En este tutorial se destaca el código de muestra de PowerShell que lleva a cabo el aprovisionamiento y la catalogación.
    * Asimismo, el tutorial usa la aplicación SaaS de ejemplo Wingtip Tickets, adaptada al patrón de aplicación por inquilino independiente.

## <a name="standalone-application-per-tenant-pattern"></a>Patrón de aplicación independiente por inquilino

El patrón de aplicación por inquilino independiente es uno de los distintos patrones para aplicaciones SaaS multiinquilino.  En este patrón se aprovisiona una aplicación independiente por cada inquilino. La aplicación consta de componentes de nivel de aplicación y una instancia de Azure SQL Database.  Cada aplicación de inquilino se puede implementar en la suscripción del proveedor.  Como alternativa, Azure ofrece un [programa de aplicaciones administradas](../../azure-resource-manager/managed-applications/overview.md) en el cual una aplicación se puede implementar en la suscripción del inquilino para que el proveedor la administre en nombre del inquilino.

   ![patrón de aplicación por inquilino](./media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern.png)

Al implementar una aplicación por inquilino, tanto la aplicación como la base de datos se aprovisionan en un grupo de recursos creado para el inquilino.  Si se usan grupos de recursos independientes se pueden aislar los recursos de la aplicación de cada inquilino, lo que les permite administrarse de forma independiente. Dentro de cada grupo de recursos, cada instancia de la aplicación se configura para que pueda obtener acceso directo a su base de datos correspondiente.  Este modelo de conexión es comparable a otros patrones que usen un catálogo para mediar en las conexiones existentes entre la aplicación y la base de datos.  Asimismo, como no existen los recursos compartidos, la base de datos de cada inquilino debe aprovisionarse con los recursos necesarios para poder manejar su carga máxima. Este patrón se suele usar en aplicaciones SaaS que tienen menos inquilinos y en las que se enfatiza ampliamente el aislamiento del inquilino y se pone menos énfasis en los costos de los recursos.

## <a name="using-a-tenant-catalog-with-the-application-per-tenant-pattern"></a>Usar un catálogo de inquilinos con el patrón de aplicación por inquilino

Mientras que la aplicación y la base de datos de cada inquilino estén totalmente aisladas, es posible que varios escenarios de administración y análisis operen entre estos inquilinos.  Por ejemplo, si se aplica un cambio de esquema en una nueva versión de la aplicación, es necesario realizar cambios en el esquema de cada base de datos del inquilino. Asimismo, los escenarios de creación de informes y de análisis deben obtener acceso a todas las bases de datos de inquilino sin importar dónde se hayan implementado.

   ![Diagrama que muestra cómo usar un catálogo de inquilinos con el patrón de aplicación por inquilino.](./media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern-with-catalog.png)

El catálogo de inquilinos contiene una asignación entre un identificador de inquilino y una base de datos de inquilino, lo que permite resolver un identificador en un nombre de servidor y de base de datos.  En la aplicación SaaS de Wingtip, el identificador del inquilino se calcula como un hash del nombre de inquilino, aunque se podrían usar otros esquemas.  Como las aplicaciones independientes no necesitan el catálogo para administrar las conexiones, el catálogo se puede usar para agregar otras acciones a un conjunto de bases de datos de inquilino. Por ejemplo, la consulta Elastic puede usar el catálogo para determinar el conjunto de bases de datos en las que se distribuyen las consultas de los informes entre inquilinos.

## <a name="elastic-database-client-library"></a>Biblioteca de cliente de Elastic Database

En la aplicación de ejemplo de Wingtip, el catálogo se implementa mediante las características de administración de particiones de la [biblioteca cliente de Elastic Database](elastic-database-client-library.md) (EDCL).  La biblioteca permite que una aplicación pueda crear, administrar y usar un mapa de particiones que se almacena en una base de datos. En el ejemplo de Wingtip Tickets, el catálogo se almacena en la base de datos del *catálogo de inquilinos*.  Igualmente, la partición asigna una clave de inquilino a la partición (base de datos) en la que se almacenan los datos de ese inquilino.  Las funciones de EDCL se encargan de administrar un *mapa de particiones global* que se almacena en tablas de la base de datos del *catálogo de inquilinos* y en un *mapa de particiones local* almacenado en cada partición.

Asimismo, se pueden usar las funciones de EDCL desde aplicaciones o scripts de PowerShell para crear y administrar las entradas del mapa de particiones. Por otro lado, se pueden usar otras funciones de EDCL para recuperar el conjunto de particiones o para conectarse a la base de datos correspondiente a la clave de inquilino.

> [!IMPORTANT]
> No modifique directamente los datos de la base de datos del catálogo o el mapa de particiones local en las bases de datos de inquilino. No se permiten actualizaciones directas debido al alto riesgo de daños en los datos. En su lugar, edite los datos de asignación solo mediante las API de EDCL.

## <a name="tenant-provisioning"></a>Aprovisionamiento de inquilinos

Cada inquilino requiere un nuevo grupo de recursos de Azure, que debe crearse antes de poder aprovisionar recursos dentro de él. Una vez creado el grupo de recursos, se puede usar una plantilla de Azure Resource Management para implementar los componentes de la aplicación y la base de datos y configurar la conexión de la base de datos. Para inicializar el esquema de la base de datos, la plantilla puede importar un archivo bacpac.  También se puede crear la base de datos como una copia de una base de datos de tipo "template".  A continuación, la base de datos se actualiza con los datos de la ubicación inicial y se registra en el catálogo.

## <a name="tutorial"></a>Tutorial

En este tutorial, aprenderá a:

* Aprovisionar un catálogo
* Registrar las bases de datos de inquilino de ejemplo que implementó anteriormente en el catálogo
* Aprovisionar a un inquilino adicional y registrarlo en el catálogo

Puede usar una plantilla de Azure Resource Manager para implementar y configurar la aplicación, crear la base de datos del inquilino y, a continuación, importar un archivo bacpac para inicializarla. Es posible que la solicitud de importación se ponga en cola durante varios minutos antes de iniciarse.

Al final de este tutorial, tendrá un conjunto de aplicaciones de inquilino independientes, cuyas bases de datos estarán registrada en el catálogo.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)
* Se implementan las tres aplicaciones de inquilino de ejemplo. Para implementar estas aplicaciones en menos de cinco minutos, consulte el artículo [Deploy and explore the Wingtip Tickets SaaS Standalone Application pattern](./saas-standaloneapp-get-started-deploy.md) (Implementar y explorar el patrón de la aplicación independiente SaaS Wingtip Tickets).

## <a name="provision-the-catalog"></a>Aprovisionar el catálogo

En esta tarea, aprenderá a aprovisionar el catálogo que se usa para registrar todas las bases de datos de inquilino. Podrá:

* **Aprovisionar la base de datos del catálogo** mediante una plantilla de administración de recursos de Azure. La base de datos se inicializa al importar un archivo bacpac.
* **Registrar las aplicaciones de inquilino de ejemplo** que implementó anteriormente.  Cada inquilino se registra mediante una clave que se creó a partir de un valor hash del nombre de inquilino.  Asimismo, el nombre del inquilino también se almacena en una tabla de extensión del catálogo.

1. En PowerShell ISE, abra *...\Learning Modules\UserConfig.psm* y actualice el valor **\<user\>** con el valor que usó al implementar las tres aplicaciones de ejemplo.  **Guarde el archivo**.
1. En PowerShell ISE, abra *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1* y establezca **$Scenario = 1**. Implemente el catálogo de inquilinos y registre a los inquilinos predefinidos.

1. Agregue un punto de interrupción colocando el cursor en cualquier lugar de la línea que dice "`& $PSScriptRoot\New-Catalog.ps1`" y presione **F9**.

    ![establecer un punto de interrupción para el seguimiento](./media/saas-standaloneapp-provision-and-catalog/breakpoint.png)

1. Para ejecutar los scripts, presione **F5**.
1.  Después de que la ejecución del script se detenga en el punto de interrupción, presione **F11** para ir al script New-Catalog.ps1.
1.  Siga paso a paso la ejecución del script mediante las opciones del menú Depurar (F10 y F11) para omitir las funciones llamadas o acceder a ellas.
    *   Para obtener más información sobre cómo depurar scripts de PowerShell, consulte [Sugerencias para trabajar con scripts de PowerShell y depurarlos](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).

Una vez completado el script, se creará el catálogo y todos los inquilinos de ejemplo se registrarán.

Ahora examinemos los recursos que ha creado.

1. Abra [Azure Portal](https://portal.azure.com/) y vaya a los grupos de recursos.  Abra el grupo de recursos **wingtip-sa-catalog-\<user\>** y compruebe el servidor del catálogo y la base de datos.
1. Abra la base de datos del portal y seleccione *Explorador de datos* en el menú izquierdo.  Haga clic en el comando de inicio de sesión y, a continuación, escriba la contraseña = **P\@ssword1**.


1. Explore el esquema de la base de datos *tenantcatalog*.
   * Los objetos del esquema `__ShardManagement` los proporciona la biblioteca de cliente Elastic Database.
   * La tabla `Tenants` y la vista `TenantsExtended` son extensiones que se agregaron en el ejemplo y que indican la manera de ampliar el catálogo para proporcionar un valor adicional.
1. Ejecute la consulta, `SELECT * FROM dbo.TenantsExtended`.

   ![explorador de datos](./media/saas-standaloneapp-provision-and-catalog/data-explorer-tenantsextended.png)

    Como alternativa al uso del Explorador de datos, puede conectarse a la base de datos desde SQL Server Management Studio. Para ello, conéctese al servidor de Wingtip:


    Tenga en cuenta que no debe editar los datos directamente en el catálogo; use siempre las API de administración de particiones.

## <a name="provision-a-new-tenant-application"></a>Aprovisionar una nueva aplicación de inquilino

En esta tarea, aprenderá a aprovisionar una aplicación de un solo inquilino. Podrá:

* **Crear un grupo de recursos** para el inquilino.
* **Aprovisionar la aplicación y la base de datos** en el nuevo grupo de recursos mediante una plantilla de administración de recursos de Azure.  Esta acción incluye la inicialización de la base de datos con el esquema común y los datos de referencia, mediante la importación de un archivo bacpac.
* **Inicializar la base de datos con información básica del inquilino**. En esta acción tendrá que especificar el tipo de ubicación que se encargará de determinar la fotografía que se usará como fondo en el sitio web de eventos.
* **Registrar la base de datos en la base de datos del catálogo**.

1. En PowerShell ISE, abra *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1* y establezca **$Scenario = 2**. Implemente el catálogo de inquilinos y registre a los inquilinos predefinidos.

1. Agregue un punto de interrupción en el script colocando el cursor en cualquier lugar de la línea 49 que dice "`& $PSScriptRoot\New-TenantApp.ps1`" y presione **F9**.
1. Para ejecutar los scripts, presione **F5**.
1.  Después de que la ejecución del script se detenga en el punto de interrupción, presione **F11** para ir al script New-Catalog.ps1.
1.  Siga paso a paso la ejecución del script mediante las opciones del menú Depurar (F10 y F11) para omitir las funciones llamadas o acceder a ellas.

Después de aprovisionar el inquilino, se abre el sitio web de eventos del nuevo inquilino.

   ![red maple racing](./media/saas-standaloneapp-provision-and-catalog/redmapleracing.png)

A continuación, puede examinar los nuevos recursos creados en Azure Portal.

   ![recursos de red maple racing](./media/saas-standaloneapp-provision-and-catalog/redmapleracing-resources.png)


## <a name="to-stop-billing-delete-resource-groups"></a>Para detener la facturación, elimine los grupos de recursos

Cuando haya terminado de utilizar el ejemplo, elimine todos los grupos de recursos que haya creado para detener la facturación asociada.

## <a name="additional-resources"></a>Recursos adicionales

- Para obtener más información acerca de las aplicaciones SaaS multiinquilino, consulte [Design patterns for multi-tenant SaaS applications](saas-tenancy-app-design-patterns.md) (Patrones de diseño para aplicaciones SaaS multiinquilino).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha obtenido información:

> [!div class="checklist"]
> * Implementación de la aplicación independiente SaaS Wingtip Tickets.
> * Información sobre los servidores y las bases de datos que componen la aplicación.
> * Información sobre cómo eliminar los recursos de ejemplo para detener la facturación relacionada con ellos.

Puede explorar cómo se usa el catálogo para admitir diversos escenarios entre inquilinos con la versión de base de datos por inquilino de la [aplicación SaaS Wingtip Tickets](./saas-dbpertenant-wingtip-app-overview.md).