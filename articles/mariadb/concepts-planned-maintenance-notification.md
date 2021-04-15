---
title: 'Notificación de mantenimiento planeado: Azure Database for MariaDB'
description: En este artículo se describe la característica Notificación de mantenimiento planeado en Azure Database for MariaDB
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: 2f6667980d35db611cfba293839ebf06535ee6a3
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107225859"
---
# <a name="planned-maintenance-notification-in-azure-database-for-mariadb"></a>Notificación de mantenimiento planeado en Azure Database for MariaDB

Obtenga información sobre cómo prepararse para eventos de mantenimiento planeado en Azure Database for MariaDB.

## <a name="what-is-a-planned-maintenance"></a>¿Qué es el mantenimiento planeado?

El servicio Azure Database for MariaDB aplica automáticamente revisiones al hardware, sistema operativo y motor de base de datos subyacentes. La revisión incluye nuevas características de servicio, seguridad y actualizaciones de software. Para el motor de MariaDB, las actualizaciones de versiones secundarias son automáticas y se incluyen como parte del ciclo de aplicación de revisión. No se requiere ninguna acción del usuario ni opciones de configuración para la aplicación de revisión. La revisión se prueba exhaustivamente y se implementa mediante procedimientos de implementación seguros.

Un mantenimiento planeado es una ventana de mantenimiento durante la que estas actualizaciones del servicio se implementan en los servidores de una región de Azure concreta. Durante el mantenimiento planeado, se crea un evento de notificación para informar a los clientes de cuándo se implementa la actualización del servicio en la región de Azure en la que se hospedan sus servidores. La duración mínima entre dos mantenimientos planeados es de 30 días. Recibirá una notificación de la siguiente ventana de mantenimiento con 72 horas de antelación.

## <a name="planned-maintenance---duration-and-customer-impact"></a>Mantenimiento planeado: duración e impacto en el cliente

Normalmente se espera que un mantenimiento planeado para una región de Azure concreta se ejecute durante 15 horas. La ventana también incluye el tiempo de búfer para ejecutar un plan de reversión si es necesario. Durante el mantenimiento planeado, puede haber reinicios o conmutaciones por error del servidor de bases de datos, lo que podría dar lugar a una breve indisponibilidad de los servidores de bases de datos para los usuarios finales. Los servidores de Azure Database for MariaDB se ejecutan en contenedores, por lo que los reinicios del servidor de bases de datos suelen ser rápidos y normalmente se espera que se completen en 60-120 segundos. El equipo de ingeniería supervisa cuidadosamente el evento de mantenimiento planeado completo, incluidos los reinicios del servidor. El tiempo de conmutación por error del servidor depende del tiempo de recuperación de la base de datos, lo que puede hacer que la base de datos se ponga en línea más tiempo si tiene mucha actividad transaccional en el servidor en el momento de la conmutación por error. Para evitar un tiempo de reinicio más largo, se recomienda evitar las transacciones de larga duración (cargas masivas) durante los eventos de mantenimiento planeado.

En resumen, mientras que el evento de mantenimiento planeado se ejecuta durante 15 horas, el impacto individual del servidor suele durar 60 segundos, en función de la actividad transaccional en el servidor. Se envía una notificación 72 horas naturales antes de que se inicie el mantenimiento planeado y otra mientras está en curso en una región concreta.

## <a name="how-can-i-get-notified-of-planned-maintenance"></a>¿Cómo se reciben las notificaciones del mantenimiento planeado?

Puede usar la característica de notificaciones de mantenimiento planeado para recibir alertas de un evento de mantenimiento planeado futuro. Recibirá la notificación sobre el próximo mantenimiento 72 horas naturales antes del evento y otra mientras está en curso en una región concreta.

### <a name="planned-maintenance-notification"></a>Notificación de mantenimiento planeado

> [!IMPORTANT]
> Actualmente, las notificaciones de mantenimiento planeado están disponibles en versión preliminar para todas las regiones **menos** Centro-oeste de EE. UU.

Las **notificaciones de mantenimiento planeado** le permiten recibir alertas de eventos de mantenimiento planeado futuros para la instancia de Azure Database for MariaDB. Estas notificaciones se integran con el mantenimiento planeado de [Service Health](../service-health/overview.md) y le permiten ver todo el mantenimiento programado para sus suscripciones en un mismo lugar. También ayuda a escalar la notificación a las audiencias adecuadas de distintos grupos de recursos, ya que puede tener distintos contactos responsables para los distintos recursos. Recibirá la notificación sobre el próximo mantenimiento 72 horas naturales antes del evento.

Intentaremos por todos los medios proporcionar la **notificación de mantenimiento planeado** con una antelación de 72 horas para todos los eventos. Sin embargo, en los casos de revisiones críticas o de seguridad, es posible que las notificaciones se envíen más cerca del evento o se omitan.

Puede comprobar la notificación de mantenimiento planeado en Azure Portal, o bien configurar alertas para recibir las notificaciones. 

### <a name="check-planned-maintenance-notification-from-azure-portal"></a>Comprobación de la notificación de mantenimiento planeado desde Azure Portal

1. En [Azure Portal](https://portal.azure.com), seleccione **Mantenimiento del servicio**.
2. Seleccione la pestaña **Mantenimiento planeado**.
3. Seleccione los valores de **Suscripción**, **Región y **Servicio** para los que quiera comprobar la notificación de mantenimiento planeado. 
   
### <a name="to-receive-planned-maintenance-notification"></a>Para recibir una notificación de mantenimiento planeado

1. En el [portal](https://portal.azure.com), seleccione **Estado del servicio**.
2. En la sección **Alertas**, seleccione **Alertas de estado**.
3. Seleccione **+ Añadir alerta de Service Health** y rellene los campos.
4. Rellene los campos obligatorios. 
5. Elija la opción de **Tipo de evento** y elija **Mantenimiento planeado** o **Seleccionar todo**.
6. En **Grupos de acciones** defina cómo quiere recibir la alerta (obtener un correo electrónico, desencadenar una aplicación lógica, etc.).  
7. Asegúrese de que Habilitar regla tras la creación esté establecido en Sí.
8. Seleccione **Crear regla de alertas** para completar la alerta.

Para conocer los pasos detallados sobre cómo crear **alertas de Service Health**, consulte [Creación de alertas del registro de actividad en notificaciones del servicio](../service-health/alerts-activity-log-service-notifications-portal.md).

## <a name="can-i-cancel-or-postpone-planned-maintenance"></a>¿Se puede cancelar o posponer el mantenimiento planeado?

Se necesita mantenimiento para mantener el servidor seguro, estable y actualizado. El evento de mantenimiento planeado no se puede cancelar ni posponer. Una vez que la notificación se ha enviado a una región de Azure concreta, no se pueden realizar cambios en la programación de revisión de los servidores individuales de esa región. La revisión se implementa para toda la región a la vez. El servicio Azure Database for MariaDB está diseñado para las aplicaciones nativas en la nube que no necesitan un control granular ni la personalización del servicio.

## <a name="are-all-the-azure-regions-patched-at-the-same-time"></a>¿Se revisan todas las regiones de Azure al mismo tiempo?

No, todas las regiones de Azure se revisan durante los intervalos de la ventana de implementación. La ventana de implementación generalmente se extiende desde las 17:00 a las 8:00 (hora local) del día siguiente, en una región concreta de Azure. Las regiones de Azure con emparejamiento geográfico se revisan en días diferentes. Para lograr la alta disponibilidad y la continuidad empresarial de los servidores de bases de datos, se recomienda aprovechar las [réplicas de lectura entre regiones](./concepts-read-replicas.md#cross-region-replication).

## <a name="retry-logic"></a>Lógica de reintento

Un error transitorio es un error que se solucionará automáticamente. Durante el mantenimiento se pueden producir [errores transitorios](./concepts-connectivity.md#transient-errors). El sistema mitiga automáticamente la mayoría de estos eventos en menos de 60 segundos. Los errores transitorios se deben controlar mediante [lógica de reintento](./concepts-connectivity.md#handling-transient-errors).


## <a name="next-steps"></a>Pasos siguientes

- Para cualquier pregunta o sugerencia que pueda tener con respecto al uso de Azure Database for MariaDB, envíe un correo electrónico al equipo de Azure Database for MariaDB en AskAzureDBforMariaDB@service.microsoft.com.
- Consulte [How to set up alerts](howto-alert-metric.md) (Configuración de alertas) para obtener instrucciones sobre cómo crear una alerta en una métrica.
- [Solución de problemas de conexión a Azure Database for MariaDB](howto-troubleshoot-common-connection-issues.md)
- [Control de los errores transitorios y conexión eficaz a Azure Database for MariaDB](concepts-connectivity.md)