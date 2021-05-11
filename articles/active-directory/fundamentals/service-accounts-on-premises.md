---
title: Introducción a las cuentas de servicio de Active Directory | Azure Active Directory
description: Introducción a los tipos de cuentas de servicio de Active Directory y cómo protegerlas.
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
ms.openlocfilehash: 4551050cd8606c577edbbdfd85debc06ac12020c
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206500"
---
# <a name="introduction-to-active-directory-service-accounts"></a>Introducción a las cuentas de servicio de Active Directory

Un servicio tiene una identidad de seguridad principal que determina los derechos de acceso para los recursos locales y de red. El contexto de seguridad de un servicio de Microsoft Win32 viene determinado por la cuenta de servicio que se usa para iniciar el servicio. Use una cuenta de servicio para:
* Identificar y autenticar un servicio.
* Iniciar un servicio correctamente.
* Acceder a código o a una aplicación, o bien ejecutarlos.
* Iniciar un proceso. 

## <a name="types-of-on-premises-service-accounts"></a>Tipos de cuentas de servicio locales

Según su caso de uso, puede usar una cuenta de servicio administrada (MSA), una cuenta de equipo o una cuenta de usuario para ejecutar un servicio. Primero tiene que probar un servicio para confirmar que puede usar una cuenta de servicio administrada. Si el servicio puede usar una MSA, tiene que usar una.

### <a name="group-managed-service-accounts"></a>Cuentas de servicio administrada por un grupo

Para los servicios que se ejecutan en su entorno local, use [cuentas de servicio administradas de grupo (gMSA)](service-accounts-group-managed.md) siempre que sea posible. Las cuentas gMSA brindan una solución de identidad única para los servicios que se ejecutan en una granja de servidores o detrás de un equilibrador de carga de red. Las cuentas gMSA también se pueden usar para los servicios que se ejecutan en un único servidor. Para obtener información sobre los requisitos para las cuentas gMSA, consulte [Introducción a las cuentas de servicio administradas de grupo](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts).

### <a name="standalone-managed-service-accounts"></a>Cuentas de servicio administradas independientes

Si no puede usar una cuenta gMSA, use una [cuenta de servicio administrada independiente (sMSA)](service-accounts-standalone-managed.md). Las cuentas sMSA requieren al menos Windows Server 2008 R2. A diferencia de las cuentas gMSA, las cuentas sMSA se ejecutan en un único servidor. Se pueden usar para varios servicios en ese servidor.

### <a name="computer-accounts"></a>Cuentas de equipo

Si no puede usar una cuenta MSA, considere la posibilidad de usar una [cuenta de equipo](service-accounts-computer.md). La cuenta LocalSystem es una cuenta local predefinida que tiene amplios permisos en el equipo local y actúa como la identidad del equipo en la red.

Los servicios que se ejecutan como una cuenta LocalSystem acceden a los recursos de red mediante las credenciales de la cuenta del equipo con el formato <nombre_de_dominio>\\<nombre_de_equipo>. Su nombre predefinido es NT AUTHORITY\SYSTEM. Puede usarlo para iniciar un servicio y proporcionar un contexto de seguridad para ese servicio.

> [!NOTE]
> Cuando se usa una cuenta de equipo, no se puede determinar qué servicio del equipo usa esa cuenta. Por lo tanto, no se puede auditar qué servicio está haciendo cambios. 

### <a name="user-accounts"></a>Cuentas de usuario

Si no puede usar una cuenta MSA, considere la posibilidad de usar una [cuenta de usuario](service-accounts-user-on-premises.md). Una cuenta de usuario puede ser una cuenta de usuario de *dominio* o una cuenta de usuario *local*.

Una cuenta de usuario de dominio permite al servicio sacar el máximo partido de las características de seguridad del servicio de Windows y Microsoft Active Directory Domain Services. El servicio tendrá permisos locales y de red concedidos para la cuenta. También tendrá los permisos de todos los grupos de los que es miembro la cuenta. Las cuentas de servicio de dominio admiten la autenticación mutua Kerberos.

Una cuenta de usuario local (formato de nombre: *.\NombreDeUsuario*) solo existe en la base de datos del Administrador de cuentas de seguridad del equipo host. No tiene un objeto de usuario en Active Directory Domain Services. No se puede autenticar una cuenta local con el dominio. Por lo tanto, un servicio que se ejecuta en el contexto de seguridad de una cuenta de usuario local no tiene acceso a los recursos de red (excepto como usuario anónimo). Los servicios que se ejecutan en el contexto de usuario local no admiten la autenticación mutua Kerberos en la que los clientes autentican el servicio. Por estos motivos, las cuentas de usuario locales no suelen ser adecuadas para los servicios habilitados para el uso de directorios.

> [!IMPORTANT]
> Las cuentas de servicio no deben ser miembros de ningún grupo con privilegios, ya que la pertenencia a grupos con privilegios otorga permisos que pueden suponer un riesgo para la seguridad. Cada servicio debe tener su propia cuenta de servicio para fines de auditoría y seguridad.

## <a name="choose-the-right-type-of-service-account"></a>Elección del tipo de cuenta de servicio correcto


| Criterio| gMSA| sMSA| Cuenta de&nbsp;equipo| Cuenta de&nbsp;usuario |
| - | - | - | - | - |
| La aplicación se ejecuta en un único servidor| Sí| Sí. Use una gMSA si es posible.| Sí. Use una cuenta MSA si es posible.| Sí. Use una cuenta MSA si es posible. |
| La aplicación se ejecuta en varios servidores| Sí| No| No. La cuenta está asociada al servidor.| Sí. Use una cuenta MSA si es posible. |
| La aplicación se ejecuta detrás de un equilibrador de carga| Sí| No| No| Sí. Úsela solo si no puede usar una cuenta gMSA. |
| La aplicación se ejecuta en Windows Server 2008 R2| No| Sí| Sí. Use una cuenta MSA si es posible.| Sí. Use una cuenta MSA si es posible. |
| La aplicación se ejecuta en Windows Server 2012| Sí| Sí. Use una gMSA si es posible.| Sí. Use una cuenta MSA si es posible.| Sí. Use una cuenta MSA si es posible. |
| Requisito para restringir la cuenta de servicio a un solo servidor| No| Sí| Sí. Use una cuenta sMSA si es posible.| No |
| | |


### <a name="use-server-logs-and-powershell-to-investigate"></a>Uso de registros de servidor y PowerShell para investigar

Puede usar registros de servidor para determinar qué servidores y en cuántos servidores se ejecuta una aplicación.

Para obtener una lista de la versión de Windows Server de todos los servidores de la red, puede ejecutar el siguiente comando de PowerShell: 

```PowerShell

Get-ADComputer -Filter 'operatingsystem -like "*server*" -and enabled -eq "true"' `

-Properties Name,Operatingsystem,OperatingSystemVersion,IPv4Address |

sort-Object -Property Operatingsystem |

Select-Object -Property Name,Operatingsystem,OperatingSystemVersion,IPv4Address |

Out-GridView

```

## <a name="find-on-premises-service-accounts"></a>Búsqueda de cuentas de servicio locales

Se recomienda agregar un prefijo, como "svc-", a todas las cuentas que use como cuentas de servicio. Esta convención de nomenclatura facilita la búsqueda y administración de las cuentas. Considere también el uso de un atributo de descripción para la cuenta de servicio y el propietario de la cuenta de servicio. La descripción puede ser un alias de equipo o un propietario del equipo de seguridad.

La búsqueda de cuentas de servicio locales es fundamental para garantizar su seguridad. Esto puede ser difícil para las cuentas que no son MSA. Se recomienda revisar todas las cuentas que tienen acceso a los recursos locales importantes, y determinar qué cuentas de equipo o de usuario pueden estar actuando como cuentas de servicio. 

Para obtener información sobre cómo buscar una cuenta de servicio, consulte el artículo sobre ese tipo de cuenta en la sección [Pasos siguientes](#next-steps).

## <a name="document-service-accounts"></a>Documentación de las cuentas de servicio

Una vez que haya encontrado las cuentas de servicio en el entorno local, documente la siguiente información: 

* **Propietario**: Persona responsable del mantenimiento de la cuenta.

* **Propósito**: La aplicación que representa la cuenta u otro propósito. 

* **Ámbitos de permiso**: Los permisos que tiene o debe tener, y los grupos de los que es miembro.

* **Perfil de riesgo**: El riesgo para su negocio si esta cuenta se pone en peligro. Si el riesgo es alto, use una MSA.

* **Duración prevista y atestación periódica**: Cuánto tiempo prevé que esta cuenta estará activa, y la frecuencia con la que el propietario debe revisar y probar su necesidad continua.

* **Seguridad de contraseñas**: Para las cuentas de usuario y de equipo local, dónde se almacena la contraseña. Asegúrese de que las contraseñas se mantengan protegidas, y documente quién tiene acceso. Considere la posibilidad de usar [Privileged Identity Management](../privileged-identity-management/pim-configure.md) para proteger las contraseñas almacenadas. 

  

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)  
* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)  
* [Protección de cuentas de equipo](service-accounts-computer.md)  
* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)  
* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)

