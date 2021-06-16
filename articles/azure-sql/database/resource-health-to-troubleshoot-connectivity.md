---
title: Uso de Azure Resource Health para supervisar el estado de una base de datos
description: Use Azure Resource Health para supervisar el estado de Azure SQL Database e Instancia administrada de Azure SQL, ayudar a diagnosticar problemas y obtener soporte técnico cuando un problema de Azure afecte a los recursos de SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: AlainDormehlMSFT
ms.author: aldorme
ms.reviewer: mathoma, wiassaf
ms.date: 03/24/2021
ms.openlocfilehash: 4be870b68c7679382d397bdd0d44a3191c3f082b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697450"
---
# <a name="use-resource-health-to-troubleshoot-connectivity-for-azure-sql-database-and-azure-sql-managed-instance"></a>Uso de Resource Health para solucionar problemas de conectividad para Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

[Resource Health](../../service-health/resource-health-overview.md#get-started) para Azure SQL Database e Instancia administrada de Azure SQL ayuda a diagnosticar problemas y obtener soporte técnico cuando un problema de Azure afecte a los recursos de SQL. Le informa acerca del mantenimiento actual y pasado de los recursos y le ayuda a mitigar los problemas. Resource Health le proporciona soporte técnico si necesita ayuda con los problemas de los servicios de Azure.

![Información general](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-overview.jpg)

## <a name="health-checks"></a>Comprobaciones de mantenimiento

Resource Health determina el estado del recurso de SQL al examinar el estado correcto o erróneo de los inicios de sesión para el recurso. Actualmente, Resource Health para el recurso de SQL Database solo examina los errores de inicio de sesión debido a errores del sistema y no a errores del usuario. El estado de Resource Health se actualiza cada 1 a 2 minutos.

## <a name="health-states"></a>Estados de mantenimiento

### <a name="available"></a>Disponible

Un estado de **Disponible** significa que Resource Health no ha detectado errores de inicio de sesión debido a errores del sistema en el recurso de SQL.

![Disponible](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-available.jpg)

### <a name="degraded"></a>Degradado

Un estado de **Degradado** significa que Resource Health ha detectado una mayoría de inicios de sesión correctos, pero también algunos errores. Probablemente estos sean errores transitorios de inicio de sesión. Para reducir el impacto de los problemas de conexión provocados por errores transitorios de inicio de sesión, implemente la [lógica de reintento](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) en el código.

![Degradado](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-degraded.jpg)

### <a name="unavailable"></a>No disponible

Un estado de **No disponible** significa que Resource Health ha detectado errores constantes de inicio de sesión al recurso de SQL. Si el recurso permanece en este estado durante un largo período, póngase en contacto con soporte técnico.

![No disponible](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-unavailable.jpg)

### <a name="unknown"></a>Desconocido

Este estado de mantenimiento indica que Resource Health no ha recibido información sobre este recurso durante más de 10 minutos. Si bien este estado no es una indicación definitiva del estado del recurso, es un punto de datos importante en el proceso de solución de problemas. Si el recurso se ejecuta según lo esperado, el estado del recurso cambia a Disponible al cabo de unos minutos. Si tiene problemas con el recurso, el estado de mantenimiento Desconocido podría sugerir que un evento de la plataforma afecta al recurso.

![Desconocido](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-unknown.jpg)

## <a name="historical-information"></a>Información histórica

Puede obtener acceso al historial de mantenimiento de hasta 14 días en la sección Historial de estado de Resource Health. La sección también contendrá el motivo del tiempo de inactividad (si está disponible) para los tiempos de inactividad notificados por Resource Health. Actualmente, Azure muestra el tiempo de inactividad para el recurso de base de datos con una granularidad de dos minutos. Es probable que el tiempo de inactividad real sea de menos de un minuto. El promedio es de 8 segundos.

### <a name="downtime-reasons"></a>Motivos del tiempo de inactividad

Cuando la base de datos experimenta un tiempo de inactividad, se realizan análisis para determinar un motivo. Cuando esté disponible, el motivo del tiempo de inactividad se notifica en la sección Historial de estado de Resource Health. Los motivos de tiempo de inactividad se publican normalmente en un plazo de 45 minutos después de un evento.

#### <a name="planned-maintenance"></a>Mantenimiento planeado

La infraestructura de Azure realiza periódicamente un mantenimiento planeado: actualización de componentes de hardware o software en el centro de datos. Mientras la base de datos se somete a mantenimiento, Azure SQL puede terminar algunas conexiones existentes y rechazar otras nuevas. Los errores de inicio de sesión sufridos durante un mantenimiento planeado suelen ser transitorios y la [lógica de reintento](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) ayuda a reducir el impacto. Si sigue experimentando errores de inicio de sesión, póngase en contacto con soporte técnico.

#### <a name="reconfiguration"></a>Reconfiguración

Las reconfiguraciones se consideran condiciones transitorias y se espera que se produzcan de vez en cuando. Estos eventos pueden desencadenarse por errores de software y hardware o de equilibrio de carga. Cualquier aplicación de producción cliente que se conecte a un servicio de base de datos en la nube debe implementar una [lógica de reintento](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) de conexión sólida, ya que podría ayudar a mitigar estas situaciones y, en general, debería hacer que los errores sean transparentes para el usuario final.

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información sobre la [lógica de reintento para errores transitorios](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors).
- [Cómo solucionar problemas, diagnosticar y evitar errores de conexión de SQL y errores transitorios de SQL Database](troubleshoot-common-connectivity-issues.md).
- Más información sobre la [configuración de alertas de Resource Health](../../service-health/resource-health-alert-arm-template-guide.md).
- Obtener una introducción a [Resource Health](../../service-health/resource-health-overview.md).
- Revisar [P+F sobre Resource Health](../../service-health/resource-health-faq.md).
