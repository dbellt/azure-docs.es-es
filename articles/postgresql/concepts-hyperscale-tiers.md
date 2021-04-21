---
title: 'Versión preliminar del nivel básico: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: 'Nivel básico de un solo nodo para Azure Database for PostgreSQL : Hiperescala (Citus)'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: c8cad5eb0983715a7cc7c18249243e93a2c498d7
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023926"
---
# <a name="basic-tier-preview"></a>Nivel básico (versión preliminar)

> [!IMPORTANT]
> El nivel básico de Hiperescala (Citus) está actualmente en versión preliminar.  Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar para Hiperescala (Citus)](hyperscale-preview-features.md).

El nivel básico de Azure Database for PostgreSQL: Hiperescala (Citus) es una manera sencilla de crear un grupo de servidores pequeño que se pueda escalar más adelante. Aunque los grupos de servidores del nivel estándar tienen un nodo de coordinación y al menos dos nodos de trabajo, el nivel básico ejecuta todo en un único nodo de base de datos.

Aparte de usar menos nodos, el nivel básico tiene todas las características del nivel estándar. Al igual que el nivel estándar, admite alta disponibilidad, réplicas de lectura y almacenamiento de tablas en columnas, entre otras características.

## <a name="choosing-basic-vs-standard-tier"></a>Elección del nivel básico frente al estándar

El nivel básico puede ser una opción de implementación económica y cómoda para el desarrollo inicial, las pruebas y la integración continua. Usa un único nodo de base de datos y presenta la misma API de SQL que el nivel estándar. Puede probar aplicaciones con el nivel básico y, posteriormente, [cambiar al nivel estándar](howto-hyperscale-scale-grow.md#add-worker-nodes) con la confianza de que la interfaz sigue siendo la misma.

El nivel básico también es adecuado para cargas de trabajo más pequeñas en producción (una vez que emerge de la versión preliminar a la disponibilidad general). Hay espacio para escalar verticalmente *dentro* del nivel básico aumentando el número de núcleos virtuales de servidor.

Cuando se requiera una mayor escala de inmediato, use el nivel estándar. Su grupo de servidores permitidos más pequeño tiene un nodo de coordinación y dos trabajadores. Puede optar por usar más nodos en función de su caso de uso, como se describe en el proceso de [tamaño inicial](howto-hyperscale-scale-initial.md).

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [aprovisionar el nivel básico](quickstart-create-hyperscale-basic-tier.md).
* Cuando esté listo, consulte cómo [cambiar](howto-hyperscale-scale-grow.md#add-worker-nodes) del nivel básico al nivel estándar.
* La opción de [almacenamiento en columnas](concepts-hyperscale-columnar.md) está disponible en el nivel básico y estándar.
