---
title: SKU para SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: SKU para SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
keywords: HLI, HANA, SKUs, S896, S224, S448, S672, Optane, SAP
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 4/16/2021
ms.author: juergent
ms.custom: H1Hack27Feb2017, references_regions
ms.openlocfilehash: 3ecbbe4d477f3e6c3c6606528c51b934b6cf534a
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718744"
---
# <a name="available-skus-for-hana-large-instances"></a>SKU disponibles para instancias grandes de HANA

El servicio BareMetal Infrastructure (certificado para cargas de trabajo de SAP HANA) basado en Rev 4.2* está disponible en las siguientes regiones:
- Oeste de Europa
- Norte de Europa
- Centro-oeste de Alemania con compatibilidad con zonas de disponibilidad
- Este de EE. UU. con compatibilidad con zonas de disponibilidad
- Este de EE. UU. 2
- Centro-sur de EE. UU.
- Oeste de EE. UU. 2 con compatibilidad con zonas de disponibilidad

El servicio BareMetal Infrastructure (certificado para cargas de trabajo de SAP HANA) basado en Rev 3* tiene disponibilidad limitada en las siguientes regiones:
- Oeste de EE. UU.
- Este de EE. UU. 
- Este de Australia 
- Sudeste de Australia
- Japón Oriental


A continuación encontrará una lista de las instancias de Azure Large disponibles.

> [!IMPORTANT]
> Tenga en cuenta la primera columna que representa el estado de la certificación de HANA para cada uno de los tipos de instancias grandes de la lista. La columna se debe correlacionar con el [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) para los SKU de Azure que empiezan por la letra **S**.



| Certificación de SAP HANA | Modelo | Memoria total | Memoria DRAM | Memoria Optane | Storage | Disponibilidad |
| --- | --- | --- | --- | --- | --- | --- |
| SÍ <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2185), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2265) | SAP HANA en Azure S96<br /> – 2 procesadores Intel® Xeon® E7-8890 v4 <br /> 48 núcleos de CPU y 96 subprocesos de CPU |  768 GB | 768 GB | --- | 3,0 TB | Disponible |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2186), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2269) | SAP HANA en Azure S224<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU |  3,0 TB | 3,0 TB | --- | 6,3 TB | Disponible |
| SÍ <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2297) | SAP HANA en Azure S224m<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU |  6,0 TB | 6,0 TB | --- | 10,5 TB | Disponible |
| SÍ <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2381) | SAP HANA en Azure S224om<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU | 6,0 TB |  3,0 TB |  3,0 TB | 10,5 TB | Disponible |
| No | SAP HANA en Azure S224oo<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU | 4,5 TB |  1,5 TB |  3,0 TB | 8,4 TB | Disponible |
| No | SAP HANA en Azure S224ooo<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU | 7,5 TB |  1,5 TB |  6,0 TB | 12,7 TB | Disponible |
| No | SAP HANA en Azure S224oom<br /> – 4 procesadores Intel® Xeon® Platinum 8276 <br /> 112 núcleos de CPU y 224 subprocesos de CPU | 9,0 TB |  3,0 TB |  6,0 TB | 14,8 TB | Disponible |
| SÍ <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1983), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2268) | SAP HANA en Azure S384<br /> – 8 procesadores Intel® Xeon® E7-8890 v4<br /> 192 núcleos de CPU y 384 subprocesos de CPU |  4,0 TB | 4,0 TB | --- | 16 TB | Disponible |
| SÍ <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2080) | SAP HANA en Azure S384m<br /> – 8 procesadores Intel® Xeon® E7-8890 v4<br /> 192 núcleos de CPU y 384 subprocesos de CPU |  6,0 TB | 6,0 TB | --- | 18 TB |  Disponible  |
| SÍ <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1984), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2267) | SAP HANA en Azure S384xm<br /> – 8 procesadores Intel® Xeon® E7-8890 v4<br /> 192 núcleos de CPU y 384 subprocesos de CPU |  8,0 TB | 8,0 TB | --- | 22 TB | Disponible |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2411), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2378) | SAP HANA en Azure S448<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 6,0 TB |  6,0 TB |  --- | 10,5 TB | Disponible (solo Rev 4.2) |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2410), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2377) | SAP HANA en Azure S448m<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 12,0 TB |  12,0 TB |  --- | 18,9 TB | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S448oo<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 9,0 TB |  3,0 TB |  6,0 TB | 14,8 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S448om<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 12,0 TB |  6,0 TB |  6,0 TB | 18,9 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S448ooo<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 15,0 TB |  3,0 TB |  12,0 TB | 23,2 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S448oom<br /> – 8 procesadores Intel® Xeon® Platinum 8276 <br /> 224 núcleos de CPU y 448 subprocesos de CPU | 18,0 TB |  6,0 TB |  12,0 TB | 27,4 TB  | Disponible (solo Rev 4.2) |
| SÍ <br /> [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2049) | SAP HANA en Azure S576m<br /> – 12 procesadores Intel® Xeon® E7-8890 v4<br /> 288 núcleos de CPU y 576 subprocesos de CPU |  12,0 TB | 12,0 TB | --- | 28 TB | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S576xm<br /> – 12 procesadores Intel® Xeon® E7-8890 v4<br /> 288 núcleos de CPU y 576 subprocesos de CPU |  18,0 TB | 18.0 | --- |  41 TB | Disponible |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2409), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2380) | SAP HANA en Azure S672<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 9,0 TB |  9,0 TB |  --- | 14,7 TB | Disponible (solo Rev 4.2) |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2408), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2379) | SAP HANA en Azure S672m<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 18,0 TB |  18,0 TB |  --- | 27,4 TB | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S672oo<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 13,5 TB |  4,5 TB |  9,0 TB | 21,1 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S672om<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 18,0 TB |  9,0 TB |  9,0 TB | 27,4 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S672ooo<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 22,5 TB |  4,5 TB |  18,0 TB | 33,7 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S672oom<br /> – 12 procesadores Intel® Xeon® Platinum 8276 <br /> 336 núcleos de CPU y 672 subprocesos de CPU | 27,0 TB |  9,0 TB |  18,0 TB | 40,0 TB  | Disponible (solo Rev 4.2) |
| SÍ <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1985) | SAP HANA en Azure S768m<br /> – 16 procesadores Intel® Xeon® E7-8890 v4<br /> 384 núcleos de CPU y 768 subprocesos de CPU |  16,0 TB | 16,0 TB | -- | 36 TB | Disponible |
| No | SAP HANA en Azure S768xm<br /> – 16 procesadores Intel® Xeon® E7-8890 v4<br /> 384 núcleos de CPU y 768 subprocesos de CPU |  24,0 TB | 24,0 TB | --- | 56 TB | Disponible |
|  SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2407), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2376)  | SAP HANA en Azure S896<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 12,0 TB |  12,0 TB |  --- | 18,9 TB | Disponible (solo Rev 4.2) |
| SÍ <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2406), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2328) | SAP HANA en Azure S896m<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 24,0 TB | 24,0 TB | -- | 35,8 TB | Disponible |
| No | SAP HANA en Azure S896oo<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 18,0 TB |  6,0 TB |  12,0 TB | 27,4 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S896om<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 24,0 TB |  12,0 TB |  12,0 TB | 35,8 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S896ooo<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 30,0 TB |  6,0 TB |  24,0 TB | 44,3 TB  | Disponible (solo Rev 4.2) |
| No | SAP HANA en Azure S896oom<br /> – 16 procesadores Intel® Xeon® Platinum 8276 <br /> 448 núcleos de CPU y 896 subprocesos de CPU | 36,0 TB |  12,0 TB |  24,0 TB | 52,7 TB  | Disponible (solo Rev 4.2) |
| SÍ <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1986) | SAP HANA en Azure S960m<br /> – 20 procesadores Intel® Xeon® E7-8890 v4<br /> 480 núcleos de CPU y 960 subprocesos de CPU |  20,0 TB | 20,0 TB | -- | 46 TB | Disponible (solo Rev 4.2) |


- Núcleos de CPU = suma de núcleos de CPU sin Hyper-Threading de la suma de procesadores de la unidad del servidor.
- Subprocesos de CPU = suma de suprocesos de proceso proporcionado por núcleos de CPU con Hyper-Threading de la suma de procesadores de la unidad del servidor. La mayoría de las unidades se configuran de forma predeterminada para usar la tecnología Hyper-Threading.
- Según las recomendaciones del proveedor, S768m, S768xm y S960m no están configuradas para ejecutar SAP HANA con la tecnología Hyper-Threading.


> [!IMPORTANT]
> Las SKU siguientes, aunque todavía se admiten, ya no se pueden adquirir: S72, S72m, S144, S144m, S192 y S192m 

Las configuraciones específicas que se elijan dependen de la carga de trabajo, los recursos de CPU y la memoria que desee. La carga de trabajo de OLTP puede usar las SKU que estén optimizadas para la carga de trabajo OLAP. 

Dos clases diferentes de hardware dividen las SKU en:

- Las series S72, S72m, S96, S144, S144m, S192, S192m, S192xm, S224, S224m, S224oo, S224om, S224ooo y S224oom se conocen como SKU de la "Clase tipo I".
- Todas las demás SKU se conocen como "Clase tipo II" de SKU.
- Si está interesado en las SKU que todavía no aparecen en el directorio de hardware de SAP, póngase en contacto con su equipo de cuenta de Microsoft para más información. 


Una demarcación completa de HANA (instancias grandes) no está asignada exclusivamente para que la use un único cliente. Esto se aplica también a los bastidores de recursos de procesos y almacenamiento conectados mediante un tejido de red implementado en Azure. La infraestructura de HANA (instancias grandes), como Azure, implementa &quot;inquilinos&quot; de clientes diferentes que están aislados entre sí en los tres niveles siguientes:

- **Network** (Red): aislamiento a través de redes virtuales dentro de la demarcación de HANA (instancias grandes).
- **Almacenamiento**: aislamiento a través de máquinas virtuales de almacenamiento que tienen volúmenes de almacenamiento asignados y aíslan los volúmenes de almacenamiento entre inquilinos.
- **Proceso**: asignación dedicada de las unidades de servidor a un único inquilino. Sin particiones de hardware ni de software de las unidades de servidor. Sin uso compartido de una sola unidad de host o servidor entre los inquilinos. 

Las implementaciones de unidades de HANA (instancias grandes) entre varios inquilinos no son visibles entre sí. Las unidades de HANA (instancias grandes) implementadas en diferentes inquilinos tampoco se pueden comunicar directamente entre sí en el nivel de la demarcación de HANA (instancias grandes). Solo las unidades de HANA (instancias grandes) dentro de un mismo inquilino se pueden comunicar entre sí en el nivel de la demarcación de HANA (instancias grandes).

Se asigna un inquilino implementado en la demarcación de instancias grandes a una suscripción de Azure con fines de facturación. Para una red, se puede acceder desde redes virtuales de otras suscripciones de Azure dentro de la misma inscripción de Azure. Si implementa con otra suscripción de Azure en la misma región de Azure, también puede elegir pedir un inquilino de instancia grande de HANA independiente.

Existen diferencias notables entre ejecutar SAP HANA en HANA (instancias grandes) y ejecutar SAP HANA en máquinas virtuales implementadas en Azure:

- No hay ningún nivel de virtualización para SAP HANA en Azure (Instancias grandes). El rendimiento proviene del hardware de reconstrucción completa subyacente.
- A diferencia de Azure, el servidor de SAP HANA en Azure (Instancias grandes) está dedicado a un cliente específico. No hay ninguna posibilidad de que un host o unidad de servidor tenga particiones de hardware o software. Como resultado, se utiliza una unidad de HANA (instancias grandes) tal cual se asignó como un conjunto a un inquilino y después al usuario. Tras un reinicio o un apagado del servidor, no se implementa el sistema operativo ni SAP HANA en otro servidor de manera automática. (Para las SKU de clase de tipo I, la única excepción es si un servidor se encuentra en problemas y se tiene que volver a implementar en otro servidor).
- A diferencia de Azure, donde los tipos de procesador de host se seleccionan para lograr una relación precio/rendimiento óptima, los tipos de procesador elegidos para SAP HANA en Azure (instancias grandes) son los de mayor rendimiento en la línea de procesadores Intel E7v3 y E7v4.

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Dimensionamiento de HLI](hana-sizing.md).
