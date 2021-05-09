---
title: Elección de los tamaños y las imágenes de máquina virtual de los grupos
description: Cómo elegir uno de los tamaños de máquina virtual y una de las versiones de sistema operativo disponibles para los nodos de proceso en grupos de Azure Batch
ms.topic: conceptual
ms.date: 03/18/2021
ms.custom: seodec18
ms.openlocfilehash: 6de7decbf40eede74dd7b92f9f1139e1b31450c8
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126262"
---
# <a name="choose-a-vm-size-and-image-for-compute-nodes-in-an-azure-batch-pool"></a>Selección de un tamaño y una imagen de máquina virtual para nodos de proceso en un grupo de Azure Batch

Cuando seleccione un tamaño de nodo para un grupo de Azure Batch, puede elegir entre casi todas las familias y tamaños de máquinas virtuales disponibles en Azure. Azure ofrece una variedad de tamaños para máquinas virtuales Windows y Linux para diferentes cargas de trabajo.

## <a name="supported-vm-series-and-sizes"></a>Series y tamaños de maquina virtual compatibles

### <a name="pools-in-virtual-machine-configuration"></a>Grupos en la configuración de máquina virtual

Los grupos de Batch en la configuración de máquina virtual son compatibles con casi todos los tamaños de [máquina virtual](../virtual-machines/sizes.md). Consulte la tabla siguiente para más información sobre los tamaños admitidos y las restricciones.

| Series de máquinas virtuales  | Tamaños admitidos |
|------------|---------|
| A básico | Todos los tamaños *excepto* Basic_A0 (A0) |
| A | Todos los tamaños, *excepto* Standard_A0, Standard_A8, Standard_A9, Standard_A10, Standard_A11 |
| Av2 | Todos los tamaños |
| B | No compatible |
| DCsv2 | Todos los tamaños |
| Dv2, DSv2 | Todos los tamaños |
| Dv3, Dsv3 | Todos los tamaños |
| Dav4, Dasv4 | Todos los tamaños |
| Ddv4, Ddsv4 |  Todos los tamaños |
| Dv4, Dsv4 | No compatible |
| Ev3, Esv3 | Todos los tamaños, excepto E64is_v3 |
| Eav4, Easv4 | Todos los tamaños |
| Edv4, Edsv4 |  Todos los tamaños |
| Ev4, Esv4 | No compatible |
| F, Fs | Todos los tamaños |
| Fsv2 | Todos los tamaños |
| G, Gs | Todos los tamaños |
| H | Todos los tamaños |
| HB | Todos los tamaños |
| HBv2 | Todos los tamaños |
| HBv3 | Standard_HB120rs_v3 (otros tamaños todavía no están disponibles) |
| HC | Todos los tamaños |
| LS | Todos los tamaños |
| Lsv2 | Todos los tamaños |
| M | Todos los tamaños |
| Mv2<sup>1</sup> | Todos los tamaños |
| NC | Todos los tamaños |
| NCv2 | Todos los tamaños |
| NCv3 | Todos los tamaños |
| NCasT4_v3 | Todos los tamaños |
| ND | Todos los tamaños |
| NDv2 | Ninguno: no disponible todavía |
| NV | Todos los tamaños |
| NVv3 | Todos los tamaños |
| NVv4 | Todos los tamaños |
| SAP HANA | No compatible |

<sup>1</sup> Estas series de máquina virtual solo se pueden usar con imágenes de máquina virtual de segunda generación.

### <a name="using-generation-2-vm-images"></a>Uso de imágenes de máquina virtual de segunda generación

Algunas series de máquina virtual, como [Mv2](../virtual-machines/mv2-series.md), solo se pueden usar con [imágenes de máquina virtual de segunda generación](../virtual-machines/generation-2.md). Las imágenes de máquina virtual de segunda generación se especifican como cualquier imagen de máquina virtual, con la propiedad "sku" de la configuración de ["imageReference"](/rest/api/batchservice/pool/add#imagereference); las cadenas "sku" tienen un sufijo del tipo "-g2" o "-gen2". Para obtener una lista de imágenes de máquina virtual admitidas por Batch, incluidas las imágenes de segunda generación, use la API ["list supported images"](/rest/api/batchservice/account/listsupportedimages), [PowerShell](/powershell/module/az.batch/get-azbatchsupportedimage) o la [CLI de Azure](/cli/azure/batch/pool/supported-images).

### <a name="pools-in-cloud-services-configuration"></a>Grupos de la configuración de Cloud Services

> [!WARNING]
> Los grupos de configuración de Cloud Services están [en desuso](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/). En su lugar, utilice los grupos de configuración de máquina virtual.

Los grupos de Batch en la configuración de Cloud Services son compatibles con todos los [tamaños de máquina virtual para Cloud Services](../cloud-services/cloud-services-sizes-specs.md)**excepto** en los siguientes casos:

| Series de máquinas virtuales  | Tamaños no compatibles |
|------------|-------------------|
| Serie A   | Extra pequeño       |
| Serie Av2 | Standard_A1_v2, Standard_A2_v2, Standard_A2m_v2 |

## <a name="size-considerations"></a>Consideraciones de tamaño

- **Requisitos de aplicación**: tenga en cuenta las características y los requisitos de la aplicación que se va a ejecutar en los nodos. Aspectos tales como si la aplicación es multiproceso y cuánta memoria consume pueden ayudar a determinar el tamaño de nodo más adecuado y rentable. Para varias instancias de [cargas de trabajo MPI](batch-mpi.md) o aplicaciones CUDA, considere la posibilidad de tamaños de máquina virtual especializados [HPC](../virtual-machines/sizes-hpc.md) o [habilitado GPU](../virtual-machines/sizes-gpu.md), respectivamente. Para obtener más información, consulte [Uso de instancias compatibles con RDMA o habilitadas para GPU en grupos de Batch](batch-pool-compute-intensive-sizes.md).

- **Tareas por nodo**: normalmente, se selecciona un tamaño de nodo bajo el supuesto de que no se ejecutará más que una sola tarea a la vez en un nodo. No obstante, puede tener ventajas [ejecutar en paralelo](batch-parallel-node-tasks.md) varias tareas y, por tanto, varias instancias de la aplicación, en varios nodos de proceso durante la ejecución del trabajo. En este caso, es habitual elegir un tamaño de nodo de varios núcleos para acomodar el aumento de la demanda por la ejecución de tareas en paralelo.

- **Niveles de carga para diferentes tareas**: todos los nodos en un grupo tienen el mismo tamaño. Si va a ejecutar aplicaciones con requisitos del sistema o niveles de carga diferentes, es recomendable usar grupos separados.

- **Disponibilidad por regiones**: una serie o tamaño de máquina virtual, podría no estar disponible en las regiones en las que cree las cuentas de Batch. Para comprobar que un tamaño está disponible, vea [Productos disponibles por región](https://azure.microsoft.com/regions/services/).

- **Cuotas**: la [cuota de núcleos](batch-quota-limit.md#resource-quotas) en su cuenta de Batch puede limitar el número de nodos de un tamaño específico que se puede agregar a un grupo de Batch. Cuando sea necesario, puede [solicitar un aumento de la cuota](batch-quota-limit.md#increase-a-quota).

- **Configuración de grupo**: por lo general, tiene más opciones de tamaño de máquina virtual cuando crea un grupo en la configuración de máquina virtual, en comparación con la configuración de Cloud Services.

## <a name="supported-vm-images"></a>Imágenes de máquina virtual admitidas

Use una de las siguientes API para devolver una lista de imágenes de máquina virtual Windows y Linux que admite actualmente el servicio Batch, incluidos los identificadores de SKU de agente de nodo de cada imagen:

- API REST del servicio Batch: [Enumerar imágenes compatibles](/rest/api/batchservice/account/listsupportedimages)
- PowerShell: [Get-AzBatchSupportedImage](/powershell/module/az.batch/get-azbatchsupportedimage)
- CLI de Azure: [az batch pool supported-images](/cli/azure/batch/pool/supported-images)

Se recomienda evitar las imágenes con fechas de final del ciclo de vida (EOL) inminentes para el soporte técnico de Batch. Estas fechas se pueden detectar con la [`ListSupportedImages` API](/rest/api/batchservice/account/listsupportedimages), [PowerShell](/powershell/module/az.batch/get-azbatchsupportedimage) o la [CLI de Azure](/cli/azure/batch/pool/supported-images). Consulte [Guía de procedimientos recomendados de Batch](best-practices.md) para más información sobre la selección de imágenes de máquina virtual del grupo de Batch.

## <a name="next-steps"></a>Pasos siguientes

- Conozca el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- Para más información acerca del uso de tamaños de máquinas virtuales de proceso intensivo, consulte [Uso de instancias compatibles con RDMA o habilitadas para GPU en grupos de Batch](batch-pool-compute-intensive-sizes.md).