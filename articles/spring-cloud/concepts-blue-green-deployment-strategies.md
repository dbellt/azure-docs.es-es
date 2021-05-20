---
title: Estrategias de implementación azul-verde en Azure Spring Cloud
description: En este tema se explican dos enfoques para las implementaciones azul-verde en Azure Spring Cloud.
author: yevster
ms.author: yebronsh
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 04/02/2021
ms.custom: devx-track-java
ms.openlocfilehash: ca385c2be61e359cc93e40cc42718096606583cf
ms.sourcegitcommit: 14cd60fe58964acd4bccb6053f36e60d5848edc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109795640"
---
# <a name="blue-green-deployment-strategies-in-azure-spring-cloud"></a>Estrategias de implementación azul-verde en Azure Spring Cloud

En este artículo se describe la compatibilidad con la implementación azul-verde en Azure Spring Cloud.

Azure Spring Cloud (nivel Estándar y superior) permite dos implementaciones para cada aplicación, aunque solo una recibe el tráfico de producción. Este patrón se conoce normalmente como implementación azul-verde. La compatibilidad de Azure Spring Cloud con la implementación azul-verde, junto con una canalización de [entrega continua (CD)](/azure/devops/learn/what-is-continuous-delivery) y rigurosas pruebas automatizadas, permite implementaciones de aplicaciones ágiles con alta confianza.

## <a name="alternating-deployments"></a>Implementaciones alternativas

La manera más sencilla de aplicar la implementación azul-verde con Azure Spring Cloud es crear dos implementaciones fijas e implementar siempre en la que no recibe el tráfico de producción. Con la tarea de Azure Spring Cloud para Azure Pipelines, puede implementarlo de esta manera con el establecimiento de la marca `UseStagingDeployment` en `true`.

Este es el funcionamiento del enfoque de implementaciones alternativas en la práctica:

Supongamos que la aplicación tiene dos implementaciones: `deployment1` y `deployment2`. Actualmente, `deployment1` se establece como implementación de producción y se ejecuta la versión `v3` de la aplicación.

Esto hace que `deployment2` sea la implementación de ensayo. Por lo tanto, cuando la canalización de entrega continua (CD) está lista para ejecutarse, implementa la siguiente versión de la aplicación, versión `v4`, en la implementación de ensayo `deployment2`.

![Dos implementaciones: deployment1 recibe el tráfico de producción.](media/spring-cloud-blue-green-patterns/alternating-deployments-1.png)

Cuando `v4` se haya iniciado en `deployment2`, puede ejecutar pruebas automatizadas y manuales en él a través de un punto de conexión de prueba privado para asegurarse de que `v4` cumple todas las expectativas.

![V4 ahora se implementa en deployment2 y se somete a pruebas.](media/spring-cloud-blue-green-patterns/alternating-deployments-2.png)

Cuando tenga confianza en `v4`, puede establecer `deployment2` como la implementación de producción para que reciba todo el tráfico de producción. `v3` seguirá ejecutándose en `deployment1` en caso de que se detecte un problema crítico que requiera reversión.

![V4 en deployment2 ahora recibe el tráfico de producción.](media/spring-cloud-blue-green-patterns/alternating-deployments-3.png)

Ahora, `deployment1` es la implementación de ensayo. Por lo tanto, la siguiente ejecución de la canalización de implementación se realiza en `deployment1`.

![V5 se implementa en deployment1.](media/spring-cloud-blue-green-patterns/alternating-deployments-4.png)

Ahora puede probar `V5` en el punto de conexión de prueba privado de `deployment1`.

![V5 se ha probado en deployment1.](media/spring-cloud-blue-green-patterns/alternating-deployments-5.png)

Por último, después de que `v5` cumpla todas sus expectativas, establezca `deployment1` como implementación de producción una vez más, para que `v5` reciba todo el tráfico de producción.

![V5 recibe el tráfico en deployment1.](media/spring-cloud-blue-green-patterns/alternating-deployments-6.png)

### <a name="tradeoffs-of-the-alternating-deployments-approach"></a>Ventajas del enfoque de implementaciones alternativas

El enfoque de implementaciones alternativas es sencillo y rápido, ya que no requiere la creación de nuevas implementaciones. Sin embargo, presenta varias desventajas, como se describe en las secciones siguientes.

#### <a name="persistent-staging-deployment"></a>Implementación de ensayo persistente

La implementación de ensayo siempre permanece en ejecución y, por tanto, consume recursos de la instancia de Azure Spring Cloud. Esto duplica eficazmente los requisitos de recursos de cada aplicación en Azure Spring Cloud.

#### <a name="the-approval-race-condition"></a>Condición de carrera de aprobación

Supongamos que en la aplicación anterior, la canalización de versión requiere la aprobación manual antes de que cada nueva versión de la aplicación pueda recibir el tráfico de producción. Esto crea el riesgo de que mientras una versión (`v6`) espera la aprobación manual en la implementación de ensayo, la canalización de implementación se ejecutará de nuevo y se sobrescribirá con una versión más reciente (`v7`). Después, cuando se concede la aprobación de `v6`, la canalización que implementó `v6` establecerá la implementación de ensayo como producción. Pero ahora será la versión `v7` no aprobada, no la versión `v6` aprobada, la que se aplica en esa implementación y recibe el tráfico.

![Condición de carrera de aprobación](media/spring-cloud-blue-green-patterns/alternating-deployments-race-condition.png)

Puede evitar la condición de carrera asegurándose de que el flujo de implementación de una versión no pueda comenzar hasta que el flujo de implementación de todas las versiones anteriores se complete o anule. Otra manera de evitar la condición de carrera de aprobación es usar el enfoque Implementaciones con nombre que se describe a continuación.

## <a name="named-deployments"></a>Implementaciones con nombre

En el enfoque de implementaciones con nombre, se crea una nueva implementación para cada nueva versión de la aplicación que se implementa. Después de probar la aplicación en su implementación personalizada, esa implementación se establece como implementación de producción. Se puede permitir que la implementación que contiene la versión anterior persista el tiempo suficiente para estar seguro de que no se necesita una reversión.

En la ilustración siguiente, la versión `v5` se ejecuta en la implementación `deployment-v5`. El nombre de implementación ahora contiene la versión porque la implementación se creó específicamente para esta versión. No hay ninguna otra implementación al principio. Ahora, para implementar la versión `v6`, la canalización de implementación crea una nueva implementación `deployment-v6` y aplica allí la versión de la aplicación `v6`.

![Implementación de una nueva versión en una implementación con nombre](media/spring-cloud-blue-green-patterns/named-deployment-1.png)

No hay riesgo de que se implemente otra versión en paralelo. En primer lugar, Azure Spring Cloud no permite la creación de una tercera implementación mientras ya existen dos implementaciones. En segundo lugar, aunque fuera posible tener más de dos implementaciones, cada implementación se identifica mediante la versión de la aplicación que contiene. Por lo tanto, la canalización que orquesta la implementación de `v6` solo intentaría establecer `deployment-v6` como implementación de producción.

![La nueva versión recibe la implementación con nombre del tráfico de producción.](media/spring-cloud-blue-green-patterns/named-deployment-2.png)

Después de que la implementación creada para la nueva versión recibe el tráfico de producción, deberá quitar la implementación que contiene la versión anterior para dejar espacio a futuras implementaciones. Es posible que quiera posponer un número de minutos u horas para poder revertir a la versión anterior si detecta un problema crítico en la nueva versión.

![Después de un período de reserva, se elimina la implementación anterior.](media/spring-cloud-blue-green-patterns/named-deployment-3.png)

### <a name="tradeoffs-of-the-named-deployments-approach"></a>Ventajas del enfoque de implementaciones con nombre

El enfoque de implementaciones con nombre tiene las siguientes ventajas:

* Impide la condición de carrera de aprobación.
* Reduce el consumo de recursos mediante la eliminación de la implementación de ensayo cuando no está en uso.

Sin embargo, también hay inconvenientes, como se describe en la sección siguiente.

#### <a name="deployment-pipeline-failures"></a>Errores de canalización de implementación

Entre el momento en que se inicia una implementación y el momento en que se elimina la implementación de ensayo, se producirá un error en los intentos adicionales de ejecutar la canalización de implementación. La canalización intentará crear una nueva implementación, lo que dará lugar a un error porque solo se permiten dos implementaciones por cada aplicación de Azure Spring Cloud.

Por lo tanto, la orquestación de la implementación debe tener los medios para reintentar un proceso de implementación con errores en un momento posterior, o los medios para asegurarse de que los flujos de implementación para cada versión permanecerán en cola hasta que se complete el flujo para todas las versiones anteriores.
