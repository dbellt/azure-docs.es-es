---
title: Recuperación ante desastres para aplicaciones SaaS con replicación geográfica
description: Obtenga información sobre cómo usar la replicación geográfica de Azure SQL Database para recuperar una aplicación SaaS multiinquilino en caso de una interrupción del servicio
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: b5702cf97c1796e2f3e7b79f61ef76eb6084293d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708389"
---
# <a name="disaster-recovery-for-a-multi-tenant-saas-application-using-database-geo-replication"></a>Recuperación ante desastres para una aplicación SaaS multiinquilino mediante la replicación geográfica de las bases de datos
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, veremos un escenario completo de recuperación ante desastres para una aplicación SaaS multiinquilino implementada con el modelo de una base de datos por inquilino. Para proteger la aplicación frente a interrupciones, use la [_replicación geográfica_](active-geo-replication-overview.md) para crear réplicas de las bases de datos de catálogo y de inquilinos en una región de recuperación alternativa. En caso de interrupción, rápidamente se produce una conmutación por error a estas réplicas para reanudar las operaciones normales de la empresa. En la conmutación por error, las bases de datos en la región original se convierten en réplicas secundarias de las bases de datos en la región de recuperación. Una vez que estas réplicas estén en línea de nuevo, se actualizan automáticamente al estado de las bases de datos en la región de recuperación. Una vez resuelta la interrupción, se producirá una conmutación por recuperación a las bases de datos en la región de producción original.

Este tutorial describe los flujos de trabajo de la conmutación por error y la conmutación por recuperación. Aprenderá a:
> [!div class="checklist"]
> 
> * Sincronizar la información de configuración de la base de datos y los grupos elásticos en el catálogo de inquilinos
> * Configurar un entorno de recuperación en una región alternativa, que incluye aplicaciones, servidores y grupos
> * Usar la _replicación geográfica_ para replicar las bases de datos de catálogo y de inquilinos en la región de recuperación
> * Conmutar por error la aplicación y las bases de datos de catálogo y de inquilinos a la región de recuperación 
> * Después, conmutar por error la aplicación y las bases de datos de catálogo y de inquilinos a la región original después de resolver la interrupción
> * Actualizar el catálogo cuando cada base de datos de inquilino conmute por error para realizar el seguimiento de la ubicación principal de cada base de datos de inquilino
> * Asegurarse de que la aplicación y la base de datos de inquilino principal siempre están colocadas en la misma región de Azure para reducir la latencia  


Antes de iniciar este tutorial, asegúrese de cumplir estos requisitos previos:
* La aplicación Wingtip Tickets SaaS Database Per Tenant está implementada. Para implementarla en menos de cinco minutos, consulte el artículo de [implementación y exploración de la aplicación Wingtip Tickets SaaS Database Per Tenant](saas-dbpertenant-get-started-deploy.md).  
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)

## <a name="introduction-to-the-geo-replication-recovery-pattern"></a>Introducción al patrón de recuperación por replicación geográfica

![Arquitectura de recuperación](./media/saas-dbpertenant-dr-geo-replication/recovery-architecture.png)

La recuperación ante desastres (DR) es una consideración importante para muchas aplicaciones, ya sea por motivos de cumplimiento o por continuidad del negocio. En caso de una interrupción del servicio prolongada, un plan de recuperación ante desastres bien preparado puede minimizar la interrupción del negocio. La replicación geográfica proporciona los menores RPO y RTO porque mantiene réplicas de las bases de datos en una región de recuperación a las que se puede conmutar por error de forma muy rápida.

Un plan de recuperación ante desastres basado en la replicación geográfica consta de tres partes diferenciadas:
* Configuración: creación y mantenimiento del entorno de recuperación
* Recuperación: conmutación por error de la aplicación y las bases de datos al entorno de recuperación en caso de interrupción
* Repatriación: conmutación por recuperación de la aplicación y las bases de datos a la región original una vez resuelta la interrupción 

Todas las partes deben evaluarse cuidadosamente, especialmente si se trabaja a escala. En general, el plan debe lograr varios objetivos:

* Configurar
    * Establecer y mantener un entorno con una imagen reflejada en la región de recuperación. Al crear los grupos elásticos y replicar alguna base de datos en este entorno de recuperación, se reserva la capacidad en la región de recuperación. En el mantenimiento del entorno se incluye la replicación de las nuevas bases de datos de inquilino a medida que se aprovisionen.  
* Recuperación
    * Si se usa un entorno de recuperación de escala reducida para minimizar los costos diarios, los grupos y las bases de datos deben escalarse verticalmente para que tengan una capacidad operativa completa en la región de recuperación
     * Habilitar el aprovisionamiento de nuevos inquilinos en la región de recuperación lo antes posible  
     * Optimizar el entorno para restaurar los inquilinos en orden de prioridad
     * Optimizar el entorno para que inquilinos vuelvan a estar en línea tan rápido como sea posible. Realice los pasos en paralelo cuando sea conveniente
     * Ser resistente a errores, contar con capacidad de reinicio y ser idempotente
     * Permitir que el proceso se pueda cancelar antes de que termine si la región original vuelve a estar en línea.
* Repatriación 
     * Conmutar por error las bases de datos desde la región de recuperación a las réplicas en la región original con un impacto mínimo en los inquilinos: sin pérdida de datos y con un tiempo mínimo sin conexión por inquilino.

En este tutorial, estos desafíos se abordan con las características de Azure SQL Database y la plataforma de Azure:

* [Plantillas de Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md), para reservar toda la capacidad necesaria tan rápido como sea posible. Las plantillas de Azure Resource Manager se utilizan para aprovisionar una imagen reflejada de los servidores de producción y los grupos elásticos en la región de recuperación.
* [Replicación geográfica](active-geo-replication-overview.md), para crear réplicas secundarias de solo lectura y replicación asincrónica para todas las bases de datos. Durante una interrupción, se produce una conmutación por error a las réplicas en la región de recuperación.  Una vez resuelta la interrupción, se produce una conmutación por recuperación a las bases de datos en la región original sin pérdida de datos.
* Operaciones de conmutación por error [asincrónicas](../../azure-resource-manager/management/async-operations.md), que se envían en el orden de prioridad de los inquilinos para minimizar el tiempo de conmutación por error si hay un gran número de bases de datos.
* [Características de recuperación de administración de particiones](elastic-database-recovery-manager.md) para cambiar las entradas de las bases de datos en el catálogo durante la recuperación y la repatriación. Estas características permiten a la aplicación conectarse a bases de datos de inquilinos independientemente de la ubicación sin tener que volver a configurar la aplicación.
* [Alias DNS de SQL Server](./dns-alias-overview.md), para habilitar el aprovisionamiento de nuevos inquilinos independientemente de la región en la que esté funcionando la aplicación. Los alias DNS también se utilizan para que el proceso de sincronización del catálogo se pueda conectar al catálogo activo, independientemente de su ubicación.

## <a name="get-the-disaster-recovery-scripts"></a>Obtención de los scripts de recuperación ante desastres 

> [!IMPORTANT]
> Al igual que todos los scripts de administración de Wingtip vales, los scripts de recuperación ante desastres tienen una calidad de muestra y no para usarse en producción. 

Los scripts de recuperación ante desastres de este tutorial y el código fuente de la aplicación Wingtip están disponibles en GitHub, en el [repositorio de Wingtip Tickets SaaS Database Per Tenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/). Consulte en las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) los pasos para descargar y desbloquear los scripts de administración de Wingtip Tickets.

## <a name="tutorial-overview"></a>Información general del tutorial
En este tutorial, usará primero la replicación geográfica para crear réplicas de la aplicación de Wingtip Tickets y sus bases de datos en una región distinta. Después, conmutará por error a esta región para simular la recuperación ante una interrupción. Cuando haya finalizado, la aplicación es totalmente funcional en la región de recuperación.

Más adelante, en un paso de repatriación diferente, se conmutarán por error el catálogo y las bases de datos de inquilinos de la región de recuperación a la región original. La aplicación y las bases de datos siguen estando disponibles durante la repatriación. Cuando haya finalizado, la aplicación es totalmente funcional en la región original.

> [!Note]
> La aplicación se recupera en la _región emparejada_ de la región en la que se implementa la aplicación. Para más información, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md).

## <a name="review-the-healthy-state-of-the-application"></a>Revisión del estado de mantenimiento de la aplicación

Antes de comenzar el proceso de recuperación, revise el estado de mantenimiento normal de la aplicación.
1. En el explorador web, abra el centro de eventos de Wingtip Tickets (http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net; sustituya &lt; usuario&gt; por el valor del usuario de la implementación).
    * Desplácese hasta la parte inferior de la página y observe el nombre y la ubicación del servidor del catálogo en el pie de página. La ubicación es la región en la que se implementó la aplicación.
    *SUGERENCIA: Mantenga el mouse sobre la ubicación para aumentar el tamaño de la pantalla.* 
    ![Estado correcto del centro de eventos en la región original](./media/saas-dbpertenant-dr-geo-replication/events-hub-original-region.png)

2. Haga clic en el inquilino Contoso Concert Hall y abra su página de eventos.
    * En el pie de página, observe el nombre del servidor del inquilino. La ubicación será la misma que la ubicación del servidor del catálogo.

3. En [Azure Portal](https://portal.azure.com), abra el grupo de recursos en el que se implementó la aplicación
    * Observe la región en la que están implementados los servidores. 

## <a name="sync-tenant-configuration-into-catalog"></a>Sincronización de la configuración del inquilino en el catálogo

En esta tarea, se inicia un proceso para sincronizar la configuración de los servidores, los grupos elásticos y las bases de datos en el catálogo de inquilinos. El proceso mantiene esta información actualizada en el catálogo.  El proceso trabaja con el catálogo activo, ya sea en la región original o en la región de recuperación. La información de configuración se usa como parte del proceso de recuperación para garantizar que el entorno de recuperación sea coherente con el entorno original y, después durante la repatriación, para asegurarse de que la región original sea coherente con los cambios realizados en el entorno de recuperación. El catálogo también se usa para realizar un seguimiento del estado de recuperación de los recursos de inquilino

> [!IMPORTANT]
> Para simplificar, el proceso de sincronización y otros procesos de recuperación y repatriación de larga duración se implementan en estos tutoriales como trabajos locales de PowerShell o sesiones que se ejecutan en el inicio de sesión de usuario de cliente. Los token de autenticación que se emiten al iniciar sesión expirarán después de varias horas y luego se producirá un error de los trabajos. En un escenario de producción, los procesos de ejecución prolongada se deben implementar como servicios confiables de Azure de algún tipo, que se ejecutan como una entidad de servicio. Consulte [Uso de Azure PowerShell para crear una entidad de servicio con un certificado](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

1. En _PowerShell ISE_, abra el archivo ...\Learning Modules\UserConfig.psm1. Reemplace `<resourcegroup>` y `<user>` en las líneas 10 y 11 por el valor usado al implementar la aplicación.  Guarde el archivo.

2. En *PowerShell ISE*, abra el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 y establezca:
    * **$DemoScenario = 1**, iniciar un trabajo en segundo plano que sincroniza la información de configuración del servidor de inquilino y del grupo con el catálogo

3. Presione **F5** para ejecutar el script de sincronización. Se abrirá una nueva sesión de PowerShell para sincronizar la configuración de los recursos del inquilino.
![Captura de pantalla que muestra la nueva sesión de PowerShell que se abre para sincronizar la configuración de los recursos del inquilino.](./media/saas-dbpertenant-dr-geo-replication/sync-process.png)

Deje la ventana de PowerShell ejecutándose en segundo plano y continúe con el resto del tutorial. 

> [!Note]
> El proceso de sincronización conecta con el catálogo mediante un alias DNS. Este alias se modifica durante la restauración y la repatriación para que apunte al catálogo activo. El proceso de sincronización mantiene actualizado el catálogo con los cambios de configuración de las bases de datos o los grupos que se han realizado en la región de recuperación.  Durante la repatriación, estos cambios se aplican a los recursos equivalentes en la región original.

## <a name="create-secondary-database-replicas-in-the-recovery-region"></a>Creación de réplicas de base de datos secundarias en la región de recuperación

En esta tarea, se inicia un proceso que implementa una instancia de aplicación duplicada y se replican el catálogo y todas las bases de datos de inquilinos a una región de recuperación.

> [!Note]
> Este tutorial agrega protección por replicación geográfica a la aplicación de ejemplo Wingtip Tickets. En un escenario de producción para una aplicación que utiliza replicación geográfica, cada inquilino se aprovisionaría con una base de datos de replicación geográfica desde el principio. Consulte [Diseño de servicios de alta disponibilidad con Azure SQL Database](designing-cloud-solutions-for-disaster-recovery.md#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime)

1. En *PowerShell ISE*, abra el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 y establezca los siguientes valores:
    * **$DemoScenario = 2**, crear el entorno de recuperación con imagen reflejada y replicar el catálogo y las bases de datos de inquilinos

2. Presione **F5** para ejecutar el script. Se abre una nueva sesión de PowerShell para crear las réplicas.
![Proceso de sincronización](./media/saas-dbpertenant-dr-geo-replication/replication-process.png)  

## <a name="review-the-normal-application-state"></a>Revisión del estado normal de la aplicación

En este punto, la aplicación se ejecuta con normalidad en la región original y ahora está protegida por replicación geográfica.  Existen réplicas secundarias de solo lectura en la región de recuperación para todas las bases de datos. 

1. En Azure Portal, examine los grupos de recursos y observe que se ha creado un grupo de recursos con el sufijo -recovery en la región de recuperación. 

2. Revise los recursos del grupo de recursos de recuperación.  

3. Haga clic en la base de datos Contoso Concert Hall en el servidor _tenants1-dpt -&lt;usuario&gt;-recovery_.  Haga clic en Replicación geográfica en el lado izquierdo. 

    ![Vínculo de replicación geográfica de Contoso Concert](./media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication.png) 

En el mapa de regiones de Azure, observe el vínculo de replicación geográfica entre la principal en la región original y la secundaria en la región de recuperación.  

## <a name="fail-over-the-application-into-the-recovery-region"></a>Conmutación por error de la aplicación en la región de recuperación

### <a name="geo-replication-recovery-process-overview"></a>Introducción al proceso de recuperación con replicación geográfica

El script de recuperación realiza las siguientes tareas:

1. Deshabilita el punto de conexión de Traffic Manager de la aplicación web en la región original. La deshabilitación del punto de conexión impide que los usuarios se conecten a la aplicación en un estado no válido si la región original pasa a estar en línea durante la recuperación.

1. Utiliza una conmutación por error forzada de la base de datos del catálogo en la región de recuperación para que sea la base de datos principal y actualiza el alias _activecatalog_ para que apunte al servidor del catálogo de recuperación.

1. Actualiza el alias _newtenant_ para que apunte al servidor del inquilino en la región de recuperación. El cambio de este alias garantiza que las bases de datos para los inquilinos nuevos se aprovisionan en la región de recuperación. 

1. Marca todos los inquilinos existentes en el catálogo de recuperación como sin conexión para evitar el acceso a las bases de datos de inquilino antes de que se realice la conmutación por error.

1. Actualiza la configuración de todos los grupos elásticos y las bases de datos únicas replicadas en la región de recuperación para reflejar su configuración en la región original. (Esta tarea solo es necesaria si los grupos o las bases de datos replicadas en el entorno de recuperación se reducen durante las operaciones normales para reducir los costos).

1. Habilita el punto de conexión de Traffic Manager de la aplicación web en la región de recuperación. La habilitación de este punto de conexión permite a la aplicación aprovisionar nuevos inquilinos. En esta fase, los inquilinos existentes siguen estando sin conexión.

1. Envía lotes de solicitudes para forzar la conmutación por error de las bases de datos en orden de prioridad.
    * Los lotes se organizan de forma que las bases de datos se conmutan por error en paralelo en todos los grupos.
    * Las solicitudes de conmutación por error se envían usando operaciones asincrónicas para ser enviadas rápidamente y para que se puedan procesar simultáneamente varias solicitudes.

   > [!Note]
   > En un escenario de interrupción, las bases de datos principales en la región original están sin conexión.  Al forzar la conmutación por error en la secundaria, se rompe la conexión con la principal sin intentar aplicar las transacciones en cola restantes. En un escenario de obtención de detalles de la recuperación ante desastres como este tutorial, si hay alguna actividad de actualización en el momento de la conmutación por error podría haber alguna pérdida de datos. Más adelante, durante la repatriación, cuando se conmutan por recuperación las bases de datos de la región de recuperación a la región original, se utiliza una conmutación por error normal para asegurarse de que no hay ninguna pérdida de datos.

1. Supervisa el servicio para determinar cuándo se han conmutado por error las bases de datos. Una vez que se ha realizado la conmutación por error de una base de datos de inquilino, actualiza el catálogo para registrar el estado de recuperación de la base de datos de inquilino y marca al inquilino como en línea.
    * La aplicación puede acceder a las bases de datos de inquilino tan pronto como se marcan como en línea en el catálogo.
    * Una suma de valores rowversion en la base de datos de inquilino se almacena en el catálogo. Este valor actúa como una huella digital que permite al proceso de repatriación determinar si la base de datos se ha actualizado en la región de recuperación.

### <a name="run-the-script-to-fail-over-to-the-recovery-region"></a>Ejecución del script para conmutar por error a la región de recuperación

Ahora suponga que hay una interrupción en la región en la que la aplicación se implementa y ejecuta el script de recuperación:

1. En *PowerShell ISE*, abra el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 y establezca los siguientes valores:
    * **$DemoScenario = 3**, recuperar la aplicación en una región de recuperación mediante la conmutación por error a las réplicas

2. Presione **F5** para ejecutar el script.  
    * El script se abre en una nueva ventana de PowerShell y luego inicia una serie de trabajos de PowerShell que se ejecutan en paralelo. Estos trabajos realizan la conmutación por error de las bases de datos de inquilino a la región de recuperación.
    * La región de recuperación es la _región emparejada_ asociada a la región de Azure en la que implementó la aplicación. Para más información, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md). 

3. Supervise el estado del proceso de recuperación en la ventana de PowerShell.
    ![Proceso de conmutación por error](./media/saas-dbpertenant-dr-geo-replication/failover-process.png)

> [!Note]
> Para explorar el código de los trabajos de recuperación, revise los scripts de PowerShell en la carpeta ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\RecoveryJobs.

### <a name="review-the-application-state-during-recovery"></a>Revisión del estado de la aplicación durante la recuperación

Mientras que el punto de conexión de la aplicación esté deshabilitado en Traffic Manager, la aplicación no está disponible. Después de la conmutación por error del catálogo a la región de recuperación y de que todos los inquilinos sean marcados como sin conexión, la aplicación se pone de nuevo en línea. Aunque la aplicación está disponible, cada inquilino aparece como sin conexión en el centro de eventos hasta que se realiza la conmutación por error de su base de datos. Es importante diseñar la aplicación para controlar las bases de datos de inquilino sin conexión.

1. Inmediatamente después de recuperar la base de datos de catálogo, actualice el centro de eventos de Wingtip Tickets en el explorador web.
   * En el pie de página, observe que el nombre del servidor de catálogo tiene ahora un sufijo _-recovery_ y se encuentra en la región de recuperación.
   * Observe que los inquilinos que no se han restaurado aún se marcan como sin conexión y no son seleccionables.  

     > [!Note]
     > Cuando se trata de solo unas pocas bases de datos para recuperar, es posible que no pueda actualizar el explorador antes de que se haya completado la recuperación, por lo que quizás no llegue a ver a los inquilinos mientras están sin conexión. 

     ![Centro de eventos sin conexión](./media/saas-dbpertenant-dr-geo-replication/events-hub-offlinemode.png) 

   * Si abre directamente la página de eventos de un inquilino sin conexión, se muestra una notificación "inquilino sin conexión". Por ejemplo, si Contoso Concert Hall está sin conexión, intente abrir http://events.wingtip-dpt.&lt.usuario&gt;.trafficmanager.net/contosoconcerthall ![ Página de Contoso sin conexión](./media/saas-dbpertenant-dr-geo-replication/dr-in-progress-offline-contosoconcerthall.png) 

### <a name="provision-a-new-tenant-in-the-recovery-region"></a>Aprovisionamiento de un nuevo inquilino en la región de recuperación
Incluso antes de que todas las bases de datos de inquilino existentes hayan realizado la conmutación por error, puede aprovisionar nuevos inquilinos en la región de recuperación.  

1. En *PowerShell ISE*, abra el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 y establezca la siguiente propiedad:
    * **$DemoScenario = 4**, aprovisionar un nuevo inquilino en la región de recuperación

2. Presione **F5** para ejecutar el script y aprovisionar el nuevo inquilino. 

3. La página de eventos de Hawthorn Hall se abre en el explorador cuando finaliza. Observe en el pie de página que la base de datos de Hawthorn Hall se aprovisiona en la región de recuperación.
    ![Página de eventos de Hawthorn Hall](./media/saas-dbpertenant-dr-geo-replication/hawthornhallevents.png) 

4. En el explorador, actualice la página del centro de eventos de Wingtip Tickets y verá a Hawthorn Hall incluido. 
    * Si ha aprovisionado a Hawthorn Hall sin esperar a la restauración de los otros inquilinos, es posible que estos todavía estén sin conexión.


## <a name="review-the-recovered-state-of-the-application"></a>Revisión del estado de recuperación de la aplicación

Cuando finaliza el proceso de recuperación, la aplicación y todos los inquilinos están totalmente funcionales en la región de recuperación. 

1. Una vez que la presentación de la ventana de la consola de PowerShell indica que se han recuperado todos los inquilinos, actualice el centro de eventos.  Todos los inquilinos aparecerán en línea, incluido el nuevo inquilino, Hawthorn Hall.

    ![inquilinos nuevos y recuperados en el centro de eventos](./media/saas-dbpertenant-dr-geo-replication/events-hub-with-hawthorn-hall.png)

2. En [Azure Portal](https://portal.azure.com), abra la lista de grupos de recursos.  
    * Observe el grupo de recursos que ha implementado, además del grupo de recursos de recuperación, con el sufijo _-recovery_.  El grupo de recursos de recuperación contiene todos los recursos creados durante el proceso de recuperación, además de los nuevos recursos creados durante la interrupción.  

3. Abra el grupo de recursos de recuperación y observe los siguientes elementos:
   * Las versiones de recuperación de los servidores de catálogo y tenants1 con el sufijo _-recovery_.  Las bases de datos de catálogo y de inquilino restauradas en estos servidores tienen los nombres utilizados en la región original.

   * El servidor SQL _tenants2-dpt-&lt;usuario&gt;-recovery_.  Este servidor se utiliza para el aprovisionamiento de nuevos inquilinos durante la interrupción.
   * La instancia de App Service llamada _events-wingtip-dpt-&lt;región-de-recuperación&gt;-&lt;usuario&gt_, que es la instancia de recuperación de la aplicación de eventos. 

     ![Recursos de recuperación de Azure](./media/saas-dbpertenant-dr-geo-replication/resources-in-recovery-region.png) 

4. Abra el servidor SQL _tenants2-dpt-&lt;usuario&gt;-recovery_.  Tenga en cuenta que contiene la base de datos _hawthornhall_ y el grupo elástico _Pool1_.  La base de datos _hawthornhall_ está configurada como una base de datos elástica en el grupo elástico _Pool1_.

5. Vuelva al grupo de recursos y haga clic en la base de datos Contoso Concert Hall en el servidor _tenants1-dpt -&lt;usuario&gt;-recovery_. Haga clic en Replicación geográfica en el lado izquierdo.

    ![Base de datos Contoso después de la conmutación por error](./media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication-after-failover.png)

## <a name="change-tenant-data"></a>Cambio de los datos del inquilino 
En esta tarea, va a actualizar una de las bases de datos de inquilino. 

1. En el explorador, busque la lista de eventos para Contoso Concert Hall y tome nota del último nombre de evento.
2. En *PowerShell ISE*, en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1, establezca el siguiente valor:
    * **$DemoScenario = 5**, eliminar un evento de un inquilino en la región de recuperación
3. Presione **F5** para ejecutar el script
4. Actualice la página de eventos de Contoso Concert Hall (http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net/contosoconcerthall; sustituya &lt; usuario&gt; por el valor del usuario en la implementación) y observe que el último evento se ha eliminado.

## <a name="repatriate-the-application-to-its-original-production-region"></a>Repatriación de la aplicación a su región de producción original

Esta tarea repatria la aplicación a su región original. En un escenario real, se iniciaría la repatriación cuando la interrupción esté resuelta.

### <a name="repatriation-process-overview"></a>Introducción al proceso de repatriación

![Arquitectura de la repatriación](./media/saas-dbpertenant-dr-geo-replication/repatriation-architecture.png)

El proceso de repatriación:
1. Cancela cualquier solicitud de restauración de base de datos pendiente.
2. Actualiza el alias _newtenant_ para que apunte al servidor del inquilino en la región de origen. El cambio de este alias garantiza que las bases de datos para los inquilinos nuevos ahora se aprovisionan en la región de origen.
3. Actualiza los datos de inquilino modificados en la región original
4. Realiza la conmutación por error de las bases de datos de inquilino en orden de prioridad.

La conmutación por error mueve de un modo eficaz la base de datos a la región original. Cuando se realiza la conmutación por error de la base de datos, se eliminan las conexiones abiertas y la base de datos no está disponible durante unos segundos. Las aplicaciones se deberían escribir incluyendo lógica de reintentos para asegurarse de que se vuelven a conectar.  Aunque esta breve desconexión no suele apreciarse, puede optar por repatriar las bases de datos fuera del horario comercial. 


### <a name="run-the-repatriation-script"></a>Ejecución del script de repatriación
Ahora imaginemos que la interrupción se resuelve y ejecuta el script de repatriación.

1. En *PowerShell ISE*, en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1.

2. verifique que el proceso de sincronización del catálogo aún se está ejecutando en su instancia de PowerShell.  Si es necesario, reinícielo estableciendo:
    * **$DemoScenario = 1**, iniciar la sincronización de la información de configuración del servidor de inquilino, el grupo y la base de datos en el catálogo
    * Presione **F5** para ejecutar el script.

3.  Luego, para iniciar el proceso de repatriación, establezca:
    * **$DemoScenario = 6**, repatriar la aplicación en su región original
    * Presione **F5** para ejecutar el script de recuperación en una nueva ventana de PowerShell.  La repatriación tardará varios minutos y se puede supervisar en la ventana de PowerShell.
    ![Proceso de repatriación](./media/saas-dbpertenant-dr-geo-replication/repatriation-process.png)

4. Mientras se ejecuta el script, actualice la página del centro de eventos (http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net).
    * Observe que todos los inquilinos están en línea y son accesibles durante este proceso.

5. Después de finalizar la repatriación, actualice el centro de eventos y abra la página de eventos de Hawthorn Hall. Observe que esta base de datos se ha repatriado a la región original.
    ![Centro de eventos repatriado](./media/saas-dbpertenant-dr-geo-replication/events-hub-repatriated.png)


## <a name="designing-the-application-to-ensure-app-and-database-are-colocated"></a>Diseño de la aplicación para asegurar que esta y la base de datos están ubicadas en el mismo lugar 
La aplicación está diseñada para conectarse siempre desde una instancia en la misma región que la base de datos de inquilino. Este diseño reduce la latencia entre la aplicación y la base de datos. Esta optimización supone que la interacción entre la aplicación y la base de datos es más activa que la interacción entre el usuario y la aplicación.  

Las bases de datos de inquilino pueden distribuirse en regiones de recuperación y originales durante algún tiempo durante la repatriación. Para cada base de datos, la aplicación busca la región en la que se encuentra la base de datos mediante una búsqueda de DNS por el nombre del servidor del inquilino. En SQL Database, el nombre del servidor es un alias. El nombre del servidor con alias contiene el nombre de la región. Si la aplicación no se encuentra en la misma región que la base de datos, se redirige a la instancia en la misma región que el servidor. El redireccionamiento a la instancia en la misma región que la base de datos minimiza la latencia entre la aplicación y la base de datos. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:
> [!div class="checklist"]
> 
> * Sincronizar la información de configuración de la base de datos y los grupos elásticos en el catálogo de inquilinos
> * Configurar un entorno de recuperación en una región alternativa, que incluye aplicaciones, servidores y grupos
> * Usar la _replicación geográfica_ para replicar las bases de datos de catálogo y de inquilinos en la región de recuperación
> * Conmutar por error la aplicación y las bases de datos de catálogo y de inquilinos a la región de recuperación 
> * Conmutar por recuperación la aplicación y las bases de datos de catálogo y de inquilinos a la región original después de resolver la interrupción

Puede aprender más acerca de las tecnologías que proporciona Azure SQL Database para habilitar la continuidad del negocio en la documentación de [Introducción a la continuidad del negocio](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Otros tutoriales basados en la aplicación SaaS de Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)