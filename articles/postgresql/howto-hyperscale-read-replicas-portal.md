---
title: 'Administración de réplicas de lectura mediante Azure Portal en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: 'Obtenga información sobre cómo administrar réplicas de lectura mediante Azure Portal para Azure Database for PostgreSQL: Hiperescala (Citus).'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 04/07/2021
ms.openlocfilehash: 7bf5cc43ca173aa787c1cde4d5727b48206872af
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315514"
---
# <a name="create-and-manage-read-replicas-in-azure-database-for-postgresql---hyperscale-citus-from-the-azure-portal"></a>Creación y administración de réplicas de lectura mediante Azure Portal para Azure Database for PostgreSQL: Hiperescala (Citus)

> [!IMPORTANT]
> Las réplicas de lectura en Hiperescala (Citus) están actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar para Hiperescala (Citus)](hyperscale-preview-features.md).

En este artículo, obtendrá información sobre cómo crear y administrar las réplicas de lectura en Hiperescala (Citus) mediante Azure Portal. Para más información acerca de las réplicas de lectura, consulte la [introducción](concepts-hyperscale-read-replicas.md).


## <a name="prerequisites"></a>Requisitos previos

Un [grupo de servidores de Hiperescala (Citus)](quickstart-create-hyperscale-portal.md) debe ser el principal.

## <a name="create-a-read-replica"></a>Creación de una réplica de lectura

Para crear una réplica de lectura, siga estos pasos:

1. Seleccione un grupo de servidores de Azure Database for PostgreSQL existente para utilizar como servidor principal. 

2. En la barra lateral del grupo de servidores, en **Administración del grupo de servidores**, seleccione **Replicación**.

3. Seleccione **Agregar réplica**.

4. Escriba un nombre para la réplica de lectura. 

5. Seleccione **Aceptar** para confirmar la creación de la réplica.

Después de crear la réplica de lectura, puede verla en la ventana **Replicación**.

> [!IMPORTANT]
>
> Revise la [sección sobre las consideraciones de la información general de Réplicas de lectura](concepts-hyperscale-read-replicas.md#considerations).
>
> Antes de actualizar la configuración de un grupo de servidores principal a un nuevo valor, actualice la configuración de réplica a un valor igual o superior. Esta acción ayuda a que la réplica haga frente a los cambios realizados en el servidor maestro.

## <a name="delete-a-primary-server-group"></a>Eliminación de un grupo de servidores principal

Para eliminar un grupo de servidores principal, se usan los mismos pasos que para eliminar un grupo de servidores de Hiperescala (Citus) independiente. 

> [!IMPORTANT]
>
> Cuando se elimina un grupo de servidores principal, la replicación se detiene en todas las réplicas de lectura. Las réplicas de lectura se convierten en grupos de servidores independientes que ahora admiten tanto lectura como escritura.

Para eliminar un grupo de servidores de Azure Portal, siga estos pasos:

1. En Azure Portal, seleccione el grupo de servidores principal de Azure Database for PostgreSQL.

2. Abra la página **Introducción** del grupo de servidores. Seleccione **Eliminar**.
 
3. Escriba el nombre del grupo de servidores principal que desea eliminar. Seleccione **Eliminar** para confirmar la eliminación del grupo de servidores principal.
 

## <a name="delete-a-replica"></a>Eliminación de una réplica

Puede eliminar una réplica de lectura de forma similar a cómo se elimina un grupo de servidores principal.

- En Azure Portal, abra la página **Introducción** para la réplica de lectura. Seleccione **Eliminar**.
 
También puede eliminar la réplica de lectura desde la ventana **Replicación** siguiendo estos pasos:

1. En Azure Portal, seleccione el grupo de servidores principal de Hiperescala (Citus).

2. En el menú del grupo de servidores, en **Administración del grupo de servidores**, seleccione **Replicación**.

3. Seleccione la réplica de lectura que desea eliminar.
 
4. Seleccione **Eliminar réplica**.
 
5. Escriba el nombre de la réplica que quiere eliminar. Seleccione **Eliminar** para confirmar la eliminación de la réplica.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre las [Réplicas de lectura en Azure Database for PostgreSQL: Hiperescala (Citus)](concepts-hyperscale-read-replicas.md).
