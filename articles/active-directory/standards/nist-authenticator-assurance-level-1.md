---
title: Alcanzar el nivel de seguridad NIST AAL1 con Azure Active Directory
description: Instrucciones para alcanzar el nivel 1 de seguridad del autenticador (AAL 1) de NIST con Azure Active Directory.
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
ms.openlocfilehash: 640c401beb42b07465f533fa14b5b7fd4b0200e5
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294200"
---
# <a name="achieving-nist-authenticator-assurance-level-1-with-azure-active-directory"></a>Alcanzar el nivel 1 de seguridad del autenticador de NIST con Azure Active Directory

El Instituto Nacional de Estándares y Tecnología (NIST, por sus siglas en inglés) desarrolla los requisitos técnicos de las agencias federales de Estados Unidos que implementan soluciones de identidad. El cumplimiento de estos requisitos también es necesario en el caso de organizaciones que trabajan con agencias federales. Este artículo le guía por los pasos necesarios para alcanzar el nivel 1 de seguridad de autenticación (AAL 1) de NIST. 

Recursos que puede ser interesante ver antes de intentar alcanzar AAL 1:
* [Introducción a NIST](nist-overview.md): conozca los distintos niveles de AAL.
* [Conceptos básicos de autenticación:](nist-authentication-basics.md): terminología importante y tipos de autenticación.
* [Tipos de autenticadores de NIST](nist-authenticator-types.md): conozca cada uno de los tipos de autenticador.
* [AAL de NIST:](nist-about-authenticator-assurance-levels.md): los componentes de los AAL, cómo se les asignan métodos de autenticación de Microsoft Azure Active Directory y descripción de los módulos de plataforma de confianza (TPM). 

## <a name="permitted-authenticator-types"></a>Tipos de autenticadores permitidos

 Para alcanzar el nivel AAL1, se puede usar cualquier [autenticador permitido](nist-authenticator-types.md) del NIST de uno o varios factores. La tabla siguiente contiene las que no se incluyen en [AAL2](nist-authenticator-assurance-level-2.md) y [AAL3](nist-authenticator-assurance-level-2.md).

| Método de autenticación de Azure AD| Tipo de autenticador de NIST |
| - | - |
| Contraseña |Secreto memorizado |
| Teléfono (SMS)|  Fuera de banda |
|  Clave de seguridad FIDO 2 <br>Aplicación Microsoft Authenticator para iOS (sin contraseña)<br>Windows Hello para empresas con TPM de software <br>Tarjeta inteligente (ADFS) |  Software criptográfico multifactor |

> [!TIP]
> Se recomienda cumplir al menos el nivel AAL 2, salvo que por razones empresariales, estándares del sector o requisitos de cumplimiento se deba cumplir con AAL3.

## <a name="fips-140-validation"></a>Validación de FIPS 140

### <a name="verifier-requirements"></a>Requisitos del comprobador

Azure AD usa el módulo criptográfico validado globalmente FIPS 140 nivel 1 de Windows   
en todas sus operaciones criptográficas relacionadas con la autenticación. Por lo tanto, es un comprobador compatible con FIPS 140 según las exigencias de las agencias gubernamentales.

## <a name="man-in-the-middle-mitm-resistance"></a>Resistencia de tipo "man in the middle" (MitM) 

Todas las comunicaciones entre el solicitante y Azure AD se realizan a través de un canal protegido autenticado para proporcionar resistencia a los ataques MitM. Así se satisfacen los requisitos de resistencia de MitM para AAL1, AAL2 y AAL3.

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md) 