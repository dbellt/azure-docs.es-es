---
title: Restauración de un servidor descartado de Azure Database for PostgreSQL
description: En este artículo se explica cómo restaurar un servidor descartado de Azure Database for PostgreSQL mediante Azure Portal.
author: Bashar-MSFT
ms.author: bahusse
ms.service: postgresql
ms.topic: how-to
ms.date: 11/03/2020
ms.openlocfilehash: 5b5bb9fd6e3d34fc4a6b0ae90a2cd76fc84e9ce1
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107366528"
---
# <a name="restore-a-dropped-azure-database-for-postgresql-server"></a>Restauración de un servidor descartado de Azure Database for PostgreSQL

Cuando se descarta un servidor, la copia de seguridad del servidor de bases de datos se puede guardar hasta cinco días en el servicio. Solo se puede acceder a la copia de seguridad de la base de datos y restaurarla desde la suscripción de Azure en la que residía originalmente el servidor. Se pueden seguir los siguientes pasos recomendados para recuperar un recurso de servidor PostgreSQL descartado en un plazo de 5 días desde el momento de la eliminación de este servidor. Los pasos recomendados solo funcionarán si la copia de seguridad del servidor todavía está disponible y no se ha eliminado del sistema. 

## <a name="pre-requisites"></a>Requisitos previos
Para restaurar un servidor descartado de Azure Database for PostgreSQL, necesita lo siguiente:
- El nombre de la suscripción de Azure que hospeda el servidor original.
- La ubicación en la que se creó el servidor.

## <a name="steps-to-restore"></a>Pasos para la restauración

1. Vaya a [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_ActivityLog/ActivityLogBlade). Seleccione el servicio **Azure Monitor** y luego **Registro de actividad**.

2. En Registro de actividad, haga clic en **Agregar filtro** como se muestra y establezca filtros para lo siguiente

    - **La suscripción** = la suscripción que hospeda el servidor eliminado.
    - **Tipo de recurso**: servidores de Azure Database for PostgreSQL (Microsoft.DBforPostgreSQL/servers)
    - **Operación**: elimine el servidor PostgreSQL (Microsoft.DBforPostgreSQL/servers/delete)
 
    ![Registro de actividad filtrado para la operación de eliminación del servidor PostgreSQL](./media/howto-restore-dropped-server/activity-log-azure.png)

3. Seleccione el evento **Eliminar servidor PostgreSQL** y luego la pestaña **JSON**. Copie los atributos `resourceId` y `submissionTimestamp` en la salida JSON. ResourceId tiene el formato siguiente: `/subscriptions/ffffffff-ffff-ffff-ffff-ffffffffffff/resourceGroups/TargetResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/deletedserver`.


 4. Vaya a la [página de creación de la API de REST del servidor PostgreSQL](/rest/api/PostgreSQL/servers/create) y seleccione la pestaña **Probar** resaltada en verde. Inicie sesión con su cuenta de Azure.

 5. Especifique las propiedades **resourceGroupName**, **serverName** (nombre del servidor eliminado) y **subscriptionId** en función del valor JSON del atributo resourceId capturado en el paso 3 anterior. La propiedad api-version está rellenada previamente y se puede dejar tal cual, como se muestra en la siguiente imagen.

    ![Creación de un servidor mediante la API de REST](./media/howto-restore-dropped-server/create-server-from-rest-api-azure.png)
  
 6. Desplácese por la sección “Cuerpo de la solicitud” y pegue lo siguiente, sustituyendo “Ubicación del servidor descartado” (p. ej., “CentralUS, EastUS, etc.), “submissionTimestamp” y “resourceId”. En el caso de "restorePointInTime", especifique un valor de "submissionTimestamp" menos **15 minutos** para asegurarse de que el comando no tiene errores.
    
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

    Por ejemplo, si la hora actual es 2020-11-02T23:59:59.0000000Z, se recomienda una restauración a un momento dado de un mínimo de 15 minutos antes, 2020-11-02T23:44:59.0000000Z.

    > [!Important]
    > Hay un límite de tiempo de cinco días después de la eliminación del servidor. Transcurrido este tiempo, es previsible un error, ya que no es posible encontrar el archivo de copia de seguridad.
    
7. Si ve el código de respuesta 201 o 202, quiere decir que la solicitud de restauración se envió correctamente. 

    La creación del servidor puede llevar tiempo según el tamaño de la base de datos y los recursos de proceso aprovisionados en el servidor original. El estado de la restauración se puede supervisar desde el registro de actividad mediante el filtrado de 
   - **La suscripción** = su suscripción.
   - **Tipo de recurso**: servidores de Azure Database for PostgreSQL (Microsoft.DBforPostgreSQL/servers) 
   - **Operación**: actualización de la creación del servidor PostgreSQL

## <a name="next-steps"></a>Pasos siguientes
- Si está intentando restaurar un servidor en un plazo de cinco días y sigue recibiendo un error después de seguir los pasos descritos anteriormente, abra un incidente de soporte técnico para obtener ayuda. Si intenta restaurar un servidor descartado pasados cinco días, se devolverá un error porque no se encontrará el archivo de copia de seguridad. No abra una incidencia de soporte técnico en este caso. El equipo de soporte técnico no puede proporcionar asistencia si la copia de seguridad se elimina del sistema. 
- Para evitar la eliminación accidental de los servidores, se recomienda encarecidamente usar [Bloqueos de recursos](https://techcommunity.microsoft.com/t5/azure-database-for-PostgreSQL/preventing-the-disaster-of-accidental-deletion-for-your-PostgreSQL/ba-p/825222).
