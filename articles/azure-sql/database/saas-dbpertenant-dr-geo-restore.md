---
title: 'Aplicaciones SaaS: Copias de seguridad con redundancia geográfica para la recuperación ante desastres'
description: Aprenda a usar copias de seguridad con redundancia geográfica de Azure SQL Database para recuperar una aplicación SaaS multiinquilino en caso de una interrupción.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.date: 01/14/2019
ms.openlocfilehash: cedcd6514f8a5708f8828cb14c6a19da34cb2c15
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708261"
---
# <a name="use-geo-restore-to-recover-a-multitenant-saas-application-from-database-backups"></a>Uso de la restauración geográfica para recuperar una aplicación SaaS multiinquilino a partir de copias de seguridad de base de datos
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, exploraremos un escenario completo de recuperación ante desastres para una aplicación SaaS multiinquilino implementada con el modelo de base de datos por inquilino. Se usará la [restauración geográfica](recovery-using-backups.md) para recuperar el catálogo y las bases de datos de inquilino a partir de copias de seguridad con redundancia geográfica mantenidas automáticamente en una región de recuperación alternativa. Después de que se resuelva la interrupción, usará la [replicación geográfica](active-geo-replication-overview.md) para repatriar las bases de datos cambiadas a su región original.

![El diagrama muestra la región original y la de recuperación, ambas incluyen una aplicación, un catálogo, imágenes de servidores y grupos originales o reflejados, copias de seguridad automáticas en el almacenamiento, con una región de recuperación que acepta la replicación geográfica de la copia de seguridad y que dispone de un servidor y un grupo para los nuevos inquilinos.](./media/saas-dbpertenant-dr-geo-restore/geo-restore-architecture.png)

La restauración geográfica es la solución de recuperación ante desastres de menor costo para Azure SQL Database. Sin embargo, la restauración desde copias de seguridad con redundancia geográfica puede producir pérdida de datos de hasta una hora. Además, puede tardar bastante tiempo, según el tamaño de cada base de datos. 

> [!NOTE]
> Recupere las aplicaciones con los menores RPO y RTO posibles usando la replicación geográfica en lugar de la restauración geográfica.

En este tutorial se exploran los flujos de trabajo de la restauración y la repatriación. Aprenderá a:
> [!div class="checklist"]
> 
> * Sincronizar la información de configuración de la base de datos y los grupos elásticos en el catálogo de inquilinos
> * Configurar un entorno de imagen reflejada en una región de recuperación que incluye aplicaciones, servidores y grupos   
> * Recuperar el catálogo y las bases de datos de inquilinos mediante la restauración geográfica
> * Usar la replicación geográfica para repatriar el catálogo de inquilino y las bases de datos de inquilino modificadas después de resolver la interrupción
> * Actualizar el catálogo a medida que se restaure (o repatríe) cada base de datos para realizar el seguimiento de la ubicación actual de la copia activa de la base de datos de cada inquilino
> * Asegurarse de que la aplicación y la base de datos de inquilinos siempre están colocadas en la misma región de Azure para reducir la latencia 
 

Antes de empezar este tutorial, debe realizar estos pasos que son requisito previo:
* Implementar la base de datos SaaS Wingtip Tickets por aplicación de inquilino Para implementarla en menos de cinco minutos, consulte el artículo [Implementación y exploración de la aplicación de base de datos SaaS Wingtip Tickets por inquilino](saas-dbpertenant-get-started-deploy.md). 
* Instale Azure PowerShell. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-geo-restore-recovery-pattern"></a>Introducción al patrón de recuperación por restauración geográfica

La recuperación ante desastres (DR) es una consideración importante para muchas aplicaciones, ya sea por motivos de cumplimiento o por continuidad del negocio. En caso de una interrupción del servicio prolongada, un plan de recuperación ante desastres bien preparado puede minimizar la interrupción del negocio. Un plan de recuperación ante desastres por restauración geográfica debe cumplir varios objetivos:
 * Reservar toda la capacidad necesaria en la región de recuperación elegida tan pronto como sea posible para asegurarse de que está disponible para restaurar las bases de datos de inquilinos
 * Establecer un entorno de recuperación de imagen reflejada que refleje la configuración original de grupos y bases de datos 
 * Permitir la cancelación del proceso de restauración antes de que termine si la región original vuelve a estar conectada.
 * Habilitar rápidamente el aprovisionamiento de inquilinos para que la incorporación de nuevos inquilinos se pueda reiniciar tan pronto como sea posible
 * Estar optimizado para restaurar los inquilinos en orden de prioridad
 * Estar optimizado para que los inquilinos vuelvan a estar en línea tan pronto como sea posible mediante la realización de pasos en paralelo cuando sea conveniente
 * Ser resistente a errores, contar con capacidad de reinicio y ser idempotente
 * Repatriar las bases de datos a su región original con un impacto mínimo en los inquilinos cuando se resuelva la interrupción.  

> [!NOTE]
> La aplicación se recupera en la región emparejada de la región en la que se implementa la aplicación. Para más información, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md).   

En este tutorial se usan características de Azure SQL Database y la plataforma de Azure para abordar estos desafíos:

* [Plantillas de Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md), para reservar toda la capacidad necesaria tan rápido como sea posible. Las plantillas de Azure Resource Manager se utilizan para aprovisionar una imagen reflejada de los servidores originales y los grupos elásticos en la región de recuperación. También se crean un servidor y un grupo independientes para el aprovisionamiento de nuevos inquilinos.
* [Biblioteca cliente de Elastic Database](elastic-database-client-library.md) (EDCL), para crear y mantener un catálogo de bases de datos de inquilinos. El catálogo ampliado incluye información de configuración de grupos y bases de datos actualizada periódicamente.
* [Características de recuperación de administración de particiones](elastic-database-recovery-manager.md) de la EDCL, para mantener las entradas de ubicación de base de datos en el catálogo durante la recuperación y repatriación.  
* [Restauración geográfica](../../key-vault/general/disaster-recovery-guidance.md), para recuperar el catálogo y las bases de datos de inquilino a partir de copias de seguridad con redundancia geográfica mantenidas automáticamente. 
* [Operaciones de restauración asincrónicas](../../azure-resource-manager/management/async-operations.md) enviadas en orden de prioridad de inquilino, que el sistema pone en cola para cada grupo y se procesan por lotes de modo que el grupo no se sobrecarga. Estas operaciones se pueden cancelar antes de la ejecución o durante esta en caso de que sea necesario.   
* [Replicación geográfica](active-geo-replication-overview.md), para repatriar las bases de datos a la región original después de la interrupción. Cuando se usa la replicación geográfica, no hay pérdida de datos y el impacto en el inquilino es mínimo.
* [Alias DNS de SQL Server](./dns-alias-overview.md), para permitir que el proceso de sincronización del catálogo se conecte al catálogo activo, independientemente de su ubicación.  

## <a name="get-the-disaster-recovery-scripts"></a>Obtención de los scripts de recuperación ante desastres

Los scripts de recuperación ante desastres de este tutorial están disponibles en el [repositorio de GitHub SaaS Wingtip Tickets, con una base de datos por inquilino](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consulte en las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) los pasos para descargar y desbloquear los scripts de administración de Wingtip Tickets.

> [!IMPORTANT]
> Al igual que todos los scripts de administración de Wingtip vales, los scripts de recuperación ante desastres tienen una calidad de muestra y no para usarse en producción.

## <a name="review-the-healthy-state-of-the-application"></a>Revisión del estado de mantenimiento de la aplicación
Antes de comenzar el proceso de recuperación, revise el estado de mantenimiento normal de la aplicación.

1. En el explorador web, abra el centro de eventos de Wingtip Tickets: http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net (sustituya &lt; usuario&gt; por el valor del usuario de la implementación).
    
   Desplácese hasta la parte inferior de la página y observe el nombre y la ubicación del servidor del catálogo en el pie de página. La ubicación es la región en la que se implementó la aplicación.    

   > [!TIP]
   > Mantenga el puntero del mouse sobre la ubicación para ampliar la pantalla.

   ![Estado correcto del centro de eventos en la región original](./media/saas-dbpertenant-dr-geo-restore/events-hub-original-region.png)

2. Seleccione el inquilino Contoso Concert Hall y abra su página de eventos.

   En el pie de página, observe el nombre del servidor del inquilino. La ubicación será la misma que la ubicación del servidor del catálogo.

   ![Región original de Contoso Concert Hall](./media/saas-dbpertenant-dr-geo-restore/contoso-original-location.png)   

3. En [Azure Portal](https://portal.azure.com), revise y abra el grupo de recursos en el que se implementó la aplicación.

   Tenga en cuenta los recursos y la región donde se implementan los componentes de servicio de aplicaciones y SQL Database.

## <a name="sync-the-tenant-configuration-into-the-catalog"></a>Sincronización de la configuración del inquilino con el catálogo

En esta tarea, se inicia un proceso para sincronizar la configuración de los servidores, los grupos elásticos y las bases de datos con el catálogo de inquilino. Esta información se utiliza posteriormente para configurar un entorno de imagen reflejada en la región de recuperación.

> [!IMPORTANT]
> Para simplificar, el proceso de sincronización y otros procesos de recuperación y repatriación de larga ejecución se implementan en estos ejemplos como trabajos locales de PowerShell o sesiones que se ejecutan en el inicio de sesión de usuario del cliente. Los tokens de autenticación que se emiten al iniciar sesión expiran después de varias horas y entonces los trabajos darán error. En un escenario de producción, los procesos de ejecución prolongada se deben implementar como servicios confiables de Azure de algún tipo, que se ejecutan como una entidad de servicio. Consulte [Uso de Azure PowerShell para crear una entidad de servicio con un certificado](../../active-directory/develop/howto-authenticate-service-principal-powershell.md). 

1. En PowerShell ISE, abra el archivo ...\Learning Modules\UserConfig.psm1. Reemplace `<resourcegroup>` y `<user>` en las líneas 10 y 11 por el valor usado al implementar la aplicación. Guarde el archivo.

2. En PowerShell ISE, abra el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1.

    En este tutorial, ejecutará cada uno de los escenarios de este script de PowerShell, por lo que debe mantener este archivo abierto.

3. Configure las opciones siguientes:

    $DemoScenario = 1: Inicie un trabajo en segundo plano que sincronice la información de configuración del servidor de inquilino y del grupo con el catálogo.

4. Para ejecutar el script de sincronización, seleccione F5. 

    Esta información se utiliza posteriormente para asegurarse de que la recuperación crea una imagen reflejada de los servidores, los grupos y las bases de datos en la región de recuperación.  

    ![Proceso de sincronización](./media/saas-dbpertenant-dr-geo-restore/sync-process.png)

Deje la ventana de PowerShell ejecutándose en segundo plano y continúe con el resto del tutorial.

> [!NOTE]
> El proceso de sincronización conecta con el catálogo mediante un alias DNS. Este alias se modifica durante la restauración y la repatriación para apuntar al catálogo activo. El proceso de sincronización mantiene actualizado el catálogo con los cambios de configuración de las bases de datos o los grupos que se han realizado en la región de recuperación. Durante la repatriación, estos cambios se aplican a los recursos equivalentes en la región original.

## <a name="geo-restore-recovery-process-overview"></a>Introducción al proceso de recuperación con restauración geográfica

El proceso de recuperación de restauración geográfica implementa la aplicación y restaura las bases de datos a partir de copias de seguridad en la región de recuperación.

El proceso de recuperación hace lo siguiente:

1. Deshabilita el punto de conexión de Azure Traffic Manager de la aplicación web en la región original. La deshabilitación del punto de conexión impide que los usuarios se conecten a la aplicación en un estado no válido si la región original pasa a estar en línea durante la recuperación.

2. Aprovisiona un servidor de catálogo de recuperación en la región de recuperación, restaura geográficamente la base de datos de catálogo y actualiza el alias activecatalog para que apunte al servidor de catálogo restaurado. El cambio del alias de catálogo garantiza que el proceso de sincronización de catálogo siempre se sincroniza con el catálogo activo.

3. Marca todos los inquilinos existentes en el catálogo de recuperación como sin conexión para evitar el acceso a bases de datos de inquilino antes de que se restauren.

4. Aprovisiona una instancia de la aplicación en la región de recuperación y la configura para utilizar el catálogo restaurado en dicha región. Para mantener la latencia al mínimo, la aplicación de ejemplo está diseñada para que siempre se conecte a una base de datos de inquilinos en la misma región.

5. Aprovisiona un servidor y un grupo elástico en los que se aprovisionan nuevos inquilinos. La creación de estos recursos garantiza que el aprovisionamiento de nuevos inquilinos no interfiere en la recuperación de los inquilinos existentes.

6. Actualiza el alias del nuevo inquilino para que apunte al servidor de nuevas bases de datos de inquilino en la región de recuperación. El cambio de este alias garantiza que las bases de datos de los inquilinos nuevos se aprovisionan en la región de recuperación.
        
7. Aprovisiona los servidores y grupos elásticos en la región de recuperación para restaurar las bases de datos de inquilino. Estos servidores y los grupos son una imagen reflejada de la configuración de la región original. El aprovisionamiento por adelantado reserva la capacidad necesaria para restaurar todas las bases de datos.

    Una interrupción en una región puede provocar una presión considerable sobre los recursos disponibles en la región emparejada. Si confía en la restauración geográfica para recuperación ante desastres, se recomienda la reserva de recursos rápidamente. Considere el uso de la replicación geográfica si es fundamental que una aplicación se deba recuperar en una región específica. 

8. Habilita el punto de conexión de Traffic Manager de la aplicación web en la región de recuperación. La habilitación de este punto de conexión permite a la aplicación aprovisionar nuevos inquilinos. En esta fase, los inquilinos existentes siguen estando sin conexión.

9. Envía lotes de las solicitudes para restaurar las bases de datos en orden de prioridad. 

    * Los lotes se organizan de forma que las bases de datos se restauran en paralelo en todos los grupos.  

    * Las solicitudes de restauración se envían de forma asincrónica para que se envíen rápidamente y se pongan en cola para su ejecución en cada grupo.

    * Dado que las solicitudes de restauración se procesan en paralelo en todos los grupos, es mejor distribuir a los inquilinos importantes entre muchos grupos. 

10. Supervisa el servicio para determinar cuándo se restauran las bases de datos. Una vez que se restaura una base de datos de inquilinos, se marca como en línea en el catálogo y se registra una suma de rowversion para dicha base de datos. 

    * La aplicación puede acceder a las bases de datos de inquilino tan pronto como se marcan como en línea en el catálogo.

    * Una suma de valores rowversion en la base de datos de inquilino se almacena en el catálogo. Esta suma actúa como una huella digital que permite al proceso de repatriación determinar si la base de datos se actualizó en la región de recuperación.

## <a name="run-the-recovery-script"></a>Ejecución del script de recuperación

> [!IMPORTANT]
> Este tutorial restaura las bases de datos a partir de copias de seguridad con redundancia geográfica. Aunque estas copias de seguridad normalmente están disponibles en 10 minutos, pueden tardar hasta una hora. El script se pausa hasta que están disponibles.

Suponga que hay una interrupción en la región en la que la aplicación se implementa y ejecuta el script de recuperación:

1. En PowerShell ISE, establezca el siguiente valor en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1:

    $DemoScenario = 2: Recupere la aplicación en una región de recuperación mediante la restauración a partir de copias de seguridad con redundancia geográfica.

2. Para ejecutar el script, seleccione F5.  

    * El script se abre en una nueva ventana de PowerShell y luego inicia un conjunto de trabajos de PowerShell que se ejecutan en paralelo. Estos trabajos restauran los servidores, los grupos y las bases de datos en la región de recuperación.

    * La región de recuperación es la región emparejada asociada a la región de Azure en la que implementó la aplicación. Para más información, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md). 

3. Supervise el estado del proceso de recuperación en la ventana de PowerShell.

    ![Captura de pantalla que muestra la ventana de PowerShell en la que puede supervisar el estado del proceso de recuperación.](./media/saas-dbpertenant-dr-geo-restore/dr-in-progress.png)

> [!NOTE]
> Para explorar el código para los trabajos de recuperación, revise los scripts de PowerShell en la carpeta ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\RecoveryJobs.

## <a name="review-the-application-state-during-recovery"></a>Revisión del estado de la aplicación durante la recuperación
Mientras que el punto de conexión de la aplicación esté deshabilitado en Traffic Manager, la aplicación no está disponible. El catálogo se restaura y todos los inquilinos se marcan como sin conexión. A continuación, se habilita el punto de conexión de la aplicación en la región de recuperación y la aplicación vuelve a estar en línea. Aunque la aplicación esté disponible, los inquilinos aparecen sin conexión en el centro de eventos hasta que sus bases de datos se restauran. Es importante diseñar la aplicación para controlar las bases de datos de inquilino sin conexión.

* Después de recuperar la base de datos de catálogo pero antes de que los inquilinos vuelvan a estar en línea, actualice el centro de eventos de Wingtip Tickets en el explorador web.

  * En el pie de página, observe que el nombre del servidor de catálogo tiene ahora un sufijo -recovery y se encuentra en la región de recuperación.

  * Observe que los inquilinos que aún no se han restaurado se marcan como sin conexión y no se pueden seleccionar.   
 
    ![Proceso de recuperación](./media/saas-dbpertenant-dr-geo-restore/events-hub-tenants-offline-in-recovery-region.png)  

  * Si abre directamente la página de eventos de un inquilino mientras el inquilino está sin conexión, la página muestra una notificación "inquilino sin conexión". Por ejemplo, si Contoso Concert Hall está desconectado, intente abrir http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net/contosoconcerthall.

    ![Captura de pantalla que muestra una página de eventos sin conexión.](./media/saas-dbpertenant-dr-geo-restore/dr-in-progress-offline-contosoconcerthall.png)

## <a name="provision-a-new-tenant-in-the-recovery-region"></a>Aprovisionamiento de un nuevo inquilino en la región de recuperación
Incluso antes de que se restauren las bases de datos de inquilino, puede aprovisionar nuevos inquilinos en la región de recuperación. Más adelante, las nuevas bases de datos de inquilinos de la región de recuperación se repatrian con las bases de datos recuperadas.   

1. En PowerShell ISE, establezca la siguiente propiedad en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1:

    $DemoScenario = 3: Aprovisione un nuevo inquilino en la región de recuperación.

2. Para ejecutar el script, seleccione F5.

3. La página de eventos de Hawthorn Hall se abre en el explorador cuando el aprovisionamiento finaliza. 

    Tenga en cuenta que la base de datos de Hawthorn Hall se encuentra en la región de recuperación.

    ![Hall Hawthorn aprovisionado en la región de recuperación](./media/saas-dbpertenant-dr-geo-restore/hawthorn-hall-provisioned-in-recovery-region.png)

4. En el explorador, actualice la página del centro de eventos de Wingtip Tickets y verá que Hawthorn Hall está incluido. 

    Si ha aprovisionado a Hawthorn Hall sin esperar a la restauración de los otros inquilinos, es posible que estos todavía estén sin conexión.

## <a name="review-the-recovered-state-of-the-application"></a>Revisión del estado de recuperación de la aplicación

Cuando finaliza el proceso de recuperación, la aplicación y todos los inquilinos están totalmente funcionales en la región de recuperación. 

1. Una vez que la presentación de la ventana de la consola de PowerShell indica que se han recuperado todos los inquilinos, actualice el centro de eventos. 

    Todos los inquilinos aparecen en línea, incluido el nuevo, Hawthorn Hall.

    ![inquilinos nuevos y recuperados en el centro de eventos](./media/saas-dbpertenant-dr-geo-restore/events-hub-with-hawthorn-hall.png)

2. Haga clic en Contoso Concert Hall y abra su página de eventos. 

    En el pie de página, tenga en cuenta que la base de datos se encuentra en el servidor de recuperación ubicado en la región de recuperación.

    ![Contoso en la región de recuperación](./media/saas-dbpertenant-dr-geo-restore/contoso-recovery-location.png)

3. En [Azure Portal](https://portal.azure.com), abra la lista de grupos de recursos.  

    Observe el grupo de recursos que ha implementado, además del grupo de recursos de recuperación, con el sufijo -recovery. El grupo de recursos de recuperación contiene todos los recursos creados durante el proceso de recuperación, además de los nuevos recursos creados durante la interrupción. 

4. Abra el grupo de recursos de recuperación y observe los siguientes elementos:

   * Las versiones de recuperación de los servidores de catálogo y tenants1 con el sufijo -recovery. Las bases de datos de catálogo y de inquilino restauradas en estos servidores tienen los nombres utilizados en la región original.

   * El servidor SQL tenants2-dpt-&lt;usuario&gt;-recovery. Este servidor se utiliza para el aprovisionamiento de nuevos inquilinos durante la interrupción.

   * La instancia de App Service llamada events-wingtip-dpt-&lt;recoveryregion&gt;-&lt;usuario&gt;, que es la instancia de recuperación de la aplicación de eventos.

     ![Recursos de Contoso en la región de recuperación](./media/saas-dbpertenant-dr-geo-restore/resources-in-recovery-region.png) 
    
5. Abra el servidor SQL tenants2-dpt-&lt;usuario&gt;-recovery. Observe que contiene la base de datos hawthornhall y el grupo elástico Pool1. La base de datos hawthornhall está configurada como una base de datos elástica en el grupo elástico Pool1.

## <a name="change-the-tenant-data"></a>Cambio de los datos del inquilino 
En esta tarea, va a actualizar una de las bases de datos de inquilino restaurada. El proceso de repatriación copia las bases de datos restauradas que se han cambiado a la región original. 

1. En el explorador, busque la lista de eventos de Contoso Concert Hall, desplácese por los eventos y observe el último evento, Seriously Strauss.

2. En PowerShell ISE, establezca el siguiente valor en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1:

    $DemoScenario = 4: Elimine un evento de un inquilino en la región de recuperación.

3. Para ejecutar el script, seleccione F5.

4. Actualice la página de eventos de Contoso Concert Hall (http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net/contosoconcerthall) y observe que falta el evento Seriously Strauss.

En este punto del tutorial, ha recuperado la aplicación, que ahora se está ejecutando en la región de recuperación. Ha aprovisionado a un nuevo inquilino en la región de recuperación y los datos modificados de uno de los inquilinos restaurados.  

> [!NOTE]
> Otros tutoriales del ejemplo no están diseñados para ejecutarse con la aplicación en el estado de recuperación. Si desea explorar otros tutoriales, asegúrese de repatriar primero la aplicación.

## <a name="repatriation-process-overview"></a>Introducción al proceso de repatriación

El proceso de repatriación revierte la aplicación y sus bases de datos a su región original después de que se resuelva una interrupción.

![Repatriación con restauración geográfica](./media/saas-dbpertenant-dr-geo-restore/geo-restore-repatriation.png)   

El proceso:

1. Detiene cualquier actividad de restauración en curso y cancela las solicitudes de restauración de base de datos pendientes o sobre la marcha.

2. Se reactiva en las bases de datos de inquilino de la región original que no se han cambiado desde la interrupción. Estas bases de datos incluyen las que aún no se han recuperado y las recuperadas pero que no cambiaron posteriormente. Las bases de datos reactivadas están exactamente igual que estaban la última vez que los inquilinos accedieron a ellas.

3. Aprovisiona una imagen reflejada del servidor y el grupo elástico del nuevo inquilino en la región original. Finalizada esta acción, el alias del nuevo inquilino se actualiza para apuntar a este servidor. La actualización del alias provoca la incorporación de un nuevo inquilino en la región original en lugar de la región de recuperación.

3. Usa la replicación geográfica para mover el catálogo a la región original desde la región de recuperación.

4. Actualiza la configuración del grupo en la región original para que sea coherente con los cambios realizados en la región de recuperación durante la interrupción.

5. Crea los servidores y grupos necesarios para hospedar cualquier base de datos nueva creada durante la interrupción.

6. Usa la replicación geográfica para repatriar las bases de datos de inquilinos restauradas que se han actualizado después de la restauración y todas las nuevas bases de datos de inquilinos aprovisionadas durante la interrupción. 

7. Limpia los recursos creados en la región de recuperación durante el proceso de restauración.

Para limitar el número de bases de datos de inquilinos que necesitan repatriarse, los pasos 1 a 3 se realizan de inmediato.  

El paso 4 solo se realiza si se ha modificado el catálogo en la región de recuperación durante la interrupción. El catálogo se actualiza si se crean nuevos inquilinos o si se modifica la configuración de la base de datos o del grupo en la región de recuperación.

Es importante que la interrupción que el paso 7 cause a los inquilinos sea mínima y no se pierdan datos. Para lograr este objetivo, el proceso usa la replicación geográfica.

Antes de que cada base de datos se replique geográficamente, se elimina la base de datos correspondiente de la región original. Después, la base de datos de la región de recuperación se replica geográficamente creando una réplica secundaria en la región original. Una vez finalizada la replicación, el inquilino se marca como sin conexión en el catálogo, lo que rompe todas las conexiones con la base de datos en la región de recuperación. Después, la base de datos conmuta por error, lo que provoca que las transacciones pendientes se procesen en la réplica secundaria, evitando así la pérdida de datos. 

En la conmutación por error, los roles de base de datos se invierten. La réplica secundaria de la región original se convierte en la base de datos principal de lectura y escritura, mientras que la base de datos de la región de recuperación se convierte en una réplica secundaria de solo lectura. La entrada del inquilino en el catálogo se actualiza para hacer referencia a la base de datos en la región original y el inquilino se marca como en línea. En este punto, la repatriación de la base de datos está completada. 

Las aplicaciones se deben escribir con lógica de reintento para garantizar que se vuelven a conectar automáticamente cuando las conexiones se rompen. Cuando se vuelven a conectar usando el catálogo a la conexión rota, se conectan a la base de datos repatriada en la región original. Aunque la breve desconexión no suele apreciarse, puede optar por repatriar las bases de datos fuera del horario comercial.

Una vez repatriada una base de datos, se puede eliminar la base de datos secundaria de la región de recuperación. Después, la base de datos de la región original vuelve a basarse en la restauración geográfica para la protección de recuperación ante desastres.

En el paso 8, se eliminan los recursos de la región de recuperación, incluidos los grupos y servidores de recuperación.

## <a name="run-the-repatriation-script"></a>Ejecución del script de repatriación
Ahora imaginemos que la interrupción se resuelve y ejecuta el script de repatriación.

Si ha seguido el tutorial, el script reactiva inmediatamente Fabrikam Jazz Club y Dogwood Dojo en la región original porque no han cambiado. Luego repatría el nuevo inquilino, Hawthorn Hall, y Contoso Concert Hall porque se ha modificado. El script también repatría el catálogo, que se actualizó cuando se aprovisionó Hawthorn Hall.
  
1. En PowerShell ISE, en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 script, verifique que el proceso de sincronización del catálogo está todavía en ejecución en su instancia de PowerShell. Si es necesario, reinícielo estableciendo:

    $DemoScenario = 1: Inicie la sincronización de la información de configuración del servidor de inquilino, el grupo y la base de datos en el catálogo.

    Para ejecutar el script, seleccione F5.

2.  Luego, para iniciar el proceso de repatriación, establezca:

    $DemoScenario = 5: Repatrie la aplicación a su región original.

    Para ejecutar el script de recuperación en una nueva ventana de PowerShell, seleccione F5. La repatriación tarda varios minutos y se puede supervisar en la ventana de PowerShell.

3. Mientras se ejecuta el script, actualice la página del centro de eventos (http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net).

    Observe que todos los inquilinos están en línea y son accesibles durante este proceso.

4. Seleccione Fabrikam Jazz Club para abrirlo. Si no modificó este inquilino, observe en el pie de página que el servidor ya se ha revertido al servidor original.

5. Abra o actualice la página de eventos de Contoso Concert Hall. Observe por el pie de página que, inicialmente, la base de datos todavía está en el servidor -recovery. 

6. Actualice la página de eventos de Contoso Concert Hall cuando finalice el proceso de repatriación y observe que la base de datos se encuentra ahora en su región original.

7. Vuelva a actualizar el centro de eventos y abra Hawthorn Hall. Tenga en cuenta que su base de datos también está ubicada en la región original. 

## <a name="clean-up-recovery-region-resources-after-repatriation"></a>Limpieza de los recursos de la región de recuperación después de la repatriación
Una vez completada la repatriación, es seguro eliminar los recursos de la región de recuperación. 

> [!IMPORTANT]
> Elimine estos recursos rápidamente para detener toda la facturación provocada por los mismos.

El proceso de restauración crea todos los recursos de recuperación en un grupo de recursos de recuperación. El proceso de limpieza elimina este grupo de recursos y quita todas las referencias a los recursos del catálogo. 

1. En PowerShell ISE, en el script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, establezca:
    
    $DemoScenario = 6: Elimine recursos obsoletos de la región de recuperación.

2. Para ejecutar el script, seleccione F5.

Después de limpiar los scripts, la aplicación vuelve donde comenzó. En este momento, puede volver a ejecutar el script o probar otros tutoriales.

## <a name="designing-the-application-to-ensure-that-the-app-and-the-database-are-co-located"></a>Diseño de la aplicación para asegurarse de que la aplicación y la base de datos están ubicadas conjuntamente 
La aplicación está diseñada para conectarse siempre desde una instancia de la misma región que la base de datos del inquilino. Este diseño reduce la latencia entre la aplicación y la base de datos. Esta optimización supone que la interacción entre la aplicación y la base de datos es más activa que la interacción entre el usuario y la aplicación.  

Las bases de datos de inquilinos pueden distribuirse entre regiones de recuperación y originales durante algún tiempo durante la repatriación. Para cada base de datos, la aplicación busca la región en la que se encuentra la base de datos mediante una búsqueda de DNS por el nombre del servidor del inquilino. El nombre del servidor es un alias. El nombre del servidor con alias contiene el nombre de la región. Si la aplicación no se encuentra en la misma región que la base de datos, se redirige a la instancia en la misma región que el servidor. El redireccionamiento a la instancia de la misma región que la base de datos minimiza la latencia entre la aplicación y la base de datos.  

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:
> [!div class="checklist"]
> 
> * Usar el catálogo de inquilino para mantener actualizada la información de configuración periódicamente, lo que permite crear un entorno de recuperación de imagen reflejada en otra región
> * Recuperar bases de datos en la región de recuperación mediante la restauración geográfica
> * Actualizar el catálogo de inquilino para reflejar las ubicaciones de las bases de datos de inquilinos restauradas 
> * Usar un alias de DNS para habilitar una aplicación para conectarse al catálogo de inquilino sin reconfiguración
> * Usar la replicación geográfica para repatriar las bases de datos recuperadas a su región original después de que se resuelve una interrupción

Pruebe el tutorial sobre la [recuperación ante desastres de una aplicación SaaS multiinquilino mediante la replicación geográfica](./saas-dbpertenant-dr-geo-replication.md) para aprender a reducir drásticamente el tiempo necesario para recuperar una aplicación multiinquilino a gran escala mediante la replicación geográfica.

## <a name="additional-resources"></a>Recursos adicionales

[Otros tutoriales basados en la aplicación SaaS de Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)