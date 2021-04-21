---
title: Administración de grupos del marketplace comercial | Azure Marketplace
description: Obtenga información acerca de cómo administrar grupos en el programa Marketplace comercial en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: varsha-sarah
ms.author: vavargh
ms.custom: contperf-fy21q2
ms.date: 04/07/2021
ms.openlocfilehash: aa75082142aa5735350cd7fed30194314b6e7368
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107108077"
---
# <a name="manage-groups-for-the-commercial-marketplace"></a>Administración de grupos para marketplace comercial

Los grupos le permiten controlar varios roles y permisos de usuario a la vez.

## <a name="add-an-existing-group"></a>Agregar un grupo existente

Para agregar un grupo que ya existe en la cuenta profesional (inquilino de Azure AD) de la organización a la cuenta del Centro de partners:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar grupos**.
1. Seleccione uno o varios grupos en la lista que aparece. Puede usar el cuadro de búsqueda para buscar grupos concretos. Si selecciona más de un grupo para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varios grupos con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
1. Cuando haya terminado de elegir los grupos, seleccione **Agregar selección**.
1. En la sección **Roles**, especifique los roles o permisos personalizados de los grupos seleccionados. Todos los miembros del grupo podrán acceder a la cuenta del Centro de partners con los permisos que aplique al grupo, independientemente de los roles y permisos asociados a su cuenta individual.
1. Seleccione **Guardar**.

Cuando se agrega un grupo existente, todos los usuarios que son miembros de ese grupo pueden acceder a su cuenta del Centro de partners con los permisos asociados al rol asignado al grupo.

## <a name="add-a-new-group"></a>Agregar un nuevo grupo

Para agregar un nuevo grupo a la cuenta del Centro de partners:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar grupos**.
1. En la siguiente página, seleccione **Nuevo grupo**.
1. Escriba el nombre para mostrar del nuevo grupo.
1. Especifique los roles o permisos personalizados del grupo. Todos los miembros del grupo podrán acceder a la cuenta del Centro de partners con los permisos que aplique aquí, independientemente de los roles y permisos asociados a su cuenta individual.
1. Seleccione los usuarios del nuevo grupo en la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios concretos.
1. Cuando haya terminado de seleccionar usuarios, seleccione **Agregar selección** para agregarlos al grupo nuevo.
1. Seleccione **Guardar**.

Este nuevo grupo se creará en la cuenta profesional (inquilino de Azure AD) de la organización, no solo en la cuenta del Centro de partners.

## <a name="remove-a-group"></a>Eliminar un grupo

Para quitar un grupo de la cuenta de trabajo (inquilino de Azure AD):
1. Vaya a **Usuarios** (en **Configuración de la cuenta**).
1. Seleccione el grupo que quiere quitar con la casilla en la columna de la derecha.
1. Elija **Quitar** de las acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar los grupos seleccionados.
