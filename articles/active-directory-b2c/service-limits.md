---
title: Restricciones y límites del servicio de Azure AD B2C
titleSuffix: Azure AD B2C
description: Consulte las restricciones y los límites del servicio Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/02/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 27fe9f4c7d63a5e0674bde6db3f387f813bed749
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111409404"
---
# <a name="azure-active-directory-b2c-service-limits-and-restrictions"></a>Restricciones y límites del servicio Azure Active Directory B2C

Este artículo contiene las restricciones de uso y otros límites de servicio para el servicio Azure Active Directory B2C (Azure AD B2C).

## <a name="end-userconsumption-related-limits"></a>Límites relacionados con el usuario final y el consumo

Los siguientes límites de servicio relacionados con el usuario final se aplican a todos los protocolos de autenticación y autorización admitidos por Azure AD B2C, como SAML, Open ID Connect, OAuth2 y ROPC.

|Category |Límite    |
|---------|---------|
|Número de solicitudes por dirección IP por inquilino de Azure AD B2C       |6000/5 min          |
|Número total de solicitudes por inquilino de Azure AD B2C     |12 000/min          |

El número de solicitudes puede variar en función del número de lecturas y escrituras de directorio que se produzcan durante el recorrido del usuario de Azure AD B2C. Por ejemplo, un recorrido de inicio de sesión simple que lee desde el directorio consta de una solicitud. Si el recorrido de inicio de sesión también debe actualizar el directorio, esta operación se cuenta como una solicitud adicional.

## <a name="azure-ad-b2c-configuration-limits"></a>Límites de configuración de Azure AD B2C

En la tabla siguiente se enumeran los límites de configuración administrativa en el servicio de Azure AD B2C.

|Category  |Límite  |
|---------|---------|
|Número de ámbitos por aplicación        |1000          |
|Número de [atributos personalizados](user-profile-attributes.md#extension-attributes) por usuario<sup>1</sup>       |100         |
|Número de redireccionamientos de direcciones URL por aplicación       |100         |
|Número de direcciones URL de cierre de sesión por aplicación        |1          |
|Límite de cadena por atributo      |250 caracteres          |
|Número de inquilinos de B2C por suscripción      |20         |
|Niveles de[herencia](custom-policy-overview.md#inheritance-model) en directivas personalizadas     |10         |
|Número de directivas por inquilino de Azure AD B2C      |200          |
|Tamaño máximo de archivo de directiva      |400 KB          |

<sup>1</sup> Consulte también [Restricciones y límites del servicio Azure AD](../active-directory/enterprise-users/directory-service-limits-restrictions.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [Guía de limitación de Microsoft Graph](/graph/throttling) 
- Más información sobre las [Diferencias de validación para las aplicaciones de Azure AD B2C](../active-directory/develop/supported-accounts-validation.md)
