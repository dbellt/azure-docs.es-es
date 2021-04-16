---
title: Protección de cuentas de servicio administradas de grupo | Azure Active Directory
description: Guía para proteger las cuentas de equipo de cuentas de servicio administradas de grupo.
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
ms.openlocfilehash: f6f91adf499c62fef56a30c5f2945ba7a90a0b29
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105640073"
---
# <a name="securing-group-managed-service-accounts"></a>Protección de cuentas de servicio administradas de grupo

Las cuentas de servicio administradas de grupo (gMSA) son cuentas de dominio administradas que se usan para proteger los servicios. Las gMSA se pueden ejecutar en un solo servidor, o en una granja de servidores, como los sistemas que están detrás de un equilibrador de carga de red (NLB) o un servidor de Internet Information Services (IIS). Una vez que configure los servicios para usar una entidad de seguridad de gMSA, Windows se encargará de la administración de contraseñas para esa cuenta.

## <a name="benefits-of-using-gmsas"></a>Ventajas de usar las gMSA

Las gMSA ofrecen una solución de identidad única con mayor seguridad, a la vez que reducen la sobrecarga administrativa de las maneras siguientes:

* **Establecimiento de contraseñas seguras**. Las gMSA usan contraseñas complejas generadas aleatoriamente de 240 bytes. La complejidad y la longitud de las contraseñas de gMSA minimizan la probabilidad de que un servicio se vea comprometido debido a ataques por fuerza bruta o de diccionario.

* **Cambio periódico de las contraseñas**. En el caso de las gMSA, la administración de contraseñas se traslada a Windows, que cambia la contraseña cada 30 días. Los administradores de servicios y de dominios ya no necesitan programar cambios de contraseña ni administrar las interrupciones del servicio para proteger las cuentas de servicio. 

* **Compatibilidad con la implementación en granjas de servidores**. La capacidad de implementar las gMSA en varios servidores permite la compatibilidad con soluciones de equilibrio de carga en las que varios hosts ejecutan el mismo servicio. 

* **Compatibilidad con la administración simplificada del nombre de entidad de seguridad de servidor (SPN)** . Puede configurar el SPN con PowerShell en el momento de la creación de la cuenta. Además, los servicios que admiten registros de SPN automáticos pueden hacerlo en la gMSA, siempre y cuando se hayan establecido correctamente los permisos de gMSA. 

## <a name="when-to-use-gmsas"></a>Cuándo usar las gMSA

Use las gMSA como el tipo de cuenta preferido para los servicios locales, a menos que un servicio, como el clúster de conmutación por error, no lo admita.

> [!IMPORTANT]
> Debe probar el servicio con las gMSA antes de la implementación en producción. Para ello, configure un entorno de prueba y asegúrese de que la aplicación puede usar la gMSA y tener acceso a los recursos a los que necesita acceder. Para más información, consulte [Compatibilidad con las cuentas de servicio administradas de grupo](/system-center/scom/support-group-managed-service-accounts).


Si un servicio no admite el uso de las gMSA, la siguiente mejor opción es usar una cuenta de servicio administrada independiente (sMSA). Las sMSA proporcionan la misma funcionalidad que una gMSA, pero solo se pueden implementar en un único servidor.

Si no puede usar una gMSA o si el servicio admite la sMSA, el servicio debe estar configurado para ejecutarse como una cuenta de usuario estándar. Los administradores de servicios y dominios deben observar los procesos de administración de contraseñas seguras para mantener protegida la cuenta.

## <a name="assess-the-security-posture-of-gmsas"></a>Evaluación de la posición de seguridad de las gMSA

Las gMSA son intrínsecamente más seguras que las cuentas de usuario estándar, que requieren la administración continua de contraseñas. Sin embargo, es importante tener en cuenta el ámbito de acceso de las gMSA al mirar su posición de seguridad general.

En la tabla siguiente se muestran posibles problemas de seguridad y mitigaciones para el uso de las gMSA.

| Problemas de seguridad| Mitigaciones |
| - | - |
| Una gMSA es un miembro de grupos con privilegios. | Revise las pertenencias a grupos. Para ello, puede crear un script de PowerShell a fin de enumerar todas las pertenencias a grupos y, a continuación, filtrar un archivo .csv resultante por los nombres de los archivos gMSA. <br>Quite la gMSA de los grupos con privilegios.<br> Conceda a la gMSA solo los derechos y permisos necesarios para ejecutar su servicio (póngase en contacto con su proveedor de servicios). 
| gMSA tiene acceso de lectura y escritura a los recursos confidenciales. | Audite el acceso a los recursos confidenciales. Archive los registros de auditoría en un SIEM, por ejemplo Azure Log Analytics o Azure Sentinel, para su análisis. Quite los permisos de recursos innecesarios si se detecta un nivel de acceso no deseado. |


## <a name="find-gmsas"></a>Búsqueda de las gMSA

Es posible que la organización ya tenga gMSA creadas. Ejecute el cmdlet de PowerShell siguiente para recuperar estas cuentas:

Para trabajar de forma eficaz, las gMSA deben estar en la unidad organizativa (UO) de las cuentas de servicio administradas.

  
![Captura de pantalla de la unidad organizativa de cuenta de servicio administrada.](./media/securing-service-accounts/secure-gmsa-image-1.png)

Para buscar las MSA de servicio que quizás no estén ahí, consulte los comandos siguientes.

**Para buscar todas las cuentas de servicio, incluidas las gMSA y las sMSA:**


```powershell

Get-ADServiceAccount -Filter *

# This PowerShell cmdlet will return all Managed Service Accounts (both gMSAs and sMSAs). An administrator can differentiate between the two by examining the ObjectClass attribute on returned accounts.

# For gMSA accounts, ObjectClass = msDS-GroupManagedServiceAccount

# For sMSA accounts, ObjectClass = msDS-ManagedServiceAccount

# To filter results to only gMSAs:

Get-ADServiceAccount –Filter * | where $_.ObjectClass -eq "msDS-GroupManagedServiceAccount"}
```

## <a name="manage-gmsas"></a>Administración de las gMSA

Puede usar los cmdlets de PowerShell de Active Directory siguientes para administrar las gMSA:

`Get-ADServiceAccount`

`Install-ADServiceAccount`

`New-ADServiceAccount`

`Remove-ADServiceAccount`

`Set-ADServiceAccount`

`Test-ADServiceAccount`

`Uninstall-ADServiceAccount`

> [!NOTE]
> A partir de Windows Server 2012, los cmdlets *-ADServiceAccount funcionan con las gMSA de manera predeterminada. Para más información sobre el uso de los cmdlets anteriores, consulte [**Introducción a las cuentas de servicio administradas de grupo**](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts).

## <a name="move-to-a-gmsa"></a>Migración a una gMSA
Las gMSA son el tipo de cuenta de servicio más seguro para las necesidades locales. Si puede migrar a una, debería hacerlo. Además, considere la posibilidad de migrar los servicios a Azure y las cuentas de servicio a Azure Active Directory.

1.  Asegúrese de que la [clave raíz KDS esté implementada en el bosque](/windows-server/security/group-managed-service-accounts/create-the-key-distribution-services-kds-root-key). Esta operación solo se realiza una vez.

2. [Cree una gMSA nueva](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts).

3. Instale la gMSA nueva en cada host que ejecuta el servicio.
   > [!NOTE] 
   > Para más información sobre la creación e instalación de gMSA en un host, antes de configurar el servicio para usar gMSA, consulte [Introducción a las cuentas de servicio administradas de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj128431(v=ws.11)).

 
4. Cambie la identidad de servicio a gMSA y especifique una contraseña en blanco.

5. Valide que el servicio funciona con la nueva identidad gMSA.

6. Elimine la identidad de la cuenta de servicio anterior.

 

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos sobre la protección de cuentas de servicio.

* [Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)

* [Protección de cuentas de servicio administradas de grupo](service-accounts-group-managed.md)

* [Protección de cuentas de servicio administradas independientes](service-accounts-standalone-managed.md)

* [Protección de cuentas de equipo](service-accounts-computer.md)

* [Protección de cuentas de usuario](service-accounts-user-on-premises.md)

* [Control de cuentas de servicio locales](service-accounts-govern-on-premises.md)