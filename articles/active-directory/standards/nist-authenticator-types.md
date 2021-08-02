---
title: Tipos de autenticadores de NIST y métodos de Azure Active Directory correspondientes
description: Explica cómo los métodos de autenticación de Azure Active Directory se alinean con los tipos de autenticadores de NIST.
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: barbaraselden
ms.author: baselden
manager: mtillman
ms.reviewer: martinco
ms.date: 4/26/2021
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f878e18a914b0319657790864d02832bf91fbe36
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111889659"
---
# <a name="nist-authenticator-types-and-aligned-azure-active-directory-methods"></a>Tipos de autenticadores de NIST y métodos de Azure Active Directory correspondientes

El proceso de autenticación comienza cuando un solicitante declara su control de uno o varios de los autenticadores que están asociados a un suscriptor. El suscriptor puede ser una persona u otra entidad.

| Tipo de autenticador del Instituto Nacional de Estándares y Tecnología (NIST)| Métodos de autenticación de Azure Active Directory (Azure AD). |
| - | - |
|  Secreto memorizado <br> (algo que sabe)|  Contraseña (cuentas en la nube)  <br>Contraseña (federada)<br> Contraseña (sincronización de hash de contraseña)<br>Contraseña (autenticación transferida) |
|Secreto de búsqueda <br> (algo que tiene)| Ninguno. Por definición, un secreto de búsqueda es los datos que no se mantienen en un sistema. |
|Fuera de banda <br>(algo que tiene)| Teléfono (SMS): no recomendado |
| Contraseña de un solo uso de factor único <br>‎(algo que tiene)| Aplicación Microsoft Authenticator (contraseña de un solo uso)  <br>Contraseña de un solo uso de factor único (a través de los fabricantes de OTP)<sup data-htmlnode="">1</sup> | 
| Contraseña multifactor de un solo uso<br>(algo que tiene + algo que sabe o algo que forma parte de usted)| Contraseña multifactor de un solo uso (a través de los fabricantes de OTP)<sup data-htmlnode="">1</sup>| 
|Software criptográfico de factor único<br>(algo que tiene)|Dispositivo móvil compatible <br> Aplicación Microsoft Authenticator (notificación) <br> Unido a Azure AD híbrido<sup data-htmlnode="">2</sup> con TPM de software<br> Unido a Azure AD<sup data-htmlnode="">2</sup> con TPM de software |
| Hardware criptográfico de factor único <br>(algo que tiene) | Unido a Azure AD<sup data-htmlnode="">2</sup> con TPM de hardware <br> Unido a Azure AD híbrido<sup data-htmlnode="">2</sup> con TPM de hardware|
|Software criptográfico multifactor<br>(algo que tiene + algo que sabe o algo que forma parte de usted) | Aplicación Microsoft Authenticator para iOS (sin contraseña)<br> Windows Hello para empresas con TPM de software |
|Hardware criptográfico multifactor <br>(algo que tiene + algo que sabe o algo que forma parte de usted) |Aplicación Microsoft Authenticator para Android (sin contraseña)<br> Windows Hello para empresas con TPM de hardware<br> Tarjeta inteligente (proveedor de identidades federadas) <br> Clave de seguridad FIDO 2 |


<sup data-htmlnode="">1</sup> tokens OATH-TOTP SHA-1 de la variedad de 30 o 60 segundos.

<sup data-htmlnode="">2</sup> Para obtener más información sobre los estados de unión de los dispositivos, consulte la [documentación de identidad del dispositivo de Azure AD](../devices/index.yml). 

## <a name="why-sms-isnt-recommended"></a>Por qué no se recomiendan los SMS 

Los mensajes de texto cumplen el estándar NIST, pero NIST no los recomienda. Los riesgos de intercambio de dispositivos, cambios de SIM, portabilidad de números y otros comportamientos pueden causar problemas. Si estas acciones se llevan a cabo de manera malintencionada, pueden dar lugar a una experiencia poco segura. Aunque no se recomienda usar SMS, son mejores que usar una contraseña por sí sola, ya que los hackers tendrán que esforzarse más. 

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Logro del nivel 1 de seguridad del autenticador de NIST con Azure Active Directory](nist-authenticator-assurance-level-1.md)

[Logro del nivel 2 de seguridad del autenticador de NIST con Azure Active Directory](nist-authenticator-assurance-level-2.md)

[Logro del nivel 3 de seguridad del autenticador de NIST con Azure Active Directory](nist-authenticator-assurance-level-3.md)