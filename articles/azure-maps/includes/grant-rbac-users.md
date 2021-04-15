---
title: Concesión de acceso basado en roles para usuarios
titleSuffix: Azure Maps
description: Uso del control de acceso basado en roles para conceder a los usuarios autorización para Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/17/2020
ms.topic: include
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 234c3358b98b7af677793fba58c1602a8bd976f0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "101102825"
---
## <a name="grant-role-based-access-for-users-to-azure-maps"></a>Concesión de acceso basado en roles para usuarios de Azure Maps

Para conceder el *control de acceso basado en rol de Azure (RBAC de Azure)* , se asignan una o varias definiciones de roles de Azure a un grupo de Azure AD o a entidades de seguridad. Para ver las definiciones de roles de Azure disponibles para Azure Maps, vaya a **Control de acceso (IAM)** . Seleccione **Roles** y busque los roles que comienzan con *Azure Maps*.

* Para administrar de forma eficaz una gran cantidad de acceso de usuarios a Azure Maps, consulte [Grupos de Azure AD](../../active-directory/fundamentals/active-directory-manage-groups.md).
* Para que los usuarios puedan autenticarse en la aplicación, se deben crear en Azure AD. Consulte [Incorporación o eliminación de usuarios mediante Azure AD](../../active-directory/fundamentals/add-users-azure-active-directory.md).

Obtenga más información en [Azure AD](../../active-directory/fundamentals/index.yml) para administrar de manera eficaz un directorio para los usuarios.

1. Vaya a su **cuenta de Azure Maps**. Seleccione **Control de acceso (IAM)**  > **Asignación de roles**.

    ![Concesión del acceso mediante RBAC de Azure](../media/how-to-manage-authentication/how-to-grant-rbac.png)

2. En la pestaña **Asignaciones de roles**, en **Rol**, seleccione una definición de roles de Azure Maps integrada, como **Lector de datos de Azure Maps** o **Colaborador de datos de Azure Maps**. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD**. Seleccione la entidad de seguridad por nombre. Después, seleccione **Guardar**.

   * Consulte los detalles sobre la [asignación de roles de Azure](../../role-based-access-control/role-assignments-portal.md).

> [!WARNING]
> Las definiciones de roles de Azure Maps integradas proporcionan un acceso de autorización muy amplio a muchas API REST de Azure Maps. Para restringir al mínimo las API para los usuarios, consulte cómo [crear una definición de roles personalizada y asignar usuarios](../../role-based-access-control/custom-roles.md) a la definición de roles personalizada. Esto proporcionará a los usuarios el privilegio mínimo necesario para la aplicación.