---
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 09/01/2020
ms.author: tomfitz
ms.openlocfilehash: c29abc456c5825a2f22b20a4893bb14eb09058af
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107984277"
---
| Resource | Límite |
| --- | --- |
| Recursos por [grupo de recursos](../articles/azure-resource-manager/management/overview.md#resource-groups) | Los recursos no están limitados por el grupo de recursos. En su lugar, están limitados por el tipo de recurso de un grupo de recursos. Consulte la fila siguiente. |
| Recursos por grupo de recursos, por tipo de recurso |800: algunos tipos de recursos pueden superar el límite de 800. Consulte [Resources not limited to 800 instances per resource group](../articles/azure-resource-manager/management/resources-without-resource-group-limit.md) (Recursos no limitados a 800 instancias por grupo de recursos). |
| Implementaciones por grupo de recursos en el historial de implementaciones |800<sup>1</sup> |
| Recursos por implementación |800 |
| Bloqueos de administración por [ámbito](../articles/azure-resource-manager/management/overview.md#understand-scope) único  |20 |
| Número de etiquetas por recurso o grupo de recursos |50 |
| Longitud de la clave de etiqueta |512 |
| Longitud del valor de la etiqueta |256 |

<sup>1</sup>Las implementaciones se eliminan automáticamente del historial a medida se aproxima al límite. Eliminar una entrada del historial de implementaciones no afecta a los recursos implementados. Para obtener más información, consulte [Eliminaciones automáticas del historial de implementaciones](../articles/azure-resource-manager/templates/deployment-history-deletions.md).

#### <a name="template-limits"></a>Límites de plantilla

| Value | Límite |
| --- | --- |
| Parámetros |256 |
| variables |256 |
| Recursos (incluido el recuento de copia) |800 |
| Salidas |64 |
| Expresión de plantilla |24 576 caracteres |
| Recursos de plantillas exportadas |200 |
| Tamaño de la plantilla |4 MB |
| Tamaño del archivo de parámetros |4 MB |

Puede superar algunos límites de plantilla utilizando una plantilla anidada. Para más información, consulte [Uso de plantillas vinculadas al implementar recursos de Azure](../articles/azure-resource-manager/templates/linked-templates.md). Para reducir el número de parámetros, variables o salidas, puede combinar varios valores en un objeto. Para más información, consulte [Objetos como parámetros](/azure/architecture/guide/azure-resource-manager/advanced-templates/objects-as-parameters).

Es posible que reciba un error con una plantilla o un archivo de parámetros de menos de 4 MB si el tamaño total de la solicitud es demasiado grande. Para más información sobre cómo simplificar la plantilla para evitar una solicitud de gran tamaño, consulte [Resolución de errores de tamaño de trabajo superado](../articles/azure-resource-manager/templates/error-job-size-exceeded.md).