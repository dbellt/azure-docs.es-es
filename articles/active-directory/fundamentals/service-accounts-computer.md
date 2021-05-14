---
title: Protección de cuentas de equipo | Azure Active Directory
description: Guía para proteger las cuentas de equipo locales.
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
ms.openlocfilehash: ec2e8cddb93e2e6fdf9ff804bbc28fd283436131
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206626"
---
# <a name="secure-on-premises-computer-accounts"></a>Protección de cuentas de equipo locales

La cuenta de equipo, o la cuenta LocalSystem, es una cuenta integrada con privilegios muy elevados que tiene acceso a prácticamente todos los recursos del equipo local. Esta cuenta no está asociada a ninguna cuenta de usuario con sesión iniciada. Los servicios que se ejecutan con LocalSystem acceden a los recursos de red mediante la presentación de las credenciales del equipo a los servidores remotos con el formato <nombre_de_dominio>\\<nombre_de_equipo>$. El nombre predefinido de una cuenta de equipo es NT AUTHORITY\SYSTEM. Puede usarlo para iniciar un servicio y proporcionar el contexto de seguridad para ese servicio.

![Captura de pantalla de una lista de servicios locales en una cuenta de equipo.](.\media\securing-service-accounts\secure-computer-accounts-image-1.png)

## <a name="benefits-of-using-a-computer-account"></a>Ventajas de usar una cuenta de equipo

Las cuentas de equipo proporcionan las siguientes ventajas:

* **Acceso local sin restricciones**: la cuenta de equipo proporciona acceso completo a los recursos locales de la máquina.

* **Administración automática de contraseñas**: se elimina la necesidad de cambiar manualmente las contraseñas. La cuenta es miembro de Active Directory y la contraseña de la cuenta se cambia automáticamente. El uso de una cuenta de equipo también elimina la necesidad de registrar el nombre de la entidad de seguridad del servicio.

* **Derechos de acceso limitados fuera de la máquina**: la lista de control de acceso (ACL) predeterminada de Active Directory Domain Services permite un acceso mínimo a las cuentas de equipo. En caso de acceso por parte de un usuario no autorizado, el servicio solo tendría acceso limitado a los recursos de la red.

## <a name="assess-the-security-posture-of-computer-accounts"></a>Evaluación de la posición de seguridad de las cuentas de equipo

En la tabla siguiente se enumeran algunas dificultades potenciales y las mitigaciones asociadas al usar una cuenta de equipo:
 
| Incidencia | Mitigación |
| - | - |
| Las cuentas de equipo están sujetas a eliminación y recreación cuando el equipo sale del dominio y vuelve a unirse a él. | Valide la necesidad de agregar un equipo a un grupo de Active Directory y compruebe qué cuenta de equipo se ha agregado a un grupo mediante los scripts de ejemplo proporcionados en este artículo.| 
| Si agrega una cuenta de equipo a un grupo, todos los servicios que se ejecutan con LocalSystem en ese equipo reciben los derechos de acceso del grupo.| Sea selectivo con respecto a las pertenencias a grupos de la cuenta de equipo. Evite convertir una cuenta de equipo en miembro de cualquier grupo de administradores de dominio, ya que el servicio asociado tiene acceso completo a AD DS. |
| El uso de valores predeterminados de red inapropiados para LocalSystem. | No suponga que la cuenta de equipo tiene el acceso limitado predeterminado a los recursos de red. En su lugar, compruebe cuidadosamente las pertenencias a grupos de la cuenta. |
| Servicios desconocidos que se ejecutan con LocalSystem. | Asegúrese de que todos los servicios que se ejecutan con la cuenta LocalSystem son servicios de Microsoft o servicios de confianza de terceros. |
| | |

## <a name="find-services-that-run-under-the-computer-account"></a>Búsqueda de los servicios que se ejecutan con la cuenta de equipo

Para encontrar los servicios que se ejecutan en el contexto de LocalSystem, use el siguiente cmdlet de PowerShell:

```powershell
Get-WmiObject win32_service | select Name, StartName | Where-Object {($_.StartName -eq "LocalSystem")}
```

Para encontrar las cuentas de equipo que son miembros de un grupo específico, ejecute el siguiente cmdlet de PowerShell:

```powershell
Get-ADComputer -Filter {Name -Like "*"} -Properties MemberOf | Where-Object {[STRING]$_.MemberOf -like "Your_Group_Name_here*"} | Select Name, MemberOf
```

Para encontrar las cuentas de equipo que son miembros de los grupos de administradores de identidades (administradores de dominio, administradores de empresas, administradores), ejecute el siguiente cmdlet de PowerShell:

```powershell
Get-ADGroupMember -Identity Administrators -Recursive | Where objectClass -eq "computer"
```

## <a name="move-from-computer-accounts"></a>Migrar de cuentas de equipo

> [!IMPORTANT]
> Las cuentas de equipo son cuentas con privilegios muy elevados y deben usarse solo cuando el servicio necesite acceso sin restricciones a los recursos locales de la máquina y no sea posible usar una cuenta de servicio administrada (MSA).

* Compruebe con el propietario del servicio si su servicio se puede ejecutar con una MSA. Si es así, use una cuenta de servicio administrada de grupo (gMSA) o una cuenta de servicio administrada independiente (sMSA).

* Use una cuenta de usuario de dominio solo con los permisos necesarios para ejecutar el servicio.

## <a name="next-steps"></a>Pasos siguientes 

Para más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)
* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)
* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)
* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)  
* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)
