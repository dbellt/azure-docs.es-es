---
title: Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)
description: Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: troubleshooting
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: eddebb6742a50ce729d436fc19ce4ff32491f75f
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490000"
---
# <a name="troubleshoot-azure-role-assignment-conditions-preview"></a>Solución de problemas de las condiciones de asignación de roles de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="symptom---condition-is-not-enforced"></a>Síntoma: no se aplica la condición

**Causa 1**

Las entidades de seguridad tienen una o varias asignaciones de roles en el mismo ámbito o en un ámbito superior.

**Solución 1**

Asegúrese de que las entidades de seguridad no tienen varias asignaciones de roles (con o sin condiciones) que concedan acceso a la misma acción de datos, lo que lleva a que no se apliquen las condiciones. Para obtener información sobre la lógica de evaluación, consulte [Cómo determina Azure RBAC si un usuario tiene acceso a un recurso](overview.md#how-azure-rbac-determines-if-a-user-has-access-to-a-resource).

**Causa 2**

La asignación de roles tiene varias acciones que conceden un permiso y la condición no tiene como destino todas las acciones. Por ejemplo, puede crear un blob si tiene las acciones de datos `/blobs/write` o `/blobs/add/action`. Si la asignación de roles tiene ambas acciones de datos y solo establece una de ellas como destino en una condición, la asignación de roles concederá el permiso para crear blobs y omitirá la condición.

**Solución 2**

Si la asignación de roles tiene varias acciones que conceden un permiso, asegúrese de que establece como destino todas las acciones pertinentes.

**Causa 3**

Cuando agrega una condición a una asignación de roles, la condición puede tardar hasta cinco minutos en aplicarse. Al agregar una condición, la actualización se notifica a los proveedores de recursos (como Microsoft.Storage). Estos aplican inmediatamente las actualizaciones en sus cachés locales para asegurarse de que tienen las asignaciones de roles más recientes. Este proceso se lleva a cabo en uno o dos minutos, pero puede tardar hasta cinco en completarse.

**Solución 3**

Espere cinco minutos y vuelva a probar la condición.

## <a name="symptom---condition-is-not-valid-error-when-adding-a-condition"></a>Síntoma: error que indica que la condición no es válida al agregarla

Al intentar agregar una asignación de roles con una condición, aparece un error similar al siguiente:

`The given role assignment condition is invalid.`

**Causa**

La condición no tiene el formato correcto. 

**Solución**

Corrija cualquier problema de [sintaxis o formato de la condición](conditions-format.md). También tiene la opción de agregar la condición mediante el [editor visual en Azure Portal](conditions-role-assignments-portal.md).

## <a name="symptom---resource-attribute-is-not-valid-error-when-adding-a-condition-using-azure-powershell"></a>Síntoma: error que indica que el atributo de recurso no es válido al agregar una condición mediante Azure PowerShell

Al intentar agregar una asignación de roles con una condición mediante Azure PowerShell, aparece un error similar al siguiente:

```
New-AzRoleAssignment : Resource attribute
Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$> is not valid.
```

**Causa**

Si la condición incluye un signo de dólar ($), debe agregarle un carácter de acento grave (\`) como prefijo.

**Solución**

Agregue un carácter de acento grave (\`) antes de cada signo de dólar. A continuación se muestra un ejemplo. Para obtener más información sobre las reglas de comillas en PowerShell, consulte [Acerca de las reglas de las comillas](/powershell/module/microsoft.powershell.core/about/about_quoting_rules).

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade'))"
```

## <a name="symptom---resource-attribute-is-not-valid-error-when-adding-a-condition-using-azure-cli"></a>Síntoma: error que indica que el atributo de recurso no es válido al agregar una condición mediante la CLI de Azure

Al intentar agregar una asignación de roles con una condición mediante la CLI de Azure, aparece un error similar al siguiente:

```
Resource attribute Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$> is not valid.
```

**Causa**

Si la condición incluye un signo de dólar ($), debe agregarle una barra diagonal inversa (\\) como prefijo.

**Solución**

Agregue una barra diagonal inversa (\\) antes de cada signo de dólar. A continuación se muestra un ejemplo. Para más información sobre las reglas de comillas en Bash, consulte [Comillas dobles](https://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html).

```azurecli
condition="((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<\$key_case_sensitive\$>] StringEquals 'Cascade'))"
```

## <a name="symptom---error-when-assigning-a-condition-string-to-a-variable-in-bash"></a>Síntoma: error al asignar una cadena de condición a una variable en Bash

Al intentar asignar una cadena de condición a una variable en Bash, aparece el mensaje `bash: !: event not found`.

**Causa**

En Bash, si la expansión del historial está habilitada, es posible que aparezca el mensaje `bash: !: event not found` debido al signo de exclamación (!).

**Solución**

Deshabilite la expansión del historial con el comando `set +H`. Para volver a habilitarla, utilice `set -H`.

## <a name="symptom---unrecognized-arguments-error-when-adding-a-condition-using-azure-cli"></a>Síntoma: error de argumentos no reconocidos al agregar una condición mediante la CLI de Azure

Al intentar agregar una asignación de roles con una condición mediante la CLI de Azure, aparece un error similar al siguiente:

`az: error: unrecognized arguments: --description {description} --condition {condition} --condition-version 2.0`

**Causa**

Es probable que esté usando una versión anterior de la CLI de Azure que no admite parámetros de condición en la asignación de roles.

**Solución**

Actualice a la versión más reciente de la CLI de Azure (2.18 o posterior). Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="symptom---condition-not-recognized-in-visual-editor"></a>Síntoma: condición no reconocida en el editor visual

Después de usar el editor de código, cambia al editor visual y aparece un mensaje similar al siguiente:

`The current expression cannot be recognized. Switch to the code editor to edit the expression or delete the expression and add a new one.`

**Causa**

Se realizaron actualizaciones en la condición que el editor visual no puede analizar.

**Solución**

Corrija cualquier problema de [sintaxis o formato de la condición](conditions-format.md). También tiene la opción de eliminar la condición e intentarlo de nuevo.

## <a name="symptom---error-when-copying-and-pasting-a-condition"></a>Síntoma: error al copiar y pegar una condición

**Causa**

Si copia una condición de un documento, podría incluir caracteres especiales y provocar errores. Algunos editores (como Microsoft Word) agregan caracteres de control al dar formato al texto que no se muestran.

**Solución**

Si tiene la seguridad de que la condición es correcta, elimine todos los espacios y retornos y, a continuación, vuelva a agregar los espacios necesarios.

## <a name="next-steps"></a>Pasos siguientes

- [Sintaxis y formato de las condiciones de asignación de roles de Azure (versión preliminar)](conditions-format.md)
- [Preguntas frecuentes sobre las condiciones de asignación de roles de Azure (versión preliminar)](conditions-faq.md)
- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
