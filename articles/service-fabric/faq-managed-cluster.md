---
title: Preguntas frecuentes sobre los clústeres administrados de Service Fabric
description: Preguntas frecuentes sobre los clústeres administrados de Service Fabric, incluidas las funcionalidades, los casos de uso y los escenarios comunes.
ms.topic: troubleshooting
ms.author: pepogors
author: peterpogorski
ms.date: 5/10/2021
ms.custom: references_regions
ms.openlocfilehash: f79146e1271436e29dcd7f911c250cab58c28707
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111956767"
---
# <a name="service-fabric-managed-clusters-frequently-asked-questions"></a>Preguntas más frecuentes sobre los clústeres administrados de Service Fabric

Aquí encontrará algunas preguntas frecuentes y sus respuestas sobre los clústeres administrados de Service Fabric.

## <a name="general"></a>General

### <a name="what-are-service-fabric-managed-clusters"></a>¿Qué son los clústeres administrados de Service Fabric?

Los clústeres administrados de Service Fabric son una evolución del modelo de recursos de clúster de Service Fabric diseñado para facilitar la implementación y administración de los clústeres. Un clúster administrado de Service Fabric usa el modelo de encapsulación de Azure Resource Manager para que un usuario solo necesite definir e implementar un único recurso de clúster en comparación con los muchos recursos independientes que deben implementar hoy (conjunto de escalado de máquinas virtuales, Load Balancer, IP, etc.).

### <a name="what-regions-are-supported"></a>¿Qué regiones se admiten?

Los clústeres administrados de Service Fabric se admiten en todas las regiones de la nube pública.

### <a name="can-i-do-an-in-place-migration-of-my-existing-service-fabric-cluster-to-a-managed-cluster-resource"></a>¿Puedo realizar una migración en contexto del clúster de Service Fabric existente a un recurso de clúster administrado?

No. Necesitará crear un recurso de clúster de Service Fabric para usar el nuevo tipo de recurso de clúster administrado de Service Fabric.

### <a name="is-there-an-additional-cost-for-service-fabric-managed-clusters"></a>¿Hay un costo adicional para los clústeres administrados de Service Fabric?

No. No se aplica ningún costo adicional asociado con un clúster administrado de Service Fabric aparte del costo de los recursos de red, almacenamiento y proceso subyacentes necesarios para el clúster.

### <a name="is-there-a-new-sla-introduced-by-the-service-fabric-managed-cluster-resource"></a>¿El recurso de clúster administrado de Service Fabric incluye algún Acuerdo de Nivel de Servicio nuevo?

El Acuerdo de Nivel de Servicio no cambia respecto al modelo de recursos de Service Fabric actual.

### <a name="what-is-the-difference-between-a-basic-and-standard-sku-cluster"></a>¿Cuál es la diferencia entre un clúster de SKU Básico y Estándar?

Un clúster de SKU Básico supone que la mayoría de las configuraciones las proporciona el proveedor de recursos de Service Fabric. Los clústeres de SKU Básico están diseñados para usarse en entornos de prueba y preproducción. Un clúster de SKU Estándar permite a los usuarios configurar el clúster para satisfacer sus necesidades de forma específica. Para más información, consulte [SKU de clúster administrado de Service Fabric](./overview-managed-cluster.md#service-fabric-managed-cluster-skus).

## <a name="cluster-deployment-and-management"></a>Implementación y administración de clústeres

### <a name="i-run-custom-script-extensions-on-my-virtual-machine-scale-set-can-i-continue-to-do-that-with-a-managed-service-fabric-resource"></a>Ejecuto extensiones de script personalizadas en mi conjunto de escalado de máquinas virtuales, ¿puedo continuar haciendo eso con un recurso de Service Fabric administrado?

Sí, puede especificar extensiones de máquina virtual en tipos de nodo de clúster administrado. Para más información, consulte [Incorporación de una extensión de conjunto de escalado a un tipo de nodo de clúster administrado de Service Fabric](how-to-managed-cluster-vmss-extension.md).

### <a name="i-want-to-have-an-internal-only-load-balancer-is-that-possible"></a>Quiero tener un equilibrador de carga solo interno, ¿es posible?

No. Actualmente no es posible tener un equilibrador de carga solo interno. Se recomienda consolidar las reglas del grupo de seguridad de red (NSG) para bloquear el tráfico entrante o saliente no deseado.

### <a name="can-i-autoscale-my-cluster"></a>¿Puedo escalar mi clúster automáticamente?

Actualmente no se admite la escalabilidad automática.

### <a name="can-i-deploy-my-cluster-across-availability-zones"></a>¿Puedo implementar mi clúster en zonas de disponibilidad?

Sí, los clústeres administrados de Service Fabric que abarcan zonas de disponibilidad se admiten en aquellas regiones de Azure que admiten zonas de disponibilidad. Para más información, consulte el artículo sobre [clústeres administrados de Service Fabric entre zonas de disponibilidad](.\service-fabric-cross-availability-zones.md).

### <a name="can-i-deploy-stateless-node-types-on-a-service-fabric-managed-cluster"></a>¿Se pueden implementar tipos de nodos sin estado en un clúster administrado de Service Fabric? 

Sí, los clústeres administrados de Service Fabric admiten tipos de nodo sin estado para todos los tipos de nodo secundario. Para más información, consulte el artículo sobre [tipos de nodos sin estado de un clúster administrado de Service Fabric](./how-to-managed-cluster-stateless-node-type.md).

### <a name="can-i-select-between-automatic-and-manual-upgrades-for-my-cluster-runtime"></a>¿Puedo seleccionar entre actualizaciones automáticas y manuales del clúster en tiempo de ejecución?

Sí, puede seleccionar entre actualizaciones automáticas y manuales. Para más información, consulte el artículo sobre las [actualizaciones de clústeres](./service-fabric-cluster-upgrade.md).

## <a name="applications"></a>Aplicaciones

### <a name="is-there-a-local-development-experience-for-service-fabric-managed-clusters"></a>¿Hay alguna experiencia de desarrollo local para los clústeres administrados de Service Fabric?

La experiencia de desarrollo local no cambia respecto a los clústeres de Service Fabric existentes. Para más información, consulte [Configuración del entorno de desarrollo](./service-fabric-get-started.md) para obtener más detalles sobre la experiencia de desarrollo local.

### <a name="can-i-deploy-my-applications-as-an-azure-resource-manager-resource"></a>¿Puedo implementar mis aplicaciones como un recurso de Azure Resource Manager?

Sí. Se ha agregado compatibilidad para implementar aplicaciones como un recurso de Azure Resource Manager (además de la implementación mediante PowerShell y la CLI). Para empezar, consulte [Implementación de una aplicación de clúster administrado de Service Fabric mediante una plantilla de ARM](how-to-managed-cluster-app-deployment-template.md).

### <a name="can-i-deploy-applications-with-managed-identities"></a>¿Puedo implementar aplicaciones con identidades administradas?

 Sí, las aplicaciones con identidades administradas se pueden implementar en los clústeres administrados de Service Fabric. Para más información, consulte el artículo sobre la [protección de identidades administradas](.\concepts-managed-identity.md).