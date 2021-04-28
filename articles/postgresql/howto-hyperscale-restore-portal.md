---
title: 'Restauración: Hiperescala (Citus) en Azure Database for PostgreSQL a través de Azure Portal'
description: En este artículo se explica cómo realizar operaciones de restauración con Hiperescala (Citus) en Azure Database for PostgreSQL mediante Azure Portal.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 04/14/2021
ms.openlocfilehash: 99e507fed35e5b74aa44bc2713550f7cf9d7b2e5
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107518888"
---
# <a name="point-in-time-restore-of-a-hyperscale-citus-server-group"></a>Restauración a un momento dado de un grupo de servidores de Hiperescala (Citus)

En este artículo se incluyen instrucciones paso a paso para realizar [restauraciones a un momento dado](concepts-hyperscale-backup.md#restore) para un grupo de servidores de Hiperescala (Citus) mediante copias de seguridad. Puede restaurar ya sea a la copia de seguridad más antigua o a un punto de restauración personalizado dentro del período de retención.

## <a name="restoring-to-the-earliest-restore-point"></a>Restauración al punto de restauración más antiguo

Siga estos pasos para restaurar el grupo de servidores de Hiperescala (Citus) a la primera copia de seguridad que se haya creado.

1.  En [Azure Portal](https://portal.azure.com/), elija el grupo de servidores que quiera restaurar.

2.  Haga clic en **Información general** en el panel izquierdo y, después, haga clic en **Restaurar**.

    > [!IMPORTANT]
    > Si el botón **Restaurar** aún no aparece para el grupo de servidores, abra una solicitud de soporte técnico de Azure.

3.  La página Restaurar le pedirá que elija entre el punto de restauración **más antiguo** y uno **personalizado**, y mostrará la fecha más antigua.

4.  Seleccione **Punto de restauración más antiguo**.

5.  Indique un nuevo nombre de grupo de servidores en el campo **Restaurar en el servidor nuevo**. Se muestran los demás campos (suscripción, grupo de recursos y ubicación), pero no se pueden editar.

6.  Haga clic en **Aceptar**.

7.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.

Por último, realice las [tareas posteriores a la restauración](#post-restore-tasks).

## <a name="restoring-to-a-custom-restore-point"></a>Restauración a un punto de restauración personalizado

Siga estos pasos para restaurar el grupo de servidores de Hiperescala (Citus) a la fecha y hora que elija.

1.  En [Azure Portal](https://portal.azure.com/), elija el grupo de servidores que quiera restaurar.

2.  Haga clic en **Información general** en el panel izquierdo y, después, haga clic en **Restaurar**.

    > [!IMPORTANT]
    > Si el botón **Restaurar** aún no aparece para el grupo de servidores, abra una solicitud de soporte técnico de Azure.

3.  La página Restaurar le pedirá que elija entre el punto de restauración **más antiguo** y uno **personalizado**, y mostrará la fecha más antigua.

4.  Elija un **punto de restauración personalizado**.

5.  Seleccione la fecha y hora del **Punto de restauración (UTC)** y especifique un nuevo nombre de grupo de servidores en el campo **Restaurar en el servidor nuevo**. Se muestran los demás campos (suscripción, grupo de recursos y ubicación), pero no se pueden editar.
 
6.  Haga clic en **Aceptar**.

7.  Se mostrará una notificación en la que se indica que se ha iniciado la operación de restauración.

Por último, realice las [tareas posteriores a la restauración](#post-restore-tasks).

## <a name="post-restore-tasks"></a>Tareas posteriores a la restauración

Cuando efectúe una restauración, debe hacer lo siguiente para que los usuarios y las aplicaciones vuelvan a conectarse:

* Si el nuevo servidor está destinado a reemplazar al servidor original, redirija los clientes y las aplicaciones de cliente al nuevo servidor.
* Asegúrese de aplicar un firewall de nivel de servidor adecuado para que se conecten los usuarios. Estas reglas no se copian del grupo de servidores original.
* Ajuste los parámetros del servidor PostgreSQL según sea necesario. Los parámetros no se copian del grupo de servidores original.
* No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
* Configure las alertas según corresponda.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de la [copia de seguridad y restauración](concepts-hyperscale-backup.md) en Hiperescala (Citus).
* Establezca  [alertas sugeridas](./howto-hyperscale-alert-on-metric.md#suggested-alerts) en los grupos de servidores de Hiperescala (Citus).
