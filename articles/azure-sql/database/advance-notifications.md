---
title: Notificaciones anticipadas (versión preliminar) para eventos de mantenimiento planeados
description: Reciba una notificación antes del mantenimiento planeado de Azure SQL Database.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service-overview
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: scott-kim-sql
ms.author: scottkim
ms.reviewer: mathoma
ms.date: 03/02/2021
ms.openlocfilehash: 1b79d9f88c61ca4bc11895e26f8ffc4fd69b8a35
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706565"
---
# <a name="advance-notifications-for-planned-maintenance-events-preview"></a>Notificaciones anticipadas para eventos de mantenimiento planeados (versión preliminar)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Notificaciones avanzadas (versión preliminar) está disponible para las bases de datos que se hayan configurado en [Ventana de mantenimiento (versión preliminar)](maintenance-window.md). Notificaciones anticipadas permite a los clientes configurar notificaciones para que se envíen hasta 24 horas antes de cualquier evento planeado.

Las notificaciones se pueden configurar para recibir mensajes de texto, correos electrónicos, notificaciones push de Azure y correos de voz cuando el mantenimiento planeado debe iniciarse en las 24 horas siguientes. Se enviarán notificaciones adicionales al iniciarse y finalizar el mantenimiento.

> [!Note]
> Aunque existe la posibilidad de elegir una ventana de mantenimiento para las instancias administradas de Azure SQL, actualmente no se dispone de notificaciones anticipadas para estas.

## <a name="create-an-advance-notification"></a>Creación de una notificación anticipada

Las notificaciones anticipadas están disponibles para las instancias de Azure SQL Database que tengan configurada su ventana de mantenimiento. 

Para habilitar una notificación, siga estos pasos:  

1. Vaya a la página [Mantenimiento planeado](https://portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/plannedMaintenance), seleccione **Alertas de estado** y luego **Añadir alerta de estado del servicio**.

    :::image type="content" source="media/advance-notifications/health-alerts.png" alt-text="opción de menú para crear una nueva alerta de estado":::

2. En la sección **Acciones**, seleccione **Add action groups** (Agregar grupos de acciones). 

    :::image type="content" source="media/advance-notifications/add-action-group.png" alt-text="opción de menú para agregar un grupo de acciones":::

3. Rellene el formulario **Crear grupo de acciones** y seleccione **Siguiente: Notificaciones**.  

    :::image type="content" source="media/advance-notifications/create-action-group.png" alt-text="formulario para crear grupo de acciones":::

1. En la pestaña **Notificaciones**, seleccione **Tipo de notificación**. La opción **Email/SMS message/Push/Voice** (Correo electrónico/mensaje SMS/notificación push/mensaje de voz) ofrece la máxima flexibilidad y es la opción recomendada. Seleccione el icono de lápiz para configurar la notificación.  

    :::image type="content" source="media/advance-notifications/notifications.png" alt-text="configuración de notificaciones":::



   1. Rellene el formulario *Agregar o Editar notificación* que se abrirá y seleccione **Aceptar**: 

   2. Las acciones y las etiquetas son opcionales. Aquí puede configurar acciones adicionales para que se desencadenen, o bien usar etiquetas para clasificar y organizar los recursos de Azure. 

   4. Compruebe los detalles de la pestaña **Revisar y crear** y seleccione **Crear**. 

7. Después de seleccionar Crear, se abrirá la pantalla de configuración de reglas de alerta y el grupo de acciones aparecerá seleccionado. Asigne un nombre a la nueva regla de alerta, elija el grupo de recursos correspondiente y seleccione **Crear regla de alerta**. 

8. Vuelva a hacer clic en el elemento de menú **Alertas de estado**. La lista de alertas ahora incluirá la nueva alerta. 


Está listo. La próxima vez que haya un evento de mantenimiento de Azure SQL planeado, recibirá una notificación anticipada.

## <a name="receiving-notifications"></a>Recibir notificaciones

En la tabla siguiente se muestran las notificaciones de información general que puede recibir: 

|Status|Descripción|
|:---|:---|
|**Implementación planeada**| Recibida 24 horas antes del evento de mantenimiento. El mantenimiento se planea en fecha, entre las 17:00 y las 8:00 (hora local) para la base de datos xyz.|
|**En curso** | Se inicia el mantenimiento de la base de datos *xyz* .| 
|**Completo** | El mantenimiento de la base de datos *xyz* ha finalizado. |

En la tabla siguiente se muestran notificaciones adicionales que se pueden enviar mientras el mantenimiento está en curso: 

|Status|Descripción|
|:---|:---|
|**Extendido** | El mantenimiento está en curso, pero no ha finalizado para la base de datos *xyz*. El mantenimiento continuará en la siguiente ventana de mantenimiento.| 
|**Canceled**| El mantenimiento se ha cancelado para la base de datos *xyz* y se volverá a programar más adelante. |
|**Bloqueado**|Durante el mantenimiento de la base de datos *xyz* se ha producido un problema. Le avisaremos cuando se reanude la operación.| 
|**Reanudado**|El problema se ha resuelto y el mantenimiento se efectuará en la siguiente ventana de mantenimiento.|


## <a name="next-steps"></a>Pasos siguientes

- [Ventana de mantenimiento](maintenance-window.md)
- [P+F sobre la ventana de mantenimiento](maintenance-window-faq.yml)
- [Introducción a las alertas en Microsoft Azure](../../azure-monitor/alerts/alerts-overview.md)
- [Envío de correo electrónico a un rol de Azure Resource Manager](../../azure-monitor/alerts/action-groups.md#email-azure-resource-manager-role)