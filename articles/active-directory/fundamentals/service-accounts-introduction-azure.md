---
title: Introducción a la protección de las cuentas de servicio de Azure Active Directory
description: Explicación de los tipos de cuentas de servicio disponibles en Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 3/1/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: d657f1df14b083631227cb7c19f64b65be8801d0
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107010472"
---
# <a name="introduction-to-securing-azure-service-accounts"></a>Introducción a la protección de cuentas de servicio de Azure

Hay tres tipos de cuentas de servicio nativas para Azure Active Directory: identidades administradas, entidades de servicio y cuentas de servicio basadas en usuario. Las cuentas de servicio son un tipo especial de cuenta que está pensado para representar una entidad no humana, como una aplicación, una API u otro servicio. Estas entidades operan en el contexto de seguridad que proporciona la cuenta de servicio. 

## <a name="types-of-azure-active-directory-service-accounts"></a>Tipos de cuentas de servicio de Azure Active Directory

En el caso de los servicios hospedados en Azure, se recomienda usar una identidad administrada, si es posible, y una entidad de servicio en caso contrario. Las identidades administradas no se pueden usar para los servicios hospedados fuera de Azure. En ese caso, se recomienda una entidad de servicio. Si puede usar una identidad administrada o una entidad de servicio, hágalo. Se recomienda no utilizar una cuenta de usuario de Azure Active Directory como cuenta de servicio. Consulte la tabla siguiente como resumen.
 

| Hospedaje de servicio| Identidad administrada| Entidad de servicio| Cuenta de usuario de Azure |
| - | - | - | - |
|El servicio se hospeda en Azure.| Sí. <br>Recomendado si el servicio <br>admite una identidad administrada.| Sí.| No se recomienda. |
| El servicio no está hospedado en Azure.| No| Sí. Se recomienda su uso.| No se recomienda. |
| El servicio es multiinquilino.| No| Sí. Se recomienda su uso.| No. |


## <a name="managed-identities"></a>Identidades administradas

Las identidades administradas son identidades de Azure Active Directory (Azure AD) seguras creadas para proporcionar identidades para los recursos de Azure. Hay [dos tipos de identidades administradas](../managed-identities-azure-resources/overview.md#managed-identity-types): 
 
* Las identidades administradas asignadas por el sistema se pueden asignar directamente a una instancia de un servicio. 

* Las identidades administradas asignadas por el usuario se pueden crear como un recurso independiente. 

Para más información, consulte [Protección de identidades administradas](service-accounts-managed-identities.md). Para obtener más información sobre las identidades administradas, consulte [¿Qué son las identidades administradas para recursos de Azure?](../managed-identities-azure-resources/overview.md)

## <a name="service-principals"></a>Entidades de servicio

Si no puede usar una identidad administrada para representar la aplicación, use una entidad de servicio. Las entidades de servicio se pueden usar con aplicaciones de un solo inquilino y de varios inquilinos. 

Una entidad de servicio es la representación local de un objeto de aplicación en un único inquilino de Azure AD. Funciona como la identidad de la instancia de la aplicación, define quién puede tener acceso a la aplicación y a qué recursos puede tener acceso la aplicación. La entidad de servicio se crea de forma local en cada inquilino donde se usa la aplicación y hace referencia al objeto de aplicación único globalmente. El inquilino protege el inicio de sesión y el acceso a los recursos de la entidad de servicio.

Existen dos mecanismos para la autenticación con entidades de servicio: certificados de cliente y secretos de cliente. Los certificados son más seguros: use certificados de cliente si es posible. A diferencia de los secretos de cliente, los certificados de cliente no se pueden insertar accidentalmente en el código.

Para obtener información sobre cómo proteger las entidades de servicio, consulte [Protección de entidades de servicio](service-accounts-principal.md).

 
## <a name="next-steps"></a>Pasos siguientes


Para obtener más información sobre la protección de las cuentas de servicio de Azure, consulte:

[Protección de identidades administradas](service-accounts-managed-identities.md)

[Protección de entidades de servicio](service-accounts-principal.md)

[Gobernanza de las cuentas de servicio de Azure](service-accounts-governing-azure.md)
