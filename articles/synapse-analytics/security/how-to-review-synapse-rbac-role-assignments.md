---
title: Cómo examinar asignaciones de roles de Synapse RBAC en Synapse Studio
description: En este artículo se describe cómo examinar asignaciones de roles de Synapse RBAC en Synapse Studio
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/1/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: acc1ed8cece087628f6f8deb41696acefe124874
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110784040"
---
# <a name="how-to-review-synapse-rbac-role-assignments"></a>Cómo examinar asignaciones de roles de Synapse RBAC

Los roles de Synapse RBAC se usan para asignar permisos a usuarios, grupos y otras entidades de seguridad para permitir el acceso y el uso de los recursos de Synapse.  [Más información](./synapse-workspace-synapse-rbac.md)

En este artículo se explica cómo revisar las asignaciones de roles actuales para un área de trabajo.

Con cualquier rol de Synapse RBAC, puede enumerar las asignaciones de roles de Synapse RBAC para todos los ámbitos, incluidas las asignaciones de objetos a los que no tiene acceso. Solo un administrador de Synapse puede conceder acceso a Synapse RBAC.  

>[!Note]
> Los usuarios invitados (usuarios de un inquilino de AD distinto) también pueden ver y administrar las asignaciones de roles, después de que se les asigne el rol de administrador de Synapse.    

## <a name="open-synapse-studio"></a>Abrir Synapse Studio  

Para revisar las asignaciones de roles, primero [abra Synapse Studio](https://web.azuresynapse.net/) y seleccione el área de trabajo. 

![Inicio de sesión en el área de trabajo](./media/common/login-workspace.png) 
 
 Una vez que haya abierto el área de trabajo, seleccione el **centro de administración** a la izquierda, expanda la sección **Seguridad** y seleccione **Control de acceso**. 

 ![Seleccione Control de acceso en la sección Seguridad de la izquierda](./media/how-to-manage-synapse-rbac-role-assignments/left-nav-security-access-control.png)

## <a name="review-workspace-role-assignments"></a>Revisar las asignaciones de roles del área de trabajo

En la pantalla de control de acceso se muestran todas las asignaciones de roles actuales para el área de trabajo, agrupadas por rol. Cada asignación incluye el nombre de entidad de seguridad, el tipo de entidad de seguridad, el rol y su ámbito.

![Pantalla de control de acceso](./media/how-to-review-synapse-rbac-role-assignments/access-control-assignments.png)

Si una entidad de seguridad tiene asignado el mismo rol en diferentes ámbitos, verá varias asignaciones para la entidad de seguridad, una para cada ámbito.  

Si se asigna un rol a un grupo de seguridad, verá que los roles se asignan de manera explícita al grupo pero no a los roles heredados de los grupos primarios.  

Puede filtrar la lista por nombre de entidad de seguridad o correo electrónico, y filtrar de forma selectiva los tipos de objeto, los roles y los ámbitos. Escriba su nombre o alias de correo electrónico en el filtro de nombre para ver los roles que tiene asignados. Solo un administrador de Synapse puede cambiar sus roles.

>[!Important] 
>Si es un miembro de un grupo al que se asignan roles de forma directa o indirecta, puede que tenga permisos que no se muestran.

>[!tip]
>Puede encontrar su pertenencia a grupos mediante Azure Active Directory en Azure Portal.  

Si crea una nueva área de trabajo, usted y la entidad de servicio de MSI del área de trabajo reciben automáticamente el rol de administrador de Synapse en el ámbito del área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [administrar asignaciones de roles RBAC de Synapse](./how-to-manage-synapse-rbac-role-assignments.md).

Obtenga información sobre [los roles que necesita para realizar tareas específicas](./synapse-workspace-understand-what-role-you-need.md)