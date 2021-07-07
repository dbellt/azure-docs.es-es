---
title: 'Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante Azure Portal: Azure RBAC'
description: En este tutorial, aprenderá a conceder acceso de usuario a los recursos de Azure mediante Azure Portal y el control de acceso basado en rol (Azure RBAC).
services: role-based-access-control
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 06/07/2021
ms.author: rolyon
ms.custom: subject-rbac-steps
ms.openlocfilehash: 5df6b591cc7ba3795104957b9cd11de814e2472a
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591933"
---
# <a name="tutorial-grant-a-user-access-to-azure-resources-using-the-azure-portal"></a>Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante Azure Portal

El [control de acceso basado en rol (RBAC)](overview.md) es la forma en la que se administra el acceso a los recursos de Azure. En este tutorial, se concede acceso a un usuario para crear y administrar máquinas virtuales en un grupo de recursos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conceder acceso a un usuario en el ámbito del grupo de recursos
> * Quitar acceso

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

1. En la lista de navegación, haga clic en **Grupos de recursos**.

1. Haga clic en **Nuevo** para abrir la página **Crear un grupo de recursos**.

   ![Página Crear un grupo de recursos.](./media/quickstart-assign-role-user-portal/resource-group.png)

1. Seleccione una suscripción.

1. En el campo del nombre del **Grupo de recursos**, escriba **example-group**, o cualquier otro nombre.

1. Haga clic en **Revisar y crear** y, después, **Crear** para crear el grupo de recursos.

1. Haga clic en **Actualizar** para actualizar la lista de grupos de recursos.

   El grupo de recursos nuevo aparece en la lista de grupos de recursos.

## <a name="grant-access"></a>Conceder acceso

En RBAC de Azure, es preciso asignar un rol de Azure para conceder acceso.

1. En la lista de **Grupos de recursos**, abra el grupo de recursos **example-group** nuevo.

1. En el menú de navegación, haga clic **en Control de acceso (IAM)** .

1. Haga clic en la pestaña **Asignaciones de roles** para ver la lista actual de asignaciones de roles.

   ![Página Control de acceso (IAM) de un grupo de recursos.](./media/shared/rg-role-assignments.png)


1. Haga clic en **Agregar** > **Agregar asignación de roles (versión preliminar)** .

   Si no tiene permisos para asignar roles, la opción Agregar asignación de roles se deshabilitará.

   ![Página Control de acceso (IAM) con el menú Agregar asignación de roles abierto.](../../includes/role-based-access-control/media/add-role-assignment-menu-generic.png)

1. En la pestaña **Rol**, seleccione el rol **Colaborador de la máquina virtual**.

    ![Página Agregar asignación de roles con la pestaña Rol seleccionada.](../../includes/role-based-access-control/media/add-role-assignment-role-generic.png)

1. En la pestaña **Miembros**, selecciónese usted mismo o seleccione otro usuario.

1. En la pestaña **Revisión y asignación**, revise la configuración de la asignación de roles.

1. Haga clic en **Revisión y asignación** para asignar el rol.

   Transcurridos unos instantes, se asigna el rol Colaborador de la máquina virtual al usuario en el ámbito del grupo de recursos example-group.

   ![Asignación del rol Colaborador de la máquina virtual.](./media/quickstart-assign-role-user-portal/vm-contributor-assignment.png)

## <a name="remove-access"></a>Eliminación de acceso

En Azure RBAC, para quitar el acceso hay que quitar una asignación de roles.

1. En la lista de asignaciones de roles, agregue una marca de verificación junto al usuario con el rol Colaborador de la máquina virtual.

1. Haga clic en **Quitar**.

   ![Mensaje de eliminación de asignaciones de roles.](./media/quickstart-assign-role-user-portal/remove-role-assignment.png)

1. En el mensaje de eliminación de asignación de roles que aparece, haga clic en **Sí**.

## <a name="clean-up"></a>Limpieza

1. En la lista de navegación, haga clic en **Grupos de recursos**.

1. Haga clic en **example-group** para abrir el grupo de recursos.

1. Haga clic en **Eliminar grupo de recursos** para eliminar el grupo de recursos.

1. En el panel **¿Está seguro de que desea eliminar?** , escriba el nombre del grupo de recursos y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Concesión de acceso de usuario a recursos de Azure mediante Azure PowerShell](tutorial-role-assignments-user-powershell.md)
