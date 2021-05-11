---
title: Niveles de garantía del autenticador de NIST con Azure Active Directory
description: Información general sobre los niveles de garantía del autenticador aplicados a Azure Active Directory
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
ms.openlocfilehash: 08660ae2d9f3b3561665489e10b29b51a9c2d512
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294171"
---
# <a name="about-authenticator-assurance-levels"></a>Acerca de los niveles de garantía del autenticador

El Instituto Nacional de Estándares y Tecnología (NIST, por sus siglas en inglés) desarrolla los requisitos técnicos para las agencias federales de Estados Unidos que implementan soluciones de identidad. La [publicación especial de NIST 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html) define las directrices técnicas para la implementación de la autenticación digital. Esto se lleva a cabo con un marco de niveles de garantía del autenticador (AAL). Dichos niveles caracterizan la solidez de la autenticación de una identidad digital. Las instrucciones también cubren la administración del ciclo de vida de los autenticadores, incluida la revocación. 

El estándar incluye los requisitos de AAL para 11 categorías de requisitos:

* Tipos de autenticadores permitidos

* Nivel de comprobación del estándar federal de procesamiento de información 140 (FIPS 140) (los requisitos de FIPS 140 se cumplen según [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) o revisiones más recientes)

* Reautenticación

* Controles de seguridad

* Resistencia de tipo "man in the middle" (MitM)

* Resistencia a la suplantación del comprobador (resistencia a la suplantación de identidad [phishing])

* Resistencia al riesgo del comprobador

* Resistencia de reproducción

* Intención de autenticación

* Directiva de retención de registros

* Controles de privacidad

## <a name="applying-nist-aals-in-your-environment"></a>Aplicación de AAL de NIST en el entorno

> [!TIP]
> Se recomienda cumplir al menos el nivel AAL 2, salvo que por razones empresariales, estándares del sector o requisitos de cumplimiento deba cumplir con AAL 3.

En general, no se recomienda el nivel AAL 1 porque acepta soluciones de solo contraseña, y las contraseñas son la forma de autenticación más fácil de poner en peligro. Consulte [Su contra$eña no importa](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984). 

Aunque NIST no requiere de resistencia a la suplantación del comprobador (también conocida como suplantación de las credenciales) hasta el nivel AAL 3, se recomienda encargarse de esta amenaza en todos los niveles. Puede seleccionar autenticadores que proporcionen resistencia a la suplantación del comprobador, por ejemplo al requerir dispositivos unidos a Azure AD o unidos a Azure AD híbrido. Si usa Office 365, puede usar la Protección contra amenazas avanzada de Office 365 y, en concreto, las [directivas de protección contra suplantación de identidad (anti-phishing)](https://docs.microsoft.com/microsoft-365/security/office-365-security/set-up-anti-phishing-policies?view=o365-worldwide).

A medida que evalúa el nivel AAL de NIST adecuado para su organización, puede plantearse si toda la organización debe cumplir los estándares de NIST, o si hay grupos específicos de usuarios y recursos que se pueden separar, y cuáles configuraciones de AAL de NIST se aplican solo a un grupo específico de usuarios y recursos. 

## <a name="security-controls-privacy-controls-records-retention-policy"></a>Controles de seguridad, controles de privacidad y directiva de retención de registros

Azure y Azure Government han obtenido una aprobación inicial P-ATO (Provisional Authority to Operate) en el [nivel de alto impacto de la publicación especial 800-53 de NIST](https://nvd.nist.gov/800-53/Rev4/impact/high) de la Junta de autorizaciones conjuntas, el grado más alto para la acreditación de FedRAMP, que autoriza el uso de Azure y Azure Government para procesar datos altamente confidenciales.

Estas certificaciones de Azure y Azure Government cumplen los controles de seguridad, los controles de privacidad y los requisitos de la directiva de retención de registros para los niveles AAL 1, AAL 2 y AAL 3.

La auditoría de FedRAMP para Azure y Azure Government ha incluido el sistema de administración de seguridad de la información, que abarca la infraestructura, el desarrollo, las operaciones, la administración y la compatibilidad con servicios dentro del ámbito. Una vez que se concede un P-ATO, un proveedor de servicios en la nube aún requiere una autorización (ATO) de cualquiera de las agencias gubernamentales con la que trabaja. En el caso de Azure, una agencia gubernamental, o las organizaciones que trabajan con ella, pueden usar la aprobación P-ATO de Azure en su propio proceso de autorización de seguridad y confiar en ella como la base para emitir una autorización ATO de agencia que también cumpla los requisitos de FedRAMP.

Azure sigue siendo compatible con más servicios en el nivel de alto impacto de FedRAMP High que cualquier otro proveedor de nube. Y aunque FedRAMP High en la nube pública de Azure satisfaga las necesidades de muchos clientes de la Administración Pública de Estados Unidos, las agencias con requisitos más estrictos seguirán confiando en Azure Government, que proporciona medidas de seguridad adicionales, como una revisión más exhaustiva del personal. Microsoft incluye en una lista todos los servicios públicos de Azure que están disponibles actualmente en Azure Government al límite de FedRAMP High, así como los servicios planeados para el año en curso.

Además, Microsoft está totalmente comprometido con [proteger y administrar los datos de los clientes](https://www.microsoft.com/trust-center/privacy/data-management) mediante directivas de retención de registros establecidas con claridad. Como empresa global con clientes en casi todos los países del mundo, Microsoft tiene una sólida cartera de cumplimiento para ayudar a los clientes. Para ver una lista completa de las ofertas de cumplimiento, visite [Oferta de cumplimiento de Microsoft](https://docs.microsoft.com/compliance/regulatory/offering-home). 

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md) 