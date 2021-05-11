---
title: Alcanzar el nivel de seguridad NIST AAL3 con Azure Active Directory
description: Instrucciones para alcanzar el nivel 3 de garantía del autenticador (AAL 3) de NIST con Azure Active Directory.
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
ms.openlocfilehash: 1d7ebba72877d5c4e5df111ab574e28156003bf9
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294212"
---
# <a name="achieving-nist-authenticator-assurance-level-3-with-the-azure-active-directory"></a>Alcanzar el nivel 3 de garantía del autenticador de NIST con Azure Active Directory

Este artículo le guía para lograr el nivel 3 de garantía del autenticación de National Institute of Standards and Technology (NIST AAL). Recursos que puede ser interesante ver antes de intentar alcanzar AAL 3:
* [Introducción a NIST](nist-overview.md): conozca los distintos niveles de AAL.
* [Conceptos básicos de autenticación](nist-authentication-basics.md): terminología importante y tipos de autenticación.
* [Tipos de autenticadores de NIST](nist-authenticator-types.md): conozca cada uno de los tipos de autenticador.
* [AAL de NIST](nist-about-authenticator-assurance-levels.md): los componentes de las AAL y cómo se asignan a ellos los métodos de autenticación de Microsoft Azure Active Directory.

## <a name="permitted-authenticator-types"></a>Tipos de autenticadores permitidos
Microsoft ofrece métodos de autenticación que le permiten satisfacer los tipos de autenticadores de NIST necesarios. Consulte nuestras recomendaciones.
 

| Métodos de autenticación de Azure AD| Tipo de autenticador de NIST |
| - | -|
| **Métodos recomendados**|    |
| Clave de seguridad FIDO2 **O**<br> Tarjeta inteligente (AD FS) **O**<br>Windows Hello para empresas con TPM de hardware| Hardware criptográfico multifactor |
| **Otros métodos**|   |
| Contraseña **Y**<br>(Unión a Azure AD híbrido con TPM de hardware **O** <br> Unión a Azure AD híbrido con TPM de hardware)| Secreto memorizado **+** hardware criptográfico de un solo factor |
| Contraseña **Y**<br>Hardware de contraseña única de un solo factor (de fabricantes de OTP) **O**<br>Unión a Azure AD híbrido con TPM de software **O** <br> Unión a Azure AD con TPM de software **O**<br> Dispositivo administrado compatible| Secreto memorizado **Y**<br>Hardware de contraseña única de un solo factor **Y**<br>Software criptográfico de un solo factor |

### <a name="our-recommendations"></a>Nuestras recomendaciones 

Recomendamos utilizar un autenticador de hardware criptográfico multifactor para lograr AAL3. La autenticación sin contraseña elimina la mayor superficie de ataque (la contraseña) y ofrece a los usuarios un método simplificado para autenticarse. Si su organización se basa completamente en la nube, se recomienda usar claves de seguridad FIDO2.

Tenga en cuenta que las claves FIDO2 y Windows Hello para empresas no se han validado en el nivel de seguridad FIPS 140 necesario y, como tales, los clientes federales tendrían que realizar una evaluación y valoración de riesgos antes de aceptar estos autenticadores como AAL3.

Para consultar instrucciones detalladas, vea [Planeamiento de una implementación de autenticación sin contraseña en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-passwordless-deployment).

Para obtener más información sobre cómo implementar Windows Hello para empresas, consulte la [guía de implementación de Windows Hello para empresas](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-deployment-guide).

## <a name="fips-140-validation"></a>Validación FIPS 140

### <a name="verifier-requirements"></a>Requisitos del comprobador

Azure AD usa el módulo criptográfico validado FIPS 140 Nivel 1 general de Windows   
en todas sus operaciones criptográficas relacionadas con la autenticación. Por lo tanto, es un comprobador compatible con FIPS 140.

### <a name="authenticator-requirements"></a>Requisitos del autenticador

Los autenticadores de hardware criptográfico de un solo factor y multifactor tienen requisitos de autenticador diferentes. 

Es necesario que los autenticadores de hardware criptográfico de un solo factor tengan: 

* FIPS 140 Nivel 1 general (o superior)

* FIPS 140 Nivel 3 Seguridad física (o superior)

Los dispositivos unidos a Azure AD y a Azure AD híbrido cumplen este requisito cuando se produce lo siguiente: 

* Ejecuta [Windows en un modo de funcionamiento aprobado por FIPS 140](https://docs.microsoft.com/windows/security/threat-protection/fips-140-validation). 

* En una máquina con un TPM que sea FIPS 140 Nivel 1 general (o superior) con FIPS 140 Nivel 3 Seguridad física. 

   * Busque TPM compatibles mediante la búsqueda de "Módulo de plataforma segura" y "TPM" en [Programa de validación de módulos criptográficos](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search).

Consulte con el proveedor de dispositivos móviles para obtener información sobre su cumplimiento con FIPS 140.

Es necesario que los autenticadores de **hardware criptográfico multifactor** tengan: 

* FIPS 140 Nivel 2 general (o superior)

* FIPS 140 Nivel 3 Seguridad física (o superior)

Las claves de seguridad FIDO2, las tarjetas inteligentes y Windows Hello para empresas pueden ayudarle a cumplir estos requisitos.

* Las claves FIDO2 son una innovación muy reciente y, como tal, todavía están en proceso de la certificación de FIPS.

* Las tarjetas inteligentes son una tecnología probada con varios productos de proveedores que cumplen los requisitos de FIPS.

   * Obtenga más información en el [Programa de validación de módulos criptográficos](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search).

**Windows Hello para empresas**

FIPS 140 requiere que todo el límite criptográfico, incluido el software, el firmware y el hardware, esté en el ámbito de la evaluación. Los sistemas operativos Windows son plataformas informáticas abiertas que se pueden emparejar con miles de combinaciones de hardware. Por lo tanto, Microsoft no puede mantener certificaciones FIPS para cada combinación. Las siguientes certificaciones individuales de los componentes deben evaluarse como parte de la evaluación de riesgos para usar WHfB como un autenticador de AAL3:

* **Microsoft Windows 10 y Microsoft Windows Server** usan el [perfil de protección aprobado por el Gobierno de EE. UU. para sistemas operativos de uso general, versión 4.2.1](https://www.niap-ccevs.org/Profile/Info.cfm?PPID=442&id=442) de National Information Assurance Partnership (NIAP). NIAP supervisa un programa nacional para evaluar productos de tecnologías de la información (TI) de productos comerciales (COTS) para cumplir los criterios comunes internacionales. 

* La **biblioteca criptográfica de Microsoft Windows** [ha logrado el nivel 1 de FIPS general del Programa de validación de módulos criptográficos (CMVP) de NIST](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/Certificate/3544). El CMVP, un esfuerzo conjunto entre el NIST y el centro canadiense para la ciberseguridad, valida el módulo criptográfico con los estándares FIPS. 

* Elija un **Módulo de plataforma segura (TPM)** que sea FIPS 140 Nivel 2 general y FIPS 140 Nivel 3 Seguridad física. **Como organización, es su responsabilidad asegurarse de que el TPM de hardware que usa satisface las necesidades del nivel de AAL que desea lograr**.   
‎Para determinar qué TPM cumplen los estándares actuales, vaya al [Programa de validación de módulos criptográficos del centro de recursos de seguridad del equipo de NIST](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/validated-modules/Search). En el campo Nombre de módulo, escriba "Módulo de plataforma segura". La lista resultante contiene TPMS de hardware que cumplen los estándares actuales.

## <a name="reauthentication"></a>Reautenticación 

En AAL3, NIST requiere una nueva autenticación cada 12 horas, independientemente de la actividad del usuario, y después de cualquier período de inactividad que dure 15 minutos o más. Se requiere la presentación de ambos factores.

Para cumplir el requisito de reautenticación independientemente de la actividad del usuario, Microsoft recomienda configurar la [frecuencia de inicio de sesión del usuario](https://aka.ms/NIST/38) en 12 horas. 

NIST también permite el uso de controles de compensación para confirmar la presencia del suscriptor:

* El tiempo de espera para la inactividad de la sesión de 15 minutos se puede conseguir bloqueando el dispositivo a nivel del sistema operativo mediante Microsoft System Center Configuration Manager (SCCM), objetos de directiva de grupo (GPO) o Intune. También debe requerir autenticación local para que el suscriptor la desbloquee.

* El tiempo de espera independientemente de la actividad se puede lograr mediante la ejecución de una tarea programada (aprovechando SCCM, GPO o Intune) que bloquea la máquina después de 12 horas, independientemente de la actividad.

## <a name="man-in-the-middle-mitm-resistance"></a>Resistencia de tipo "man in the middle" (MitM) 

Todas las comunicaciones entre el solicitante y Azure AD se realizan a través de un canal protegido autenticado para proporcionar resistencia a los ataques MitM. Así se satisfacen los requisitos de resistencia de MitM para AAL1, AAL2 y AAL3.

## <a name="verifier-impersonation-resistance"></a>Resistencia a la suplantación del comprobador

Todos los métodos de autenticación de Azure AD que cumplen AAL3 utilizan los autenticadores criptográficos que enlazan la salida del autenticador con la sesión específica que se va a autenticar. Para ello, usan una clave privada controlada por el reclamador para la que el comprobador conoce la clave pública. Esto satisface los requisitos de resistencia a la suplantación del comprobador para AAL3.

## <a name="verifier-compromise-resistance"></a>Resistencia al riesgo del comprobador

Todos los métodos de autenticación de Azure AD que cumplen AAL3 usan un autenticador criptográfico que requiere que el comprobador almacene una clave pública correspondiente a una clave privada que el autenticador conserva o que almacene la salida del autenticador esperada con los algoritmos hash validados por FIPS 140. Puede encontrar más detalles en [Consideraciones sobre la seguridad de los datos de Azure AD](https://aka.ms/AADDataWhitepaper).

## <a name="replay-resistance"></a>Resistencia de reproducción

Todos los métodos de autenticación de Azure AD en AAL3 utilizan nonce o desafíos y son resistentes a los ataques de reproducción, ya que el comprobador detectará fácilmente las transacciones de autenticación reproducidas, puesto que no contendrán los datos apropiados de nonce o temporalidad.

## <a name="authentication-intent"></a>Intención de autenticación

El objetivo de la intención de autenticación es que sea más difícil que los autenticadores físicos conectados directamente (por ejemplo, dispositivos criptográficos multifactor) se usen sin el conocimiento del sujeto, por ejemplo, mediante malware en el punto de conexión.

NIST permite usar controles de compensación para mitigar el riesgo de malware. Cualquier dispositivo compatible con Intune que ejecuta la Protección del sistema de Windows Defender y ATP de Windows Defender cumple este requisito de mitigación.

## <a name="next-steps"></a>Pasos siguientes 

[Introducción a NIST](nist-overview.md)

[Más información sobre los AAL](nist-about-authenticator-assurance-levels.md)

[Conceptos básicos sobre autenticación](nist-authentication-basics.md)

[Tipos de autenticadores de NIST](nist-authenticator-types.md)

[Alcanzar el nivel AAL1 de NIST con Azure AD](nist-authenticator-assurance-level-1.md)

[Alcanzar el nivel AAL2 de NIST con Azure AD](nist-authenticator-assurance-level-2.md)

[Alcanzar el nivel AAL3 de NIST con Azure AD](nist-authenticator-assurance-level-3.md) 
