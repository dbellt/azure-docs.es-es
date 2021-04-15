---
title: Evitación de la aceleración automática del inicio de sesión en Azure AD mediante la directiva de detección del dominio de inicio
description: Obtenga información acerca de cómo evitar la aceleración automática de sugerencias de dominio en IDP federados.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/12/2021
ms.author: iangithinji
ms.reviewer: hirsin
ms.openlocfilehash: b89e0e1c8bd8109fac8b4b7c05a845a3e234b617
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375557"
---
# <a name="disable-auto-acceleration-to-a-federated-idp-during-user-sign-in-with-home-realm-discovery-policy"></a>Deshabilitación de la aceleración automática en un IDP federado durante el inicio de sesión de usuario con la directiva de detección del dominio de inicio

La [directiva de detección del dominio de inicio](/graph/api/resources/homeRealmDiscoveryPolicy) (HRD) ofrece a los administradores varias maneras de controlar cómo y dónde se autentican sus usuarios. La sección `domainHintPolicy` de la directiva de HRD se usa para ayudar a migrar usuarios federados a credenciales administradas en la nube como [FIDO](../authentication/howto-authentication-passwordless-security-key.md), asegurándose de que siempre visitan la página de inicio de sesión de Azure AD y no se aceleran automáticamente a un IDP federado como consecuencia de las sugerencias de dominio.

Esta directiva es necesaria en situaciones en las que un administrador no puede controlar o actualizar las sugerencias para agregar dominios durante el inicio de sesión.  Por ejemplo, `outlook.com/contoso.com` envía al usuario a una página de inicio de sesión con el parámetro `&domain_hint=contoso.com` anexado, con el fin de acelerar automáticamente al usuario de manera directa al IDP federado del dominio `contoso.com`. Los usuarios con credenciales administradas enviadas a un IDP federado no pueden iniciar sesión con sus credenciales administradas, lo que reduce la seguridad y les genera frustración por experiencias de inicio de sesión aleatorias. Los administradores que implementan credenciales administradas [también deben configurar esta directiva](#suggested-use-within-a-tenant) para asegurarse de que los usuarios siempre puedan usar sus credenciales administradas.

## <a name="domainhintpolicy-details"></a>Detalles de DomainHintPolicy

La sección DomainHintPolicy de la directiva de HRD es un objeto JSON que permite que un administrador opte por que ciertos dominios y aplicaciones no reciban sugerencias de dominio.  En la práctica, esto indica a la página de inicio de sesión de Azure AD que se comporte como si en la solicitud de inicio de sesión no estuviera presente un parámetro `domain_hint`.

### <a name="the-respect-and-ignore-policy-sections"></a>Secciones de la directiva de respetar y omitir

|Sección | Significado | Valores |
|--------|---------|--------|
|`IgnoreDomainHintForDomains` |Si se envía esta sugerencia de dominio en la solicitud, omítala. |Matriz de direcciones de dominio (por ejemplo, `contoso.com`). También admite `all_domains`.|
|`RespectDomainHintForDomains`| Si se envía esta sugerencia de dominio en la solicitud, respétela incluso si `IgnoreDomainHintForApps` indica que la aplicación de la solicitud no debería acelerarse automáticamente. Se usa para ralentizar el lanzamiento de sugerencias de dominio en desuso dentro de la red: puede indicar que algunos dominios deben seguir acelerados. | Matriz de direcciones de dominio (por ejemplo, `contoso.com`). También admite `all_domains`.|
|`IgnoreDomainHintForApps`| Si una solicitud de esta aplicación incluye una sugerencia de dominio, omítala. | Matriz de identificadores de aplicación (GUID). También admite `all_apps`.|
|`RespectDomainHintForApps` |Si una solicitud de esta aplicación incluye una sugerencia de dominio, respétela incluso si `IgnoreDomainHintForDomains` incluye ese dominio. Se usa para asegurarse de que algunas aplicaciones siguen funcionando si detecta que se interrumpen sin sugerencias de dominio. | Matriz de identificadores de aplicación (GUID). También admite `all_apps`.|

### <a name="policy-evaluation"></a>Evaluación de directiva

La lógica de DomainHintPolicy se ejecuta en cada solicitud entrante que contiene una sugerencia de dominio y se acelera en función de dos fragmentos de datos de la solicitud: el dominio de la sugerencia de dominios y el identificador de cliente (la aplicación). En resumen: el "respeto" de un dominio o una aplicación tiene prioridad sobre una instrucción para "omitir" una sugerencia de dominio para un dominio o aplicación determinados.

1. En ausencia de una directiva de sugerencia de dominio, o si ninguna de las 4 secciones hacen referencia a la sugerencia de aplicación o dominio mencionada, [se evaluará el resto de la directiva de HRD](configure-authentication-for-federated-users-portal.md#priority-and-evaluation-of-hrd-policies).
1. Si alguna de las secciones `RespectDomainHintForApps` o `RespectDomainHintForDomains` (o ambas) incluye la sugerencia de aplicación o dominio en la solicitud, el usuario se acelerará automáticamente al IDP federado según se solicite.
1. Si alguna de las secciones `IgnoreDomainHintsForApps` o `IgnoreDomainHintsForDomains` (o ambas) hace referencia a la sugerencia de aplicación o dominio en la solicitud, y no se hace referencia a ellas en las secciones de "respeto", la solicitud no se acelerará automáticamente y el usuario permanecerá en la página de inicio de sesión de Azure AD para proporcionar un nombre de usuario.

Una vez que un usuario ha escrito un nombre de usuario en la página de inicio de sesión, puede usar sus credenciales administradas, si tienen alguna registrada.  Si deciden no usar una credencial administrada o no tienen ninguna registrada, se les dirigirá a su IDP federado para la indicación de credenciales como de costumbre.

## <a name="suggested-use-within-a-tenant"></a>Uso sugerido dentro de un inquilino

Los administradores de dominios federados deben configurar esta sección de la directiva de HRD en un plan de cuatro fases. El objetivo de este plan es que, finalmente, todos los usuarios de un inquilino tengan la oportunidad de utilizar sus credenciales administradas independientemente del dominio o de la aplicación, salvo aquellas aplicaciones que tengan una fuerte dependencia del uso de `domain_hint`.  Este plan ayuda a los administradores a encontrar esas aplicaciones, eximirlas de la nueva directiva y seguir implementando el cambio en el resto del inquilino.

1. Escoja un dominio para realizar este cambio inicialmente.  Este será su dominio de prueba, por lo que debe elegir uno que pueda ser más receptivo a los cambios en la experiencia de usuario (es decir, ver una página de inicio de sesión diferente).  Se omitirán todas las sugerencias de dominio de todas las aplicaciones que usen este nombre de dominio. [Configure](#configuring-policy-through-graph-explorer) esta directiva en la directiva de HRD predeterminada del inquilino:

```json
 "DomainHintPolicy": { 
    "IgnoreDomainHintForDomains": [ "testDomain.com" ], 
    "RespectDomainHintForDomains": [], 
    "IgnoreDomainHintForApps": [], 
    "RespectDomainHintForApps": [] 
} 
```

2. Recopile comentarios de los usuarios del dominio de prueba. Recoja los detalles de las aplicaciones que se han interrumpido como resultado de este cambio: tienen una dependencia de uso de la sugerencia de dominio y deben actualizarse. Por ahora, agréguelas a la sección `RespectDomainHintForApps`:

```json
 "DomainHintPolicy": { 
    "IgnoreDomainHintForDomains": [ "testDomain.com" ], 
    "RespectDomainHintForDomains": [], 
    "IgnoreDomainHintForApps": [], 
    "RespectDomainHintForApps": ["app1-clientID-Guid", "app2-clientID-Guid] 
} 
```

3. Continúe expandiendo la implementación de la directiva a los nuevos dominios y recopile más comentarios.

```json
 "DomainHintPolicy": { 
    "IgnoreDomainHintForDomains": [ "testDomain.com", "otherDomain.com", "anotherDomain.com"], 
    "RespectDomainHintForDomains": [], 
    "IgnoreDomainHintForApps": [], 
    "RespectDomainHintForApps": ["app1-clientID-Guid", "app2-clientID-Guid] 
} 
```

4. Complete la implementación: diríjase a todos los dominios, excepto a aquellos que deban seguir acelerados:

```json
 "DomainHintPolicy": { 
    "IgnoreDomainHintForDomains": [ "*" ], 
    "RespectDomainHintForDomains": ["guestHandlingDomain.com"], 
    "IgnoreDomainHintForApps": [], 
    "RespectDomainHintForApps": ["app1-clientID-Guid", "app2-clientID-Guid] 
} 
```

Una vez completado el paso 4, todos los usuarios, excepto los de `guestHandlingDomain.com`, pueden iniciar sesión en la página de inicio de sesión de Azure AD, incluso cuando las sugerencias de dominio podrían provocar una aceleración automática en un IDP federado.  La excepción a esto es si la aplicación que solicita el inicio de sesión es una de las exentas; para esas aplicaciones, todas las sugerencias de dominio seguirán siendo aceptadas.

## <a name="configuring-policy-through-graph-explorer"></a>Configuración de la Directiva mediante el Probador de Graph

Establezca la [directiva de HRD](/graph/api/resources/homeRealmDiscoveryPolicy) como de costumbre, con Microsoft Graph.  

1. Conceda el permiso Policy.ReadWrite.ApplicationConfiguration en el [Probador de Graph](https://developer.microsoft.com/graph/graph-explorer).  
1. Use la dirección URL `https://graph.microsoft.com/v1.0/policies/homeRealmDiscoveryPolicies`.
1. Publique mediante POST la nueva directiva en esta dirección URL o aplique una revisión PATCH a `/policies/homerealmdiscoveryPolicies/{policyID}` si sobrescribe una existente.

Contenido de POST o PATCH:

```json
{
    "displayName":"Home Realm Discovery Domain Hint Exclusion Policy",
    "definition":[
        "{\"HomeRealmDiscoveryPolicy\" : {\"DomainHintPolicy\": { \"IgnoreDomainHintForDomains\": [ \"Contoso.com\" ], \"RespectDomainHintForDomains\": [], \"IgnoreDomainHintForApps\": [\"sample-guid-483c-9dea-7de4b5d0a54a\"], \"RespectDomainHintForApps\": [] } } }"
    ],
    "isOrganizationDefault":true
}
```

Asegúrese de usar barras diagonales para provocar el escape de la sección JSON de `Definition` al usar Graph.  

`isOrganizationDefault` debe ser true, pero el valor de displayName y la definición pueden cambiar.

## <a name="next-steps"></a>Pasos siguientes

* [Habilitación del inicio de sesión con clave de seguridad sin contraseña](../authentication/howto-authentication-passwordless-security-key.md)
* [Habilitación del inicio de sesión sin contraseña con la aplicación Microsoft Authenticator](../authentication/howto-authentication-passwordless-phone.md)