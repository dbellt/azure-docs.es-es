---
title: Directivas de acceso de Azure Video Analyzer
description: En este artículo se explica cómo Azure Video Analyzer usa tokens JWT en directivas de acceso para proteger vídeos.
ms.topic: reference
ms.date: 05/10/2021
ms.openlocfilehash: 4c0a4cf4ed02f6c1ddbcecd2351e8ffa0f4ad887
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388494"
---
# <a name="access-policies"></a>Directivas de acceso

Las directivas de acceso definen los permisos y la duración del acceso a un recurso de vídeo determinado de Video Analyzer. Estas directivas de acceso permiten un mayor control y flexibilidad al permitir que los tokens JWT de terceros (clientes que no son de AAD) proporcionen autorización a las API de cliente que habilitan: 

- acceso a metadatos de vídeo. 
- acceso a streaming de vídeos. 

## <a name="access-policy-definition"></a>Definición de una directiva de acceso

```json
"name": "accesspolicyname1", 
"properties": { 
    "role": "Reader", 
    "authentication": { 
        "@type": "#Microsoft.VideoAnalyzer.JwtAuthentication", 
        "issuers": [ 
            "issuer1", 
            "issuer2" 
        ], 
        "audiences": [ 
            "audience1" 
        ], 
        "claims": [ 
            { 
                "name":"claimname1", 
                "value":"claimvalue1" 
            }, 
            { 
                "name":"claimname2", 
                "value":"claimvalue2" 
            } 
        ], 
        "keys": [ 
            { 
                "@type": "#Microsoft.VideoAnalyzer.RsaTokenKey", 
                "alg": "RS256", 
                "kid": "123", 
                "n": "YmFzZTY0IQ==", 
                "e": "ZLFzZTY0IQ==" 
            }, 
            { 
                "@type": "#Microsoft.VideoAnalyzer.EccTokenKey", 
                "alg": "ES256", 
                "kid": "124", 
                "x": "XX==", 
                "y": "YY==" 
            } 
        ] 
    } 
} 
```

> [!NOTE] 
> Solo se requiere un tipo de clave. 

### <a name="roles"></a>Roles

Actualmente, solo se admite el rol de lector.

### <a name="issuer-matching-rules"></a>Reglas de coincidencia del emisor

Se pueden especificar varios problemas en la directiva, mientras que un solo emisor se puede especificar en el token.  El emisor coincide si el emisor del token se encuentra entre los emisores especificados en la directiva.

### <a name="audience-matching-rules"></a>Reglas de coincidencia de audiencia

Si el valor de audiencia es ${System.Runtime.BaseResourceUrlPattern} para el recurso de vídeo, la audiencia que se proporciona en el token JWT debe coincidir con la dirección URL del recurso base. Si no es así, la audiencia del token debe coincidir con la audiencia de la directiva de acceso.

### <a name="claims-matching-rules"></a>Reglas de coincidencia de notificaciones

Se pueden especificar varias notificaciones en la directiva de acceso y en el token JWT.  Todas las notificaciones de una directiva de acceso debe proporcionarse en el token para pasar la validación; sin embargo, el token JWT puede tener notificaciones adicionales que no aparecen en la directiva de acceso.

### <a name="keys"></a>Claves

Se admiten dos tipos de claves: RSA y ECC.

[RSA](https://wikipedia.org/wiki/RSA_(cryptosystem))

* @type- \#Microsoft.VideoAnalyzer.RsaTokenKey
* alg: algoritmo.  Puede ser 256, 384 o 512 
* kid: id. clave
* n: módulo
* e: exponente público 

[ECC](https://wikipedia.org/wiki/Elliptic-curve_cryptography)        

* @type- \#Microsoft.VideoAnalyzer.EccTokenKey
* alg: algoritmo.  Puede ser 256, 384 o 512
* kid: id. clave
* x: valor de coordenada.
* y: valor de coordenada.

### <a name="token-validation-process"></a>Proceso de validación de tokens

Los clientes deben crear sus propios tokens JWT y se validarán mediante el método siguiente:

- En la lista de directivas que coinciden con el id. clave, validamos:
  - Firma de token
  - Expiración del token
  - Emisor
  - Público
  - Notificaciones adicionales

### <a name="policy-audience-and-token-matching-examples"></a>Ejemplos de coincidencia de tokens y audiencia de la directiva:

| **Audiencia de la directiva**                      | URL solicitada                         | Dirección URL de token                            | Resultado |
| ---------------------------------------- | ------------------------------------- | ------------------------------------ | ------ |
| (Cualquier literal)                            | (CUALQUIERA)                                 | (Coincide)                              | Conceder  |
| (Cualquier literal)                            | (CUALQUIERA)                                 | (No coincide)                          | Denegar   |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos                   | https://fqdn/videos/*                | Conceder  |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos                   | https://fqdn/videos/{videoName}      | Denegar   |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{videoName}       | https://fqdn/vid*                    | Conceder  |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{videoName}       | https://fqdn/videos/*                | Conceder  |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{videoName}       | https://fqdn/videos/{baseVideoName}* | Conceder  |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{videoName}       | https://fqdn/videos/{videoName}      | Conceder  |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{videoName}Suffix | https://fqdn/videos/{videoName}      | Denegar   |
| ${System.Runtime.BaseResourceUrlPattern} | https://fqdn/videos/{otherVideoName}  | https://fqdn/videos/{videoName}      | Denegar   |

> [!NOTE]  
> Video Analyzer admite un máximo de 20 directivas.  ${System.Runtime.BaseResourceUrlPattern} permite una mayor flexibilidad para acceder a recursos específicos mediante una directiva de acceso y varios tokens.  A continuación, estos tokens permiten el acceso a distintos recursos de Video Analyzer en función de la audiencia. 

## <a name="creating-an-access-policy"></a>Creación de una directiva de acceso

Hay dos maneras de crear una directiva de acceso.

### <a name="in-the-azure-portal"></a>En Azure Portal

1. Inicie sesión en Azure Portal y vaya al grupo de recursos donde se encuentra la cuenta de Video Analyzer.
2. Seleccione el recurso de Video Analyzer.
3. En Video Analyzer, seleccione Directivas de acceso

   :::image type="content" source="./media/access-policies/access-policies-menu.png" alt-text="Menú Directivas de acceso en Azure Portal":::
4. Haga clic en Nuevo y escriba lo siguiente:

   - Nombre de directiva de acceso: cualquier nombre
   - Emisor: debe coincidir con el emisor del token JWT. 
   - Audiencia: audiencia del token JWT: ${System.Runtime.BaseResourceUrlPattern} es el valor predeterminado. 
   - Tipo de clave: kty 
   - Algoritmo: alg
   - Id. clave: kid 
   - Valor N/X 
   - Valor E/Y 

   :::image type="content" source="./media/access-policies/access-policies-portal.png" alt-text="Directiva de acceso en Azure Portal":::
5. Haga clic en `Save`.

### <a name="create-access-policy-via-api"></a>Creación de una directiva de acceso a través de la API

Consulte API de Azure Resource Manager (ARM) 

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
