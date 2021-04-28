---
title: SKU de BareMetal para cargas de trabajo de Oracle
description: Obtenga información acerca de las SKU para las cargas de trabajo de Oracle BareMetal Infrastructure.
ms.topic: reference
ms.subservice: workloads
ms.date: 04/15/2021
ms.openlocfilehash: 42ff26b9ea9bcc022f1ddbf3dddcb041b2cea3a2
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588860"
---
# <a name="baremetal-skus-for-oracle-workloads"></a>SKU de BareMetal para cargas de trabajo de Oracle

En este artículo, examinaremos las SKU de BareMetal Infrastructure especializadas para cargas de trabajo de Oracle.

Las SKU de BareMetal Infrastructure para Oracle van de dos sockets hasta cuatro sockets. También puede elegir entre varios núcleos de CPU y tamaños de memoria para cumplir los requisitos de la carga de trabajo. La siguiente es una tabla que resume las características de las SKU disponibles.
 
| **Hardware con** **certificación de Oracle** | **Modelo** | **Memoria total** | **Storage** | **Disponibilidad** |
| --- | --- | --- | --- | --- |
| SÍ | SAP HANA en Azure S32m: 2 procesadores Intel® Xeon® I6234 de 16 núcleos de CPU y 32 subprocesos de CPU | 1,5 TB | --- | Disponible |
| SÍ | SAP HANA en Azure S64m: 4 procesadores Intel® Xeon® I6234 de 32 núcleos de CPU y 64 subprocesos de CPU | 3,0 TB | --- | Disponible |
| SÍ | SAP HANA en Azure S96: 2 procesadores Intel® Xeon® E7-8890 v4 de 48 núcleos de CPU y 96 subprocesos de CPU | 768 GB | 3,0 TB | Disponible |
| SÍ | SAP HANA en Azure S224: 4 procesadores Intel® Xeon® Platinum 8276 de 112 núcleos de CPU y 224 subprocesos de CPU | 3,0 TB | 6,3 TB | Disponible |
| SÍ | SAP HANA en Azure S224m: 4 procesadores Intel® Xeon® Platinum 8276 de 112 núcleos de CPU y 224 subprocesos de CPU | 6,0 TB | 10,5 TB | Disponible |

- Núcleos de CPU = suma de los núcleos de CPU que no tienen Hyper-Threading (el número total de procesadores físicos) de la unidad de servidor. 
- Subprocesos de CPU = suma de subprocesos de proceso que proporcionan los núcleos de CPU con Hyper-Threading (el número total de procesadores lógicos) de la unidad de servidor. La mayoría de las unidades se configuran de forma predeterminada para usar la tecnología Hyper-Threading.
- Los servidores están dedicados a los clientes.
- El cliente tiene acceso raíz (sin hipervisor).
- Los servidores no están directamente en las redes virtuales de Azure.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca del almacenamiento que ofrece BareMetal Infrastructure para Oracle.

> [!div class="nextstepaction"]
> [Almacenamiento en BareMetal para cargas de trabajo de Oracle](oracle-baremetal-storage.md)
