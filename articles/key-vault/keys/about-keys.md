---
title: 'Acerca de las claves: Azure Key Vault'
description: Información general de los detalles para desarrolladores y la interfaz de REST de Azure Key Vault para claves.
services: key-vault
author: amitbapat
manager: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: overview
ms.date: 02/17/2021
ms.author: ambapat
ms.openlocfilehash: b9565095a40052a940d7a7b31f0fd3a27e0e75c2
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815007"
---
# <a name="about-keys"></a>Acerca de las claves

Azure Key Vault proporciona dos tipos de recursos para almacenar y administrar claves criptográficas. Los almacenes admiten claves protegidas por software y protegidas con HSM (módulo de seguridad de hardware). Los HSM administrados solo admiten claves protegidas con HSM. 

|Tipo de recurso|Métodos de protección de claves|URL base del punto de conexión del plano de datos|
|--|--|--|
| **Almacenes** | Protegidas mediante software<br/><br/>y<br/><br/>Protegidas con HSM (con SKU Premium)</li></ul> | https://{vault-name}.vault.azure.net |
| **HSM administrados** | Protegidas con HSM | https://{hsm-name}.managedhsm.azure.net |
||||

- **Almacenes**: los almacenes proporcionan una solución de administración de claves de bajo costo, fácil de implementar, multiinquilino, resistente a zona (donde esté disponible) y que resulta adecuada para la mayoría de los escenarios de aplicaciones en la nube.
- **HSM administrados**: HSM administrado proporciona HSM de un solo inquilino, resistentes a zona (donde esté disponible) y de alta disponibilidad para almacenar y administrar las claves criptográficas. Es más adecuado para aplicaciones y escenarios de uso que administran claves de alto valor. También ayuda a cumplir los requisitos de seguridad, cumplimiento normativo y regulación más estrictos. 

> [!NOTE]
> Los almacenes también le permiten almacenar y administrar varios tipos de objetos como secretos, certificados y claves de cuentas de almacenamiento además de las claves criptográficas.

Las claves criptográficas en Key Vault se representan como objetos de clave web JSON [JWK]. Las especificaciones de notación de objetos JavaScript (JSON) y de firma y cifrado de objetos JavaScript (JOSE) son:

-   [Clave web JSON (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key)  
-   [Cifrado web JSON (JWE)](http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption)  
-   [Algoritmos web JSON (JWA)](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms)  
-   [Firma web JSON (JWS)](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature) 

Las especificaciones de JWK/JWA base también se han ampliado para habilitar los tipos de clave únicos para las implementaciones de Azure Key Vault y HSM administrado. 

Las claves protegidas con HSM (también conocidas como claves HSM) se procesan en un HSM (módulo de seguridad de hardware) y siempre permanecen dentro del límite de la protección del mismo. 

- Los almacenes usan HSM validados con **FIPS 140-2 nivel 2** para proteger las claves HSM en la infraestructura de back-end del HSM compartido. 
- HSM administrado usa módulos HSM validados con **FIPS 140-2 nivel 3** para proteger las claves. Cada grupo de HSM es una instancia aislada de un solo inquilino con su propio [dominio de seguridad](../managed-hsm/security-domain.md) que proporciona un aislamiento criptográfico completo de todos los demás HSM que comparten la misma infraestructura de hardware.

Estas claves están protegidas en grupos de HSM de un solo inquilino. Puede importar una clave RSA, EC y simétrica de forma temporal o mediante la exportación desde un dispositivo HSM compatible. También puede generar claves en los grupos de HSM. Al importar claves de HSM mediante el método descrito en la [especificación Bring Your Own Key](../keys/byok-specification.md), se permite el transporte seguro del material de la clave a los grupos de HSM administrados. 

Para más información acerca de los límites geográficos, consulte [Centro de confianza de Microsoft Azure](https://azure.microsoft.com/support/trust-center/privacy/)

## <a name="key-types-and-protection-methods"></a>Tipos de clave y métodos de protección

Key Vault admite claves RSA y EC. HSM administrado admite claves RSA, EC y simétricas. 

### <a name="hsm-protected-keys"></a>Claves protegidas con HSM

|Tipo de clave|Almacenes (solo SKU Premium)|HSM administrados|
|--|--|--|
|**EC-HSM**: clave de curva elíptica | Compatible | Compatible|
|**RSA-HSM**: Clave RSA|Compatible|Compatible|
|**oct-HSM**: clave simétrica|No compatible|Compatible|
|||

### <a name="software-protected-keys"></a>Claves protegidas con software

|Tipo de clave|Almacenes|HSM administrados|
|--|--|--|
**RSA**: clave RSA "protegida con software"|Compatible|No compatible
**EC**: clave de curva elíptica "protegida con software"|Compatible|No compatible
|||

### <a name="compliance"></a>Cumplimiento normativo

|Tipo de clave y destino|Cumplimiento normativo|
|---|---|
|Claves protegidas por software en almacenes (SKU Premium y Estándar) | FIPS 140-2 nivel 1|
|Claves protegidas con HSM en almacenes (SKU Premium)| FIPS 140-2 nivel 2|
|Claves protegidas con HSM en HSM administrado|FIPS 140-2 nivel 3|
|||



Consulte [Tipos de claves, algoritmos y operaciones](about-keys-details.md) para más información sobre cada tipo de clave, algoritmos, operaciones, atributos y etiquetas.

## <a name="next-steps"></a>Pasos siguientes
- [Acerca de Key Vault](../general/overview.md)
- [Acerca de HSM administrado](../managed-hsm/overview.md)
- [Información acerca de los secretos](../secrets/about-secrets.md)
- [Información acerca de los certificados](../certificates/about-certificates.md)
- [Introducción a la API REST de Key Vault](../general/about-keys-secrets-certificates.md)
- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
