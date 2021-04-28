---
title: 'Mantenimiento programado de Azure Database for MySQL: servidor flexible'
description: 'En este artículo se explica la característica de mantenimiento programado de Azure Database for MySQL: servidor flexible.'
author: niklarin
ms.author: nlarin
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 424402db1933c0a20ddd25a6e5af11d84d0775a8
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107481164"
---
# <a name="scheduled-maintenance-in-azure-database-for-mysql--flexible-server"></a>Mantenimiento programado de Azure Database for MySQL: servidor flexible

Azure Database for MySQL: servidor flexible realiza un mantenimiento periódico para conservar la base de datos administrada protegida, estable y actualizada. Durante el mantenimiento, el servidor obtiene nuevas características, actualizaciones y revisiones.

> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está en versión preliminar.

## <a name="select-a-maintenance-window"></a>Selección de una ventana de mantenimiento

Puede programar el mantenimiento durante un día concreto de la semana y una ventana de tiempo dentro de ese día. También puede dejar que el sistema elija un día y una hora de esa ventana de tiempo automáticamente. En cualquier caso, el sistema avisa cinco días antes de ejecutar cualquier mantenimiento. El sistema también comunica cuándo se ha iniciado el mantenimiento y cuándo ha finalizado correctamente.

Las notificaciones sobre el próximo mantenimiento programado se pueden:

* Enviar por correo electrónico a una dirección concreta
* Enviar por correo electrónico a un rol de Azure Resource Manager
* Enviar en un mensaje de texto (SMS) a dispositivos móviles
* Insertar como una notificación en una aplicación de Azure
* Entregar como un mensaje de voz

A la hora de especificar las preferencias de la programación de mantenimiento, puede elegir un día de la semana y una ventana de tiempo. Si no lo hace, el sistema elige horas entre las 23:00 y las 7:00 de la hora de la región del servidor. Puede definir diferentes programaciones para cada servidor flexible de la suscripción de Azure.

> [!IMPORTANT]
> Normalmente, hay al menos 30 días entre los eventos de mantenimiento programado correctos de un servidor.
>
> Sin embargo, en el caso de una actualización de emergencia crítica, como una vulnerabilidad grave, la ventana de notificación podría ser inferior a cinco días. La actualización crítica se puede aplicar al servidor aunque se haya realizado un mantenimiento programado correcto durante los últimos 30 días.

Se puede actualizar la configuración de programación en cualquier momento. Si hay un mantenimiento programado para el servidor flexible y se actualizan las preferencias de programación, el lanzamiento actual continuará según lo programado, y el cambio de configuración de programación se aplicará cuando finalice correctamente para el siguiente mantenimiento programado.

Puede definir una programación administrada por el sistema o una programación personalizada para cada servidor flexible de la suscripción a Azure.  
* Con la programación personalizada, puede elegir el día de la semana y una ventana de una hora para la ventana de mantenimiento del servidor.  
* Con la programación administrada por el sistema, el sistema elegirá cualquier ventana de una hora entre las 23:00 y las 7:00 según la hora de la región del servidor.  

Como parte del lanzamiento de los cambios, primero aplicamos las actualizaciones a los servidores configurados con programaciones administradas por el sistema y, después, las aplicaciones a los servidores con programaciones personalizadas después de un intervalo mínimo de siete días dentro de una región determinada. Si planea recibir actualizaciones anticipadas en la línea de servidores de los entornos de desarrollo y pruebas, se recomienda configurar una programación administrada por el sistema para los servidores que se usan en los entornos de desarrollo y pruebas. De este modo, podrá recibir primero la actualización más reciente en el entorno de desarrollo/pruebas para las pruebas y evaluaciones de validación. Si se produce algún comportamiento o cambios importantes, tendrá tiempo para solucionarlos antes de que la misma actualización se implemente en los servidores de producción con una programación administrada personalizada. La actualización comienza a implementarse en los servidores flexibles de programación personalizada después de siete días y se aplica al servidor en la ventana de mantenimiento definida. En este momento, no hay ninguna opción para aplazar la actualización una vez enviada la notificación. La programación personalizada se recomienda solo para los entornos de producción. 

En raras ocasiones, el sistema puede cancelar el evento de mantenimiento, o bien este puede no completarse correctamente. Si se produce un error en la actualización, esta se revertirá y se restaurará la versión anterior de los archivos binarios. En estos escenarios de actualización con errores, es posible que el servidor se reinicie durante la ventana de mantenimiento. Si se produce un error en la actualización o se cancela, el sistema creará una notificación sobre el evento de mantenimiento con errores o cancelado, respectivamente, para notificárselo. El siguiente intento de mantenimiento se programará según la configuración de programación actual, y el usuario recibirá una notificación al respecto con cinco días de antelación. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [cambiar la programación de mantenimiento](how-to-maintenance-portal.md).
* Obtenga información sobre cómo [obtener notificaciones sobre próximos eventos de mantenimiento](../../service-health/service-notifications.md) con Azure Service Health.
* Obtenga información sobre cómo [configurar alertas sobre próximos eventos de mantenimiento programado](../../service-health/resource-health-alert-monitor-guide.md).
