---
title: Protección de cuentas de servicio basadas en el usuario | Azure Active Directory
description: Una guía para proteger las cuentas de servicio basadas en usuario.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 2/15/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 79e0dc10aa9cb5fb67812cca31d2cd892afcccbe
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208102"
---
# <a name="secure-user-based-service-accounts-in-active-directory"></a>Protección de cuentas de servicio basadas en el usuario

El uso de cuentas de usuario locales es el enfoque tradicional para ayudar a proteger los servicios que se ejecutan en Windows. Use estas cuentas como último recurso cuando el servicio no admita cuentas de servicio administradas de grupo (gMSA) ni cuentas de servicio administradas independientes (sMSA). Para obtener información sobre cómo seleccionar el mejor tipo de cuenta para usar, consulte [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md). 

También puede querer investigar si puede mover el servicio para usar una cuenta de servicio de Azure, como una identidad administrada o una entidad de servicio. 

Puede crear cuentas de usuario locales para proporcionar un contexto de seguridad para los servicios y permisos que las cuentas necesitan para acceder a los recursos locales y de red. Las cuentas de usuario locales requieren la administración manual de contraseñas, de forma similar a cualquier otra cuenta de usuario de Active Directory. Los administradores de servicios y dominios deben observar los procesos seguros de administración de contraseñas para ayudar a mantener protegidas estas cuentas.

Cuando cree una cuenta de usuario como cuenta de servicio, úsela solo para un único servicio. Asígnele un nombre que deje claro que se trata de una cuenta de servicio y para qué servicio está destinada. 

## <a name="benefits-and-challenges"></a>Ventajas y desafíos

Las cuentas de usuario locales pueden proporcionar importantes ventajas. Son el tipo de cuenta más versátil para usar con los servicios. Todas las directivas que rigen las cuentas de usuario normales pueden controlar las cuentas de usuario que se usan como cuentas de servicio. Pero debe usarlas solo si no puede usar una MSA. Evalúe también si una cuenta de equipo es una mejor opción. 

Los desafíos asociados con el uso de cuentas de usuario locales se resumen en la tabla siguiente:

| Desafío | Mitigación |
| - | - |
| La administración de contraseñas es un proceso manual que puede debilitar la seguridad y provocar tiempo de inactividad del servicio.| <li>Asegúrese de que la complejidad de las contraseñas y el cambio de contraseñas se rijan por un proceso sólido que garantice actualizaciones periódicas con contraseñas seguras.<li>Coordine el cambio de contraseñas con una actualización de contraseñas en el servicio, lo que ayudará a reducir el tiempo de inactividad del servicio. |
| La identificación de las cuentas de usuario locales que actúan como cuentas de servicio puede ser difícil. | <li>Documente y mantenga registros de las cuentas de servicio que hay implementadas en su entorno.<li>Realice un seguimiento del nombre de cuenta y de los recursos a los que tienen acceso asignado.<li>Considere la posibilidad de agregar un prefijo "svc-" a todas las cuentas de usuario que se usan como cuentas de servicio. |
| | |


## <a name="find-on-premises-user-accounts-used-as-service-accounts"></a>Búsqueda de cuentas de usuario locales que se utilizan como cuentas de servicio

Las cuentas de usuario locales son como cualquier otra cuenta de usuario de Active Directory. Puede ser difícil encontrar estas cuentas, ya que no hay ningún atributo único de una cuenta de usuario que la identifique como cuenta de servicio. 

Se recomienda crear una convención de nomenclatura fácilmente identificable para cualquier cuenta de usuario que use como cuenta de servicio. Por ejemplo, podría agregar "svc-" como prefijo y nombrar el servicio como "svc-HRDataConnector".

Puede usar algunos de los criterios siguientes para buscar estas cuentas de servicio. Sin embargo, es posible que este enfoque no encuentre todas las cuentas, por ejemplo:

* Cuentas de confianza para la delegación.  
* Cuentas con nombres de entidad de seguridad de servicio.  
* Cuentas con contraseñas configuradas para que no expiren nunca.

Para buscar las cuentas de usuario locales que ha creado para los servicios, puede ejecutar los siguientes comandos de PowerShell.

Para buscar cuentas de confianza para la delegación:

```PowerShell

Get-ADObject -Filter {(msDS-AllowedToDelegateTo -like '*') -or (UserAccountControl -band 0x0080000) -or (UserAccountControl -band 0x1000000)} -prop samAccountName,msDS-AllowedToDelegateTo,servicePrincipalName,userAccountControl | select DistinguishedName,ObjectClass,samAccountName,servicePrincipalName, @{name='DelegationStatus';expression={if($_.UserAccountControl -band 0x80000){'AllServices'}else{'SpecificServices'}}}, @{name='AllowedProtocols';expression={if($_.UserAccountControl -band 0x1000000){'Any'}else{'Kerberos'}}}, @{name='DestinationServices';expression={$_.'msDS-AllowedToDelegateTo'}}

```

Para buscar cuentas que tengan nombres de entidad de seguridad de servicio:

```PowerShell

Get-ADUser -Filter * -Properties servicePrincipalName | where {$_.servicePrincipalName -ne $null}

```

Para buscar cuentas con contraseñas configuradas para que no expiren nunca:

```PowerShell

Get-ADUser -Filter * -Properties PasswordNeverExpires | where {$_.PasswordNeverExpires -eq $true}

```

También puede auditar el acceso a recursos confidenciales y archivar los registros de auditoría en un sistema de administración de eventos e información de seguridad (SIEM). Si usa sistemas como Azure Log Analytics o Azure Sentinel, puede buscar y analizar cuentas de servicio.

## <a name="assess-the-security-of-on-premises-user-accounts"></a>Evaluación de la seguridad de las cuentas de usuario locales

Puede evaluar la seguridad de las cuentas de usuario locales que se usan como cuentas de servicio mediante los siguientes criterios:

* ¿Cuál es la directiva de administración de contraseñas?  
* ¿La cuenta pertenece a algún grupo con privilegios?  
* ¿Tiene la cuenta permisos de lectura y escritura a recursos importantes?

### <a name="mitigate-potential-security-issues"></a>Mitigación de posibles problemas de seguridad

En la tabla siguiente se resumen los posibles problemas de seguridad y las mitigaciones para las cuentas de usuario locales:

| Problema de seguridad | Mitigación |
| - | - |
| Administración de contraseñas.| <li>Asegúrese de que la complejidad de las contraseñas y el cambio de las contraseñas se rijan por un proceso sólido que garantice actualizaciones periódicas y requisitos de contraseña seguros.<li>Coordine los cambios de contraseñas con una actualización de contraseñas para minimizar el tiempo de inactividad del servicio. |
| La cuenta pertenece a grupos con privilegios.| <li>Revise las pertenencias a grupos.<li>Quite la cuenta de los grupos con privilegios.<li>Conceda a la cuenta solo los derechos y permisos necesarios para ejecutar su servicio (consulte al proveedor de servicios). Por ejemplo, es posible que pueda denegar el inicio de sesión localmente o denegar el inicio de sesión interactivo. |
| La cuenta tiene permisos de lectura y escritura para recursos confidenciales.| <li>Audite el acceso a recursos confidenciales.<li>Archive los registros de auditoría en un SIEM (Azure Log Analytics o Azure Sentinel) para su análisis.<li>Corrija los permisos de recursos si se detecta un nivel de acceso no deseado. |
| | |


## <a name="move-to-more-secure-account-types"></a>Migración a tipos de cuenta más seguros

Microsoft no recomienda que se usen cuentas de usuario locales como cuentas de servicio. Para cualquier servicio que use este tipo de cuenta, evalúe si se puede configurar para usar una cuenta gMSA o sMSA en su lugar.

Además, evalúe si el propio servicio se puede mover a Azure para poder usar tipos de cuentas de servicio más seguras. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)  
* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)  
* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)  
* [Protección de cuentas de equipo](service-accounts-computer.md)  
* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)

 
