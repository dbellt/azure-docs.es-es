---
title: Actualización de un grupo de servidores de Hiperescala (Citus) en Azure Database for PostgreSQL
description: En este artículo se describe cómo actualizar PostgreSQL y Citus en Azure Database for PostgreSQL - Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 4/5/2021
ms.openlocfilehash: 161204bf02ac36c1f5a3969cf57c61e98560c9b5
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107518905"
---
# <a name="upgrade-hyperscale-citus-server-group"></a>Actualización de un grupo de servidores de Hiperescala (Citus)

En estas instrucciones se describe cómo actualizar a una nueva versión principal de PostgreSQL en todos los nodos del grupo de servidores.

## <a name="test-the-upgrade-first"></a>Prueba de la actualización

La actualización de PostgreSQL provoca más cambios de los que podría imaginar, ya que Hiperescala (Citus) también actualizará las [extensiones de base de datos](concepts-hyperscale-extensions.md), incluida la extensión de Citus.
Se recomienda encarecidamente probar la aplicación con la nueva versión de PostgreSQL y Citus antes de actualizar el entorno de producción.

Una manera cómoda de probar es realizar una copia del grupo de servidores mediante la [restauración a un momento dado](concepts-hyperscale-backup.md#restore). Actualice la copia y pruebe la aplicación con ella. Una vez que haya comprobado que todo funciona correctamente, actualice el grupo de servidores de origen.

## <a name="upgrade-a-server-group-in-the-azure-portal"></a>Actualización de un grupo de servidores en Azure Portal

1. En la sección **Información general** de un grupo de servidores de Hiperescala (Citus), seleccione el botón **Actualizar**.
1. Aparecerá un cuadro de diálogo que muestra la versión actual de PostgreSQL y de Citus.
   Elija una nueva versión de PostgreSQL en la lista **Upgrade to** (Actualizar a).
1. Compruebe que el valor de **Versión de Citus después de la actualización** es el esperado.
   Este valor cambia en función de la versión de PostgreSQL seleccionada.
1. Seleccione el botón **Actualizar** para continuar.

## <a name="next-steps"></a>Pasos siguientes

* Conozca las [versiones de PostgreSQL admitidas](concepts-hyperscale-versions.md).
* Consulte [qué extensiones](concepts-hyperscale-extensions.md) se empaquetan con cada versión de PostgreSQL en los grupos de servidores de Hiperescala (Citus).
