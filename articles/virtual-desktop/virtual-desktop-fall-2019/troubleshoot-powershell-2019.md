---
title: 'PowerShell de Azure Virtual Desktop (clásico): Azure'
description: Solución de problemas de PowerShell al configurar un entorno de inquilinos de Azure Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 6ce64aa104ca261845740f9b448d5d4e7afd10e5
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751656"
---
# <a name="azure-virtual-desktop-classic-powershell"></a>PowerShell de Azure Virtual Desktop (clásico)

> [!IMPORTANT]
> Este contenido se aplica a Azure Virtual Desktop (clásico), que no admite objetos de Azure Resource Manager. Si está intentando administrar objetos de Azure Virtual Desktop para Azure Resource Manager, consulte [este artículo](../troubleshoot-powershell.md).

Use este artículo para resolver los problemas y errores al usar PowerShell con Azure Virtual Desktop. Para más información sobre PowerShell para Servicios de Escritorio remoto, consulte [PowerShell para Azure Virtual Desktop](/powershell/windows-virtual-desktop/overview).

## <a name="provide-feedback"></a>Envío de comentarios

Visite la [comunidad técnica de Azure Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para analizar el servicio Azure Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="powershell-commands-used-during-azure-virtual-desktop-setup"></a>Comandos de PowerShell usados durante la configuración de Azure Virtual Desktop

En esta sección se enumeran los comandos de PowerShell que se usan normalmente durante la configuración de Azure Virtual Desktop, y proporciona maneras de resolver los problemas que pueden surgir al usarlos.

### <a name="error-add-rdsappgroupuser-command----the-specified-userprincipalname-is-already-assigned-to-a-remoteapp-app-group-in-the-specified-host-pool"></a>Error: Comando Add-RdsAppGroupUser: The specified UserPrincipalName is already assigned to a RemoteApp app group in the specified Host Pool (El UserPrincipalName especificado ya está asignado a un grupo de aplicaciones de RemoteApp en el grupo host especificado)

```powershell
Add-RdsAppGroupUser -TenantName <TenantName> -HostPoolName <HostPoolName> -AppGroupName 'Desktop Application Group' -UserPrincipalName <UserName>
```

**Causa:** El nombre de usuario usado ya se ha asignado a un grupo de aplicaciones de un tipo diferente. Los usuarios no pueden asignarse a un grupo de aplicaciones tanto remoto como de escritorio en el mismo grupo host de sesión.

**Solución:** Si el usuario necesita tanto aplicaciones remotas como escritorio remoto, cree grupos host diferentes o conceda al usuario acceso al escritorio remoto, lo que permitirá el uso de cualquier aplicación en la VM del host de sesión.

### <a name="error-add-rdsappgroupuser-command----the-specified-userprincipalname-doesnt-exist-in-the-azure-active-directory-associated-with-the-remote-desktop-tenant"></a>Error: Comando Add-RdsAppGroupUser: The specified UserPrincipalName doesn't exist in the Azure Active Directory associated with the Remote Desktop tenant (El UserPrincipalName especificado no existe en la instancia de Azure Active Directory asociada con el inquilino de Escritorio remoto)

```powershell
Add-RdsAppGroupUser -TenantName <TenantName> -HostPoolName <HostPoolName> -AppGroupName "Desktop Application Group" -UserPrincipalName <UserPrincipalName>
```

**Causa:** El usuario especificado por el valor de UserPrincipalName no se encuentra en la instancia de Azure Active Directory vinculada al inquilino de Azure Virtual Desktop.

**Solución:** Confirme los elementos de la lista siguiente.

- El usuario está sincronizado con Azure Active Directory.
- El usuario no está vinculado a comercio de negocio a consumidor (B2C) ni de negocio a negocio (B2B).
- El inquilino de Azure Virtual Desktop está vinculado a la instancia correcta de Azure Active Directory.

### <a name="error-get-rdsdiagnosticactivities----user-isnt-authorized-to-query-the-management-service"></a>Error: Get-RdsDiagnosticActivities: User isn't authorized to query the management service (El usuario no está autorizado para consultar el servicio de administración)

```powershell
Get-RdsDiagnosticActivities -ActivityId <ActivityId>
```

**Causa:** Parámetro TenantName

**Solución:** Emita Get-RdsDiagnosticActivities con -TenantName \<TenantName>.

### <a name="error-get-rdsdiagnosticactivities----the-user-isnt-authorized-to-query-the-management-service"></a>Error: Get-RdsDiagnosticActivities: User isn't authorized to query the management service (El usuario no está autorizado para consultar el servicio de administración)

```powershell
Get-RdsDiagnosticActivities -Deployment -username <username>
```

**Causa:** Uso del conmutador -Deployment.

**Corrección:** Solo los administradores de implementación pueden usar el conmutador Deployment. Estos administradores suelen ser miembros del equipo de Servicios de Escritorio remoto/Azure Virtual Desktop. Reemplace el conmutador -Deployment por -TenantName \<TenantName>.

### <a name="error-new-rdsroleassignment----the-user-isnt-authorized-to-query-the-management-service"></a>Error: New-RdsRoleAssignment: The user isn't authorized to query the management service (El usuario no está autorizado para consultar el servicio de administración)

**Causa 1:** La cuenta usada no tiene permisos de propietario de Servicios de Escritorio remoto en el inquilino.

**Corrección 1:** Un usuario con permisos de propietario de Servicios de Escritorio remoto debe ejecutar la asignación de roles.

**Causa 2:** La cuenta usada tiene permisos de propietario de Servicios de Escritorio remoto, pero no forma parte de la instancia de Azure Active Directory del inquilino o no tiene permisos para consultas en Azure Active Directory donde se encuentra el usuario.

**Corrección 2:** Un usuario con permisos de Active Directory debe ejecutar la asignación de roles.

> [!NOTE]
> New-RdsRoleAssignment no puede conceder permisos a un usuario que no existe en Azure Active Directory (AD).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Azure Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview-2019.md).
- Para solucionar problemas durante la creación de un grupo de inquilinos y de hosts en un entorno de Azure Virtual Desktop, consulte [Creación de los grupos de inquilinos y de host](troubleshoot-set-up-issues-2019.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Azure Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).
- Para solucionar problemas con conexiones de cliente de Azure Virtual Desktop, consulte [Conexiones de servicios de Azure Virtual Desktop](troubleshoot-service-connection-2019.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](../troubleshoot-client.md).
- Para más información sobre el servicio, consulte [Entorno de Azure Virtual Desktop](environment-setup-2019.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](../../azure-resource-manager/templates/deployment-history.md).
