---
title: Mantenimiento y actualizaciones
description: Información general sobre el mantenimiento y las actualizaciones de las máquinas virtuales que se ejecutan en Azure.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/22/2020
ms.author: shants
ms.openlocfilehash: 8b332b1871c92d36821e8f797ceac5b6697e5dc1
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632194"
---
# <a name="maintenance-for-virtual-machines-in-azure"></a>Mantenimiento de máquinas virtuales en Azure

Azure actualiza periódicamente su plataforma para mejorar la confiabilidad, el rendimiento y la seguridad de la infraestructura de host para las máquinas virtuales. El objetivo de estas actualizaciones va desde la aplicación de revisiones a componentes de software en el entorno de hospedaje hasta la actualización de los componentes de red o la retirada de hardware. 

Las actualizaciones raramente afectan a las máquinas virtuales hospedadas. Cuando las actualizaciones tienen algún efecto, Azure elige el método menos agresivo en las actualizaciones:

- Si la actualización no requiere un reinicio, la máquina virtual se pone en pausa mientras se actualiza el host, o bien se migra directamente a un host ya actualizado. 
- Si el mantenimiento requiere un reinicio, se notifica al usuario el mantenimiento planeado. Asimismo, Azure proporciona un período de tiempo en el que puede iniciar el mantenimiento a la hora que le sea más conveniente. El período de mantenimiento de autoservicio es normalmente de 35 días, a menos que el mantenimiento sea urgente. Azure está invirtiendo en tecnologías para reducir los casos en los que el mantenimiento planificado de la plataforma requiere el reinicio de la máquina virtual. Para obtener instrucciones sobre la administración del mantenimiento planeado, vea Control de las notificaciones de mantenimiento planeado mediante la [CLI](maintenance-notifications-cli.md) de Azure, [PowerShell](maintenance-notifications-powershell.md) o el [portal](maintenance-notifications-portal.md).

En esta página se describe cómo Azure realiza ambos tipos de mantenimiento. Para más información sobre eventos no planeados (interrupciones), consulte el artículo sobre la [administración de la disponibilidad de máquinas virtuales para Windows](./availability.md) o el artículo correspondiente para [Linux](./availability.md).

Dentro de una máquina virtual, puede obtener una notificación sobre el próximo mantenimiento si [usa Scheduled Events para Windows](./windows/scheduled-events.md) o para [Linux](./linux/scheduled-events.md).



## <a name="maintenance-that-doesnt-require-a-reboot"></a>Mantenimiento que no requiere un reinicio

La mayoría de las actualizaciones de plataforma no afectan a las máquinas virtuales del cliente. Cuando no es posible realizar una actualización sin causar impacto, Azure elige el mecanismo de actualización con menor impacto en las máquinas virtuales del cliente. 

La mayor parte del mantenimiento sin impacto pone en pausa la máquina virtual durante menos de 10 segundos. En algunos casos, Azure usa mecanismos de mantenimiento de conservación de memoria. Estos mecanismos ponen en pausa la máquina virtual durante un máximo de 30 segundos y conservan la memoria en la RAM. La máquina virtual se reanuda y su reloj se sincroniza automáticamente. 

El mantenimiento de conservación de memoria funciona para más del 90 por ciento de las máquinas virtuales de Azure. No funciona para las series H, M, N y G. Azure usa cada vez más las tecnologías de migración en vivo y mejora el mecanismo de mantenimiento que conserva la memoria para así reducir la duración de la pausa.  

Estas operaciones de mantenimiento que no requieren un reinicio se aplican por dominio de error cada vez. También se detienen si reciben señales de estado de advertencia de las herramientas de supervisión de la plataforma. 

Estos tipos de actualizaciones pueden afectar a algunas aplicaciones. Cuando la máquina virtual se migra en vivo a otro host, es posible que algunas cargas de trabajo sensibles sufran una pequeña degradación del rendimiento en los pocos minutos anteriores a la pausa de la máquina virtual. Para preparar el mantenimiento de la máquina virtual y reducir el impacto durante el mantenimiento de Azure, intente [usar Scheduled Events para Windows](./windows/scheduled-events.md) o [Linux](./linux/scheduled-events.md) para estas aplicaciones. 

Para un mayor control sobre todas las actividades de mantenimiento, incluidas las actualizaciones sin impacto y sin reinicio, puede usar la característica de control de mantenimiento. Debe usar instancias de [Azure Dedicated Host](./dedicated-hosts.md) o una [máquina virtual aislada](../security/fundamentals/isolation-choices.md). El control de mantenimiento ofrece la opción de omitir todas las actualizaciones de la plataforma y aplicarlas a la hora que elija durante un período sucesivo de 35 días. Para obtener más información, consulte [Control de las actualizaciones con el control de mantenimiento y la CLI de Azure](maintenance-control.md).


### <a name="live-migration"></a>Migración en vivo

La migración en vivo es una operación que no requiere un reinicio y que conserva la memoria para la máquina virtual. Provoca una pausa o inmovilización, y normalmente no dura más de 5 segundos. Excepto para las series G, M, N y H, todas las máquinas virtuales de infraestructura como servicio (IaaS) son aptas para la migración en vivo. Las máquinas virtuales aptas representan más del 90 por ciento de las máquinas virtuales IaaS que se implementan en la flota de Azure. 

> [!NOTE]
> No recibirá una notificación en Azure Portal para las operaciones de migración en vivo que no requieren un reinicio. Para ver una lista de migraciones en vivo que no requieren un reinicio, [consulte los eventos programados](./windows/scheduled-events.md#query-for-events).

La plataforma Azure inicia la migración en vivo en los escenarios siguientes:
- Mantenimiento planeado
- Error de hardware
- Optimizaciones de asignación

Algunos escenarios de mantenimiento planeado usan la migración en vivo, y se puede usar Scheduled Events para saber de antemano cuando se iniciarán las operaciones de migración en vivo.

La migración en vivo también se puede usar para mover máquinas virtuales cuando los algoritmos de Azure Machine Learning predicen un error inminente de hardware o cuando se quieran optimizar las asignaciones de máquina virtual. Para obtener más información sobre el modelado predictivo que detecta las instancias de hardware degradado, vea [Improving Azure VM resiliency with predictive machine learning and live migration](https://azure.microsoft.com/blog/improving-azure-virtual-machine-resiliency-with-predictive-ml-and-live-migration/?WT.mc_id=thomasmaurer-blog-thmaure) (Mejora de la resistencia de las máquinas virtuales de Azure con la migración en vivo y el aprendizaje automático predictivo). Las notificaciones de migración en vivo aparecen en los registros de Monitor y Service Health de Azure Portal, así como en Scheduled Events si utiliza estos servicios.



## <a name="maintenance-that-requires-a-reboot"></a>Mantenimiento que requiere un reinicio

En el raro caso en que las máquinas virtuales deban reiniciarse para realizar el mantenimiento planeado, recibirá una notificación de antemano. El mantenimiento planeado tiene dos fases: la fase de autoservicio y una fase de mantenimiento programado.

Durante la *fase de autoservicio*, que normalmente tarda cuatro semanas, se inicia el mantenimiento en las máquinas virtuales. Como parte del autoservicio, puede consultar cada máquina virtual para ver su estado y el resultado de la última solicitud de mantenimiento.

Cuando inicia el mantenimiento de autoservicio, su máquina virtual se vuelve a implementar en un nodo ya actualizado. Como la máquina virtual se reinicia, se pierde el disco temporal y se actualizan las direcciones IP dinámicas asociadas con la interfaz de red virtual.

Si surge un error durante el mantenimiento de autoservicio, la operación se detiene, la máquina virtual no se actualiza y se le ofrece la posibilidad de reintentar el mantenimiento de autoservicio. 

Cuando finaliza la fase de autoservicio, comienza la *fase de mantenimiento programado*. Durante esta fase, puede seguir realizando consultas dentro de la fase de mantenimiento, pero no podrá iniciar el mantenimiento usted mismo.

Para obtener más información sobre la administración del mantenimiento que requiere un reinicio, consulte **Control de las notificaciones de mantenimiento planeado** mediante la [CLI](maintenance-notifications-cli.md) de Azure, [PowerShell](maintenance-notifications-powershell.md) o el [portal](maintenance-notifications-portal.md). 

### <a name="availability-considerations-during-scheduled-maintenance"></a>Consideraciones sobre disponibilidad durante el mantenimiento programado 

Si decide esperar a la fase de mantenimiento programado, hay algunas cosas que debe tener en cuenta para mantener la máxima disponibilidad de las máquinas virtuales. 

#### <a name="paired-regions"></a>Regiones emparejadas

Cada región de Azure se empareja con otra región de la misma proximidad geográfica. Juntas, forman un par de regiones. Durante la fase de mantenimiento programado, Azure solo actualiza las máquinas virtuales en una sola región perteneciente a un par de regiones. Por ejemplo, al actualizar las máquinas virtuales de la zona Centro-norte de EE. UU., Azure no actualizará las máquinas virtuales de Centro-sur de EE. UU. al mismo tiempo. Sin embargo, otras regiones como Norte de Europa pueden estar en mantenimiento al mismo tiempo que el Este de EE. UU. Comprender cómo funcionan los pares de regiones puede ayudar a distribuir mejor las máquinas virtuales entre regiones. Para más información, consulte [Regiones de Azure](../best-practices-availability-paired-regions.md).

#### <a name="availability-sets-and-scale-sets"></a>Conjuntos de disponibilidad y conjuntos de escalado

Al implementar una carga de trabajo en máquinas virtuales de Azure, puede crear las máquinas virtuales dentro de un *conjunto de disponibilidad* para proporcionar alta disponibilidad a la aplicación. Mediante los conjuntos de disponibilidad, puede asegurarse de que durante una interrupción o en los eventos de mantenimiento que requieren un reinicio, al menos una máquina virtual estará disponible.

En un conjunto de disponibilidad, las máquinas virtuales individuales se reparten entre un máximo de 20 dominios de actualización. Durante el mantenimiento planeado, solo un dominio de actualización se ve actualizado en un momento determinado. Los dominios de actualización no necesariamente se actualizan de forma secuencial. 

Los *conjuntos de escalado* de máquinas virtuales son un recurso informático de Azure que se puede usar para implementar y administrar un conjunto de máquinas virtuales idénticas como un recurso único. El conjunto de escalado se implementa automáticamente entre dominios de actualización, como las máquinas virtuales de un conjunto de disponibilidad. Igual que sucede con los conjuntos de disponibilidad, cuando se usan conjuntos de escalado, solo un dominio de actualización se actualiza en un momento determinado durante el mantenimiento programado.

Para más información sobre la configuración de máquinas virtuales para alta disponibilidad, consulte el artículo sobre la [administración de la disponibilidad de las máquinas virtuales para Windows](./availability.md) o el artículo correspondiente para [Linux](./availability.md).

#### <a name="availability-zones"></a>Zonas de disponibilidad

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay tres zonas independientes como mínimo en todas las regiones habilitadas. 

Una zona de disponibilidad es una combinación de un dominio de error y un dominio de actualización. Si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas no se actualizan al mismo tiempo.

Cada actualización de infraestructura implementa la zona por zona, dentro de una única región. Sin embargo, puede hacer que la implementación continúe en la zona 1 y una implementación distinta lo haga en la zona 2, al mismo tiempo. No todas las implementaciones se serializan. Sin embargo, una sola implementación únicamente implementa una zona cada vez para reducir el riesgo.

## <a name="next-steps"></a>Pasos siguientes 

Puede usar la [CLI de Azure](maintenance-notifications-cli.md), [Azure PowerShell](maintenance-notifications-powershell.md) o el [portal](maintenance-notifications-portal.md) para administrar el mantenimiento planeado.
