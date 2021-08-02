---
title: Notas de la versión de Azure HDInsight.
description: Notas más recientes de la versión de Azure HDInsight. Obtenga sugerencias de desarrollo y detalles sobre Hadoop, Spark, R Server, Hive, etc.
ms.custom: references_regions
ms.service: hdinsight
ms.topic: conceptual
ms.date: 06/02/2021
ms.openlocfilehash: 61b32852de0cb15db3860b72561f87e56db8f40d
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111903858"
---
# <a name="azure-hdinsight-release-notes"></a>Notas de la versión de Azure HDInsight

Este artículo proporciona información sobre las **últimas** actualizaciones de versión de Azure HDInsight. Para más información sobre versiones anteriores, vea [Notas de la versión (en archivo) de los componentes de Hadoop en Azure HDInsight](hdinsight-release-notes-archive.md).

## <a name="summary"></a>Resumen

Azure HDInsight es uno de los servicios más populares entre los clientes de empresa para el análisis de código abierto en Azure.

Si quiere suscribirse a las notas de la versión, vea las versiones de [este repositorio de GitHub](https://github.com/hdinsight/release-notes/releases).


## <a name="release-date-06022021"></a>Fecha de lanzamiento: 02/06/2021

Esta versión se aplica a HDInsight 3.6 y HDInsight 4.0. La versión de HDInsight se pone a disposición de todas las regiones durante varios días. Esta fecha de lanzamiento indica la fecha de lanzamiento de la primera región. Si no ve los cambios siguientes, espere unos días a que la versión se active en su región.

Las versiones del sistema operativo de esta versión son:
- HDInsight 3.6: Ubuntu 16.04.7 LTS
- HDInsight 4.0: Ubuntu 18.04.5 LTS

## <a name="new-features"></a>Nuevas características
### <a name="os-version-upgrade"></a>Actualización de la versión del sistema operativo
Como se comenta en el [ciclo de versiones de Ubuntu](https://ubuntu.com/about/release-cycle),el kernel de Ubuntu 16.04 alcanzará el final del ciclo de vida (EOL) en abril de 2021. Hemos empezado a implementar la nueva imagen de clúster de HDInsight 4.0 que se ejecuta en Ubuntu 18.04 en esta versión. Los clústeres de HDInsight 4.0 recién creados se ejecutarán en Ubuntu 18.04 de forma predeterminada una vez que esté disponible. Los clústeres existentes en Ubuntu 16.04 se ejecutarán tal y como están con soporte técnico completo.

HDInsight 3.6 seguirá funcionando en Ubuntu 16.04. Cambiará a soporte técnico Basic (desde el soporte técnico Standard) a partir del 1 de julio de 2021. Para más información sobre las fechas y las opciones de soporte técnico, consulte las [versiones de Azure HDInsight](./hdinsight-component-versioning.md#supported-hdinsight-versions). Ubuntu 18.04 no será compatible con HDInsight 3.6. Si desea usar Ubuntu 18.04, deberá migrar los clústeres a HDInsight 4.0. 

Debe quitar y volver a crear los clústeres si desea mover los clústeres existentes de HDInsight 4.0 a Ubuntu 18.04. Planee crear o volver a crear los clústeres una vez sea compatible con Ubuntu 18.04.

Después de crear el nuevo clúster, puede conectarse mediante SSH al clúster y ejecutar `sudo lsb_release -a` para comprobar que se ejecuta en Ubuntu 18.04. Se recomienda probar primero las aplicaciones en las suscripciones de prueba antes de pasar a producción. [Obtenga más información sobre la actualización de Ubuntu 18.04 de HDInsight](./hdinsight-ubuntu-1804-qa.md).

### <a name="scaling-optimizations-on-hbase-accelerated-writes-clusters"></a>Optimizaciones de escalado en clústeres de escritura acelerada de HBase
HDInsight ha realizado algunas mejoras y optimizaciones en el escalado de clústeres habilitados para escritura acelerada de HBase. [Obtenga más información sobre la escritura acelerada de HBase](./hbase/apache-hbase-accelerated-writes.md).

## <a name="deprecation"></a>Desuso
No hay desuso en esta versión.

## <a name="behavior-changes"></a>Cambios de comportamiento
### <a name="disable-stardard_a5-vm-size-as-head-node-for-hdinsight-40"></a>Deshabilitación del tamaño Stardard_A5 de máquina virtual como nodo principal para HDInsight 4.0
El nodo principal del clúster de HDInsight es responsable de inicializar y administrar el clúster. El tamaño Standard_A5 de máquina virtual tiene problemas de confiabilidad como nodo principal para HDInsight 4.0. A partir de esta versión, los clientes no podrán crear nuevos clústeres con el tamaño Standard_A5 de máquina virtual como nodo principal. Puede usar otras máquinas virtuales de 2 núcleos, como E2_v3 o E2s_v3. Los clústeres existentes se ejecutarán tal cual. Se recomienda encarecidamente una máquina virtual de 4 núcleos como mínimo para que el nodo principal garantice la alta disponibilidad y confiabilidad de los clústeres de HDInsight de producción.

### <a name="network-interface-resource-not-visible-for-clusters-running-on-azure-virtual-machine-scale-sets"></a>El recurso de la interfaz de red no es visible para clústeres que se ejecutan en conjuntos de escalado de máquinas virtuales de Azure.
HDInsight se está migrando gradualmente a conjuntos de escalado de máquinas virtuales de Azure. Los clientes ya no pueden ver las interfaces de red de las máquinas virtuales de los clústeres que usan conjuntos de escalado de máquinas virtuales de Azure.

## <a name="upcoming-changes"></a>Próximos cambios
En las próximas versiones, se realizarán los siguientes cambios.

### <a name="hdinsight-interactive-query-only-supports-schedule-based-autoscale"></a>HDInsight Interactive Query solo admite el escalado automático basado en programación

A medida que los escenarios de los clientes se vuelven más maduros y diversos, vamos identificando algunas limitaciones con la escala automática basada en la carga de Interactive Query (LLAP). Estas limitaciones se deben a la naturaleza de la dinámica de las consultas de LLAP, problemas de precisión en la predicción de carga futura y problemas en la redistribución de tareas del programador LLAP. Debido a estas limitaciones, es posible que los usuarios vean que sus consultas se ejecutan más lentamente en clústeres LLAP cuando el escalado automático está habilitado. El impacto en el rendimiento puede superar las ventajas relativas al costo de la escalabilidad automática.

A partir de julio de 2021, la carga de trabajo de Interactive Query en HDInsight solo admite la escalabilidad automática basada en programación. Ya no puede habilitar el escalado automático en clústeres Interactive Query nuevos. Los clústeres en ejecución existentes pueden seguir ejecutándose con las limitaciones conocidas descritas anteriormente. 

Microsoft recomienda pasar a un escalado automático basado en programación para LLAP.  Puede analizar el patrón de uso actual del clúster a través del panel de Grafana Hive. Para más información, consulte [Escalado automático de clústeres de Azure HDInsight](hdinsight-autoscale-clusters.md). 

### <a name="basic-support-for-hdinsight-36-starting-july-1-2021"></a>Soporte técnico Basic para HDInsight 3.6 a partir del 1 de julio de 2021
A partir del 1 de julio de 1, 2021, Microsoft ofrecerá [soporte técnico Basic](hdinsight-component-versioning.md#support-options-for-hdinsight-versions) para determinados tipos de clúster de HDInsight 3.6. El plan de soporte técnico Basic estará disponible hasta el 3 de abril de 2022. Se inscribirá automáticamente en soporte técnico Basic a partir del 1 de julio de 2021. No es necesario realizar ninguna acción para participar. Consulte [nuestra documentación](hdinsight-36-component-versioning.md) sobre los tipos de clúster que se incluyen en soporte técnico Basic. 

No se recomienda crear nuevas soluciones en HDInsight 3.6, inmovilice los cambios en entornos existentes de 3.6. Se recomienda [migrar los clústeres a HDInsight 4.0](hdinsight-version-release.md#how-to-upgrade-to-hdinsight-40). Más información sobre [las novedades de HDInsight 4.0](hdinsight-version-release.md#whats-new-in-hdinsight-40).

### <a name="vm-host-naming-will-be-changed-on-july-1-2021"></a>La nomenclatura del host de máquina virtual se cambiará el 1 de julio de 2021.
Ahora HDInsight usa máquinas virtuales de Azure para aprovisionar el clúster. El servicio se migra gradualmente a [conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/overview.md). Esta migración cambiará el formato de nombre de FQDN del nombre de host del clúster y no se garantiza que los números del nombre de host estén ordenados secuencialmente. Si desea obtener los nombres de FQDN de cada nodo, consulte [Buscar los nombres de host de los nodos de clúster](./find-host-name.md).

### <a name="move-to-azure-virtual-machine-scale-sets"></a>Traslado a conjuntos de escalado de máquinas virtuales
Ahora HDInsight usa máquinas virtuales de Azure para aprovisionar el clúster. El servicio se migra gradualmente a [conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/overview.md). Todo el proceso puede tardar meses. Después de migrar las regiones y las suscripciones, los clústeres de HDInsight recién creados se ejecutarán en conjuntos de escalado de máquinas virtuales sin acciones del cliente. No se espera ningún cambio importante.

## <a name="bug-fixes"></a>Corrección de errores
HDInsight continúa realizando mejoras en la confiabilidad y el rendimiento del clúster. 

## <a name="component-version-change"></a>Cambio de versión de componentes
En [este documento](./hdinsight-component-versioning.md) puede encontrar las versiones actuales de los componentes para HDInsight 4.0 y HDInsight 3.6.
