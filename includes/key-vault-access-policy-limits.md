---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 08/27/2020
ms.author: msmbaldwin
ms.openlocfilehash: 20ef9a4f30aafee562096e584c4b80fef236b062
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "108749739"
---
Key Vault admite hasta 1024 entradas de directiva de acceso, donde cada entrada concede un conjunto de permisos distinto a una entidad de servicio particular: Debido a esta limitación, se recomienda asignar directivas de acceso a grupos de usuarios, siempre que sea posible, en lugar de a usuarios individuales. El uso de grupos facilita la administración de permisos para varias personas de la organización. Para más información, consulte [Administración del acceso a recursos y aplicaciones con grupos en Azure Active Directory](../articles/active-directory/fundamentals/active-directory-manage-groups.md).

Para la información completa sobre el control de acceso a Key Vault, consulte [Seguridad de Azure Key Vault: Administración de identidades](../articles/key-vault/general/security-features.md#identity-management).