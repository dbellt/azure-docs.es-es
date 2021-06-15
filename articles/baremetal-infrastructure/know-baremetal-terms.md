---
title: Conozca los términos y condiciones de Azure BareMetal Infrastructure
description: Conozca los términos y condiciones de Azure BareMetal Infrastructure.
ms.topic: conceptual
ms.date: 04/06/2021
ms.openlocfilehash: 91270b2c16a1b642b697fd4ffb6a9c635608110e
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576321"
---
# <a name="know-the-terms-for-baremetal-infrastructure"></a>Conozca los términos y condiciones de BareMetal Infrastructure

En este artículo, se tratarán algunos términos importantes relacionados con BareMetal Infrastructure.

- **Revisión**: Hay dos revisiones de demarcación diferentes para las demarcaciones de BareMetal Infrastructure (instancias grandes de HANA). Estas difieren en la arquitectura y la proximidad de los hosts de Azure Virtual Machine:
    - "Revisión 3" (Rev 3): diseño original implementado a mediados de 2016.
    - "Revisión 4.2" (Rev 4.2): es un nuevo diseño que proporciona una mayor proximidad a los hosts de máquina virtual de Azure, con una latencia de red muy baja entre las VM de Azure y las instancias grandes de HANA. Los recursos de Azure Portal se conocen como "BareMetal Infrastructure" y los clientes pueden acceder a sus recursos como instancias de BareMetal desde Azure Portal.

- **Demarcación**: define el tamaño de la implementación interna de Microsoft de instancias de BareMetal. Para poder implementar instancias, se debe implementar una demarcación de instancias de BareMetal que conste de bastidores de proceso, red y almacenamiento en una ubicación del centro de datos. Este tipo de implementación se denomina demarcación de instancias de BareMetal.

- **Inquilino**: un cliente que implementa una demarcación de instancias de BareMetal se aísla como *inquilino.* Un inquilino se aísla en el nivel de redes, almacenamiento y proceso de otros inquilinos. Las unidades de almacenamiento y proceso asignadas a los distintos inquilinos no pueden verse ni comunicarse entre sí en el nivel de la demarcación de instancias de BareMetal. Un cliente puede elegir que las implementaciones se realicen en diferentes inquilinos. Aún así, no hay ninguna comunicación entre los inquilinos en el nivel de la demarcación de instancias de BareMetal.

## <a name="next-steps"></a>Pasos siguientes

Ahora que se le ha presentado terminología importante de BareMetal Infrastructure, es posible que desee aprender lo siguiente:
- Más detalles de [BareMetal Infrastructure](concepts-baremetal-infrastructure-overview.md).
- Cómo [conectarse a instancias de BareMetal Infrastructure en Azure](connect-baremetal-infrastructure.md).

