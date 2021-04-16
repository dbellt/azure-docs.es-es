---
title: Residencia de los datos de autenticación multifactor de Azure AD
description: Conozca los datos personales y de la organización que almacena la autenticación multifactor de Azure AD relativos a usted y sus usuarios, y los datos que permanecen en el país o región de origen.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: inbarc
ms.collection: M365-identity-device-management
ms.openlocfilehash: 26c8df67a8cfed92e478caacca1171b7a48fa9ca
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105932903"
---
# <a name="data-residency-and-customer-data-for-azure-ad-multifactor-authentication"></a>Residencia de datos y datos de clientes en la autenticación multifactor de Azure AD

Azure Active Directory (Azure AD) almacena los datos de los clientes en una ubicación geográfica en función de la dirección que una organización proporciona cuando se suscribe a un servicio en línea de Microsoft, como Microsoft 365 o Azure. Para más información sobre dónde se almacenan los datos de cliente, consulte [Dónde se encuentran tus datos](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) de Microsoft Trust Center.

La autenticación multifactor de Azure AD basada en la nube y el Servidor Azure Multi-Factor Authentication procesan y almacenan los personales y de la organización. En este artículo se describe qué datos se almacenan y dónde.

El servicio de autenticación multifactor de Azure AD tiene centros de datos en Estados Unidos, Europa y Asia Pacífico. Las siguientes actividades se originan de los centros de datos regionales, excepto donde se indique:

* La autenticación multifactor mediante llamadas telefónicas procede de centros de datos de Estados Unidos y se enruta mediante proveedores globales.
* Las solicitudes de autenticación de usuario de uso general de otras regiones se procesan actualmente en función de la ubicación del usuario.
* Las notificaciones push que usan la aplicación Microsoft Authenticator se procesan actualmente en centros de datos regionales según la ubicación del usuario. Los servicios de dispositivo específicos del fabricante, como Apple Push Notification Service, pueden estar fuera de la ubicación del usuario.

## <a name="personal-data-stored-by-azure-ad-multifactor-authentication"></a>Datos personales almacenados por la autenticación multifactor de Azure AD

Los datos personales son información de nivel de usuario que concierne a una persona específica. Los siguientes almacenes de datos contienen información personal:

* Usuarios bloqueados
* Usuarios omitidos
* Solicitudes de cambio de token de dispositivo de Microsoft Authenticator
* Informes de actividad de autenticación multifactor: almacene la actividad de autenticación multifactor de los componentes locales de autenticación multifactor: extensión de NPS, adaptador de AD FS y servidor MFA.
* Activaciones de Microsoft Authenticator

Esta información se conserva durante 90 días.

La autenticación multifactor de Azure AD no registra datos personales como nombres de usuario, números de teléfono o direcciones IP. Sin embargo, *UserObjectId* identifica los intentos de autenticación para los usuarios. Los datos de registro se almacenan durante 30 días.

### <a name="data-stored-by-azure-ad-multifactor-authentication"></a>Datos almacenados por la autenticación multifactor de Azure AD

En el caso de las nubes públicas de Azure, excepto la autenticación de Azure AD B2C, la extensión NPS y el adaptador de servicios de federación de Active Directory (AD FS) para Windows Server 2016 o 2019, se almacenan los siguientes datos personales:

| Tipo de evento                           | Tipo de almacén de datos |
|--------------------------------------|-----------------|
| Token OATH                           | Registros de autenticación multifactor     |
| SMS unidireccional                          | Registros de autenticación multifactor     |
| Llamada de voz                           | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor<br/>Usuarios bloqueados (si se notificó un fraude) |
| Notificación de Microsoft Authenticator | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor<br/>Usuarios bloqueados (si se notificó un fraude)<br/>Solicitudes de cambio cuando cambia el token de dispositivo de Microsoft Authenticator |

Respecto a Microsoft Azure Government, Microsoft Azure Alemania, Microsoft Azure ofrecido por 21Vianet, la autenticación de Azure AD B2C, la extensión NPS y el adaptador de AD FS para Windows Server 2016 o 2019, se almacenan los siguientes datos personales:

| Tipo de evento                           | Tipo de almacén de datos |
|--------------------------------------|-----------------|
| Token OATH                           | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor |
| SMS unidireccional                          | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor |
| Llamada de voz                           | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor<br/>Usuarios bloqueados (si se notificó un fraude) |
| Notificación de Microsoft Authenticator | Registros de autenticación multifactor<br/>Almacén de datos de informes de actividad de autenticación multifactor<br/>Usuarios bloqueados (si se notificó un fraude)<br/>Solicitudes de cambio cuando cambia el token de dispositivo de Microsoft Authenticator |

### <a name="data-stored-by-azure-multifactor-authentication-server"></a>Datos almacenados por el Servidor Azure Multi-Factor Authentication

Si implementa y ejecuta el Servidor Azure Multi-Factor Authentication, se almacenan los siguientes datos personales.

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft dejó de ofrecer el Servidor Multi-Factor Authentication en las implementaciones nuevas. Los nuevos clientes que quieren solicitar la autenticación multifactor a sus usuarios deben usar la autenticación multifactor de Azure AD basada en la nube. Los clientes actuales que hayan activado el Servidor Multi-Factor Authentication antes del 1 de julio de 2019 podrán descargar la versión y las actualizaciones más recientes, así como generar credenciales de activación como de costumbre.

| Tipo de evento                           | Tipo de almacén de datos |
|--------------------------------------|-----------------|
| Token OATH                           | Registros de autenticación multifactor<br />Almacén de datos de informes de actividad de autenticación multifactor |
| SMS unidireccional                          | Registros de autenticación multifactor<br />Almacén de datos de informes de actividad de autenticación multifactor |
| Llamada de voz                           | Registros de autenticación multifactor<br />Almacén de datos de informes de actividad de autenticación multifactor<br />Usuarios bloqueados (si se notificó un fraude) |
| Notificación de Microsoft Authenticator | Registros de autenticación multifactor<br />Almacén de datos de informes de actividad de autenticación multifactor<br />Usuarios bloqueados (si se notificó un fraude)<br />Solicitudes de cambio cuando el token de dispositivo de Microsoft Authenticator cambia |

## <a name="organizational-data-stored-by-azure-ad-multifactor-authentication"></a>Datos de la organización almacenados por la autenticación multifactor de Azure AD

Los datos de la organización consisten en información de inquilino que puede exponer una opción de configuración o cómo está configurado el entorno. La configuración de inquilino de las siguientes páginas de autenticación de Azure Portal puede almacenar datos de la organización, como los umbrales de bloqueo o información que identifica al autor de llamada de las solicitudes de autenticación telefónica entrantes:

* Bloqueo de cuenta
* Alerta de fraude
* Notificaciones
* Configuración de la llamada telefónica

En el caso del Servidor Azure Multi-Factor Authentication, las siguientes páginas de Azure Portal pueden contener datos de la organización:

* Configuración del servidor
* Omisión por única vez
* Reglas de caché
* Estado del Servidor Multi-Factor Authentication

## <a name="multifactor-authentication-logs-location"></a>Ubicación de registros de autenticación multifactor

En la tabla siguiente se muestra la ubicación de los registros de servicio de las nubes públicas.

| Nube pública| Registros de inicio de sesión | Informe de actividad de la autenticación multifactor        | Registros del servicio de autenticación multifactor       |
|-------------|--------------|----------------------------------------|------------------------|
| Estados Unidos          | Estados Unidos           | Estados Unidos                                     | Estados Unidos                     |
| Europa      | Europa       | Estados Unidos                                     | Europa <sup>2</sup>    |
| Australia   | Australia    | Estados Unidos<sup>1</sup>                         | Australia <sup>2</sup> |

<sup>1</sup>Los registros de código OATH se almacenan en Australia.

<sup>2</sup>Los registros de llamadas de voz del servicio de autenticación multifactor se almacenan en Estados Unidos.

En la tabla siguiente se muestra la ubicación de los registros de servicio de las nubes soberanas.

| Nube soberana                      | Registros de inicio de sesión                         | Informe de actividad de autenticación multifactor (incluye datos personales)| Registros del servicio de autenticación multifactor |
|--------------------------------------|--------------------------------------|-------------------------------|------------------|
| Microsoft Azure Alemania              | Alemania                              | Estados Unidos                            | Estados Unidos               |
| Azure China 21Vianet                 | China                                | Estados Unidos                            | Estados Unidos               |
| Nube de Microsoft Government           | Estados Unidos                                   | Estados Unidos                            | Estados Unidos               |

Los informes de actividad de la autenticación multifactor contienen información personal, como el nombre principal de usuario (UPN) y el número de teléfono completo.

Los registros del servicio de autenticación multifactor se usan para operar el servicio.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre qué datos de los usuarios recopilan la autenticación multifactor de Azure AD basada en la nube y el Servidor Azure Multi-Factor Authentication, consulte [Recopilación de datos de usuario de la autenticación multifactor de Azure AD](howto-mfa-reporting-datacollection.md).
