---
title: Protección de cuentas de servicio administradas independientes | Azure Active Directory
description: Guía para proteger las cuentas de servicio administradas independientes.
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
ms.openlocfilehash: 146cee9ddd6f0cef5ca23272ad9991af847f30ba
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108072160"
---
# <a name="securing-standalone-managed-service-accounts"></a>Protección de las cuentas de servicio administradas independientes

Las cuentas de servicio administradas independientes (sMSA) son cuentas de dominio administradas que se usan para proteger uno o varios servicios que se ejecutan en un servidor. No se pueden reutilizar en varios servidores. Las sMSA proporcionan administración automática de contraseñas, administración simplificada de nombres de entidad de seguridad de servicio (SPN) y la capacidad de delegar la administración a otros administradores. 

En Active Directory, las sMSA se asocian a un servidor específico que ejecuta un servicio. Puede encontrar estas cuentas en el complemento Usuarios y equipos de Active Directory de Microsoft Management Console.

![Una captura de pantalla del complemento Usuarios y equipos de Active Directory que muestra la unidad organizativa de las cuentas de servicio administradas.](./media/securing-service-accounts/secure-standalone-msa-image-1.png)

Las cuentas de servicio administradas se presentaron con el Esquema de Active Directory de Windows Server 2008R2 y requieren un nivel de sistema operativo mínimo de Windows Server 2008R2. 

## <a name="benefits-of-using-smsas"></a>Ventajas de usar las sMSA

Las sMSA ofrecen mayor seguridad que las cuentas de usuario que se usan como cuentas de servicio, a la vez que reducen la sobrecarga administrativa de las maneras siguientes:

* Establecimiento de contraseñas seguras. Las sMSA usan contraseñas complejas generadas aleatoriamente de 240 bytes. La complejidad y la longitud de las contraseñas de sMSA minimizan la probabilidad de que un servicio se vea comprometido debido a ataques por fuerza bruta o de diccionario.

* Cambio periódico de las contraseñas. Windows cambia la contraseña de las sMSA automáticamente cada 30 días. No es necesario que los administradores de servicios y dominios programen los cambios de contraseña ni administren el tiempo de inactividad asociado.

* Simplificación de la administración de SPN. Los nombres de entidad de seguridad de servicio se actualizan automáticamente si el nivel funcional de dominio (DFL) es Windows Server 2008 R2. Por ejemplo, el nombre de entidad de seguridad de servicio se actualiza automáticamente en los escenarios siguientes:

   * Se cambia el nombre de la cuenta del equipo host. 

   * Se cambia el nombre DNS del equipo host.

   * Al agregar o quitar un parámetro sam-accountname o dns-hostname mediante [PowerShell](/powershell/module/activedirectory/set-adserviceaccount).

## <a name="when-to-use-smsas"></a>Cuándo se deben usar las sMSA

Las sMSA pueden simplificar las tareas de administración y seguridad. Use las sMSA cuando tenga uno o varios servicios implementados en un solo servidor y no pueda usar una gMSA. 

> [!NOTE] 
> Si bien puede usar las sMSA para más de un servicio, se recomienda que cada servicio tenga su propia identidad para fines de auditoría. 

Si el creador del software no puede indicarle si puede usar una MSA, deberá probar la aplicación. Para hacerlo, cree un entorno de prueba y asegúrese de que pueda acceder a todos los recursos requeridos. Consulte el artículo sobre cómo [crear e instalar un sMSA](/archive/blogs/askds/managed-service-accounts-understanding-implementing-best-practices-and-troubleshooting) para instrucciones detalladas.

### <a name="assess-security-posture-of-smsas"></a>Evaluación de la posición de seguridad de las sMSA

Las sMSA son intrínsecamente más seguras que las cuentas de usuario estándar, que requieren la administración continua de contraseñas. Sin embargo, es importante tener en cuenta el ámbito de acceso de las sMSA como parte de su posición de seguridad general.

En la tabla siguiente se muestra cómo mitigar los posibles problemas de seguridad que plantean las sMSA.

| Problemas de seguridad| Mitigaciones |
| - | - |
| Una sMSA es un miembro de grupos con privilegios|Quite la sMSA de los grupos con privilegios elevados (como administradores de dominio). <br> Use el modelo con menos privilegios y conceda a la sMSA solo los derechos y los permisos que requiere para ejecutar el servicio. <br> Si no está seguro de los permisos necesarios, consulte al creador del servicio. |
| Una sMSA tiene acceso de lectura y escritura a los recursos confidenciales.|Audite el acceso a recursos confidenciales. Archive los registros de auditoría en un SIEM (Azure Log Analytics o Azure Sentinel) para su análisis. <br> Corrija los permisos de recursos si se detecta un nivel de acceso no deseado. |
| De manera predeterminada, la frecuencia de sustitución de contraseñas de una sMSA es de 30 días.| La directiva de grupo se puede usar para ajustar la duración en función de los requisitos de seguridad de la empresa. <br> * Puede establecer la duración de la expiración de la contraseña con la ruta de acceso siguiente. <br>Configuración del equipo\Directivas\Configuración de Windows\Configuración de seguridad\Opciones de seguridad\Miembro de dominio: antigüedad máxima de la contraseña de cuenta de máquina |



### <a name="challenges-with-smsas"></a>Desafíos con las sMSA

Los desafíos asociados con las sMSA son los siguientes:

| Desafíos| Mitigaciones |
| - | - |
| Se pueden usar en un solo servidor.| Use las gMSA si necesita utilizar la cuenta en los distintos servidores. |
| No se pueden usar en distintos dominios.| Use las gMSA si necesita utilizar la cuenta en los distintos dominios. |
| No todas las aplicaciones admiten las sMSA.| Si es posible, use las gMSA. Si no usa una cuenta de usuario estándar ni una cuenta de equipo como recomienda el creador de la aplicación. |


## <a name="find-smsas"></a>Búsqueda de las gMSA

En cualquier controlador de dominio, ejecute DSA.msc y expanda el contenedor Cuentas de servicio administradas para ver todas las sMSA. 

El comando de PowerShell siguiente devuelve todas las sMSA y gMSA del dominio de Active Directory. 

`Get-ADServiceAccount -Filter *`

El comando siguiente solo devuelve las sMSA del dominio de Active Directory.

`Get-ADServiceAccount -Filter * | where { $_.objectClass -eq "msDS-ManagedServiceAccount" }`

## <a name="manage-smsas"></a>Administración de las sMSA

Puede usar los cmdlets de PowerShell de Active Directory siguientes para administrar las sMSA:

`Get-ADServiceAccount`

` Install-ADServiceAccount`

` New-ADServiceAccount`

` Remove-ADServiceAccount`

`Set-ADServiceAccount`

`Test-ADServiceAccount`

`Ininstall-ADServiceAccount`

## <a name="move-to-smsas"></a>Migración a las sMSA

Si un servicio de aplicación admite las sMSA, pero no las gMSA, y actualmente utiliza una cuenta de usuario o una cuenta de equipo para el contexto de seguridad, [cree e instale una sMSA](/archive/blogs/askds/managed-service-accounts-understanding-implementing-best-practices-and-troubleshooting) en el servidor. 

Idealmente, migre los recursos a Azure y use Identidades administradas de Azure o entidades de servicio.

 

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos sobre la protección de cuentas de servicio.

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)

* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)

* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)

* [Protección de cuentas de equipo](service-accounts-computer.md)

* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)

* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)

