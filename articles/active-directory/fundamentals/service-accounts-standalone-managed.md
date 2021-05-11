---
title: Protección de las cuentas de servicio administradas independientes | Azure Active Directory
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
ms.openlocfilehash: ea3bd3e6fc971901bf69c053088678e8f0f718d0
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206680"
---
# <a name="secure-standalone-managed-service-accounts"></a>Protección de cuentas de servicio administradas independientes

Las cuentas de servicio administradas independientes (sMSA) son cuentas de dominio administradas que se usan para ayudar a proteger uno o varios servicios que se ejecutan en un servidor. No se pueden reutilizar en varios servidores. Las sMSA proporcionan administración automática de contraseñas, administración simplificada de nombres de entidad de seguridad de servicio (SPN) y la capacidad de delegar la administración a otros administradores. 

En Active Directory, las sMSA se asocian a un servidor específico que ejecuta un servicio. Puede encontrar estas cuentas en el complemento Usuarios y equipos de Active Directory de Microsoft Management Console.

![Captura de pantalla del complemento Usuarios y equipos de Active Directory que muestra la unidad organizativa de las cuentas de servicio administradas.](./media/securing-service-accounts/secure-standalone-msa-image-1.png)

Las cuentas de servicio administradas se introdujeron con el esquema de Active Directory de Windows Server 2008 R2 y requieren como mínimo Windows Server 2008 R2. 

## <a name="benefits-of-using-smsas"></a>Ventajas de usar las sMSA

Las sMSA ofrecen mayor seguridad que las cuentas de usuario que se usan como cuentas de servicio. A su vez, además, ayudan a reducir la sobrecarga administrativa de las maneras siguientes:

* **Establecimiento de contraseñas seguras**: las sMSA usan contraseñas complejas generadas aleatoriamente de 240 bytes. La complejidad y la longitud de las contraseñas de sMSA minimizan la probabilidad de que un servicio se vea comprometido debido a ataques por fuerza bruta o de diccionario.

* **Cambio periódico de las contraseñas**: Windows cambia la contraseña de las sMSA automáticamente cada 30 días. No es necesario que los administradores de servicios y dominios programen los cambios de contraseña ni administren el tiempo de inactividad asociado.

* **Simplificación de la administración de SPN**: los nombres de entidad de seguridad de servicio se actualizan automáticamente si el nivel funcional de dominio es Windows Server 2008 R2. Por ejemplo, el nombre de entidad de seguridad de servicio se actualiza de forma automática cuando:
   * Se cambia el nombre de la cuenta del equipo host.  
   * Se cambia el nombre del servidor de nombres de dominio (DNS) del equipo host.  
   * Se agregan o se quitan otros parámetros sam-accountname o dns-hostname mediante [PowerShell](/powershell/module/activedirectory/set-adserviceaccount).

## <a name="when-to-use-smsas"></a>Cuándo se deben usar las sMSA

Las sMSA pueden simplificar las tareas de administración y seguridad. Use las sMSA cuando tenga uno o varios servicios implementados en un solo servidor y no pueda usar una cuenta de servicio administrada de grupo (gMSA). 

> [!NOTE] 
> Aunque puede usar las sMSA para más de un servicio, se recomienda que cada servicio tenga su propia identidad para fines de auditoría. 

Si el creador del software no puede indicarle si puede usar una MSA o no, deberá probar la aplicación. Para hacerlo, cree un entorno de prueba y asegúrese de que pueda acceder a todos los recursos requeridos. Para obtener más información, consulte el artículo sobre cómo [crear e instalar una sMSA](/archive/blogs/askds/managed-service-accounts-understanding-implementing-best-practices-and-troubleshooting).

### <a name="assess-the-security-posture-of-smsas"></a>Evaluación de la posición de seguridad de las sMSA

Las sMSA son intrínsecamente más seguras que las cuentas de usuario estándar, que requieren la administración continua de contraseñas. Sin embargo, es importante tener en cuenta el ámbito de acceso de las sMSA como parte de su posición de seguridad general.

Consulte la tabla siguiente para ver cómo mitigar los posibles problemas de seguridad que plantean las sMSA:

| Problema de seguridad| Mitigación |
| - | - |
| Una sMSA es miembro de grupos con privilegios. | <li>Quite la sMSA de los grupos con privilegios elevados, como los administradores de dominio.<li>Use el modelo con *menos privilegios* y conceda a la sMSA solo los derechos y los permisos que requiera para ejecutar el servicio.<li>Si no está seguro de los permisos necesarios, consulte al creador del servicio. |
| Una sMSA tiene acceso de lectura y escritura a los recursos confidenciales. | <li>Audite el acceso a recursos confidenciales.<li>Archive los registros de auditoría en un programa de Administración de eventos e información de seguridad (SIEM), como Azure Log Analytics o Azure Sentinel, para su análisis.<li>Corrija los permisos de recursos si se detecta un nivel de acceso no deseado. |
| De manera predeterminada, la frecuencia de sustitución de contraseñas de una sMSA es de 30 días. | La directiva de grupo se puede usar para ajustar la duración en función de los requisitos de seguridad de la empresa. Puede establecer la duración de la expiración de la contraseña con la ruta de acceso siguiente:<br>*Configuración del equipo\Directivas\Configuración de Windows\Configuración de seguridad\Opciones de seguridad*. Para un miembro del dominio, use la **duración máxima de contraseña de cuenta de equipo**. |
| | |



### <a name="challenges-with-smsas"></a>Desafíos con las sMSA

Los desafíos asociados con las sMSA son los siguientes:

| Desafío| Mitigación |
| - | - |
| Las sMSA se pueden usar en un solo servidor. | Use una gMSA si necesita utilizar la cuenta en los distintos servidores. |
| Las sMSA no se pueden usar entre dominios. | Use una gMSA si necesita utilizar la cuenta entre distintos dominios. |
| No todas las aplicaciones admiten las sMSA. | Use una gMSA si es posible. De lo contrario, utilice una cuenta de usuario estándar o una cuenta de equipo, como recomienda el creador de la aplicación. |
| | |


## <a name="find-smsas"></a>Búsqueda de las gMSA

En cualquier controlador de dominio, ejecute DSA.msc y, a continuación, expanda el contenedor de cuentas de servicio administradas para ver todas las sMSA. 

Ejecute el comando de PowerShell siguiente para devolver todas las sMSA y gMSA del dominio de Active Directory: 

`Get-ADServiceAccount -Filter *`

Ejecute el comando siguiente para devolver solo las sMSA del dominio de Active Directory:

`Get-ADServiceAccount -Filter * | where { $_.objectClass -eq "msDS-ManagedServiceAccount" }`

## <a name="manage-smsas"></a>Administración de las sMSA

Para administrar las sMSA, puede usar los cmdlets siguientes de Active Directory PowerShell:

`Get-ADServiceAccount`  
` Install-ADServiceAccount`  
` New-ADServiceAccount`  
` Remove-ADServiceAccount`  
`Set-ADServiceAccount`  
`Test-ADServiceAccount`  
`Ininstall-ADServiceAccount`

## <a name="move-to-smsas"></a>Migración a las sMSA

Si un servicio de aplicación admite las sMSA, pero no las gMSA y, actualmente, utiliza una cuenta de usuario o una cuenta de equipo para el contexto de seguridad, [cree e instale una sMSA](/archive/blogs/askds/managed-service-accounts-understanding-implementing-best-practices-and-troubleshooting) en el servidor. 

Lo ideal sería que migrara los recursos a Azure y usara Identidades administradas de Azure o entidades de servicio.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)  
* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)  
* [Protección de cuentas de equipo](service-accounts-computer.md)  
* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)  
* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)
