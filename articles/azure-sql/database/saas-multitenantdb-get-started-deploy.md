---
title: Implementación de una aplicación SaaS de base de datos multiinquilino con particiones
description: Implemente y explore la aplicación de base de datos multiinquilino SaaS Wingtip Tickets con particiones que muestra los patrones de SaaS con Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 10/16/2018
ms.openlocfilehash: dd762f332b11e61948bcea5128af65aead193de2
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699125"
---
# <a name="deploy-and-explore-a-sharded-multi-tenant-application"></a>Implementación y exploración de una aplicación multiinquilino con particiones
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, se implementa y explora una aplicación SaaS multiinquilino de ejemplo denominada Wingtip Tickets. La aplicación Wingtip Tickets está diseñada para presentar las características de Azure SQL Database que simplifican la implementación de los escenarios de SaaS.

Esta implementación de la aplicación Wingtips Tickets usa un patrón de base de datos multiinquilino con particiones. El particionamiento se realiza por identificador del inquilino. Los datos del inquilino se distribuyen a una base de datos determinada según los valores del identificador del inquilino. 

Este patrón de base de datos permite almacenar uno o varios inquilinos en cada partición o base de datos. Para optimizar la disminución del costo, varios inquilinos pueden compartir cada base de datos. O bien puede optimizar el aislamiento, haciendo que cada base de datos solo almacene un inquilino. Puede tomar la opción de optimización por separado para cada inquilino específico. Puede tomar la opción la primera vez que se almacena el inquilino, o bien puede decidirlo más adelante. El diseño de la aplicación le permite funcionar bien de cualquier modo.

## <a name="app-deploys-quickly"></a>La aplicación se implementa rápidamente

La aplicación se ejecuta en la nube de Azure y usa Azure SQL Database. La sección de implementación que sigue proporciona el botón **Implementar en Azure** azul. Cuando se presiona el botón, la aplicación se implementa totalmente en su suscripción a Azure en cinco minutos. Tiene acceso completo para trabajar con los componentes individuales de la aplicación.

La aplicación se implementa con los datos para los tres inquilinos de ejemplo. Los inquilinos se almacenan juntos en una base de datos multiinquilino.

Cualquier usuario puede descargar el código fuente de C# y PowerShell para Wingtip Tickets desde [el repositorio de GitHub][link-github-wingtip-multitenantdb-55g].

## <a name="learn-in-this-tutorial"></a>En este tutorial

> [!div class="checklist"]
> - Implementación de la aplicación SaaS Wingtip Tickets.
> - Dónde obtener el código fuente de la aplicación y los scripts de administración.
> - Información sobre los servidores y las bases de datos que componen la aplicación.
> - Cómo se asignan los inquilinos a sus datos con el *catálogo*.
> - Cómo aprovisionar un nuevo inquilino.
> - Cómo supervisar la actividad de los inquilinos en la aplicación.

Hay disponible una serie de tutoriales relacionados que se basan en esta implementación inicial. En los tutoriales se explora una variedad de patrones de diseño y administración de SaaS. Cuando trabaja con los tutoriales, se recomienda que recorra los scripts proporcionados para ver cómo se implementan los distintos patrones de SaaS.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

- Tiene instalada la versión más reciente de Azure PowerShell. Para más información, consulte [Introducción a Azure PowerShell][link-azure-get-started-powershell-41q].

## <a name="deploy-the-wingtip-tickets-app"></a>Implementación de la aplicación Wingtip Tickets

### <a name="plan-the-names"></a>Planeamiento de los nombres

En los pasos de esta sección, proporciona un valor de *usuario* que se utiliza para garantizar que los nombres de recursos sean globalmente únicos, y un nombre para el *grupo de recursos* que contiene todos los recursos creados mediante una implementación de la aplicación. Para una persona llamada *Ann Finley*, sugerimos:
- *Usuario:* **af1** *(sus iniciales más un dígito. Use un valor diferente (por ejemplo, af2) si implementa la aplicación una segunda vez).*
- *Grupo de recursos:* **wingtip-mt-af1** *(wingtip-mt indica que esta es la aplicación multiinquilino con particiones. Anexar el nombre de usuario af1 correlaciona el nombre del grupo de recursos con los nombres del recurso que contiene).*

Elija los nombres ahora y escríbalos. 

### <a name="steps"></a>Pasos

1. Haga clic en el botón **Implementar en Azure** azul siguiente.
   - Se abrirá Azure Portal con la plantilla de implementación de la aplicación SaaS Wingtip Tickets.

     [![Botón Implementar en Azure.][image-deploy-to-azure-blue-48d]][link-aka-ms-deploywtp-mtapp-52k]

1. Especifique los valores necesarios de los parámetros para la implementación.

    > [!IMPORTANT]
    > Para esta demostración, no utilice grupos de recursos, servidores o grupos preexistentes. En su lugar, elija **Crear nuevo grupo de recursos**. Elimine este grupo de recursos cuando haya terminado con la aplicación para detener la facturación relacionada con él.
    > No use esta aplicación, ni ninguno de los recursos que se crean, para producción. Algunos aspectos de la configuración del firewall del servidor y de la autenticación son poco seguras (intencionadamente) para facilitar la demostración.

    - En **Grupo de recursos**: seleccione **Crear nuevo** y, luego, proporcione un **nombre** para el grupo de recursos (distingue mayúsculas de minúsculas).
        - Seleccione una **ubicación** en la lista desplegable.
    - Como **usuario** se recomienda elegir un **usuario** breve.

1. **Implemente la aplicación**.

    - Haga clic para aceptar los términos y condiciones.
    - Haga clic en **Comprar**.

1. Supervise el estado de implementación con un clic en **Notificaciones**, que es el icono de la campaña que aparece a la derecha del cuadro de búsqueda. La implementación de la aplicación Wingtip tarda aproximadamente cinco minutos.

   ![implementación correcta](./media/saas-multitenantdb-get-started-deploy/succeeded.png)

## <a name="download-and-unblock-the-management-scripts"></a>Descarga y desbloqueo de los scripts de administración

Durante la implementación de la aplicación, descargue el código fuente y los scripts de administración de la aplicación.

> [!NOTE]
> Es posible que Windows bloquee el contenido ejecutable (scripts, archivos DLL) al descargar y extraer archivos ZIP de un origen externo. Cuando extraiga los scripts de un archivo ZIP, use estos pasos para desbloquear el archivo ZIP antes de extraerlo. Al desbloquear el archivo .zip se asegura de que los scripts estén autorizados para ejecutarse.

1. Vaya al [repositorio de GitHub de WingtipTicketsSaaS-MultiTenantDb](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb).
2. Haga clic en **Clone or download** (Clonar o descargar).
3. Haga clic en **Download ZIP** (Descargar archivos ZIP) y guarde el archivo.
4. Haga clic con el botón derecho en el archivo **WingtipTicketsSaaS-MultiTenantDb-master.zip** y seleccione **Propiedades**.
5. En la pestaña **General**, seleccione **Desbloquear** y haga clic en **Aplicar**.
6. Haga clic en **OK**.
7. Extraiga los archivos.

Los scripts se encuentra en la carpeta *..\\WingtipTicketsSaaS-MultiTenantDb-master\\Learning Modules\\* .

## <a name="update-the-configuration-file-for-this-deployment"></a>Actualizar el archivo de configuración para esta implementación

Antes de ejecutar cualquier script, establezca los valores *grupo de recursos* y *usuario* en **UserConfig.psm1**. Establezca estas variables en los mismos valores establecidos durante la implementación.

1. Abra ...\\Learning Modules\\*UserConfig.psm1* en *PowerShell ISE*.
2. Actualice *ResourceGroupName* y *Name* con los valores específicos para la implementación (solo en las líneas 10 y 11).
3. Guarde los cambios.

Todos los scripts usan los valores establecidos en este archivo, por lo que es importante que sean precisos. Si se vuelve a implementar la aplicación, debe elegir valores diferentes para el usuario y el grupo de recursos. Luego, vuelva a actualizar el archivo UserConfig.psm1 con los valores nuevos.

## <a name="run-the-application"></a>Ejecución de la aplicación

En la aplicación Wingtip, los inquilinos son lugares. Una ubicación puede ser un auditorio, un club deportivo o cualquier otra ubicación que hospede los eventos. Las ubicaciones se registran en Wingtip como clientes y se genera un identificador de inquilino para cada una de ellas. Cada ubicación enumera sus próximos eventos en Wingtip, de manera que el público pueda comprar entradas.

Cada ubicación obtiene una aplicación web personalizada para mostrar los eventos y vender entradas. Cada aplicación web es independiente y está aislada de los demás inquilinos. Internamente en Azure SQL Database, de manera predeterminada, los datos para cada inquilino se almacenan en una base de datos particionada de varios inquilinos. Todos los datos se etiquetan con el identificador de inquilino.

Una página web central de **Event Hubs** proporciona una lista de vínculos a los inquilinos de la implementación concreta. Siga estos pasos para conocer la página web del **centro de eventos** y una aplicación web individual:

1. Abra **Events Hub** en el explorador web:
   - http://events.wingtip-mt.&lt;user&gt;.trafficmanager.net &nbsp; *(sustituya &lt; user&gt; por el valor del usuario de la implementación).*

     ![events hub](./media/saas-multitenantdb-get-started-deploy/events-hub.png)

2. Haga clic en **Fabrikam Jazz Club** en el **centro de eventos**.

   ![Eventos](./media/saas-multitenantdb-get-started-deploy/fabrikam.png)

### <a name="azure-traffic-manager"></a>Administrador de tráfico de Azure

Para controlar la distribución de las solicitudes entrantes, la aplicación Wingtip usa [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md). La página de eventos de cada inquilino incluye el nombre del inquilino en la dirección URL. La dirección URL también incluye el valor del usuario en cuestión. Cada dirección URL sigue el formato que se muestra mediante los siguientes pasos:

- http://events.wingtip-mt.&lt;usuario&gt;.trafficmanager.net/*fabrikamjazzclub*

1. La aplicación de eventos analiza el nombre del inquilino de la dirección URL. Este nombre del inquilino es *fabrikamjazzclub* en la dirección URL del ejemplo anterior.
2. A continuación, la aplicación analiza el nombre del inquilino para crear una clave de acceso a un catálogo mediante [Shard Map Management](elastic-scale-shard-map-management.md).
3. La aplicación busca la clave en el catálogo y obtiene la ubicación correspondiente de la base de datos del inquilino.
4. La aplicación usa la información de la ubicación para buscar la base de datos que contiene todos los datos del inquilino y acceder a ella.

### <a name="events-hub"></a>Centro de eventos

1. El **centro de eventos** enumera todos los inquilinos que se encuentran registrados en el catálogo y sus ubicaciones.
2. **Events Hub** usa metadatos extendidos en el catálogo para recuperar el nombre del inquilino asociado con cada asignación para construir las direcciones URL.

En un entorno de producción, lo habitual es crear un registro DNS CNAME que [apunte a un dominio de empresa de Internet](../../traffic-manager/traffic-manager-point-internet-domain.md) para el perfil de Traffic Manager.

## <a name="start-generating-load-on-the-tenant-databases"></a>Empiece a generar carga en las bases de datos de los inquilinos

Ahora que se implementó la aplicación, vamos a ponerla en marcha. El script *Demo-LoadGenerator* de PowerShell inicia una carga de trabajo que se ejecuta para cada inquilino. La carga real en muchas aplicaciones de SaaS normalmente es esporádica e imprevisible. Para simular este tipo de carga, el generador produce una carga distribuida entre todos los inquilinos. La carga incluye ráfagas aleatorias en cada inquilino que se producen a intervalos aleatorios. El modelo de carga tarda varios minutos en surgir, por lo que es mejor permitir que el generador se ejecute durante al menos tres o cuatro minutos antes de supervisar la carga.

1. En *PowerShell ISE*, abra el script ...\\Learning Modules\\Utilities\\*Demo-LoadGenerator.ps1*.
2. Presione **F5** para ejecutar el script e iniciar el generador de carga (por ahora deje los valores predeterminados de los parámetros).

El script *Demo-LoadGenerator.ps1* abre otra sesión de PowerShell donde se ejecuta el generador de carga. El generador de carga se ejecuta en esta sesión como una tarea en primer plano que invoca trabajos de generación de carga en segundo plano, uno para cada inquilino.

Una vez que se inicia la tarea en primer plano, se mantiene en un estado de invocación de trabajo. La tarea inicia trabajos en segundo plano adicionales para cualquier inquilino nuevo que se aprovisione posteriormente.

Cerrar la sesión de PowerShell detiene todos los trabajos.

Es posible que desee reiniciar la sesión del generador de carga para usar otros valores de parámetro. Si es así, cierre la sesión de generación de PowerShell y, luego, vuelva a ejecutar *Demo-LoadGenerator.ps1*.

## <a name="provision-a-new-tenant-into-the-sharded-database"></a>Aprovisionamiento de un inquilino nuevo en la base de datos con particiones

La implementación inicial incluye tres inquilinos de ejemplo en la base de datos *Tenants1*. Vamos a crear otro inquilino para ver cómo afecta a la aplicación implementada. En este paso, para crear un inquilino se presiona una tecla:

1. Abra ...\\Learning Modules\\Provision and Catalog\\*Demo-ProvisionTenants.ps1* en *PowerShell ISE*.
2. Presione **F5** (no **F8**) para ejecutar el script (deje por ahora los valores predeterminados).

   > [!NOTE]
   > Solo debe ejecutar los scripts de PowerShell al presionar la tecla **F5**, no **F8**, para ejecutar una parte determinada del script. El problema con **F8** es que la variable *$PSScriptRoot* no se evalúa. Esta variable es necesaria para que muchos scripts vayan a carpetas, invoquen otros scripts o importen módulos.

El nuevo inquilino Red Maple Racing se agrega a la base de datos *Tenants1* y se registra en el catálogo. El sitio de **eventos** de venta de entradas del nuevo inquilino se abre en el explorador:

![Nuevo inquilino](./media/saas-multitenantdb-get-started-deploy/red-maple-racing.png)

Actualice el **centro de eventos** y el nuevo inquilino aparecerá en la lista.

## <a name="provision-a-new-tenant-in-its-own-database"></a>Aprovisionamiento de un inquilino nuevo en su propia base de datos

El modelo multiinquilino con particiones permite elegir entre aprovisionar un inquilino nuevo en una base de datos que contenga otros inquilinos o en su propia base de datos. Un inquilino aislado en su propia base de datos disfruta de las siguientes ventajas:

- El rendimiento de la base de datos del inquilino se puede administrar sin poner en peligro las necesidades de otros inquilinos.
- Si es necesario, la base de datos puede restaurarse a un momento anterior en el tiempo, ya que los demás inquilinos no resultan afectados.

Puede poner los clientes de prueba de evaluación gratuita o los económicos en bases de datos de varios inquilinos. Podría colocar a cada inquilino premium en su propia base de datos específica. Si crea grandes cantidades de bases de datos que contengan solo un inquilino, puede administrarlas todas de manera colectiva en un grupo elástico para optimizar los costos de los recursos.

Ahora aprovisionaremos otro inquilino, esta vez, en su propia base de datos:

1. En ...\\Learning Modules\\Provision and Catalog\\*Demo-ProvisionTenants.ps1*, cambie *$TenantName* a **Salix Salsa**, *$VenueType* a **dance** y *$Scenario* a **2**.

2. Presione **F5** para volver a ejecutar el script.
    - Al presionar **F5** se aprovisiona el inquilino nuevo en una base de datos independiente. La base de datos y el inquilino se registran en el catálogo. Luego, el explorador se abre en la página de eventos del inquilino.

   ![Página de eventos de Salix Salsa](./media/saas-multitenantdb-get-started-deploy/salix-salsa.png)

   - Desplácese hasta la parte inferior de la página. En el banner verá el nombre de la base de datos en que están almacenados los datos del inquilino.

3. Actualice el **centro de eventos** y los dos inquilinos nuevos aparecerán en la lista.

## <a name="explore-the-servers-and-tenant-databases"></a>Exploración de los servidores y las bases de datos de inquilino

Veamos ahora algunos de los recursos que se implementaron:

1. En [Azure Portal](https://portal.azure.com), navegue a la lista de los grupos de recursos. Abra el grupo de recursos que creó cuando implementó la aplicación.

   ![resource group](./media/saas-multitenantdb-get-started-deploy/resource-group.png)

2. Haga clic en el servidor **catalog-mt&lt;usuario&gt;** . El servidor del catálogo contiene dos bases de datos denominadas *tenantcatalog* y *basetenantdb*. La base de datos *basetenantdb* es una base de datos de plantilla vacía. Se copia para crear una base de datos de inquilino nueva, ya sea para que la usen muchos inquilinos o solo uno.

   ![Servidor de catálogo](./media/saas-multitenantdb-get-started-deploy/catalog-server.png)

3. Vuelva al grupo de recursos y seleccione el servidor *tenants1-mt* que contiene las bases de datos de inquilino.
    - La base de datos tenants1 es una base de datos multiinquilino en la que se almacenan los tres inquilinos originales, además del primer inquilino que agregó. Está configurada como una base de datos estándar de 50 DTU.
    - La base de datos **salixsalsa** contiene el lugar de baile de Salix Salsa como el único inquilino. Está configurada como una base de datos de edición Estándar con 50 DTU como valor predeterminado.

   ![servidor de inquilinos](./media/saas-multitenantdb-get-started-deploy/tenants-server.png)

## <a name="monitor-the-performance-of-the-database"></a>Supervisión del rendimiento de la base de datos

Si el generador de carga lleva varios minutos en ejecución, debería haber suficiente telemetría para examinar las funcionalidades de supervisión de bases de datos integradas en Azure Portal.

1. Navegue al servidor **tenants1-mt&lt;usuario&gt;** y haga clic en **tenants1** para ver el uso de recursos para la base de datos que tiene cuatro inquilinos en ella. Cada inquilino está sujeto a una carga pesada esporádica desde el generador de carga:

   ![supervisión de tenants1](./media/saas-multitenantdb-get-started-deploy/monitor-tenants1.png)

   El gráfico de uso de DTU ilustra bien cómo una base de datos multiinquilino puede admitir una carga de trabajo impredecible a través de varios inquilinos. En este caso, el generador de carga aplica una carga esporádica de aproximadamente 30 DTU a cada inquilino. Esta carga equivale al 60 % del uso de una base de datos de 50 DTU. Valores máximos que superan el 60 % son el resultado de la carga que se aplica a más de un inquilino al mismo tiempo.

2. Vaya al servidor **tenants1-mt&lt;usuario&gt;** y haga clic en la base de datos **salixsalsa**. Puede ver el uso de los recursos en esta base de datos que contiene un solo inquilino.

   ![base de datos salixsalsa](./media/saas-multitenantdb-get-started-deploy/monitor-salix.png)

El generador de carga aplica una carga similar a cada inquilino, independientemente de la base de datos en que se encuentra cada inquilino. Con solo un inquilino en la base de datos **salixsalsa**, puede ver que la base de datos podría admitir una carga mucho mayor que la base de datos con varios inquilinos. 

### <a name="resource-allocations-vary-by-workload"></a>Las asignaciones de recursos varían según la carga de trabajo

En ocasiones, una base de datos de varios inquilinos requiere más recursos para un buen rendimiento que una de un solo inquilino, pero no siempre. La asignación de recursos óptima depende de las características de la carga de trabajo concreta de los inquilinos del sistema.

Las cargas de trabajo generadas por el script generador de carga solo tienen fines ilustrativos.

## <a name="additional-resources"></a>Recursos adicionales

- Para más información acerca de aplicaciones SaaS multiinquilino, consulte [Modelos de diseño para las aplicaciones SaaS multiinquilino](saas-tenancy-app-design-patterns.md).

- Para más información sobre los grupos elásticos, consulte:

  - [Los grupos elásticos ayudan a administrar y escalar varias bases de datos de Azure SQL Database](elastic-pool-overview.md)
  - [Escalado horizontal con Azure SQL Database](elastic-scale-introduction.md)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha obtenido información:

> [!div class="checklist"]
> - Implementación de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets
> - Información sobre los servidores y las bases de datos que componen la aplicación
> - Los inquilinos se asignan a sus datos con el *catálogo*
> - Aprovisionamiento de inquilinos nuevos en una base de datos multiinquilino o en una base de datos de inquilino único.
> - Visualización del uso de grupos para supervisar la actividad de los inquilinos
> - Información sobre cómo eliminar los recursos de ejemplo para detener la facturación relacionada con ellos.

Ahora pruebe el [tutorial de aprovisionamiento del catálogo](saas-multitenantdb-provision-and-catalog.md).


<!--  Link references.

A [series of related tutorials] is available that build upon this initial deployment.
[link-wtp-overivew-jumpto-saas-tutorials-97j]: saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials

-->

[link-aka-ms-deploywtp-mtapp-52k]: https://aka.ms/deploywtp-mtapp


[link-azure-get-started-powershell-41q]: /powershell/azure/get-started-azureps

[link-github-wingtip-multitenantdb-55g]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/



<!--  Image references.

[image-deploy-to-azure-blue-48d]: https://aka.ms/deploywtp-mtapp "Button for Deploy to Azure."
-->

[image-deploy-to-azure-blue-48d]: media/saas-multitenantdb-get-started-deploy/deploy.png "Botón Implementar en Azure."