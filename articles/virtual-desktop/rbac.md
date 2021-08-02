---
title: 'Roles integrados de Azure Virtual Desktop: Azure'
description: Información general sobre los roles integrados de Azure Virtual Desktop disponibles para el control de acceso basado en roles de Azure.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: f19035ea08e95d5535903467ddbc11fc53c0deb7
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753114"
---
# <a name="built-in-roles-for-azure-virtual-desktop"></a>Roles integrados de Azure Virtual Desktop

Azure Virtual Desktop usa controles de acceso basado en roles (RBAC) de Azure para asignar roles a usuarios y administradores. Estos roles proporcionan permisos de administrador para llevar a cabo determinadas tareas. Para más información sobre los roles integrados de RBAC de Azure, consulte [Roles integrados de Azure](../role-based-access-control/built-in-roles.md).

Los roles integrados estándar de Azure son Propietario, Colaborador y Lector. Sin embargo, Azure Virtual Desktop tiene roles adicionales que le permiten separar roles de administración para grupos de hosts, grupos de aplicaciones y áreas de trabajo. Esta separación le permite tener un control más preciso sobre las tareas administrativas. Estos roles se denominan conforme a los roles estándar de Azure y a la metodología de privilegios mínimos.

Azure Virtual Desktop no tiene un rol Propietario específico. Sin embargo, puede usar un rol Propietario estándar para los objetos de servicio.

## <a name="desktop-virtualization-contributor"></a>Colaborador de virtualización del escritorio

El rol Colaborador de virtualización del escritorio le permite administrar todos los aspectos de la implementación. Sin embargo, no le concede acceso a los recursos de proceso. También necesitará el rol Administrador de acceso de usuario para publicar grupos de aplicaciones para usuarios o grupos de usuarios.


- Microsoft.DesktopVirtualization/\* 
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="desktop-virtualization-reader"></a>Lector de virtualización del escritorio

El rol Lector de virtualización del escritorio le permite ver todo lo que hay en la implementación, pero no le permite realizar ningún cambio.

- Microsoft.DesktopVirtualization/\*/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-contributor"></a>Colaborador del grupo de hosts

El rol Colaborador del grupo de hosts le permite administrar todos los aspectos de los grupos de hosts, incluido el acceso a los recursos. Para crear máquinas virtuales, necesitará un rol de colaborador adicional, el rol Colaborador de la máquina virtual. Necesitará roles de colaborador de AppGroup y del área de trabajo para crear un grupo de hosts mediante el portal o puede usar el rol Colaborador de virtualización del escritorio.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/hostpools/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-reader"></a>Lector del grupo de hosts

El rol Lector del grupo de hosts le permite ver todo lo que hay en el grupo de hosts, pero no le permite realizar ningún cambio.

- Microsoft.DesktopVirtualization/hostpools/\*/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-contributor"></a>Colaborador del grupo de aplicaciones

El rol Colaborador del grupo de aplicaciones le permite administrar todos los aspectos de los grupos de aplicaciones. Necesitará el rol Administrador de acceso de usuario para publicar grupos de aplicaciones para usuarios o grupos de usuarios.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/applicationgroups/\*
- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-reader"></a>Lector del grupo de aplicaciones

El rol Lector del grupo de aplicaciones le permite ver todo lo que hay en el grupo de aplicaciones y no le permitirá realizar ningún cambio.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/applicationgroups/\*/read
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-contributor"></a>Colaborador del área de trabajo

El rol Colaborador del área de trabajo le permite administrar todos los aspectos de las áreas de trabajo. Para más información sobre las aplicaciones agregadas a los grupos de aplicaciones, también debe tener asignado el rol Lector del grupo de aplicaciones.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/workspaces/\*
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-reader"></a>Lector del área de trabajo

El rol Lector del área de trabajo le permite ver todo lo que hay en el área de trabajo, pero no le permite realizar ningún cambio.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/workspaces/read
- Microsoft.DesktopVirtualization/applicationgroups/read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="user-session-operator"></a>Operador de sesión de usuario

El rol Operador de sesión de usuario le permite enviar mensajes, desconectar sesiones y usar la función "cerrar sesión" para cerrar sesiones fuera del host de sesión. Sin embargo, este rol no le permite realizar operaciones de administración del host de sesión como quitar el host de sesión, cambiar el modo de purga, etc. Con este rol puede ver las asignaciones, pero no puede modificar los administradores. Se recomienda asignar este rol a grupos de hosts específicos. Si concede este permiso en un nivel de grupo de recursos, el administrador tendrá permiso de lectura en todos los grupos de hosts de un grupo de recursos.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/usersessions/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="session-host-operator"></a>Operador de host de sesión

El rol Operador del host de sesión permite ver y quitar hosts de sesión, así como cambiar el modo de purga. No pueden agregar hosts de sesión mediante Azure Portal porque no tienen permiso de escritura en los objetos del grupo de hosts. Si el token de registro es válido (es decir, se ha generado y no ha expirado), puede usar este rol para agregar hosts de sesión al grupo de hosts fuera de Azure Portal si el administrador tiene permisos de proceso mediante el rol Colaborador de la máquina virtual.

En la lista siguiente se describen los permisos a los que puede acceder este rol:

- Microsoft.DesktopVirtualization/hostpools/read
- Microsoft.DesktopVirtualization/hostpools/sessionhosts/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*
