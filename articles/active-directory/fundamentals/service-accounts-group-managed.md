---
title: Protección de cuentas de servicio administradas de grupo | Azure Active Directory
description: Guía para proteger las cuentas de equipo de cuentas de servicio administradas de grupo (gMSA).
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
ms.openlocfilehash: 9df8679079d4d3496ceeff4ac8a4acd9142ba867
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206572"
---
# <a name="secure-group-managed-service-accounts"></a>Protección de cuentas de servicio administradas de grupo

Las cuentas de servicio administradas de grupo (gMSA) son cuentas de dominio administradas que se usan para ayudar a proteger los servicios. Las gMSA se pueden ejecutar en un solo servidor o en una granja de servidores, como los sistemas que están detrás de un equilibrador de carga de red o un servidor de Internet Information Services (IIS). Una vez que configure los servicios para usar una entidad de seguridad de gMSA, el sistema operativo Windows se encargará de la administración de contraseñas para esa cuenta.

## <a name="benefits-of-using-gmsas"></a>Ventajas de usar las gMSA

Las gMSA ofrecen una solución de identidad única con mayor seguridad. A la vez, para ayudar a reducir la sobrecarga administrativa:

* **Establecen contraseñas seguras**: Las gMSA usan contraseñas complejas generadas aleatoriamente de 240 bytes. La complejidad y la longitud de las contraseñas de gMSA minimizan la probabilidad de que un servicio se vea comprometido debido a ataques por fuerza bruta o de diccionario.

* **Cambian las contraseñas periódicamente**: Las gMSA trasladan la administración de contraseñas al sistema operativo Windows, que cambia la contraseña cada 30 días. Los administradores de servicios y de dominios ya no necesitan programar cambios de contraseña ni administrar las interrupciones del servicio para ayudar a proteger las cuentas de servicio. 

* **Admiten la implementación en granjas de servidores**: La capacidad de implementar las gMSA en varios servidores permite la compatibilidad con soluciones de equilibrio de carga en las que varios hosts ejecutan el mismo servicio. 

* **Admiten la administración simplificada de nombres de entidad de seguridad de servicio (SPN)** : Puede configurar un SPN mediante PowerShell al crear una cuenta. Además, los servicios que admiten registros de SPN automáticos pueden hacerlo en la gMSA, siempre y cuando los permisos de gMSA se hayan establecido correctamente. 

## <a name="when-to-use-gmsas"></a>Cuándo usar las gMSA

Use las gMSA como el tipo de cuenta preferido para los servicios locales, a menos que un servicio, como el clúster de conmutación por error, no lo admita.

> [!IMPORTANT]
> Debe probar el servicio con las gMSA antes de implementarlo en producción. Para ello, configure un entorno de prueba y asegúrese de que la aplicación pueda usar la gMSA y luego acceda a los recursos a los que necesita acceder. Para más información, consulte [Compatibilidad con las cuentas de servicio administradas de grupo](/system-center/scom/support-group-managed-service-accounts).


Si un servicio no admite el uso de gMSA, la siguiente mejor opción es usar una cuenta de servicio administrada independiente (sMSA). Una sMSA proporciona la misma funcionalidad que una gMSA, pero solo se puede implementar en un único servidor.

Si no puede usar una gMSA o sMSA que el servicio admita, tiene que configurarlo para que se ejecute como cuenta de usuario estándar. Los administradores de servicios y dominios deben observar los procesos de administración de contraseñas seguras para ayudar a mantener protegida la cuenta.

## <a name="assess-the-security-posture-of-gmsas"></a>Evaluación de la posición de seguridad de las gMSA

Las cuentas gMSA son intrínsecamente más seguras que las cuentas de usuario estándar, que requieren la administración continua de contraseñas. Sin embargo, es importante tener en cuenta el ámbito de acceso de una gMSA analizando su posición de seguridad general.

En la tabla siguiente se muestran posibles problemas de seguridad y mitigaciones para el uso de gMSA:

| Problema de seguridad| Mitigación |
| - | - |
| Una gMSA es un miembro de grupos con privilegios. | <li>Revise las pertenencias a grupos. Para ello, puede crear un script de PowerShell para enumerar todas las pertenencias a grupos. A continuación, puede filtrar un archivo .csv resultante por los nombres de los archivos de gMSA.<li>Quite la gMSA de los grupos con privilegios.<li>Conceda a la gMSA solo los derechos y permisos necesarios para ejecutar su servicio (póngase en contacto con su proveedor de servicios). 
| gMSA tiene acceso de lectura y escritura a los recursos confidenciales. | <li>Audite el acceso a los recursos confidenciales.<li>Archive los registros de auditoría en un SIEM, como Azure Log Analytics o Azure Sentinel, para su análisis.<li>Quite los permisos de recursos innecesarios si detecta un nivel de acceso no deseado. |
| | |


## <a name="find-gmsas"></a>Búsqueda de las gMSA

Es posible que su organización ya haya creado gMSA. Para recuperar estas cuentas, ejecute el siguiente cmdlet de PowerShell:

```powershell
Get-ADServiceAccount 
Install-ADServiceAccount 
New-ADServiceAccount 
Remove-ADServiceAccount 
Set-ADServiceAccount 
Test-ADServiceAccount 
Uninstall-ADServiceAccount
```


Para trabajar de forma eficaz, las gMSA deben estar en la unidad organizativa (UO) de la cuenta de servicio administrada.

  
![Captura de pantalla de una cuenta gMSA en la unidad organizativa de la cuenta de servicio administrada.](./media/securing-service-accounts/secure-gmsa-image-1.png)

Para buscar los MSA de servicio que podrían no estar en la lista, ejecute los siguientes comandos:

```powershell

Get-ADServiceAccount -Filter *

# This PowerShell cmdlet will return all managed service accounts (both gMSAs and sMSAs). An administrator can differentiate between the two by examining the ObjectClass attribute on returned accounts.

# For gMSA accounts, ObjectClass = msDS-GroupManagedServiceAccount

# For sMSA accounts, ObjectClass = msDS-ManagedServiceAccount

# To filter results to only gMSAs:

Get-ADServiceAccount –Filter * | where $_.ObjectClass -eq "msDS-GroupManagedServiceAccount"}
```

## <a name="manage-gmsas"></a>Administración de las gMSA

Para administrar las gMSA, puede usar los siguientes cmdlets de Active Directory PowerShell:

`Get-ADServiceAccount`

`Install-ADServiceAccount`

`New-ADServiceAccount`

`Remove-ADServiceAccount`

`Set-ADServiceAccount`

`Test-ADServiceAccount`

`Uninstall-ADServiceAccount`

> [!NOTE]
> A partir de Windows Server 2012, los cmdlets *-ADServiceAccount funcionan con las gMSA de manera predeterminada. Para obtener más información sobre el uso de los cmdlets anteriores, consulte [Introducción a las cuentas de servicio administradas de grupo](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts).

## <a name="move-to-a-gmsa"></a>Migración a una gMSA
Las cuentas gMSA son el tipo de cuenta de servicio más seguro para las necesidades locales. Si puede migrar a una, debería hacerlo. Además, considere la posibilidad de migrar los servicios a Azure y las cuentas de servicio a Azure Active Directory. Para pasarse a una cuenta gMSA, haga lo siguiente:

1. Asegúrese de que la [clave raíz del Servicio de distribución de claves (KDS)](/windows-server/security/group-managed-service-accounts/create-the-key-distribution-services-kds-root-key) esté implementada en el bosque. Esta operación solo se realiza una vez.

1. [Cree una gMSA nueva](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts).

1. Instale la nueva gMSA en cada host que ejecuta el servicio.
   > [!NOTE] 
   > Para obtener más información sobre la creación e instalación de una gMSA en un host, antes de configurar el servicio para usar la gMSA, consulte [Introducción a las cuentas de servicio administradas de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj128431(v=ws.11)).

1. Cambie la identidad de servicio a gMSA y especifique una contraseña en blanco.

1. Valide que el servicio funciona con la nueva identidad gMSA.

1. Elimine la identidad de la cuenta de servicio anterior.

 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la protección de las cuentas de servicio, consulte los artículos siguientes:

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)  
* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)  
* [Protección de cuentas de equipo](service-accounts-computer.md)  
* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)  
* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)
