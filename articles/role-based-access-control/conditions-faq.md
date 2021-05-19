---
title: Preguntas frecuentes sobre las condiciones de asignación de roles de Azure (versión preliminar)
description: Preguntas frecuentes sobre las condiciones de asignación de roles de Azure (versión preliminar)
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: conceptual
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: 93eb1afb061182105766ce4d7c864601883ec0cc
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490055"
---
# <a name="faq-for-azure-role-assignment-conditions-preview"></a>Preguntas frecuentes sobre las condiciones de asignación de roles de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿Puede elegir los nombres de contenedor de almacenamiento o la ruta de acceso del blob en el generador de condiciones en Azure Portal?**

Debe escribir el nombre del contenedor de almacenamiento, la ruta de acceso del blob, el nombre de etiqueta o los valores de la condición. No hay ninguna experiencia de selección para los valores de atributo.

**¿Puede agrupar expresiones?**

Si agrega tres o más expresiones para una acción de destino, debe definir la agrupación lógica de esas expresiones en el editor de código, Azure PowerShell o la CLI de Azure. Una agrupación lógica de `a AND b OR c` puede ser `(a AND b) OR c` o `a AND (b OR c )`.

**¿Puede agregar condiciones de asignaciones de roles en el ámbito del grupo de administración?**
  
Azure Portal no permite editar ni ver una condición en el ámbito del grupo de administración. La columna **Condition** no se muestra para el ámbito del grupo de administración. Azure PowerShell y la CLI de Azure permiten agregar condiciones en el ámbito del grupo de administración.

**¿Se admiten condiciones mediante Azure AD Privileged Identity Management (PIM) para recursos de Azure en versión preliminar?**

No.

**¿Se admiten condiciones para los administradores clásicos?**

No. 

**¿Puede agregar condiciones a asignaciones de roles personalizadas?**

Sí, siempre que el rol personalizado incluya [acciones que admitan condiciones](conditions-format.md#actions).
 
**¿Aumentan las condiciones la latencia de acceso a Azure Storage Blob?**

No, en función de nuestro punto de referencia, no se espera que las condiciones agreguen ninguna latencia de usuario.

**¿Qué nuevas propiedades se han introducido en el esquema de asignación de roles para admitir condiciones?**

Estas son las nuevas propiedades de condición:

- `condition`: instrucción de condición compilada mediante una o varias acciones de definición de roles y atributos.
- `conditionVersion`: número de versión de una condición. El valor predeterminado es 2.0 y es la única versión admitida públicamente.

También hay una nueva propiedad de descripción para las asignaciones de roles:

- `description`: descripción de la asignación de roles que se puede usar para describir la condición.

**¿Se aplica una condición a toda la asignación de roles o a acciones específicas?**

Una condición solo se aplica a las acciones específicas de destino.

**¿Cuáles son los límites de una condición?**

Una condición puede tener hasta 8 KB de tamaño.

**¿Cuáles son los límites de una descripción?**

Una descripción puede tener hasta 2 KB de tamaño.

**¿Es posible crear una asignación de roles con y sin una condición, pero con la misma tupla de entidad de seguridad, definición de roles y ámbito?**

No, si intenta crear esta asignación de roles, se muestra un error.

**¿Las condiciones de las asignaciones de roles ofrecen un efecto de denegación explícito?**

No, las condiciones de las asignaciones de roles no tienen un efecto de denegación explícito. Las condiciones de las asignaciones de roles filtran el acceso concedido en una asignación de roles, lo que puede dar lugar a que no se permita el acceso. El efecto de denegación explícito forma parte de las asignaciones de denegación.

## <a name="next-steps"></a>Pasos siguientes

- [Sintaxis y formato de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-format.md)
- [Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-troubleshoot.md)
