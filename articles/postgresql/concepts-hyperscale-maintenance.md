---
title: 'Mantenimiento programado en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: 'En este artículo se describe la característica de mantenimiento programado de Azure Database for PostgreSQL: Hiperescala (Citus).'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 336b8ff034122373c9bd824d76c110c808e73010
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315586"
---
# <a name="scheduled-maintenance-in-azure-database-for-postgresql--hyperscale-citus"></a>Mantenimiento programado en Azure Database for PostgreSQL: Hiperescala (Citus)

Azure Database for PostgreSQL: Hiperescala (Citus) realiza un mantenimiento periódico para conservar la base de datos administrada protegida, estable y actualizada.  Durante el mantenimiento, todos los nodos del grupo de servidores obtienen nuevas características, actualizaciones y revisiones.

Las principales características del mantenimiento programado de Hiperescala (Citus) son:

* Las actualizaciones se aplican al mismo tiempo en todos los nodos del grupo de servidores.
* Las notificaciones acerca del próximo mantenimiento se publican en Azure Service Health con cinco días de antelación.
* Normalmente, hay al menos 30 días entre los eventos de mantenimiento correctos de un grupo de servidores.
* El día preferido de la semana y el período de tiempo dentro de ese día para el inicio del mantenimiento se pueden definir individualmente para cada grupo de servidores.

## <a name="selecting-a-maintenance-window-and-notification-about-upcoming-maintenance"></a>Selección de una ventana de mantenimiento y notificación sobre el próximo mantenimiento

Puede programar el mantenimiento durante un día concreto de la semana y una ventana de tiempo en ese día. También puede dejar que el sistema elija un día y una hora de esa ventana de tiempo automáticamente. En cualquier caso, el sistema avisa cinco días antes de ejecutar cualquier mantenimiento. El sistema también comunica cuándo se ha iniciado el mantenimiento y cuándo ha finalizado correctamente.

Las notificaciones acerca del próximo mantenimiento se publican en Azure Service Health y también se pueden:

* Enviar por correo electrónico a una dirección concreta
* Enviar por correo electrónico a un rol de Azure Resource Manager
* Enviar en un mensaje de texto (SMS) a dispositivos móviles
* Insertar como una notificación en una aplicación de Azure
* Entregar como un mensaje de voz

A la hora de especificar las preferencias de la programación de mantenimiento, puede elegir un día de la semana y una ventana de tiempo. Si no lo hace, el sistema elige horas entre las 23:00 y las 7:00 de la hora de la región del servidor. Puede definir diferentes programaciones para cada grupo de servidores de Hiperescala (Citus) de la suscripción de Azure.

> [!IMPORTANT]
> Normalmente, hay al menos 30 días entre los eventos de mantenimiento programado correctos de un grupo de servidores.
>
> Sin embargo, en el caso de una actualización de emergencia crítica, como una vulnerabilidad grave, la ventana de notificación podría ser inferior a cinco días. La actualización crítica se puede aplicar al servidor aunque se haya realizado un mantenimiento programado correcto durante los últimos 30 días.

Se puede actualizar la configuración de programación en cualquier momento. Si hay un mantenimiento programado para el grupo de servidores de Hiperescala (Citus) y actualiza la programación, los eventos existentes continuarán según lo programado originalmente. El cambio de configuración tendrá efecto después de completar correctamente los eventos existentes.

Si se produce un error en el mantenimiento o se cancela, el sistema generará una notificación.
Intentará realizar el mantenimiento de nuevo según la configuración de programación actual y le notificará cinco días antes del siguiente evento de mantenimiento.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [cambiar la programación de mantenimiento](howto-hyperscale-maintenance.md).
* Obtenga información sobre cómo [obtener notificaciones sobre próximos eventos de mantenimiento](../service-health/service-notifications.md) con Azure Service Health.
* Obtenga información sobre cómo [configurar alertas sobre próximos eventos de mantenimiento programado](../service-health/resource-health-alert-monitor-guide.md).
