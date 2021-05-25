---
title: Clústeres administrados de Service Fabric
description: Los clústeres administrados de Service Fabric son una evolución del modelo de recursos de clúster de Azure Service Fabric que agiliza la implementación y la administración de clústeres.
ms.topic: overview
ms.date: 5/10/2021
ms.openlocfilehash: a412899f4aa37ce2257a3351c3e27da4d5d3add2
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685268"
---
# <a name="service-fabric-managed-clusters"></a>Clústeres administrados de Service Fabric

Los clústeres administrados de Service Fabric son una evolución del modelo de recursos de clúster de Azure Service Fabric que agiliza la experiencia de implementación y administración de clústeres.

La plantilla del modelo de recursos de Azure (ARM) para los clústeres de Service Fabric tradicionales requiere la definición de un recurso de clúster junto con una serie de recursos de soporte técnico. Todos ellos deben estar "conectados" correctamente (durante la implementación y a lo largo del ciclo de vida del clúster) para que el clúster y los servicios funcionen correctamente. Por el contrario, el modelo de encapsulación para clústeres administrados de Service Fabric consta de un único recurso de *clúster administrado de Service Fabric*. Todos los recursos subyacentes para el clúster se abstraen y se administran mediante Azure en su nombre.

**Modelo de clúster tradicional de Service Fabric**
![Modelo de clúster tradicional de Service Fabric][sf-composition]

**Modelo de clúster administrado de Service Fabric**
![Modelo de clúster encapsulado de Service Fabric][sf-encapsulation]

En lo que se refiere al tamaño y la complejidad, la plantilla de ARM para un clúster administrado de Service Fabric consiste en alrededor de 100 líneas de JSON, frente a las 1000 líneas necesarias aproximadamente para definir un clúster de Service Fabric típico:

| Recursos de Service Fabric | Recursos de clúster administrado de Service Fabric |
|----------|-----------|
| Clúster de Service Fabric | Clúster administrado de Service Fabric |
| Conjuntos de escalado de máquinas virtuales | |
| Equilibrador de carga | |
| Dirección IP pública | |
| Cuentas de almacenamiento | |
| Virtual network | |

Los clústeres administrados de Service Fabric proporcionan una serie de ventajas respecto a los clústeres tradicionales:

**Implementación y administración simplificadas del clúster**
- Implementación y administración de un único recurso de Azure
- Administración y rotación automática de certificados
- Operaciones de escalado simplificadas

**Prevención de errores operativos**
- Prevención de errores de coincidencia de configuración con recursos subyacentes
- Bloqueo de operaciones no seguras (como la eliminación de un nodo de inicialización)

**Procedimientos recomendados de manera predeterminada**
- Configuración simplificada de la confiabilidad y la durabilidad

No hay ningún costo adicional para los clústeres administrados de Service Fabric más allá del costo de los recursos subyacentes necesarios para el clúster, y el mismo acuerdo de nivel de servicio de Service Fabric se aplica a los clústeres administrados.

> [!NOTE]
> No hay ninguna ruta de migración desde clústeres de Service Fabric existentes a clústeres administrados. Tendrá que crear un nuevo clúster administrado de Service Fabric para usar este nuevo tipo de recurso.

## <a name="service-fabric-managed-cluster-skus"></a>SKU de clúster administrado de Service Fabric

Los clústeres administrados de Service Fabric están disponibles en las SKU de nivel básico y estándar.

| Característica | Básico | Estándar |
| ------- | ----- | -------- |
| Recurso de red (SKU para [Load Balancer](../load-balancer/skus.md), [IP pública](../virtual-network/public-ip-addresses.md)) | Básico | Estándar |
| Número mínimo de nodos (instancia de máquina virtual) | 3 | 5 |
| Número máximo de nodos por tipo de nodo | 100 | 100 |
| Número máximo de tipos de nodos | 1 | 20 |
| Adición o eliminación de tipos de nodos | No | Sí |
| Redundancia de zona | No | Sí |

## <a name="feature-support"></a>Compatibilidad de características

Las funcionalidades de los clústeres administrados seguirán expandiéndose. Actualmente es compatible con:

* [Implementación de aplicación con plantillas de Resource Manager](how-to-managed-cluster-app-deployment-template.md)
* [Secretos de aplicación](how-to-managed-cluster-application-secrets.md)
* [Actualizaciones automáticas de las imágenes del sistema operativo](how-to-managed-cluster-configuration.md#enable-automatic-os-image-upgrades)
* [Expansión de zonas de disponibilidad](how-to-managed-cluster-availability-zones.md)
* [Cifrado de disco](how-to-enable-managed-cluster-disk-encryption.md) y selección de [tipo de disco administrado](how-to-managed-cluster-managed-disk.md)
* Compatibilidad con identidades administradas para [tipos de nodos](how-to-managed-identity-managed-cluster-virtual-machine-scale-sets.md) de clúster administrados y [autenticación de aplicaciones](how-to-managed-cluster-application-managed-identity.md)
* [Reglas de grupo de seguridad de red y otras opciones de red](how-to-managed-cluster-networking.md)
* [Tipos de nodo solamente sin estado](how-to-managed-cluster-stateless-node-type.md)
* [Extensiones de conjunto de escalado de máquinas virtuales](how-to-managed-cluster-vmss-extension.md) para tipos de nodo

## <a name="next-steps"></a>Pasos siguientes

Para empezar a utilizar los clústeres administrados de Service Fabric, pruebe el manual de inicio rápido:

> [!div class="nextstepaction"]
> [Creación de un clúster administrado de Service Fabric](quickstart-managed-cluster-template.md)

[sf-composition]: ./media/overview-managed-cluster/sfrp-composition-resource.png
[sf-encapsulation]: ./media/overview-managed-cluster/sfrp-encapsulated-resource.png