---
title: 'Requisitos de TLS y del conjunto de cifrado: Azure AD B2C'
titleSuffix: Azure AD B2C
description: Notas para desarrolladores sobre los requisitos de TLS y del conjunto de cifrado HTTPS al interactuar con puntos de conexión de API web.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/30/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 3732f53c3a4e77a1a10363cb53d898e6edc661db
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960438"
---
# <a name="azure-active-directory-b2c-tls-and-cipher-suite-requirements"></a>Requisitos de TLS y del conjunto de cifrado de Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) se conecta a los puntos de conexión mediante [conectores de API](api-connectors-overview.md) y [proveedores de identidades](oauth2-technical-profile.md) dentro de [flujos de usuario](user-flow-overview.md). En este artículo se abordan los requisitos de TLS y del conjunto de cifrado para los puntos de conexión.

Los puntos de conexión configurados con conectores de API y proveedores de identidades deben publicarse en un URI HTTPS accesible públicamente. Antes de establecer una conexión segura con el punto de conexión, el protocolo y el cifrado se negocian entre Azure AD B2C y el punto de conexión según las funcionalidades de ambos lados de la conexión.

Azure AD B2C debe poderse conectar a los puntos de conexión mediante la seguridad de la capa de transporte (TLS) y los conjuntos de cifrado, como se describe en este artículo.

## <a name="tls-versions"></a>Versiones de TLS

TLS versión 1.2 es un protocolo criptográfico que proporciona autenticación y cifrado de datos entre servidores y clientes. El punto de conexión debe admitir la comunicación segura a través de la **versión 1.2 de TLS**. Las versiones 1.0 y 1.1 de TLS están en desuso. 

## <a name="cipher-suites"></a>Conjuntos de cifrado

Los conjuntos de cifrado son conjuntos de algoritmos criptográficos. Proporcionan información esencial sobre cómo comunicar datos de forma segura cuando se usa el protocolo HTTPS a través de TLS.

El punto de conexión debe admitir al menos uno de los siguientes cifrados:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

## <a name="endpoints-in-scope"></a>Puntos de conexión en el ámbito

Los siguientes puntos de conexión que se usan en el entorno de Azure AD B2C deben cumplir los requisitos descritos en este artículo:

- [Conectores de API](api-connectors-overview.md) 
- OAuth1
    - Punto de conexión de token 
    - Punto de conexión de información del usuario
- Proveedores de identidades de OAuth2 y OpenID Connect
    - Punto de conexión de detección de OpenID Connect
    - Punto de conexión JWKS de OpenID Connect
    - Punto de conexión de token 
    - Punto de conexión de información del usuario
- [Sugerencia de token de identificador](id-token-hint.md)
    - Punto de conexión de detección de OpenID Connect
    - Punto de conexión JWKS de OpenID Connect
- Punto de conexión de metadatos del [proveedor de identidades SAML](saml-service-provider.md)
- Punto de conexión de metadatos del [proveedor de servicios SAML](identity-provider-generic-saml.md)

## <a name="check-your-endpoint-compatibility"></a>Comprobación de la compatibilidad del punto de conexión

Para comprobar que los puntos de conexión cumplen los requisitos descritos en este artículo, realice una prueba mediante una herramienta de cifrado y examen de TLS. Pruebe el punto de conexión con [SSLLABS](https://www.ssllabs.com/ssltest/analyze.html).


## <a name="next-steps"></a>Pasos siguientes

Consulte también los artículos siguientes:

- [Solución de problemas de aplicaciones que no admiten TLS 1.2](../cloud-services/applications-dont-support-tls-1-2.md)
- [Conjuntos de cifrado en TLS/SSL (Schannel SSP)](/windows/win32/secauthn/cipher-suites-in-schannel)
- [Habilitación de TLS 1.2](/mem/configmgr/core/plan-design/security/enable-tls-1-2)
- [Solución del problema con TLS 1.0](/security/engineering/solving-tls1-problem)