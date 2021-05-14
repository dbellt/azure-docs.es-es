---
title: Réplicas de lectura - Azure Database for PostgreSQL - Hiperescala (Citus)
description: En este artículo se describe la característica de réplica de lectura de Azure Database for PostgreSQL - Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 08629ed8ce361f5d7ac4bb4d2e148ae5b20fff02
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315568"
---
# <a name="read-replicas-in-azure-database-for-postgresql---hyperscale-citus"></a>Réplicas de lectura en Azure Database for PostgreSQL - Hiperescala (Citus)

> [!IMPORTANT]
> Las réplicas de lectura en Hiperescala (Citus) están actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar para PostgreSQL - Hiperescala (Citus)](hyperscale-preview-features.md).

Con la característica de réplica de lectura, puede replicar datos de un grupo de servidores de Hiperescala (Citus) a otro de solo lectura. Las réplicas se actualizan **de manera asincrónica** mediante la tecnología de replicación física de PostgreSQL. Puede replicar desde el servidor principal a un número ilimitado de réplicas.

Las réplicas son nuevos grupos de servidores que se administran de forma similar a los grupos de servidores normales de Hiperescala (Citus). En cada réplica de lectura, se le cobra por el proceso aprovisionado en núcleos virtuales y el almacenamiento aprovisionado en GB/mes.

Vea cómo [crear y administrar réplicas](howto-hyperscale-read-replicas-portal.md).

## <a name="when-to-use-a-read-replica"></a>Casos en los que utilizar las réplicas de lectura

La finalidad de la característica de réplica de lectura es ayudar a mejorar el rendimiento y la escala de las cargas de trabajo que conllevan un gran número de operaciones de lectura. Las cargas de trabajo de lectura se pueden aislar en las réplicas, mientras que las cargas de trabajo de escritura se pueden dirigir al servidor principal.

Un caso habitual es que las cargas de trabajo de análisis e inteligencia empresarial utilicen la réplica de lectura como origen de datos para los informes.

Dado que las réplicas son de solo lectura, no reducen directamente las cargas de capacidad de escritura en el servidor principal.

### <a name="considerations"></a>Consideraciones

La característica está pensada para escenarios donde el retraso de replicación es aceptable y para consultas de descarga. No está diseñada para escenarios de replicación sincrónica en los que se espera que los datos de réplica estén actualizados. Habrá un retraso medible entre el servidor principal y la réplica. El retraso puede ser de minutos o incluso horas, según la carga de trabajo y la latencia entre el servidor principal y la réplica.  Los datos de la réplica se vuelven finalmente coherentes con los datos del servidor principal. Use esta característica con cargas de trabajo que puedan admitir este retraso. 

## <a name="create-a-replica"></a>Creación de una réplica

Cuando se inicia el flujo de trabajo de creación de la réplica, se crea un grupo de servidores en blanco de Hiperescala (Citus). Este nuevo grupo de servidores se rellena con los datos que estaban en el grupo principal. El tiempo que se tarda en crear la réplica depende de la cantidad de datos en el servidor principal y del tiempo que ha transcurrido desde la última copia de seguridad completa semanal. Puede oscilar desde unos minutos hasta varias horas.

La característica de réplica de lectura usa la replicación física de PostgreSQL (replicación no lógica). El modo predeterminado es la replicación de streaming mediante ranuras de replicación.
Cuando es necesario, se usa el trasvase de registros para actualizarse.

Aprenda a [crear una réplica de lectura en Azure Portal](howto-hyperscale-read-replicas-portal.md).

## <a name="connect-to-a-replica"></a>Conexión a una réplica

Al crear una réplica, esta no hereda las reglas de firewall del grupo de servidores principal. Estas reglas se deben configurar de forma independiente para la réplica.

La réplica hereda la cuenta de administrador (“citus”) del grupo de servidores principal.
Todas las cuentas de usuario se replican en las réplicas de lectura. Solo se puede conectar a una réplica de lectura a través de las cuentas de usuario disponibles en el servidor principal.

Puede conectarse al nodo de coordinación de la réplica usando su nombre de host y una cuenta de usuario válida, tal y como haría en un grupo de servidores normal de Hiperescala (Citus).
En un servidor denominado **myreplica** con el nombre de usuario administrador **citus**, puede conectarse al nodo de coordinación de la réplica usando psql:

```bash
psql -h c.myreplica.postgres.database.azure.com -U citus@myreplica -d postgres
```

Cuando se le solicite, escriba la contraseña de la cuenta de usuario.

## <a name="considerations"></a>Consideraciones

En esta sección se resumen las consideraciones sobre la característica de réplica de lectura.

### <a name="new-replicas"></a>Nuevas réplicas

Una réplica de lectura se crea como un nuevo grupo de servidores de Hiperescala (Citus). Un grupo de servidores ya existente no puede convertirse en una réplica. No se puede crear una réplica de otra réplica de lectura.

### <a name="replica-configuration"></a>Configuración de réplicas

Una réplica se crea usando la misma configuración de proceso, almacenamiento y nodo de trabajo que el grupo principal. Una vez creada una réplica, se pueden cambiar varios valores de configuración, incluido el período de retención de almacenamiento y copia de seguridad. Otros valores no se pueden cambiar en las réplicas, como el tamaño de almacenamiento y el número de nodos de trabajo.

Recuerde mantener réplicas con la solidez necesaria para soportar los cambios que llegan desde el grupo principal. Por ejemplo, asegúrese de ampliar la potencia de proceso en las réplicas si hace lo propio en el grupo principal.

La réplica no hereda reglas de firewall ni valores de parámetros del servidor principal, ni cuando se crea ni con posterioridad.

### <a name="regions"></a>Regions

Los grupos de servidores de Hiperescala (Citus) solo admiten replicación en la misma región.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [crear y administrar réplicas de lectura en Azure Portal](howto-hyperscale-read-replicas-portal.md).
