---
services: storage
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/29/2020
ms.author: normesta
ms.custom: include file
ms.openlocfilehash: 97b47b48a183951002d04cd58b08d5e2b9051eda
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96017697"
---
| Mechanism | Ámbito |Límites | Nivel de permiso admitido |
|---|---|---|---|
| Azure RBAC | Cuentas de almacenamiento, contenedores. <br>Asignaciones de roles de Azure entre recursos en el nivel de suscripción o de grupo de recursos. | 2000 asignaciones de roles de Azure en una suscripción | Roles de Azure (integrados o personalizados) |
| ACL| Directorio, archivo |32 entradas de ACL (realmente 28 entradas de ACL) por archivo y por directorio. Las ACL de acceso y predeterminadas tienen su propio límite de 32 entradas de ACL. |Permiso de ACL|
