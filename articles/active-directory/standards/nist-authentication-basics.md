---
title: Conceptos básicos de autenticación de NIST en Azure Active Directory
description: En este artículo se define terminología importante y se describen los módulos de plataforma segura y los factores de autenticación para NIST.
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
ms.openlocfilehash: eacdb19dcd5121f1d7fecdf6a3cefbafd609c9a9
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111889839"
---
# <a name="nist-authentication-basics"></a>Conceptos básicos de la autenticación de NIST 

Para comprender las directrices del Instituto Nacional de Estándares y Tecnología (NIST), es necesario conocer la terminología. También debe comprender los factores tecnológicos y de autenticación del Módulo de plataforma segura (TPM). En este artículo se proporciona esa información. 

## <a name="terminology"></a>Terminología

En estos artículos de NIST se emplea la terminología siguiente.

|Término| Definición. Los términos *en cursiva* se definen en esta tabla.|
| - | - |
| Aserción| Instrucción de un *comprobador* a un *usuario de confianza* que contiene información sobre el *suscriptor*. Una aserción puede contener atributos comprobados. |
|Authentication| Proceso de comprobación de la identidad de un *sujeto*. |
| Factor de autenticación| Algo que sabe, algo que tiene o algo que es. Cada *autenticador* tiene uno o varios factores de autenticación. |
| Authenticator| Algo que el *demandante* posee y controla y que se usa para autenticar la identidad del *demandante*. |
| Demandante| Un *sujeto* cuya identidad se va a comprobar mediante uno o varios protocolos de *autenticación*. |
|Credential:| Un objeto o una estructura de datos que enlaza de forma autoritativa una identidad con al menos un *autenticador* poseído y controlado por un *suscriptor*. |
| Proveedor de servicios de credenciales (CSP)| Entidad de confianza que emite o registra *autenticadores de suscriptor* y emite *credenciales* electrónicas para los *suscriptores*. |
|Usuario de confianza| Entidad que se basa en la *aserción de un comprobador* o en los *autenticadores del demandante* y sus *credenciales*, normalmente para conceder acceso a un sistema. |
|  Asunto| Una persona, organización, dispositivo, hardware, red, software o servicio. |
| Suscriptor| Una entidad que ha recibido una *credencial* o un *autenticador* de un *CSP*. |
|Módulo de plataforma segura  | Un TPM es un módulo resistente a alteraciones que realiza operaciones criptográficas, incluida la generación de claves. |
|  Comprobador| Una entidad que comprueba la identidad del *demandante* comprobando la posesión y el control del demandante de los *autenticadores*. |


## <a name="about-trusted-platform-module-technology"></a>Acerca de la tecnología del Módulo de plataforma segura

La tecnología del Módulo de plataforma segura está diseñada para ofrecer funciones relacionadas con la seguridad y el hardware. Un chip TPM, o TPM de hardware, es un procesador criptográfico seguro que le ayuda con acciones como generar, almacenar y limitar el uso de claves criptográficas. 

Microsoft proporciona información significativa sobre cómo funcionan los TPM con Windows. Para obtener más información, vea [Módulo de plataforma segura](/windows/security/information-protection/tpm/trusted-platform-module-top-node). 

Un TPM de software es un emulador que imita la funcionalidad de TPM de hardware. 

 ## <a name="authentication-factors-and-their-strengths"></a>Factores de autenticación y sus puntos fuertes

Los factores de autenticación se pueden agrupar en tres categorías:

![Gráfico que proporciona ejemplos de factores de autenticación, agrupados por algo que sabe, algo que tiene y algo que es.](media/nist-authentication-basics/nist-authentication-basics-0.png)

La solidez de un factor de autenticación viene determinada por la seguridad que puede tener de que se trata de algo que solo el suscriptor conoce, tiene o es.

NIST proporciona pocas orientaciones sobre la solidez relativa de los factores de autenticación. En el resto de esta sección se describe cómo se evalúan esos puntos fuertes en Microsoft. 

**Algo que sabe**. Las contraseñas, el elemento más habitual empleado para determinar *algo que sabe*, representan la mayor superficie de ataque. Las siguientes mitigaciones mejoran la confianza en la afinidad con el suscriptor. Son eficaces para evitar ataques de contraseña, como ataques por fuerza bruta, interceptación e ingeniería social:

* [Requisitos de complejidad de la contraseña](https://www.microsoft.com/research/wp-content/uploads/2016/06/Microsoft_Password_Guidance-1.pdf)

* [Contraseñas prohibidas](../authentication/tutorial-configure-custom-password-protection.md)

* [Identificación de credenciales filtradas](../identity-protection/overview-identity-protection.md)

* [Almacenamiento con hash seguro](https://aka.ms/AADDataWhitepaper)

* [Bloqueo de cuenta](../authentication/howto-password-smart-lockout.md)

**Algo que tiene**. La solidez de *algo que tiene* se basa en la probabilidad de que el suscriptor lo mantenga en su poder y en la dificultad de que un atacante obtenga acceso a ello. Por ejemplo, cuando intenta protegerse frente a amenazas internas, un dispositivo móvil personal o una clave de hardware tendrán una afinidad mayor. Por lo tanto, será más seguro que un equipo de escritorio en una oficina.

**Algo que es**. La facilidad con la que un atacante puede obtener una copia de *algo que es*, o suplantar una biométrica, es importante. El NIST está redactando un marco para la biometría. En la actualidad, el NIST no acepta la biometría como método de autenticación independiente. Debe ser un factor dentro de la autenticación multifactor. Se coloca esta precaución porque la biometría es probabilística por naturaleza. Es decir, usa algoritmos que determinan la probabilidad de afinidad. La biometría no proporciona necesariamente una coincidencia exacta, como hacen las contraseñas. Para más información, consulte este documento sobre la [solidez de la función para los autenticadores: biometría](https://pages.nist.gov/SOFA/SOFA.html) (SOFA-B). 

SOFA-B intenta presentar un marco para cuantificar la solidez de la biometría para los siguientes aspectos:
- Tasa de coincidencias falsa.
- Tasa de errores falsa.
- Tasa de errores de detección de ataques de presentación.
- Esfuerzo necesario para realizar un ataque. 

## <a name="single-factor-authentication"></a>Autenticación de factor único

Puede implementar la autenticación de factor único mediante un autenticador de factor único que comprueba *algo que sabe* o *algo que es*. Un factor relacionado con *algo que es* se acepta como factor de autenticación, pero no se acepta como autenticador por sí mismo. 

![Gráfico que muestra cómo funciona la autenticación de un solo factor.](media/nist-authentication-basics/nist-authentication-basics-1.png)

## <a name="multifactor-authentication"></a>Autenticación multifactor

Puede implementar la autenticación multifactor mediante un autenticador multifactor o mediante dos autenticadores de factor único. Un autenticador multifactor requiere dos factores de autenticación para completar una única transacción de autenticación.

### <a name="multifactor-authentication-by-using-two-single-factor-authenticators"></a>Autenticación multifactor mediante dos autenticadores de factor único

La autenticación multifactor requiere dos factores de autenticación diferentes. Estos autenticadores pueden ser independientes. Por ejemplo: 

* Secreto memorizado (contraseña) y fuera de banda (SMS)

* Secreto memorizado (contraseña) y contraseña única (hardware o software)

Estos métodos realizan dos transacciones de autenticación independientes con Azure Active Directory (Azure AD).

![Gráfico que describe la autenticación multifactor a través de dos autenticadores independientes.](media/nist-authentication-basics/nist-authentication-basics-2.png)


### <a name="multifactor-authentication-by-using-a-single-multifactor-authenticator"></a>Autenticación multifactor mediante un autenticador de factor único

La autenticación multifactor requiere un factor de autenticación (*algo que sabe* o *algo que es*) para desbloquear un segundo factor de autenticación. La experiencia del usuario suele ser más fácil que la de varios autenticadores independientes.

![Gráfico que muestra la autenticación multifactor mediante un autenticador multifactor único.](media/nist-authentication-basics/nist-authentication-basics-3a.png)

Un ejemplo es la aplicación Microsoft Authenticator que se usa en el modo sin contraseña. Con este método, el usuario intenta acceder a un recurso protegido (usuario de confianza) y recibe una notificación en la aplicación Authenticator. El usuario responde a la notificación proporcionando un factor biométrico (*algo que es*) o un PIN (*algo que sabe*). Este factor desbloquea la clave criptográfica en el teléfono (*algo que tiene*), que el comprobador valida a continuación.

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Cómo alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Cómo alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Cómo alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md)