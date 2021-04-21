---
title: archivo de inclusión
description: archivo de inclusión
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 03/19/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: a00291182059506aeab9cde965fa4cbd5177ecf7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "80132089"
---
Si un usuario no tiene el acceso necesario para aplicar etiquetas, puede asignarle el rol **Colaborador de etiquetas**. Para más información, consulte el [Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante RBAC y Azure Portal](../articles/role-based-access-control/quickstart-assign-role-user-portal.md).

1. Para ver las etiquetas de un recurso o un grupo de recursos, busque etiquetas existentes en la información general. Si no ha aplicado etiquetas anteriormente, la lista está vacía.

   ![Visualización de las etiquetas por recurso o grupo de recursos](./media/resource-manager-tag-resources/view-tags.png)

1. Para agregar una etiqueta, seleccione **Haga clic aquí para agregar etiquetas**.

1. Proporcione un nombre y un valor.

   ![Agregar etiqueta](./media/resource-manager-tag-resources/add-tag.png)

1. Continúe agregando etiquetas según sea necesario. Cuando haya terminado, seleccione **Guardar**.

   ![Guardar etiquetas](./media/resource-manager-tag-resources/save-tags.png)

1. Las etiquetas aparecen ahora en la información general.

   ![Mostrar etiquetas](./media/resource-manager-tag-resources/view-new-tags.png)

1. Para agregar o eliminar una etiqueta, seleccione **cambiar**.

1. Para eliminar una etiqueta, seleccione el icono de papelera. Después, seleccione **Guardar**.

   ![Eliminar etiqueta](./media/resource-manager-tag-resources/delete-tag.png)

Para asignar etiquetas a varios recursos de forma masiva:

1. En cualquier lista de recursos, seleccione la casilla de los recursos a los que quiere asignar la etiqueta. Después, seleccione **Asignar etiquetas**.

   ![Seleccionar varios recursos](./media/resource-manager-tag-resources/select-multiple-resources.png)

1. Agregue nombres y valores. Cuando haya terminado, seleccione **Guardar**.

   ![Seleccionar Asignar](./media/resource-manager-tag-resources/select-assign.png)

Para ver todos los recursos con una etiqueta:

1. En el menú de Azure Portal, busque **etiquetas**. Seleccione el servicio entre las opciones disponibles.

   ![Buscar por etiqueta](./media/resource-manager-tag-resources/find-tags-general.png)

1. Seleccione la etiqueta para ver los recursos.

   ![Seleccionar etiqueta](./media/resource-manager-tag-resources/select-tag.png)

1. Se muestran todos los recursos que contienen esa etiqueta.

   ![Visualización de recursos por etiqueta](./media/resource-manager-tag-resources/view-resources-by-tag.png)
