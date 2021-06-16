---
title: Alta disponibilidad y recuperación ante desastres para Oracle en BareMetal
description: Obtenga información sobre la alta disponibilidad y la recuperación ante desastres para Oracle en Azure BareMetal Infrastructure.
ms.topic: how-to
ms.subservice: baremetal-oracle
ms.date: 04/15/2021
ms.openlocfilehash: 3b23774604b5126321d85892647e73118e08260f
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110575621"
---
# <a name="high-availability-and-disaster-recovery-for-oracle-on-baremetal"></a>Alta disponibilidad y recuperación ante desastres para Oracle en BareMetal

En este artículo, veremos los conceptos básicos de la alta disponibilidad y recuperación ante desastres. A continuación, presentaremos cómo puede lograr una alta disponibilidad y recuperación ante desastres en un entorno de Oracle en BareMetal Infrastructure.

## <a name="high-availability-vs-disaster-recovery"></a>Alta disponibilidad frente a recuperación ante desastres

Tanto la alta disponibilidad como la recuperación ante desastres proporcionan cobertura, pero frente a distintos tipos de errores. Usan distintas características y opciones de la Oracle Database.

La alta disponibilidad permite que un sistema supere varios errores sin afectar a la experiencia de usuario de la aplicación. Entre las características comunes de un sistema de alta disponibilidad se incluyen:

- Hardware redundante que no tiene un único punto de error.
- Recuperación automática de errores no críticos, como unidades de disco con errores o cables de red defectuosos.
- La capacidad de realizar cambios de hardware y software sin ningún efecto perceptible en el procesamiento.
- Cumplimiento o superación de los objetivos de tiempo de recuperación (RTO) y los objetivos de punto de recuperación (RPO).

La característica más común de Oracle que se usa para la alta disponibilidad es [Oracle Real Application Clusters (RAC)](https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/introduction-to-oracle-rac.html#GUID-5A1B02A2-A327-42DD-A1AD-20610B2A9D92).

La recuperación ante desastres le protege frente a errores localizados irrecuperables que dañarían la estrategia principal de alta disponibilidad. En el ecosistema de Oracle, se proporciona a través de la replicación de base de datos, también conocida como [Oracle Data Guard.](https://docs.oracle.com/en/database/oracle/oracle-database/19/sbydb/preface.html#GUID-B6209E95-9DA8-4D37-9BAD-3F000C7E3590)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las características de alta disponibilidad para Oracle:

> [!div class="nextstepaction"]
> [Características de alta disponibilidad para Oracle en BareMetal Infrastructure](high-availability-features.md)
