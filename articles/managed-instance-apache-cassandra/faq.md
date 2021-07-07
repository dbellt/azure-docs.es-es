---
title: Preguntas más frecuentes sobre Azure Managed Instance for Apache Cassandra en Azure Portal
description: Preguntas más frecuentes sobre Azure Managed Instance for Apache Cassandra. En este artículo se abordan preguntas sobre cuándo usar instancias administradas, sus ventajas, los límites de rendimiento, las regiones admitidas y otros detalles de configuración.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.openlocfilehash: 6953ffb5542e2a0d8acb9e404b2df0f923a08145
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111744024"
---
# <a name="frequently-asked-questions-about-azure-managed-instance-for-apache-cassandra-preview"></a>Preguntas más frecuentes sobre Azure Managed Instance for Apache Cassandra (versión preliminar).

En este artículo se abordan las preguntas más frecuentes sobre Azure Managed Instance for Apache Cassandra. Aprenderá cuándo usar instancias administradas, sus ventajas, los límites de rendimiento, las regiones admitidas y sus detalles de configuración.

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="general-faq"></a>Preguntas más frecuentes generales

### <a name="what-are-the-benefits-azure-managed-instance-for-apache-cassandra"></a>¿Cuáles son las ventajas de Azure Managed Instance for Apache Cassandra?

La base de datos de Apache Cassandra es la opción adecuada cuando se necesita escalabilidad y alta disponibilidad sin poner en peligro el rendimiento. Es una excelente plataforma para datos críticos debido a una escalabilidad lineal y una tolerancia a errores comprobada en la infraestructura de hardware estándar o en la nube. Azure Managed Instance for Apache Cassandra es un servicio para administrar instancias de centros de datos de Apache Cassandra de código abierto implementados en Azure.

Puede usarse en su totalidad en la nube o como parte de una nube híbrida y un clúster local. Este servicio es una excelente opción si desea tener un control y una configuración específicos en Apache Cassandra de código abierto, sin ninguna sobrecarga de mantenimiento.

### <a name="why-should-i-use-this-service-instead-of-azure-cosmos-db-cassandra-api"></a>¿Por qué debo usar este servicio en lugar de Azure Cosmos DB Cassandra API?

El equipo de Azure Cosmos DB es el que ha creado Azure Managed Instance for Apache Cassandra. Es un servicio administrado independiente para implementar, mantener y escalar clústeres y centros de datos de Apache Cassandra de código abierto. [Azure Cosmos DB Cassandra API](../cosmos-db/cassandra-introduction.md) por otro lado es una plataforma como servicio que proporciona una capa de interoperabilidad para el protocolo de conexión de Apache Cassandra. Si su expectativa es que la plataforma se comporte exactamente de la misma manera que cualquier clúster de Apache Cassandra, debe elegir el servicio Managed Instance. Para más información, consulte el artículo [Azure Managed Instance for Apache Cassandra frente a Azure Cosmos DB Cassandra API](compare-cosmosdb-managed-instance.md).

### <a name="is-azure-managed-instance-for-apache-cassandra-dependent-on-azure-cosmos-db"></a>¿Depende Azure Managed Instance for Apache Cassandra de Azure Cosmos DB?

No, no hay ninguna dependencia en cuanto a la arquitectura entre Azure Managed Instance for Apache Cassandra y el back-end de Azure Cosmos DB. 

#### <a name="can-i-deploy-azure-managed-instance-for-apache-cassandra-in-any-region"></a>¿Puedo implementar Azure Managed Instance for Apache Cassandra en cualquier región?

La instancia administrada estará disponible en un número limitado de regiones durante la versión preliminar.

### <a name="what-are-the-storage-and-throughput-limits-of-azure-managed-instance-for-apache-cassandra"></a>¿Cuáles son los límites de almacenamiento y rendimiento de Azure Managed Instance for Apache Cassandra?

Estos límites dependen de las SKU de máquina virtual que elija.

### <a name="how-are-cassandra-repairs-carried-out-in-azure-managed-instance-for-apache-cassandra"></a>¿Cómo se realizan las reparaciones de Cassandra en Azure Managed Instance for Apache Cassandra?

Se usa [cassandra-reaper.io](http://cassandra-reaper.io/), y se configura para que se ejecute automáticamente.

### <a name="what-is-the-cost-of-azure-managed-instance-for-apache-cassandra"></a>¿Cuál es el costo de Azure Managed Instance for Apache Cassandra?

Los cargos de Managed Instance se basan en el costo de la máquina virtual subyacente, con un pequeño margen de beneficio. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/managed-instance-apache-cassandra/)para obtener más información.

### <a name="can-i-use-yaml-file-settings-to-configure-behavior"></a>¿Puedo usar la configuración de archivos de YAML para configurar el comportamiento?

Sí, puede insertar configuraciones de archivo YAML como parte de una implementación de plantilla de Azure Resource Manager.

### <a name="how-can-i-monitor-infrastructure-along-with-throughput"></a>¿Cómo puedo supervisar la infraestructura junto con el rendimiento?

El servidor [Prometheus](https://prometheus.io/docs/introduction/overview/) se hospeda para supervisar la actividad en el clúster y expone un punto de conexión. Esto mantiene 10 minutos o 10 GB de datos (lo que suceda primero). Para usar esta supervisión, debe configurar una [federación](https://prometheus.io/docs/prometheus/latest/federation/) y una herramienta de panel adecuada, como Grafana.

### <a name="does-azure-managed-instance-for-apache-cassandra-provide-full-backups"></a>¿Proporciona Azure Managed Instance for Apache Cassandra copias de seguridad completas?

Sí, proporciona copias de seguridad completas para Azure Storage y se restaura en un nuevo clúster.

### <a name="how-can-i-migrate-data-from-my-existing-apache-cassandra-cluster-to-azure-managed-instance-for-apache-cassandra"></a>¿Cómo puedo migrar datos desde mi clúster de Apache Cassandra existente a Azure Managed Instance for Apache Cassandra?

Azure Managed Instance for Apache Cassandra admite todas las características de Apache Cassandra para replicar y transmitir datos entre los centros de datos.

### <a name="can-i-pair-an-on-premises-apache-cassandra-cluster-with-the-azure-managed-instance-for-apache-cassandra"></a>¿Puedo emparejar un clúster de Apache Cassandra local con Azure Managed Instance for Apache Cassandra?

Sí, puede configurar un clúster híbrido con centros de datos insertados de Azure Virtual Network implementados por el servicio. Los centros de datos de Managed Instance pueden comunicarse con centros de datos locales dentro del mismo anillo del clúster.

### <a name="where-can-i-give-feedback-on-azure-managed-instance-for-apache-cassandra-features"></a>¿Dónde puedo enviar comentarios sobre las características de Azure Managed Instance for Apache Cassandra?

Proporcione comentarios a través de los [comentarios de voz de los usuarios](https://feedback.azure.com/forums/263030-azure-cosmos-db?category_id=398548) mediante la categoría "Managed Apache Cassandra".

Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.

## <a name="deployment-specific-faq"></a>Preguntas más frecuentes específicas sobre la implementación

### <a name="will-the-managed-instance-support-node-addition-cluster-status-and-node-status-commands"></a>¿Admitirá Managed Instance los comandos de estado de nodo, estado de clústeres y adición de nodos?

Todos los comandos de *solo lectura* de Nodetool, como `status`, están disponibles mediante la CLI de Azure. Sin embargo, las operaciones como la *adición de nodos* no están disponibles, ya que nosotros administramos el estado de los nodos de la instancia administrada. En el modo híbrido, puede conectarse al clúster con *Nodetool*. Sin embargo, no se recomienda usar Nodetool, ya que podría desestabilizar el clúster. También puede invalidar cualquier Acuerdo de Nivel de Servicio de respaldo a la producción relacionado con el estado de los centros de recursos de la instancia administrada en el clúster.

### <a name="what-happens-with-various-settings-for-table-metadata"></a>¿Qué ocurre con las distintas opciones de los metadatos de una tabla?

La configuración de metadatos de tabla como, por ejemplo, el filtro de Bloom, almacenamiento en caché, oportunidad de reparación de lectura, gc_grace y memtable_flush_period de compresión son totalmente compatibles con cualquier entorno de Apache Cassandra autohospedado.

## <a name="next-steps"></a>Pasos siguientes

Para ver las preguntas más frecuentes sobre otras API, consulte:

* [Creación de un clúster de instancia administrada desde Azure Portal](create-cluster-portal.md)
* [Implementación de un clúster de Apache Spark administrado con Azure Databricks](deploy-cluster-databricks.md)
* [Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure](manage-resources-cli.md)