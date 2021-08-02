---
title: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 06/07/2021
ms.author: tamram
ms.openlocfilehash: 7f2b15a57b42f87e749060f510dafff732a1d9e0
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111904592"
---
Antes de asignar un rol de Azure RBAC a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible. Los roles de Azure RBAC definidos en un ámbito más amplio los heredan los recursos que están debajo de ellos.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos de cola de Azure, empezando por el ámbito más restringido:

- **Una cola individual.** En este ámbito, se aplica una asignación de roles a los mensajes de la cola, así como las propiedades y los metadatos de la cola.
- **La cuenta de almacenamiento.** En este ámbito, una asignación de roles se aplica a todas las colas y sus mensajes.
- **El grupo de recursos.** En este ámbito, se aplica una asignación de roles a todas las colas de todas las cuentas de almacenamiento del grupo de recursos.
- **La suscripción.** En este ámbito, se aplica una asignación de roles a todas las colas de todas las cuentas de almacenamiento de todos los grupos de recursos de la suscripción.
- **Un grupo de administración.** En este ámbito, se aplica una asignación de roles a todas las colas de todas las cuentas de almacenamiento de todos los grupos de recursos de todas las suscripciones del grupo de administración.

Para obtener más información sobre el ámbito de las asignaciones de roles de RBAC de Azure, consulte [Información sobre el ámbito de RBAC de Azure](../articles/role-based-access-control/scope-overview.md).
