---
title: Preguntas comunes sobre la supervisión de Azure Site Recovery
description: Obtención de respuestas a preguntas comunes sobre la supervisión de Azure Site Recovery mediante la supervisión integrada y Azure Monitor (Log Analytics)
ms.service: site-recovery
services: site-recovery
ms.date: 07/31/2019
ms.topic: conceptual
ms.openlocfilehash: 715fd3a30dcc37a26b07f2ca1c6a0f2ce168a23a
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581721"
---
# <a name="common-questions-about-site-recovery-monitoring"></a>Preguntas comunes sobre la supervisión de Site Recovery

Este artículo responde a preguntas comunes sobre la supervisión de Azure [Site Recovery](site-recovery-overview.md) mediante la supervisión integrada de Site Recovery Azure Monitor (Log Analytics).

## <a name="general"></a>General

### <a name="how-is-the-rpo-value-logged-different-from-the-latest-available-recovery-point"></a>¿En qué se diferencia el valor de RPO registrado del último punto de recuperación disponible?

Site Recovery utiliza un proceso asincrónico de varios pasos para replicar máquinas en Azure.

- En el penúltimo paso de la replicación, los cambios recientes en la máquina junto con los metadatos se copian en una cuenta de registro/almacenamiento en caché.
- Estos cambios, junto con la etiqueta que identifica un punto recuperable, se escriben en la cuenta de almacenamiento o el disco administrado de la región de destino.
- Site Recovery ahora puede generar un punto recuperable para la máquina virtual.
- En este momento, se ha cumplido el RPO para los cambios cargados en la cuenta de almacenamiento hasta ahora. En otras palabras, el RPO de la máquina en este momento equivale a la cantidad de tiempo transcurrido desde la marca de tiempo correspondiente al punto recuperable.
- Ahora, Site Recovery selecciona los datos cargados desde la cuenta de almacenamiento y los aplica a los discos de réplica creados para la máquina.
- A continuación, Site Recovery genera un punto de recuperación y hace que este punto esté disponible para la recuperación en la conmutación por error.
- Por lo tanto, el último punto de recuperación disponible indica la marca de tiempo correspondiente al punto de recuperación más reciente que ya se ha procesado y aplicado a los discos de réplica.


Una hora del sistema incorrecta en la máquina de origen de replicación o en los servidores de infraestructura local distorsionará el valor calculado de RPO. Para crear informes de RPO precisos, asegúrese de que el reloj del sistema sea también preciso en todos los servidores y máquinas.



## <a name="inbuilt-site-recovery-logging"></a>Registro de Site Recovery integrado


### <a name="why-is-the-vm-count-in-the-vault-infrastructure-view-different-from-the-total-count-shown-in-replicated-items"></a>¿Por qué el número de máquinas virtuales de la vista de la infraestructura de almacén se muestra diferente del recuento total de los elementos replicados?

La vista de la infraestructura de almacén tiene un ámbito por escenarios de replicación. Solo las máquinas del escenario de replicación seleccionado actualmente se incluyen en el recuento para la vista. Además, solo contamos las máquinas virtuales que están configuradas para replicarse en Azure. Las máquinas conmutadas por error o aquellas que vuelven a replicarse en un sitio local no se cuentan en la vista.

### <a name="why-is-the-count-of-replicated-items-in-essentials-different-from-the-total-count-of-replicated-items-on-the-dashboard"></a>¿Por qué el número de elementos replicados en Essentials es diferente al número total de elementos replicados en el panel?

Solo las máquinas para las que se ha completado la replicación inicial se incluyen en el recuento que se muestra en Essentials. El total de los elementos replicados, incluye todas las máquinas del almacén, incluidas aquellas para las que la replicación inicial está en curso.

## <a name="azure-monitor-logging"></a>Registro de Azure Monitor


### <a name="how-often-does-site-recovery-send-resource-logs-to-azure-monitor-log"></a>¿Con qué frecuencia Site Recovery envía registros de recursos al registro de Azure Monitor? 

- AzureSiteRecoveryReplicationStats y AzureSiteRecoveryRecoveryPoints se envían cada quince minutos.  
- AzureSiteRecoveryReplicationStats y AzureSiteRecoveryRecoveryPoints se envían cada cinco minutos. 
- AzureSiteRecoveryJobs se envía en el desencadenador y la finalización de un trabajo.
- AzureSiteRecoveryEvents se envía cada vez que se genera un evento. 
- AzureSiteRecoveryReplicatedItems se envía siempre que se produce algún cambio en el entorno. Normalmente, el tiempo de actualización de datos es 15 minutos después de un cambio. 

### <a name="how-long-is-data-kept-in-azure-monitor-logs"></a>¿Cuánto tiempo se conservan los datos en los registros de Azure Monitor? 

De forma predeterminada, la retención es de treinta y un días. Puede aumentar el período en la sección **Usage and Estimated Cost** (Uso y costo estimado)
 en el área de trabajo de Log Analytics. Haga clic en **Retención de datos** y elija el intervalo.

### <a name="whats-the-size-of-the-resource-logs"></a>¿Cuál es el tamaño de los registros de recursos? 

Normalmente, el tamaño de un registro es de 15-20 KB. 


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo supervisar con la [supervisión integrada de Site Recovery](site-recovery-monitor-and-troubleshoot.md) o [Azure Monitor](monitor-log-analytics.md).


