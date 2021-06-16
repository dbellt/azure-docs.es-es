---
title: Restauración de una base de datos en una aplicación SaaS multiinquilino
description: Obtenga información sobre cómo restaurar una base de datos de Azure SQL Database de inquilino único después de la eliminación accidental de datos
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 12/04/2018
ms.openlocfilehash: 0832e7c8bac9677ac2130c4695604597233c8881
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699107"
---
# <a name="restore-a-single-tenant-with-a-database-per-tenant-saas-application"></a>Restauración de un solo inquilino con una aplicación SaaS de base de datos por inquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

El modelo de base de datos por inquilino facilita la restauración de un inquilino único a un momento dado anterior sin afectar a otros inquilinos.

En este tutorial, conocerá dos patrones de recuperación de datos:

> [!div class="checklist"]
> * Restauración de una base de datos en una base de datos en paralelo (lado a lado).
> * Restauración de una base de datos en contexto, reemplazando la base de datos existente.

| Patrón | Descripción |
|:--|:--|
| Restauración en una base de datos en paralelo | Este patrón se puede usar para tareas como la revisión, la auditoría, el cumplimiento, etc. para permitir que un inquilino inspeccione los datos desde un punto anterior. La base de datos actual del inquilino sigue en línea y sin modificaciones. |
| Restauración en contexto | Este patrón se usa habitualmente para recuperar un inquilino a un punto anterior, después de que un inquilino elimina o daña datos de manera accidental. La base de datos original se deja sin conexión y se reemplaza por la base de datos restaurada. |
|||

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* Se implementa la aplicación SaaS de Wingtip. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de Wingtip](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-saas-tenant-restore-patterns"></a>Introducción a los patrones de restauración de inquilino de SaaS

Hay dos patrones simples para restaurar los datos de un inquilino individual. Debido a que las bases de datos de inquilino están aisladas entre sí, restaurar un inquilino no afecta los datos de los demás inquilinos. La característica de restauración a un momento dado (PITR) de Azure SQL Database se usa en ambos patrones. Esta característica siempre crea una base de datos nueva.

* **Restauración en paralelo**: en el primer patrón, se crea una base de datos en paralelo junto a la base de datos actual del inquilino. A continuación, el inquilino recibe acceso de solo lectura a la base de datos restaurada. Los datos restaurados se pueden revisar y potencialmente usar para sobrescribir los valores de datos actuales. Del diseñador de la aplicación depende determinar cómo el inquilino accede a la base de datos restaurada y las opciones de recuperación que se proporcionan. En algunos escenarios, es posible que todo lo que se necesite sea simplemente permitir que el inquilino revise los datos en un punto anterior.

* **Restauración en contexto**: el segundo patrón resulta útil si se perdieron o dañaron datos y el inquilino quiere revertir a un punto anterior. El inquilino queda sin conexión mientras se restaura la base de datos. La base de datos original se elimina y se cambia el nombre de la base de datos restaurada. La cadena de copia de seguridad de la base de datos original sigue siendo accesible después de la eliminación, para que pueda restaurar la base de datos a un momento dado anterior si es necesario.

Si la base de datos usa la [replicación geográfica activa](active-geo-replication-overview.md) y la restauración en paralelo, se recomienda copiar los datos requeridos desde la copia restaurada a la base de datos original. Si reemplaza la base de datos original por la restaurada, debe volver a configurar y sincronizar la replicación geográfica.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtención de los scripts de la aplicación Wingtip Tickets SaaS Database Per Tenant

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Si desea conocer los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets, consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md).

## <a name="before-you-start"></a>Antes de comenzar

Cuando se crea una base de datos, puede tardar entre 10 y 15 minutos hasta que la primera copia de seguridad completa esté disponible para restaurar desde ahí. Si acaba de instalar la aplicación, es posible que deba esperar unos minutos antes de probar este escenario.

## <a name="simulate-a-tenant-accidentally-deleting-data"></a>Simulación de la eliminación accidental de los datos de un inquilino

Para mostrar estos escenarios de recuperación, primero elimine de manera "accidental" un evento en una de las bases de datos de inquilino. 

### <a name="open-the-events-app-to-review-the-current-events"></a>Apertura de la aplicación Eventos para revisar los eventos actuales

1. Abra Events Hub (http://events.wtp.&lt;usuario&gt;.trafficmanager.net) y seleccione **Contoso Concert Hall**.

   ![Centro de eventos](./media/saas-dbpertenant-restore-single-tenant/events-hub.png)

2. Desplace la lista de eventos y anote el último evento de la lista.

   ![Aparece el último evento](./media/saas-dbpertenant-restore-single-tenant/last-event.png)

### <a name="accidentally-delete-the-last-event"></a>Eliminación "accidental" del último evento

1. En PowerShell ISE, abra ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\*Demo-RestoreTenant.ps1*, y establezca el valor siguiente:

   * **$DemoScenario** = **1**, *eliminación del último evento (sin venta de entradas)* .
2. Presione F5 para ejecutar el script y eliminar el último evento. Aparece el mensaje de confirmación siguiente:

   ```Console
   Deleting last unsold event from Contoso Concert Hall ...
   Deleted event 'Seriously Strauss' from Contoso Concert Hall venue.
   ```

3. Se abre la página de eventos de Contoso. Desplácese hacia abajo y compruebe que el evento desapareció. Si todavía está en la lista, seleccione **Actualizar**  y compruebe que ya no aparece.
   ![Último evento eliminado](./media/saas-dbpertenant-restore-single-tenant/last-event-deleted.png)

## <a name="restore-a-tenant-database-in-parallel-with-the-production-database"></a>Restauración de una base de datos de inquilino en paralelo con la base de datos de producción

En este ejercicio se restaura la base de datos de Contoso Concert Hall a un momento específico antes de que se eliminara el evento. Este escenario supone que desea revisar los datos eliminados en una base de datos en paralelo.

 El script *Restore-TenantInParallel.ps1* crea una base de datos de inquilino en paralelo denominada *ContosoConcertHall\_old*, con una entrada de catálogo en paralelo. Este patrón de restauración es más adecuado para recuperarse de una pérdida de datos sin importancia. También puede usar este patrón si necesita revisar los datos con fines de cumplimiento y auditoría. Es el enfoque que se recomienda cuando se usa la [replicación geográfica activa](active-geo-replication-overview.md).

1. Complete la sección en que se [simula que un usuario elimina datos de manera accidental](#simulate-a-tenant-accidentally-deleting-data).
2. En PowerShell ISE, abra ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\_Demo-RestoreTenant.ps1_.
3. Establezca **$DemoScenario** = **2**, *Restore tenant in parallel* (Restaurar el inquilino en paralelo).
4. Presione F5 para ejecutar el script.

El script restaura la base de datos de inquilino a un momento dado antes de que eliminara el evento. La base de datos se restaura a una base de datos nueva denominada _ContosoConcertHall\_old_. Los metadatos de catálogo que existen en esta base de datos restaurada se eliminar y, luego, la base de datos se agrega al catálogo mediante una clave construida a partir del nombre *ContosoConcertHall\_old*.

El script de demostración abre la página de eventos de esta base de datos de inquilino en el explorador. En la dirección URL ```http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall_old```, tenga en cuenta que esta página muestra los datos de la base de datos restaurada, donde se agrega *_old* al nombre.

Desplace los eventos que aparecen en el explorador y confirmar que se restauró el evento que se eliminó en la sección anterior.

Exponer al inquilino restaurado como un inquilino adicional, con su propia aplicación de eventos, puede que no sea la forma en que proporciona acceso de inquilino a los datos restaurados. Sí sirve para ilustrar el patrón de restauración. Habitualmente, proporciona acceso de solo lectura a los datos antiguos y retiene la base de datos restaurada durante un período definido. En el ejemplo, puede eliminar la entrada de inquilino restaurado una vez que finalice mediante la ejecución del escenario _Remove restored tenant_ (Quitar el inquilino restaurado).

1. Establezca **$DemoScenario** = **4**, *Remove restored tenant* (Quitar el inquilino restaurado).
2. Presione F5 para ejecutar el script.
3. La entrada *ContosoConcertHall\_old* se eliminó del catálogo. Cierre la página de eventos de este inquilino en el explorador.

## <a name="restore-a-tenant-in-place-replacing-the-existing-tenant-database"></a>Restauración de un inquilino en contexto, reemplazando la base de datos de inquilino existente

En este ejercicio se restaura el inquilino de Contoso Concert Hall a un momento antes de que se eliminara el evento. El script *Restore-TenantInPlace* restaura una base de datos de inquilino a una base de datos nueva y elimina la original. Este patrón de restauración es más adecuado para la recuperación después de daños graves en los datos y el inquilino podría tener que adecuarse a la pérdida de datos importantes.

1. En PowerShell ISE, abra el archivo **Demo-RestoreTenant.ps1**.
2. Establezca **$DemoScenario** = **5**, *Restore tenant in place* (Restaurar el inquilino en contexto).
3. Presione F5 para ejecutar el script.

El script restaura la base de datos de inquilino a un punto antes de que se eliminara el evento. En primer lugar, deja sin conexión al inquilino de Contoso Concert Hall para evitar futuras actualizaciones. Luego, se crea una base de datos en paralelo mediante la restauración a partir del punto de restauración. Luego, la base de datos restaurada recibe una marca de tiempo como nombre para garantizar que el nombre de la base de datos no entre en conflicto con el nombre de la base de datos de inquilino existente. A continuación, se elimina la base de datos de inquilino anterior y el nombre de la base de datos restaurada se cambia al nombre de la base de datos original. Por último, Contoso Concert Hall se vuelve a conectar para permitir que la aplicación tenga acceso a la base de datos restaurada.

La base de datos se restauró correctamente a un momento dado antes de que se eliminara el evento. Cuando se abra la página de **eventos**, confirme que se restauró el último evento.

Después de que restaure la base de datos, tardará otros 10 a 15 minutos antes de que la primera copia de seguridad completa esté disponible para volver a realizar la restauración.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Restauración de una base de datos en una base de datos en paralelo (lado a lado).
> * Restauración de una base de datos en contexto.

Prueba del tutorial [Administración de un esquema de base de datos de inquilino](saas-tenancy-schema-management.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Otros tutoriales basados en la aplicación SaaS de Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Introducción a la continuidad empresarial con Azure SQL Database](business-continuity-high-availability-disaster-recover-hadr-overview.md)
* [Más información sobre las copias de seguridad de SQL Database](automated-backups-overview.md)