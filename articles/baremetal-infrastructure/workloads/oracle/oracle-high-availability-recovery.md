---
title: Recuperación de la base de datos de Oracle en la Infraestructura de Azure sin sistema operativo
description: Obtenga información sobre cómo recuperar la base de datos de Oracle en la Infraestructura de Azure sin sistema operativo.
ms.topic: reference
ms.subservice: baremetal-oracle
ms.date: 04/15/2021
ms.openlocfilehash: c93159476403ac01ea9622b438fb36e5c423ecb4
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578488"
---
# <a name="recover-your-oracle-database-on-azure-baremetal-infrastructure"></a>Recuperación de la base de datos de Oracle en la Infraestructura de Azure sin sistema operativo

Aunque ninguna sola tecnología protege frente a todos los escenarios de error, combinar características permite a los administradores de bases de datos recuperar una base de datos en prácticamente cualquier situación.

## <a name="causes-of-database-failure"></a>Causas de los errores de una base de datos

Los errores de una base de datos pueden producirse por muchas razones, pero normalmente se incluyen en varias categorías:

- Errores de manipulación de datos
- Pérdida de registros de rehacer en línea
- Pérdida de archivos de control de base de datos
- Pérdida de archivos de datos de la base de datos
- Datos físicos dañados

## <a name="choose-your-method-of-recovery"></a>Elección de un método de recuperación

El tipo de recuperación depende del tipo de error. Supongamos que se descarta un objeto o que los datos se modifican incorrectamente. La solución más rápida normalmente consiste en efectuar una operación de base de datos de flashback. En otros casos, la recuperación a través de una instantánea de Azure NetApp Files puede proporcionar la recuperación que desee. El árbol de decisión de la ilustración siguiente muestra escenarios comunes de error y recuperación si se aplican todas las opciones de protección de datos descritas anteriormente.

:::image type="content" source="media/oracle-high-availability/db-recovery-decision-tree.png" alt-text="Diagrama del árbol de decisión de recuperación de la base de datos." lightbox="media/oracle-high-availability/db-recovery-decision-tree.png":::

Tenga en cuenta que este árbol de decisión de ejemplo solo se ve desde la perspectiva de un administrador de base de datos. Cada implementación puede tener requisitos diferentes, lo que podría cambiar el orden de las opciones. Por ejemplo, cambiar un rol de base de datos a otra región mediante Data Guard puede tener un efecto negativo en el rendimiento de la aplicación. El objetivo de tiempo de recuperación (RTO) del método de recuperación de instantáneas podría ser inferior. Para asegurarse de que se cumplan los requisitos de RTO/RPO, se recomienda llevar a cabo estas operaciones y documentar los procedimientos para ejecutarlas cuando sea necesario.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la infraestructura sin sistema operativo:

- [¿Qué es la infraestructura sin sistema operativo en Azure?](../../concepts-baremetal-infrastructure-overview.md)
- [Conexión a instancias de la infraestructura sin sistema operativo en Azure](../../connect-baremetal-infrastructure.md)
