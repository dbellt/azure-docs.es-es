---
title: Novedades de Azure Cache for Redis
description: Actualizaciones recientes para Azure Cache for Redis
author: yegu-ms
ms.service: cache
ms.topic: reference
ms.date: 09/28/2020
ms.author: yegu
ms.openlocfilehash: 21ee51ac53147482210aa0bda410e11d12e5f479
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111413796"
---
# <a name="whats-new-in-azure-cache-for-redis"></a>Novedades de Azure Cache for Redis

## <a name="azure-tls-certificate-change"></a>Cambio en los certificados TLS de Azure

Microsoft está actualizando los servicios de Azure para que usen los certificados de servidor TLS de un conjunto diferente de entidades de certificación (CA). Este cambio se implementa en fases desde el 13 de agosto de 2020 hasta el 26 de octubre de 2020 (estimado). Azure realiza este cambio porque [los certificados de entidad de certificación actuales no cumplen uno de los requisitos de la base de referencia del foro CA/Browser](https://bugzilla.mozilla.org/show_bug.cgi?id=1649951). El problema se comunicó el 1 de julio de 2020 y se aplica a varios proveedores populares de infraestructura de clave pública (PKI) en todo el mundo. La mayoría de los certificados TLS que utilizan los servicios de Azure actuales proceden de la PKI de *Baltimore CyberTrust Root*. El servicio de Azure Cache for Redis seguirá encadenado a Baltimore CyberTrust Root. Sin embargo, son sus nuevas entidades de certificación intermedias (ICA) las que emitirán los certificados de servidor TLS a partir del 12 de octubre de 2020.

> [!NOTE]
> Este cambio se limita a los servicios de las [regiones públicas de Azure](https://azure.microsoft.com/global-infrastructure/geographies/). Excluye las nubes soberanas (por ejemplo, China) o gubernamentales.
>
>

### <a name="does-this-change-affect-me"></a>¿Me afecta este cambio?

Esperamos que la mayoría de los clientes de Azure Cache for Redis no se vea afectada por el cambio. La aplicación puede resultar afectada si especifica explícitamente una lista de certificados aceptables, una práctica conocida como "asignación de certificados". Si está asignada a un certificado intermedio o de hoja en lugar de a Baltimore CyberTrust Root, debe **tomar medidas inmediatas** para cambiar la configuración del certificado.

En la siguiente tabla se proporciona información acerca de los certificados que se están implementando. En función del certificado que utilice la aplicación, es posible que deba actualizarlo a fin de evitar la pérdida de conectividad para la instancia de Azure Cache for Redis.

| Tipo de entidad de certificación | Current | Tras la implementación (12 de octubre de 2020) | Acción |
| ----- | ----- | ----- | ----- |
| Root | Huella digital: d4de20d05e66fc53fe1a50882c78db2852cae474<br><br> Expiración: Lunes, 12 de mayo de 2025, 4:59:00 p.m.<br><br> Nombre del firmante:<br> CN = Baltimore CyberTrust Root<br> OU = CyberTrust<br> O = Baltimore<br> C = IE | Sin cambios | Ninguno |
| Intermediarios | Huellas digitales:<br> CN = Microsoft IT TLS CA 1<br> Thumbprint: 417e225037fbfaa4f95761d5ae729e1aea7e3a42<br><br> CN = Microsoft IT TLS CA 2<br> Thumbprint: 54d9d20239080c32316ed9ff980a48988f4adf2d<br><br> CN = Microsoft IT TLS CA 4<br> Thumbprint: 8a38755d0996823fe8fa3116a277ce446eac4e99<br><br> CN = Microsoft IT TLS CA 5<br> Thumbprint: Ad898ac73df333eb60ac1f5fc6c4b2219ddb79b7<br><br> Expiración: ‎Viernes, ‎20 de ‎mayo de ‎2024 a las 5:52:38 a.m.<br><br> Nombre del firmante:<br> OU = Microsoft IT<br> O = Microsoft Corporation<br> L = Redmond<br> S = Washington<br> C = US<br> | Huellas digitales:<br> CN = Microsoft RSA TLS CA 01<br> Thumbprint: 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a<br><br> CN = Microsoft RSA TLS CA 02<br> Huella digital: b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75<br><br> Expiración: ‎Martes, ‎8 de ‎octubre de ‎2024 12:00:00 a.m.<br><br> Nombre del firmante:<br> O = Microsoft Corporation<br> C = US<br> | Requerido |

### <a name="what-actions-should-i-take"></a>¿Qué medidas debo tomar?

Si la aplicación utiliza el almacén de certificados del sistema operativo o asigna la raíz de Baltimore entre otros, no es necesario realizar ninguna acción. 

Si la aplicación asigna cualquier certificado TLS intermedio o de hoja, se recomienda asignar las raíces siguientes:

| Certificado | Huella digital |
| ----- | ----- |
| [Baltimore Root CA](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 |
| [Digicert Global Root G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |

> [!TIP]
> Se espera que los certificados intermedio y de hoja cambien con frecuencia. Se recomienda no tener una dependencia con ellos. En su lugar, asigne la aplicación a un certificado raíz, ya que se implementa con menos frecuencia.
>
>

Para continuar con la asignación de certificados intermedios, agregue lo siguiente a la lista de certificados intermedios asignados, que incluye algunos más para minimizar futuros cambios:

| Nombre común de la CA | Huella digital |
| ----- | ----- |
| [Microsoft RSA TLS CA 01](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2001.crt) | 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a |
| [Microsoft RSA TLS CA 02](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2002.crt) | b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75 |
| [Microsoft Azure TLS Issuing CA 01](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2001.cer) | 2f2877c5d778c31e0f29c7e371df5471bd673173 |
| [Microsoft Azure TLS Issuing CA 02](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2002.cer) | e7eea674ca718e3befd90858e09f8372ad0ae2aa |
| [Microsoft Azure TLS Issuing CA 05](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2005.cer) | 6c3af02e7f269aa73afd0eff2a88a4a1f04ed1e5 |
| [Microsoft Azure TLS Issuing CA 06](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.cer) | 30e01761ab97e59a06b41ef20af6f2de7ef4f7b0 |

Si la aplicación valida el certificado en el código, tiene que modificarlo para que reconozca las propiedades (por ejemplo, emisores, huella digital) de los certificados recién asignados. Esta comprobación adicional debe cubrir todos los certificados asignados para que estén mejor preparados para el futuro.

## <a name="next-steps"></a>Pasos siguientes

Si tiene más preguntas, póngase en contacto con nosotros a través del departamento de [soporte técnico](https://azure.microsoft.com/support/options/).  
