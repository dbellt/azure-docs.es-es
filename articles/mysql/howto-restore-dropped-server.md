---
title: Restauración de un servidor de Azure Database for MySQL eliminado
description: En este artículo se describe cómo restaurar un servidor que se ha eliminado en Azure Database for MySQL mediante Azure Portal.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/09/2020
ms.openlocfilehash: 98036b1a12aafde21a318ad8f84f3eba08d3e0a6
ms.sourcegitcommit: 12f15775e64e7a10a5daebcc52154370f3e6fa0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108001527"
---
# <a name="restore-a-deleted-azure-database-for-mysql-server"></a>Restauración de un servidor de Azure Database for MySQL eliminado

Cuando se elimina un servidor, la copia de seguridad del servidor de bases de datos se puede conservar hasta cinco días en el servicio. Solo se puede acceder a la copia de seguridad de la base de datos y restaurarla desde la suscripción de Azure en la que residía originalmente el servidor. Se pueden seguir los siguientes pasos recomendados para recuperar un recurso del servidor MySQL eliminado en un plazo de cinco días desde el momento de la eliminación del servidor. Los pasos recomendados solo funcionarán si la copia de seguridad del servidor todavía está disponible y no se ha eliminado del sistema. 

## <a name="pre-requisites"></a>Requisitos previos
Para restaurar un servidor de Azure Database for MySQL que se ha eliminado, necesitará lo siguiente:
- El nombre de la suscripción de Azure que hospeda el servidor original.
- La ubicación en la que se creó el servidor.

## <a name="steps-to-restore"></a>Pasos para la restauración

1. Vaya al [Registro de actividad](https://ms.portal.azure.com/#blade/Microsoft_Azure_ActivityLog/ActivityLogBlade) en la hoja del monitor de Azure Portal. 

2. En el registro de actividad, haga clic en **Agregar filtro** tal como se muestra, y establezca los siguientes filtros para: 

    - **La suscripción** = la suscripción que hospeda el servidor eliminado.
    - **El tipo de recurso** = servidores de Azure Database for MySQL (Microsoft.DBforMySQL/servers). 
    - **La operación** = eliminar el servidor de MySQL (Microsoft.DBforMySQL/servers/delete). 
 
     [![Registro de actividad filtrado para la operación de eliminación del servidor de MySQL](./media/howto-restore-dropped-server/activity-log.png)](./media/howto-restore-dropped-server/activity-log.png#lightbox)
   
 3. Haga doble clic en el evento Eliminar servidor de MySQL y haga clic en la pestaña JSON; a continuación, anote los atributos "resourceId" y "submissionTimestamp" en la salida JSON. El valor de resourceId tiene el siguiente formato: /subscriptions/ffffffff-ffff-ffff-ffff-ffffffffffff/resourceGroups/TargetResourceGroup/providers/Microsoft.DBforMySQL/servers/deletedserver.
 
 4. Vaya a la [página de la API de REST del servidor](/rest/api/mysql/flexibleserver(preview)/servers/create) y haga clic en la pestaña "Pruébelo" que se encuentra resaltada en verde; a continuación, inicie sesión con su cuenta de Azure.
 
 5. Proporcione los valores de resourceGroupName, serverName (nombre del servidor eliminado) y subscriptionId (derivado del atributo resourceId capturado en el paso 3), mientras que la versión de la API se rellena previamente tal como se muestra en la imagen.
 
     [![Creación de un servidor mediante la API de REST](./media/howto-restore-dropped-server/create-server-from-rest-api.png)](./media/howto-restore-dropped-server/create-server-from-rest-api.png#lightbox)
  
 6. Desplácese hacia abajo a la sección de cuerpo de la solicitud y pegue lo siguiente:
 
    ```json
    {
        "location": "Dropped Server Location",  
        "properties": 
            {
                "restorePointInTime": "submissionTimestamp - 15 minutes",
                "createMode": "PointInTimeRestore",
                "sourceServerId": "resourceId"
            }
    }
    ```
7. Reemplace los siguientes valores en el cuerpo de la solicitud anterior:
   * "Dropped server Location" por la región de Azure donde se creó originalmente el servidor eliminado
   * "submissionTimestamp" y "resourceId" por los valores capturados en el paso 3. 
   * En el caso de "restorePointInTime", especifique un valor de "submissionTimestamp" menos **15 minutos** para asegurarse de que el comando no tiene errores.
   
8. Si ve el código de respuesta 201 o 202, quiere decir que la solicitud de restauración se envió correctamente. 

9. La creación del servidor puede llevar tiempo según el tamaño de la base de datos y los recursos de proceso aprovisionados en el servidor original. El estado de la restauración se puede supervisar desde el registro de actividad mediante el filtrado de 
   - **La suscripción** = su suscripción.
   - **El tipo de recurso** = servidores de Azure Database for MySQL (Microsoft.DBforMySQL/servers). 
   - **La operación** = actualización de la creación del servidor de MySQL.

## <a name="next-steps"></a>Pasos siguientes
- Si está intentando restaurar un servidor en un plazo de cinco días y sigue recibiendo un error después de seguir los pasos descritos anteriormente, abra un incidente de soporte técnico para obtener ayuda. Si intenta restaurar un servidor eliminado pasados cinco días, se devolverá un error porque no se encontrará el archivo de copia de seguridad. No abra una incidencia de soporte técnico en este caso. El equipo de soporte técnico no puede proporcionar asistencia si la copia de seguridad se elimina del sistema. 
- Para evitar la eliminación accidental de los servidores, se recomienda encarecidamente usar [Bloqueos de recursos](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/preventing-the-disaster-of-accidental-deletion-for-your-mysql/ba-p/825222).
