---
title: Notas de la versión de Azure HDInsight.
description: Notas más recientes de la versión de Azure HDInsight. Obtenga sugerencias de desarrollo y detalles sobre Hadoop, Spark, R Server, Hive, etc.
ms.custom: references_regions
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/07/2021
ms.openlocfilehash: 399ba119e7a58a04bd623d8b08c28dd860304b99
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632230"
---
# <a name="azure-hdinsight-release-notes"></a>Notas de la versión de Azure HDInsight

Este artículo proporciona información sobre las **últimas** actualizaciones de versión de Azure HDInsight. Para más información sobre versiones anteriores, vea [Notas de la versión (en archivo) de los componentes de Hadoop en Azure HDInsight](hdinsight-release-notes-archive.md).

## <a name="summary"></a>Resumen

Azure HDInsight es uno de los servicios más populares entre los clientes de empresa para el análisis de código abierto en Azure.

Si quiere suscribirse a las notas de la versión, vea las versiones de [este repositorio de GitHub](https://github.com/hdinsight/release-notes/releases).

## <a name="price-correction-for-hdinsight-dv2-virtual-machines"></a>Corrección de precios para máquinas virtuales HDInsight Dv2

El 25 de abril de 2021 se corrigió un error de precios para la serie de máquinas virtuales Dv2 en HDInsight. El error de precios produjo un cargo reducido en las facturas de algunos clientes antes del 25 de abril y, con la corrección, los precios ahora coinciden con lo que se había anunciado en la página de precios de HDInsight y en la calculadora de precios de HDInsight. El error de precios afectaba a los clientes de las siguientes regiones que usaban máquinas virtuales Dv2:

- Centro de Canadá
- Este de Canadá
- Este de Asia
- Norte de Sudáfrica
- Sudeste de Asia
- Centro de Emiratos Árabes Unidos

A partir del 25 de abril de 2021, figurará en su cuenta la cantidad corregida de las máquinas virtuales Dv2. Las notificaciones de los clientes se enviaron a los propietarios de la suscripción antes del cambio. Puede usar la calculadora de precios, la página de precios de HDInsight o la hoja de creación de un clúster de HDInsight de Azure Portal para ver los costos corregidos de las máquinas virtuales Dv2 en su región.

No es necesaria ninguna otra acción por su parte. La corrección de precios solo se aplicará para el uso a partir del 25 de abril de 2021 en las regiones especificadas, y no a ningún uso anterior a esta fecha. Para asegurarse de que tiene la solución más eficaz y rentable, se recomienda revisar los precios, VCPU y RAM de los clústeres Dv2 y comparar las especificaciones de Dv2 con las máquinas virtuales Ev3 para ver si la solución se beneficiaría del uso de una de las series de máquinas virtuales más recientes.

## <a name="release-date-03242021"></a>Fecha de lanzamiento: 24/03/2021

Esta versión se aplica a HDInsight 3.6 y HDInsight 4.0. La versión de HDInsight se pone a disposición de todas las regiones durante varios días. Esta fecha de lanzamiento indica la fecha de lanzamiento de la primera región. Si no ve los cambios siguientes, espere unos días a que la versión se active en su región.

## <a name="new-features"></a>Nuevas características
### <a name="spark-30-preview"></a>Versión preliminar de Spark 3.0
HDInsight agregó compatibilidad con [Spark 3.0.0](https://spark.apache.org/docs/3.0.0/) a HDInsight 4.0 como una característica en vista previa. 

### <a name="kafka-24-preview"></a>Versión preliminar de Kafka 2.4
HDInsight agregó compatibilidad con [Kafka 2.4.1](http://kafka.apache.org/24/documentation.html) a HDInsight 4.0 como una característica en vista previa.

### <a name="eav4-series-support"></a>Compatibilidad con la serie Eav4
HDInsight ha agregado compatibilidad con la serie Eav4 en esta versión. Obtenga más información sobre la [serie Dav4 aquí](../virtual-machines/eav4-easv4-series.md). La serie está disponible en las siguientes regiones: 

* Este de Australia
* Sur de Brasil
* Centro de EE. UU.
* Este de Asia
* Este de EE. UU.
* Japón Oriental
* Sudeste de Asia
* Sur de Reino Unido
* Oeste de Europa
* Oeste de EE. UU. 2

### <a name="moving-to-azure-virtual-machine-scale-sets"></a>Movimiento a conjuntos de escalado de máquinas virtuales
Ahora HDInsight usa máquinas virtuales de Azure para aprovisionar el clúster. El servicio se migra gradualmente a [conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/overview.md). Todo el proceso puede tardar meses. Después de migrar las regiones y las suscripciones, los clústeres de HDInsight recién creados se ejecutarán en conjuntos de escalado de máquinas virtuales sin acciones del cliente. No se espera ningún cambio importante.

## <a name="deprecation"></a>Desuso
No hay desuso en esta versión.

## <a name="behavior-changes"></a>Cambios de comportamiento
### <a name="default-cluster-version-is-changed-to-40"></a>La versión predeterminada del clúster cambiará a la 4.0
La versión predeterminada del clúster de HDInsight cambiará de la 3.6 a la 4.0. Para más información sobre las versiones disponibles, consulte [este artículo](./hdinsight-component-versioning.md). Más información sobre las novedades de [HDInsight 4.0](./hdinsight-version-release.md).

### <a name="default-cluster-vm-sizes-are-changed-to-ev3-series"></a>Los tamaños de la máquina virtual del clúster predeterminado cambian en la serie Ev3 
Los tamaños de la máquina virtual del clúster predeterminado cambia de la serie D a la serie Ev3. Este cambio se aplica a los nodos principales y los nodos de trabajo. Para evitar que este cambio afecte a los flujos de trabajo probados, especifique los tamaños de máquina virtual que desea usar en la plantilla de Resource Manager.

### <a name="network-interface-resource-not-visible-for-clusters-running-on-azure-virtual-machine-scale-sets"></a>El recurso de la interfaz de red no es visible para clústeres que se ejecutan en conjuntos de escalado de máquinas virtuales de Azure.
HDInsight se está migrando gradualmente a conjuntos de escalado de máquinas virtuales de Azure. Los clientes ya no pueden ver las interfaces de red de las máquinas virtuales de los clústeres que usan conjuntos de escalado de máquinas virtuales de Azure.

## <a name="upcoming-changes"></a>Próximos cambios
En las próximas versiones, se realizarán los siguientes cambios.

### <a name="hdinsight-interactive-query-only-supports-schedule-based-autoscale"></a>HDInsight Interactive Query solo admite el escalado automático basado en programación

A medida que los escenarios de los clientes se vuelven más maduros y diversos, vamos identificando algunas limitaciones con la escala automática basada en la carga de Interactive Query (LLAP). Estas limitaciones se deben a la naturaleza de la dinámica de las consultas de LLAP, problemas de precisión en la predicción de carga futura y problemas en la redistribución de tareas del programador LLAP. Debido a estas limitaciones, es posible que los usuarios vean que sus consultas se ejecutan más lentamente en clústeres LLAP cuando el escalado automático está habilitado. El impacto en el rendimiento puede superar las ventajas relativas al costo de la escala automática.

A partir del 15 de mayo de 2021, la carga de trabajo en HDInsight solo admite el escalado automático basado en programación. Ya no puede habilitar el escalado automático en clústeres Interactive Query nuevos. Los clústeres en ejecución existentes pueden seguir ejecutándose con las limitaciones conocidas descritas anteriormente. 

Microsoft recomienda pasar a un escalado automático basado en programación para LLAP.  Puede analizar el patrón de uso actual del clúster a través del panel de Grafana Hive. Para más información, consulte [Escalado automático de clústeres de Azure HDInsight](hdinsight-autoscale-clusters.md). 

### <a name="os-version-upgrade"></a>Actualización de la versión del sistema operativo
Los clústeres de HDInsight se ejecutan actualmente en Ubuntu 16.04 LTS. Como se comenta en el [ciclo de versiones de Ubuntu](https://ubuntu.com/about/release-cycle),el kernel de Ubuntu 16.04 alcanzará el final del ciclo de vida (EOL) en abril de 2021. Comenzaremos a implementar la nueva imagen de clúster de HDInsight 4.0 que se ejecuta en Ubuntu 18.04 en mayo de 2021. Los clústeres de HDInsight 4.0 recién creados se ejecutarán en Ubuntu 18.04 de forma predeterminada una vez que esté disponible. Los clústeres existentes en Ubuntu 16.04 se ejecutarán tal y como están con soporte técnico completo.

HDInsight 3.6 seguirá funcionando en Ubuntu 16.04. El soporte técnico estándar terminará el 30 de junio de 2021 y cambiará a soporte técnico Basic a partir del 1 de julio de 2021. Para más información sobre las fechas y las opciones de soporte técnico, consulte las [versiones de Azure HDInsight](./hdinsight-component-versioning.md#supported-hdinsight-versions). Ubuntu 18.04 no será compatible con HDInsight 3.6. Si desea usar Ubuntu 18.04, deberá migrar los clústeres a HDInsight 4.0. 

Debe quitar y volver a crear los clústeres si desea mover los existentes a Ubuntu 18.04. Planee crear o volver a crear el clúster una vez sea compatible con Ubuntu 18.04. Se enviará otra notificación cuando la nueva imagen esté disponible en todas las regiones.

Se recomienda encarecidamente probar previamente las acciones de script y las aplicaciones personalizadas implementadas en nodos perimetrales en una máquina virtual Ubuntu 18.04. Puede [crear una máquina virtual Ubuntu Linux sencilla en la versión 18.04-LTS](https://azure.microsoft.com/resources/templates/101-vm-simple-linux/) y, a continuación, crear y usar un [par de claves de Secure Shell (SSH)](../virtual-machines/linux/mac-create-ssh-keys.md#ssh-into-your-vm) en la máquina virtual para ejecutar y probar las acciones de script y las aplicaciones personalizadas implementadas en los nodos perimetrales.

### <a name="disable-stardard_a5-vm-size-as-head-node-for-hdinsgiht-40"></a>Deshabilitación del tamaño Stardard_A5 de máquina virtual como nodo principal para HDInsgiht 4.0
El nodo principal del clúster de HDInsight es responsable de inicializar y administrar el clúster. El tamaño Standard_A5 de máquina virtual tiene problemas de confiabilidad como nodo principal para HDInsight 4.0. A partir de la próxima versión de mayo de 2021, los clientes no podrán crear nuevos clústeres con el tamaño Standard_A5 de máquina virtual como nodo principal. Puede usar otras máquinas virtuales de 2 núcleos, como E2_v3 o E2s_v3. Los clústeres existentes se ejecutarán tal cual. Se recomienda encarecidamente una máquina virtual de 4 núcleos como mínimo para que el nodo principal garantice la alta disponibilidad y confiabilidad de los clústeres de HDInsight de producción.

### <a name="basic-support-for-hdinsight-36-starting-july-1-2021"></a>Soporte técnico Basic para HDInsight 3.6 a partir del 1 de julio de 2021
A partir del 1 de julio de 1, 2021, Microsoft ofrecerá [soporte técnico Basic](hdinsight-component-versioning.md#support-options-for-hdinsight-versions) para determinados tipos de clúster de HDInsight 3.6. El plan de soporte técnico Basic estará disponible hasta el 3 de abril de 2022. Se inscribirá automáticamente en soporte técnico Basic a partir del 1 de julio de 2021. No es necesario realizar ninguna acción para participar. Consulte [nuestra documentación](hdinsight-36-component-versioning.md) sobre los tipos de clúster que se incluyen en soporte técnico Basic. 

No se recomienda crear nuevas soluciones en HDInsight 3.6, inmovilice los cambios en entornos existentes de 3.6. Se recomienda [migrar los clústeres a HDInsight 4.0](hdinsight-version-release.md#how-to-upgrade-to-hdinsight-40). Más información sobre [las novedades de HDInsight 4.0](hdinsight-version-release.md#whats-new-in-hdinsight-40).

## <a name="bug-fixes"></a>Corrección de errores
HDInsight continúa realizando mejoras en la confiabilidad y el rendimiento del clúster. 

## <a name="component-version-change"></a>Cambio de versión de componentes
Se ha agregado compatibilidad con Spark 3.0.0 y Kafka 2.4.1 como versión preliminar. En [este documento](./hdinsight-component-versioning.md) puede encontrar las versiones actuales de los componentes para HDInsight 4.0 y HDInsight 3.6.

## <a name="recommanded-features"></a>Características recomendadas
### <a name="service-tags"></a>Etiquetas de servicio
Las etiquetas de servicio simplifican la restricción del acceso de red a los servicios de Azure para las máquinas virtuales y las redes virtuales de Azure. Las etiquetas de servicio en las reglas del grupo de seguridad de red (NSG) permiten o impiden el tráfico a un servicio específico de Azure. La regla se puede establecer globalmente o por región de Azure. Azure proporciona el mantenimiento de las direcciones IP subyacentes a cada etiqueta. Las etiquetas de servicio de HDInsight de los grupos de seguridad de red (NSG) son grupos de direcciones IP para los servicios de mantenimiento y administración. Estos grupos ayudan a minimizar la complejidad de la creación de reglas de seguridad. Los clientes de HDInsight pueden habilitar la etiqueta de servicio mediante Azure Portal, PowerShell y la API de REST. Para más información, consulte [Etiquetas de servicio del grupo de seguridad de red (NSG) para Azure HDInsight](./hdinsight-service-tags.md).
