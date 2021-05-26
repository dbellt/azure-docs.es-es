---
title: 'Preguntas más frecuentes: credenciales verificables de Azure (versión preliminar)'
description: Encuentre respuestas a preguntas frecuentes sobre las credenciales verificables.
author: barclayn
manager: davba
ms.service: active-directory
ms.subservice: verifiable-credentials
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: c9253ce81b64b45d0ab8a72605eeba4c0d49f3f6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110457897"
---
# <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

Esta página contiene las preguntas más frecuentes sobre las credenciales verificables y las identidades descentralizadas. Las preguntas están organizadas en las siguientes secciones.

- [Vocabulario y conceptos básicos](#the-basics)
- [Preguntas conceptuales sobre las identidades descentralizadas](#conceptual-questions)
- [Preguntas sobre el uso de la versión preliminar de las credenciales verificables](#using-the-preview)

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory son una característica que se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="the-basics"></a>Conceptos básicos

### <a name="what-is-a-did"></a>¿Qué es un DID? 

Los identificadores descentralizados (DID) son identificadores que se pueden usar para proteger el acceso a los recursos, firmar y verificar las credenciales, y facilitar el intercambio de datos entre aplicaciones. A diferencia de los nombres de usuario y las direcciones de correo electrónico tradicionales, los DID son propiedad de la propia entidad (ya sea una persona, un dispositivo o una empresa). Los DID existen de forma independiente a cualquier organización externa o intermediario de confianza. En la [especificación de los identificadores descentralizados de W3C](https://www.w3.org/TR/did-core/), se explica con más detalle.

### <a name="why-do-we-need-a-did"></a>¿Por qué se necesita un DID?

La confianza digital hace del todo necesario que los participantes posean y controlen sus identidades, y la identidad comienza con el identificador.
En una época en la que todos los días se producen vulneraciones de sistemas y ataques a los jugosos repositorios centralizados de identificadores, la descentralización de identidades se está convirtiendo en una necesidad de seguridad perentoria para los consumidores y las empresas.
Las personas que poseen y controlan sus identidades pueden intercambiar datos y pruebas verificables. Los entornos de credenciales distribuidos permiten automatizar muchos de los procesos empresariales que actualmente se realizan manualmente y requieren una gran cantidad de trabajo.

### <a name="what-is-a-verifiable-credential"></a>¿Qué es una credencial verificable? 

Las credenciales forman parte de nuestra vida diaria: utilizamos carnets de conducir para confirmar que somos capaces de conducir un vehículo de motor, grados universitarios para indicar nuestro nivel educativo y pasaportes emitidos por los Gobiernos para viajar de un país a otro. Las credenciales verificables constituyen un mecanismo para expresar este tipo de credenciales en la Web de una forma criptográficamente segura que respete la privacidad y pueda verificarse automáticamente. En [la especificación de credenciales verificables de W3C](https://www.w3.org/TR/vc-data-model//), se explica con más detalle.


## <a name="conceptual-questions"></a>Preguntas conceptuales

### <a name="what-happens-when-a-user-loses-their-phone-can-they-recover-their-identity"></a>¿Qué ocurre cuando un usuario pierde su teléfono? ¿Puede recuperar su identidad?

Existen varios mecanismos de recuperación que se pueden ofrecer a los usuarios, cada uno con sus propias ventajas. En la actualidad, estamos evaluando diferentes opciones y enfoques de diseño de recuperación que resulten cómodos y seguros, al tiempo que respeten la privacidad y la capacidad de autogobierno de los usuarios.

### <a name="how-can-a-user-trust-a-request-from-an-issuer-or-verifier-how-do-they-know-a-did-is-the-real-did-for-an-organization"></a>¿Cómo puede un usuario confiar en una solicitud de un emisor o comprobador? ¿Cómo puede saber si el DID de una organización es real?

Hemos implementado [la especificación de configuración de DID conocidos de Decentralized Identity Foundation](https://identity.foundation/.well-known/resources/did-configuration/) para conectar un DID a los nombres de dominio más conocidos de un sistema existente. Todos los DID que se crean utilizando las credenciales verificables de Azure Active Directory tienen la opción de incluir un nombre de dominio raíz que se codificará en el documento del DID. Consulte este artículo sobre la [vinculación de dominios al identificador distribuido](how-to-dnsbind.md) para más información.  

### <a name="why-does-the-verifiable-credential-preview-use-ion-as-its-did-method-and-therefore-bitcoin-to-provide-decentralized-public-key-infrastructure"></a>¿Por qué la versión preliminar de las credenciales verificables utiliza ION como método de DID, y por tanto Bitcoin, para proporcionar una infraestructura de clave pública descentralizada?

ION es una red de identificadores de nivel 2 descentralizada y escalable que no utiliza permisos y que se ejecuta sobre Bitcoin. Esta escalabilidad la consigue sin incluir ningún token de "criptoactivos" especial, ningún validador de confianza ni ningún mecanismo de consenso centralizado. Usamos Bitcoin como sustrato base de nivel 1 por la seguridad que ofrece la red descentralizada a la hora de proporcionar un alto grado de inmutabilidad en un sistema de registro de eventos cronológico.

## <a name="using-the-preview"></a>Uso de la versión preliminar

### <a name="why-must-i-use-nodejs-for-the-verifiable-credentials-preview-any-plans-for-other-programming-languages"></a>¿Por qué debo usar NodeJS con la versión preliminar de las credenciales verificables? ¿Hay planes de incluir otros lenguajes de programación? 

Hemos elegido NodeJS porque es una plataforma muy popular para los desarrolladores de aplicaciones. Vamos a publicar una API REST que permitirá a los desarrolladores emitir y verificar las credenciales. 

### <a name="is-any-of-the-code-used-in-the-preview-open-source"></a>¿Todos el código que se utiliza en la versión preliminar es abierto?

Sí. Los repositorios siguientes son los componentes de código abierto de nuestros servicios.

1. [SideTree, en GitHub](https://github.com/decentralized-identity/sidetree)
2. [VC SDK para Node, en GitHub](https://github.com/microsoft/VerifiableCredentials-Verification-SDK-Typescript)
3. [Android SDK para crear carteras de identidades descentralizadas, en GitHub](https://github.com/microsoft/VerifiableCredential-SDK-Android)
4. [iOS SDK para crear carteras de identidades descentralizadas, en GitHub](https://github.com/microsoft/VerifiableCredential-SDK-iOS)


## <a name="what-are-the-licensing-requirements"></a>¿Cuáles son los requisitos de licencias?

Para poder utilizar la versión preliminar de las credenciales verificables, se necesita una licencia Azure AD P2. Se trata de un requisito temporal, ya que esperamos que los precios de este servicio se facturen en función del uso. 


## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para personalizar las credenciales verificables de Azure Active Directory](credential-design.md)
