---
title: Conceptos básicos de autenticación de NIST en Azure Active Directory
description: Explicaciones de la terminología y los factores de autenticación de NIST.
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
ms.openlocfilehash: 028ce5fc6ae3cef586803e1b04948006b58fd17e
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294218"
---
# <a name="nist-authentication-basics"></a>Conceptos básicos de la autenticación de NIST 

Para entender las directrices de NIST es necesario conocer bien la terminología y los conceptos de los módulos de plataforma de confianza (TPM) y los factores de autenticación.

## <a name="terminology"></a>Terminología

En estos artículos relacionados con NIST se emplea la terminología siguiente.

|Término| Definición: los términos *en cursiva* se definen en esta tabla.|
| - | - |
| Aserción| Una instrucción de un *comprobador* a un *usuario de confianza* que contiene información sobre el *suscriptor*. Puede contener atributos comprobados. |
|Authentication| Proceso de comprobación de la identidad de un *sujeto*. |
| Factor de autenticación| Algo que sabe, algo que tiene o algo que es: cada *autenticador* tiene uno o varios factores de autenticación. |
| Authenticator| Algo que el *demandante* posee y controla y que se usa para autenticar la identidad del *demandante*. |
| Demandante| Un *sujeto* cuya identidad se va a comprobar mediante uno o varios protocolos de autenticación. |
|Credential:| Un objeto o una estructura de datos que enlaza de forma autoritativa una identidad con al menos un *autenticador* poseído y controlado por un *suscriptor*. |
| Proveedor de servicios de credenciales (CSP)| Entidad de confianza que emite o registra *autenticadores de suscriptor* y emite *credenciales* electrónicas para los *suscriptores*. |
|Usuario de confianza| Una entidad que se basa en la *aserción de un comprobador* o en los *autenticadores del demandante* y sus *credenciales*, normalmente para conceder acceso a un sistema. |
|  Asunto| Una persona, organización, dispositivo, hardware, red, software o servicio. |
| Suscriptor| Una entidad que ha recibido una *credencial* o un *autenticador* de un *CSP*. |
|Módulo de plataforma segura (TPM)  | Un TPM es un módulo resistente a alteraciones que realiza operaciones criptográficas, incluida la generación de claves. |
|  Comprobador| Una entidad que comprueba la identidad del *demandante* comprobando la posesión y el control del demandante de los *autenticadores*. |


## <a name="about-trusted-platform-modules"></a>Acerca de los módulos de plataforma segura 

La tecnología del Módulo de plataforma segura (TPM) está diseñado para ofrecer funciones relacionadas con la seguridad y el hardware. Un chip TPM, o TPM de hardware, es un procesador criptográfico seguro que le ayuda con acciones como generar, almacenar y limitar el uso de claves criptográficas. 

Microsoft proporciona información significativa sobre cómo funcionan los TPM con Microsoft Windows. Para obtener más información, vea este artículo sobre el [Módulo de plataforma segura](https://docs.microsoft.com/windows/security/information-protection/tpm/trusted-platform-module-top-node). 

Un TPM de software es un emulador que imita esta funcionalidad. 

 ## <a name="authentication-factors-and-their-strengths"></a>Factores de autenticación y sus puntos fuertes

Los factores de autenticación se pueden agrupar en tres categorías. En la tabla siguiente se presenta un ejemplo de los tipos de factores de cada agrupación.

![Representación pictográfica de algo que conoce, algo que posee y algo que es.](media/nist-authentication-basics/nist-authentication-basics-0.png)

La solidez de un factor de autenticación viene determinada por la seguridad que podamos tener de que se trata de algo que solo el suscriptor conoce, tiene o es.

En el NIST hay pocas orientaciones sobre la solidez relativa de los factores de autenticación. En Microsoft, evaluamos los puntos fuertes como se indica a continuación. 

**Algo que sabe**: las contraseñas, el elemento más habitual empleado para determinar "algo que conoce", representan la mayor superficie de ataque. Las siguientes mitigaciones mejoran la confianza en la afinidad con el suscriptor y son eficaces para evitar ataques de contraseña, como ataques por fuerza bruta, interceptación e ingeniería social:

* [Requisitos de complejidad de la contraseña](https://www.microsoft.com/research/wp-content/uploads/2016/06/Microsoft_Password_Guidance-1.pdf)

* [Contraseñas prohibidas](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-configure-custom-password-protection)

* [Identificación de credenciales filtradas](https://docs.microsoft.com/azure/active-directory/identity-protection/overview-identity-protection)

* [Almacenamiento con hash seguro](https://aka.ms/AADDataWhitepaper)

* [Bloqueo de cuenta](https://docs.microsoft.com/azure/active-directory/authentication/howto-password-smart-lockout)

**Algo que tiene**: la solidez de "algo que tiene" se basa en la probabilidad de que el suscriptor lo mantenga en su poder y en la dificultad de que un atacante obtenga acceso a ello. Por ejemplo, un dispositivo móvil personal o una clave de hardware tendrán una afinidad mayor, y por tanto serán más seguros, que el equipo de escritorio de una oficina respecto a la protección frente a una amenaza interna.

**Algo que es**: la facilidad con la que un atacante puede obtener una copia de algo que es, o suplantar una biométrica, es importante. El NIST está redactando un marco para la biometría. En la actualidad, el NIST no aceptará la biometría como método de autenticación independiente. Debe ser un factor dentro de la autenticación multifactor. Esto se debe a que la biometría es, por naturaleza, probabilística. Es decir, usa algoritmos que determinan la probabilidad de que se trate de la misma persona. No es necesariamente una coincidencia exacta, como lo es una contraseña. Consulte este documento sobre la [solidez de la función para los autenticadores: biometría](https://pages.nist.gov/SOFA/SOFA.html) (SOFA-B). SOFA-B intenta presentar un marco para cuantificar la solidez de la biometría en términos de tasas de falsas coincidencias, falsedad, tasa de errores, tasa de errores de detección de ataques de presentación y esfuerzo requerido para iniciar un ataque. 

## <a name="single-factor-authentication"></a>Autenticación de factor único

La autenticación de factor único se puede lograr mediante un autenticador de factor único que constituye algo que sabe o algo que es. Aunque un factor de autenticación de "algo que es" se acepta como factor de autenticación, no se acepta como autenticador por sí mismo. 

![Imagen conceptual de la autenticación de factor único.](media/nist-authentication-basics/nist-authentication-basics-1.png)

## <a name="multi-factor-authentication"></a>Autenticación multifactor

La autenticación multifactor se puede lograr mediante un autenticador multifactor o una combinación de dos autenticadores de un solo factor. Un autenticador multifactor requiere dos factores de autenticación para ejecutar una única transacción de autenticación.

### <a name="multi-factor-authentication-using-two-single-factor-authenticators"></a>Autenticación multifactor mediante dos autenticadores de factor único

La autenticación multifactor requiere dos factores de autenticación diferentes. Pueden ser dos autenticadores independientes, como 

* Secreto memorizado [contraseña] y fuera de banda [SMS]

* Secreto memorizado [contraseña] y contraseña única [hardware o software]

Estos métodos realizan dos transacciones de autenticación independientes con Azure AD.

![Imagen conceptual de la autenticación multifactor mediante dos autenticadores independientes.](media/nist-authentication-basics/nist-authentication-basics-2.png)


### <a name="multi-factor-authentication-using-a-single-multi-factor-authenticator"></a>Autenticación multifactor mediante un único autenticador multifactor

La autenticación multifactor requiere un factor de autenticación (algo que sabe o algo que es) para desbloquear un segundo factor de autenticación. Esto suele ser una experiencia de usuario más sencilla que varios autenticadores independientes.

![Imagen conceptual de la autenticación multifactor con un único autenticador multifactor.](media/nist-authentication-basics/nist-authentication-basics-3a.png)

Un ejemplo es la aplicación Microsoft Authenticator que se usa en el modo sin contraseña. Con este método, el usuario intenta acceder a un recurso protegido (usuario de confianza) y recibe una notificación en su aplicación de autenticador. El usuario responde a una notificación proporcionando un factor biométrico (algo que es) o un PIN (algo que sabe), que luego desbloquea la clave criptográfica en el teléfono (algo que tiene) que posteriormente valida el comprobador.

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md) 