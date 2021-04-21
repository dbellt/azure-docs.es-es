---
title: 'Tamaño inicial de un grupo de servidores de Hiperescala (Citus): Azure Database for PostgreSQL'
description: Elija el tamaño inicial adecuado para su caso de uso.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 04/07/2021
ms.openlocfilehash: 8b87cfc8276d13ccc7e12a4901489ea0b1e770a5
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107012513"
---
# <a name="pick-initial-size-for-hyperscale-citus-server-group"></a>Elección del tamaño inicial para un grupo de servidores de Hiperescala (Citus)

El tamaño de un grupo de servidores, tanto en su número de nodos como en su capacidad de hardware, es [fácil de cambiar](howto-hyperscale-scale-grow.md). Sin embargo, sigue siendo necesario elegir un tamaño inicial para un nuevo grupo de servidores. Estas son algunas sugerencias para una opción razonable.

## <a name="use-cases"></a>Casos de uso

Hiperescala (Citus) se usa con frecuencia de las maneras siguientes.

### <a name="multi-tenant-saas"></a>Aplicaciones SaaS multiinquilino

Al migrar a Hiperescala (Citus) desde una instancia existente de base de datos de PostgreSQL de un solo nodo, elija un clúster en el que el número de núcleos virtuales de trabajo y la RAM total sean iguales a los de la instancia original. En estos escenarios se han detectado mejoras de rendimiento de dos a tres veces mayores porque el particionamiento mejora el uso de recursos, lo que permite, por ejemplo, índices más pequeños.

El recuento de núcleo virtual es realmente la única decisión. La asignación de RAM se determina actualmente según el recuento de núcleos virtuales, como se describe en la página [Opciones de configuración de Hiperescala (Citus)](concepts-hyperscale-configuration-options.md).
El nodo de coordinación no requiere tanta RAM como los trabajo, pero no hay ninguna manera de elegir RAM y núcleos virtuales de manera independiente.

### <a name="real-time-analytics"></a>Análisis en tiempo real

Total de núcleos virtuales: cuando los datos de trabajo se ajustan a la RAM, se puede esperar una mejora de rendimiento lineal en Hiperescala (Citus) proporcional al número de núcleos de trabajo. Para determinar el número adecuado de núcleos virtuales para sus necesidades, tenga en cuenta la latencia actual de las consultas en la base de datos de un solo nodo y la latencia necesaria en Hiperescala (Citus). Divida la latencia actual según la latencia deseada y redondee el resultado.

RAM del trabajo: el mejor caso sería proporcionar suficiente memoria para que la mayor parte del espacio de trabajo se ajuste a la memoria. El tipo de consultas que utiliza la aplicación afecta a los requisitos de memoria. Puede ejecutar EXPLAIN ANALYZE en una consulta para determinar cuánta memoria se requiere. Recuerde que los núcleos virtuales y la RAM se escalan a la vez, tal y como se describe en el artículo sobre [opciones de configuración de Hiperescala (Citus)](concepts-hyperscale-configuration-options.md).

## <a name="choosing-a-hyperscale-citus-tier"></a>Elección de un nivel de Hiperescala (Citus)

> [!IMPORTANT]
> El nivel básico de Hiperescala (Citus) está actualmente en versión preliminar.  Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar para Hiperescala (Citus)](hyperscale-preview-features.md).

En las secciones anteriores se da una idea de cuántos núcleos virtuales y cuánta RAM se necesitan para cada caso de uso. Puede satisfacer estas demandas a través de una opción entre dos niveles de Hiperescala (Citus): el nivel básico y el estándar.

El nivel básico usa un único nodo de base de datos para realizar el procesamiento, mientras que el nivel estándar permite más nodos. De lo contrario, los niveles son idénticos y ofrecen las mismas características. En algunos casos, los núcleos virtuales y el espacio en disco de un solo nodo se pueden escalar para que sean suficientes y, en otros casos, requiere la cooperación de varios nodos.

Para ver una comparación de los niveles, consulte la página de conceptos del [nivel básico](concepts-hyperscale-tiers.md).

## <a name="next-steps"></a>Pasos siguientes

- [Escalado de un grupo de servidores](howto-hyperscale-scale-grow.md)
- Más información sobre el grupo de servidores en [opciones de rendimiento](concepts-hyperscale-configuration-options.md).
