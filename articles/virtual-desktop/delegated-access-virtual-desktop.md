---
title: 'Acceso delegado en Azure Virtual Desktop: Azure'
description: Se describe cómo delegar funcionalidades administrativas en una implementación de Azure Virtual Desktop y se incluyen ejemplos.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: femila
ms.openlocfilehash: 2367a2840e64956d3dfca41ce8ee5e31c9f354c7
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751998"
---
# <a name="delegated-access-in-azure-virtual-desktop"></a>Acceso delegado en Azure Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop con objetos de Azure Resource Manager. Si usa Azure Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/delegated-access-virtual-desktop-2019.md).

Azure Virtual Desktop presenta un modelo de acceso delegado que permite definir la cantidad de acceso que puede tener un usuario determinado mediante la asignación de un rol. Una asignación de roles consta de tres componentes: entidad de seguridad, definición de rol y ámbito. El modelo de acceso delegado de Azure Virtual Desktop se basa en el modelo de Azure RBAC. Para más información sobre asignaciones de roles específicas y sus componentes, consulte la [información general sobre el control de acceso basado en roles de Azure](../role-based-access-control/built-in-roles.md).

El acceso delegado de Azure Virtual Desktop admite los siguientes valores para cada elemento de la asignación de roles:

* Entidad de seguridad
    * Usuarios
    * Grupos de usuarios
    * Entidades de servicio
* Definición de roles
    * Roles integrados
    * Roles personalizados
* Ámbito
    * Grupos de host
    * Grupos de aplicaciones
    * Áreas de trabajo

## <a name="powershell-cmdlets-for-role-assignments"></a>Cmdlets de PowerShell para asignaciones de roles

Antes de empezar, asegúrese de seguir las instrucciones que se indican en [Configuración del módulo de PowerShell](powershell-module.md) para configurar el módulo de PowerShell de Azure Virtual Desktop, en caso de que no lo haya hecho aún.

Azure Virtual Desktop usa el control de acceso basado en rol de Azure (Azure RBAC) al publicar grupos de aplicaciones para usuarios o grupos de usuarios. El rol Usuario de la virtualización del escritorio se asigna al usuario o grupo de usuarios, y el ámbito es el grupo de aplicaciones. Este rol concede al usuario un acceso especial a los datos en el grupo de aplicaciones.

Ejecute el siguiente cmdlet para agregar usuarios de Azure Active Directory a un grupo de aplicaciones:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Ejecute el siguiente cmdlet para agregar grupos de usuarios de Azure Active Directory a un grupo de aplicaciones:

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

## <a name="next-steps"></a>Pasos siguientes

Para una lista más completa de cmdlets de PowerShell que puede usar cada rol, consulte la [referencia de PowerShell](/powershell/windows-virtual-desktop/overview).

Para obtener una lista completa de los roles que se admiten en Azure RBAC, consulte [Roles integrados de Azure](../role-based-access-control/built-in-roles.md).

Para tener una guía para configurar un entorno de Azure Virtual Desktop, consulte [Entorno de Azure Virtual Desktop](environment-setup.md).
