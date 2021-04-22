---
title: Conozca los términos y condiciones de Azure BareMetal Infrastructure
description: Conozca los términos y condiciones de Azure BareMetal Infrastructure.
ms.topic: conceptual
ms.subservice: workloads
ms.date: 04/06/2021
ms.openlocfilehash: 53a601cc4556198479d8ca5d7495942d4dc2762c
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580135"
---
# <a name="know-the-terms-for-baremetal-infrastructure"></a>Conozca los términos y condiciones de BareMetal Infrastructure

En este artículo, se tratarán algunos términos importantes relacionados con BareMetal Infrastructure.

- **Revisión**: hay una revisión de marca original conocida como Revisión 3 (Rev 3) y dos revisiones de marca adicionales para los sellos de instancias de BareMetal. Cada sello difiere en la arquitectura y la proximidad de los hosts de máquina virtual de Azure:
    - **Revisión 4** (Rev 4): un nuevo diseño que proporciona una mayor proximidad de los hosts de máquina virtual (VM) de Azure y reduce la latencia entre las VM de Azure y las instancias de SAP HANA. 
    - **Revisión 4.2** (Rev 4.2): la infraestructura BareMetal Infrastructure cuyo nombre se ha cambiado más reciente que usa la arquitectura Rev 4 existente. Rev 4 proporciona mayor proximidad a los hosts de máquina virtual (VM) de Azure. Presenta mejoras significativas en la latencia de red entre las máquinas virtuales de Azure y las instancias de BareMetal implementadas en los sellos o filas de Rev 4. Puede tener acceso a sus instancias de BareMetal y administrarlas en Azure Portal.    

- **Demarcación**: define el tamaño de la implementación interna de Microsoft de instancias de BareMetal. Para poder implementar instancias, se debe implementar una demarcación de instancias de BareMetal que conste de bastidores de proceso, red y almacenamiento en una ubicación del centro de datos. Este tipo de implementación se denomina demarcación de instancias de BareMetal.

- **Inquilino**: un cliente que implementa una demarcación de instancias de BareMetal se aísla como *inquilino.* Un inquilino se aísla en el nivel de redes, almacenamiento y proceso de otros inquilinos. Las unidades de almacenamiento y proceso asignadas a los distintos inquilinos no pueden verse ni comunicarse entre sí en el nivel de la demarcación de instancias de BareMetal. Un cliente puede elegir que las implementaciones se realicen en diferentes inquilinos. Aún así, no hay ninguna comunicación entre los inquilinos en el nivel de la demarcación de instancias de BareMetal.

## <a name="next-steps"></a>Pasos siguientes
Ahora que se le ha presentado terminología importante de BareMetal Infrastructure, es posible que desee aprender lo siguiente:
- Más detalles de [BareMetal Infrastructure](concepts-baremetal-infrastructure-overview.md).
- Cómo [conectarse a instancias de BareMetal Infrastructure en Azure](connect-baremetal-infrastructure.md).

