---
title: Logro del nivel de seguridad NIST AAL2 con Azure Active Directory
description: Instrucciones para alcanzar el nivel 2 de garantía del autenticador (AAL2) de NIST con Azure Active Directory.
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
ms.openlocfilehash: aaa40141eaa0617c34a0ae8c4a7cf396d624c1b6
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294213"
---
# <a name="achieving-nist-authenticator-assurance-level-2-with-azure-active-directory"></a>Logro del nivel 2 de garantía del autenticador de NIST con Azure Active Directory

El Instituto Nacional de Estándares y Tecnología (NIST, por sus siglas en inglés) desarrolla los requisitos técnicos para las agencias federales de Estados Unidos que implementan soluciones de identidad. El cumplimiento de estos requisitos también es necesario en el caso de organizaciones que trabajan con agencias federales. Este artículo le guía por los pasos necesarios para alcanzar el nivel 2 de garantía de autenticación (AAL2) de NIST. 

Recursos que puede ser interesante ver antes de intentar alcanzar AAL2:
* [Introducción a NIST](nist-overview.md): conozca los distintos niveles de AAL.
* [Conceptos básicos de autenticación:](nist-authentication-basics.md): terminología importante y tipos de autenticación.
* [Tipos de autenticadores de NIST](nist-authenticator-types.md): conozca cada uno de los tipos de autenticador.
* [AAL de NIST](nist-about-authenticator-assurance-levels.md): los componentes de las AAL y cómo se asignan a ellos los métodos de autenticación de Microsoft Azure Active Directory.

## <a name="permitted-authenticator-types"></a>Tipos de autenticadores permitidos


| Método de autenticación de Azure AD| Tipo de autenticador de NIST | 
| - | - |
| **Métodos recomendados** |   | 
| Aplicación Microsoft Authenticator para iOS (sin contraseña)<br>Windows Hello para empresas con TPM de software | Software criptográfico multifactor |
| Clave de seguridad FIDO 2<br>Aplicación Microsoft Authenticator para Android (sin contraseña)<br>Windows Hello para empresas con TPM de hardware<br>Tarjeta inteligente (ADFS) | Hardware criptográfico multifactor |
| **Otros métodos** |  |
| Contraseña y teléfono (SMS) | Secreto memorizado + fuera de banda |
| Contraseña + aplicación Microsoft Authenticator (OTP)<br>Contraseña + SF OTP | Secreto memorizado + contraseña única de un solo factor |
| Contraseña + Azure AD unido al TPM de software <br>Contraseña + dispositivo móvil compatible<br>Contraseña + Azure AD híbrido unido al TPM de software <br>Contraseña + aplicación Microsoft Authenticator (notificación) | Secreto memorizado + SW de criptografía de factor único |
| Contraseña + Azure AD unido con TPM de hardware <br>Contraseña + Azure AD híbrido unido con TPM de hardware | Secreto memorizado + hardware criptográfico de un solo factor |


### <a name="our-recommendations"></a>Nuestras recomendaciones

Recomendamos el uso de autenticadores de hardware o software criptográfico multifactor para lograr AAL2. La autenticación sin contraseña elimina la mayor superficie de ataque (la contraseña) y ofrece a los usuarios un método simplificado para autenticarse. 

Para obtener instrucciones detalladas sobre cómo seleccionar un método de autenticación sin contraseña, vea [Planeamiento de una implementación de autenticación sin contraseña en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-passwordless-deployment).

Para obtener más información sobre cómo implementar Windows Hello para empresas, consulte la [guía de implementación de Windows Hello para empresas](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-deployment-guide).

## <a name="fips-140-validation"></a>Validación de FIPS 140

La siguiente información es una guía para lograr la validación de FIPS 140.

### <a name="verifier-requirements"></a>Requisitos del comprobador

Azure AD usa el módulo criptográfico validado globalmente del nivel 1 de FIPS 140 de Windows   
en todas sus operaciones criptográficas relacionadas con la autenticación. Por lo tanto, es un comprobador compatible con FIPS 140 según las exigencias de las agencias gubernamentales.

### <a name="authenticator-requirements"></a>Requisitos del autenticador

Los autenticadores criptográficos de las agencias gubernamentales deben tener la validación *general FIPS 140, nivel 1.* Esto no es un requisito para las agencias no gubernamentales. Los siguientes autenticadores de Azure AD cumplen el requisito cuando se ejecutan en [Windows en un modo de operación aprobado por FIPS 140](https://docs.microsoft.com/windows/security/threat-protection/fips-140-validation).

* Contraseña

* Azure AD unido con software o TPM de hardware

* Azure AD híbrido unido con software o TPM de hardware

* Windows Hello para empresas con software o TPM de hardware

* Tarjeta inteligente (ADFS) 

Las claves de seguridad de FIDO2 y la aplicación Microsoft Authenticator (en todos sus modos: notificación, OTP y sin contraseña) no cumplen los requisitos de las agencias gubernamentales para la validación general del nivel 1 de FIPS 140 a partir de este artículo.

* La aplicación Microsoft Authenticator usa criptografía aprobada por FIPS 140; sin embargo, no está validada de forma global por el nivel 1 de FIPS 140. 

* Las claves FIDO2 son una innovación muy reciente y, como tal, todavía están en proceso de la certificación de FIPS.

## <a name="reauthentication"></a>Reautenticación 

En AAL2, NIST requiere una nueva autenticación cada 12 horas, independientemente de la actividad del usuario, y después de cualquier período de inactividad que dure 30 minutos o más. Es obligatoria la presentación de algo que sabe o algo que es, porque el secreto de sesión es algo que tiene.

Para cumplir el requisito de reautenticación independientemente de la actividad del usuario, Microsoft recomienda configurar la [frecuencia de inicio de sesión del usuario](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-session-lifetime) en 12 horas. 

NIST también permite el uso de controles de compensación para confirmar la presencia del suscriptor:

* El tiempo de espera para la inactividad de la sesión de 30 minutos se puede conseguir bloqueando el dispositivo a nivel del sistema operativo mediante Microsoft System Center Configuration Manager (SCCM), objetos de directiva de grupo (GPO) o Intune. También debe requerir autenticación local para que el suscriptor la desbloquee.

* El tiempo de espera independientemente de la actividad se puede lograr mediante la ejecución de una tarea programada (aprovechando SCCM, GPO o Intune) que bloquea la máquina después de 12 horas, independientemente de la actividad.

## <a name="man-in-the-middle-mitm-resistance"></a>Resistencia de tipo "man in the middle" (MitM) 

Todas las comunicaciones entre el solicitante y Azure AD se realizan a través de un canal protegido autenticado para proporcionar resistencia a los ataques MitM. Así se satisfacen los requisitos de resistencia de MitM para AAL1, AAL2 y AAL3.

## <a name="replay-resistance"></a>Resistencia de reproducción

Todos los métodos de autenticación de Azure AD en AAL2 utilizan nonce o desafíos y son resistentes a los ataques de reproducción, ya que el verificador detectará fácilmente las transacciones de autenticación reproducidas, puesto que no contendrán los datos apropiados de nonce o temporalidad.

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md)  