---
title: 'Azure Database for PostgreSQL - Hiperescala (Citus): mantenimiento programado desde Azure Portal'
description: Aprenda a configurar el mantenimiento programado para Azure Database for PostgreSQL - Hiperescala (Citus) desde Azure Portal.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.topic: how-to
ms.date: 04/07/2021
ms.openlocfilehash: 04dd37e3e9abbfbec7badb036802e645cc7732b0
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108117"
---
# <a name="manage-scheduled-maintenance-settings-for-azure-database-for-postgresql--hyperscale-citus"></a>Administración de la configuración del mantenimiento programado para Azure Database for PostgreSQL - Hiperescala (Citus)

En la suscripción de Azure es posible especificar las opciones de mantenimiento de cada grupo de servidores de Hiperescala (Citus). Las opciones incluyen la programación de mantenimiento y la configuración de notificación para eventos de mantenimiento próximos y finalizados.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesita:

- Un [grupo de servidores de Azure Database for PostgreSQL - Hiperescala (Citus)](quickstart-create-hyperscale-portal.md)

## <a name="specify-maintenance-schedule-options"></a>Especificación de opciones de programación de mantenimiento

1. En la página del grupo de servidores de Hiperescala (Citus), bajo el título **Configuración**, elija **Mantenimiento** para abrir las opciones de mantenimiento programado.
2. La programación predeterminada (administrada por el sistema) es un día de la semana aleatorio y una franja de 30 minutos para el mantenimiento que se inicia entre las 23:00 y las 7:00 ([hora de la región de Azure](https://go.microsoft.com/fwlink/?linkid=2143646) del grupo de servidores). Si quiere personalizar esta programación, elija **Programación personalizada**. Después, puede seleccionar el día de la semana que prefiera y una franja de 30 minutos para la hora de inicio del mantenimiento.

## <a name="notifications-about-scheduled-maintenance-events"></a>Notificaciones sobre eventos de mantenimiento programado

Puede usar Azure Service Health para [ver notificaciones](../service-health/service-notifications.md) sobre el próximo mantenimiento programado o el que ya se ha realizado en el grupo de servidores de Hiperescala (Citus). También puede [configurar](../service-health/resource-health-alert-monitor-guide.md) alertas en Azure Service Health para recibir notificaciones sobre los eventos de mantenimiento.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [mantenimiento programado en Azure Database for PostgreSQL: Hiperescala (Citus)](concepts-hyperscale-maintenance.md)
* Información sobre [Azure Service Health](../service-health/overview.md)
