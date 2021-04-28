---
title: 'Copia de seguridad y restauración: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: Protección de datos frente a daños o eliminaciones accidentales
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/14/2021
ms.openlocfilehash: 7681e9c28bbbbcec06bcc1cf2bf469f1b4189d79
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107520180"
---
# <a name="backup-and-restore-in-azure-database-for-postgresql---hyperscale-citus"></a>Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)

Azure Database for PostgreSQL: Hiperescala (Citus) crea automáticamente copias de seguridad de cada nodo y las almacena en un almacenamiento con redundancia local. Las copias de seguridad se pueden usar para restaurar el grupo de servidores de Hiperescala (Citus) a una hora especificada.
Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.

## <a name="backups"></a>Copias de seguridad

Al menos una vez al día, Azure Database for PostgreSQL realiza copias de seguridad de instantáneas de los archivos de datos y del registro de transacciones de la base de datos. Las copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención. (El período de retención es actualmente de 35 días para todos los grupos de servidores). Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

En las regiones de Azure que admiten zonas de disponibilidad, las instantáneas de copia de seguridad se almacenan en tres zonas de disponibilidad. Siempre que haya al menos una zona de disponibilidad en línea, se puede restaurar el grupo de servidores de Hiperescala (Citus).

Los archivos de copia de seguridad no se pueden exportar. Solo se pueden usar para operaciones de restauración en Azure Database for PostgreSQL.

### <a name="backup-storage-cost"></a>Costo del almacenamiento de copia de seguridad

Para obtener información sobre los precios del almacenamiento de copia de seguridad actual, consulte la [página de precios](https://azure.microsoft.com/pricing/details/postgresql/hyperscale-citus/) de Azure Database for PostgreSQL: Hiperescala (Citus).

## <a name="restore"></a>Restauración

Puede restaurar un grupo de servidores de Hiperescala (Citus) a un momento dado de cualquiera de los últimos 35 días.  La restauración a un momento dado es útil en diversos escenarios. Por ejemplo, cuando un usuario elimina accidentalmente los datos, elimina una tabla importante o la base de datos, o si una aplicación sobrescribe accidentalmente los datos correctos con datos incorrectos.

> [!IMPORTANT]
> Los grupos de servidores eliminados de Hiperescala (Citus) no se pueden restaurar. Si elimina el grupo de servidores, todos los nodos que pertenecen a este se eliminan y no se pueden recuperar. Para proteger los recursos del grupo de servidores, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los [bloqueos de administración](../azure-resource-manager/management/lock-resources.md).

El proceso de restauración crea un nuevo grupo de servidores en la misma región, suscripción y grupo de recursos de Azure que el original. El grupo de servidores tiene la misma configuración del original: el mismo número de nodos, de núcleos virtuales, tamaño de almacenamiento, roles de usuario, versión de PostgreSQL y versión de la extensión de Citus.

La configuración del firewall y los parámetros del servidor de PostgreSQL no se conservan del grupo de servidores original sino que se restablecen a los valores predeterminados. El firewall impedirá todas las conexiones. Tendrá que ajustar manualmente estos valores después de la restauración. En general, puede consultar nuestra lista de [tareas posteriores a la restauración](howto-hyperscale-restore-portal.md#post-restore-tasks) sugeridas.

## <a name="next-steps"></a>Pasos siguientes

* Consulte los pasos para [restaurar un grupo de servidores](howto-hyperscale-restore-portal.md) en Azure Portal.
* Obtenga más información acerca de las  [zonas de disponibilidad de Azure](../availability-zones/az-overview.md).
