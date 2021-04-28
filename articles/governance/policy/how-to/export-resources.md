---
title: Exportación de recursos de Azure Policy
description: Obtenga más información sobre cómo exportar recursos de Azure Policy a GitHub, por ejemplo, definiciones y asignaciones de directivas.
ms.date: 03/31/2021
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ca7e5de5238bab50e31b25d6a0bf5068aad397a8
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108073654"
---
# <a name="export-azure-policy-resources"></a>Exportación de recursos de Azure Policy

En este artículo se ofrece información sobre cómo exportar los recursos de Azure Policy existentes. La exportación de los recursos es útil y se recomienda para la copia de seguridad, pero también es un paso importante en su recorrido por la gobernanza de la nube y el tratamiento de [policy-as-code](../concepts/policy-as-code.md). Los recursos de Azure Policy se pueden exportar a través de [Azure Portal](#export-with-azure-portal), la [CLI de Azure](#export-with-azure-cli), [Azure PowerShell](#export-with-azure-powershell) y cada uno de los SDK admitidos.

## <a name="export-with-azure-portal"></a>Exportación con Azure Portal

Para exportar una definición de directiva de Azure Portal, siga estos pasos:

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

1. Seleccione **Definiciones** en el lado izquierdo de la página de Azure Policy.

1. Use el botón **Exportar definiciones**, o bien seleccione el botón de los puntos suspensivos que encontrará en la fila de la definición de una directiva y, a continuación, seleccione **Exportar definición**.

1. Seleccione el botón **Iniciar sesión con GitHub**. Si aún no se ha autenticado en GitHub para autorizar a Azure Policy a exportar el recurso, revise el acceso que [Acciones de GitHub](https://github.com/features/actions) requiere en la nueva ventana que se abre y seleccione **Autorizar AzureGitHubActions** para continuar con el proceso de exportación. Una vez completado este paso, la nueva ventana se cerrará automáticamente.

1. En la pestaña **Aspectos básicos**, establezca las siguientes opciones y,a continuación, seleccione la pestaña **Directivas** o el botón **Siguiente: Directivas** situado en la parte inferior de la página.

   - **Filtro del repositorio**: establezca esta opción en _Mis repositorios_ si solo quiere ver los repositorios que posee, o bien en _Todos los repositorios_ para ver todos aquellos a los que concedió acceso a Acciones de GitHub.
   - **Repositorio**: establezca esta opción en el repositorio en el que quiera exportar los recursos de Azure Policy.
   - **Rama**: establezca la rama del repositorio. Usar una rama distinta de la predeterminada es una buena manera de validar las actualizaciones antes de combinarlas en el código fuente.
   - **Directorio**: se trata de la _carpeta de nivel raíz_ en la que se exportarán los recursos de Azure Policy. Las subcarpetas de este directorio se crean en función de los recursos que se exportan.

1. En la pestaña **Directivas**, establezca el ámbito de búsqueda; para ello, seleccione el botón de los puntos suspensivos y elija una combinación de grupos de administración, suscripciones o grupos de recursos.
   
1. Use el botón **Agregar definiciones de directiva** para realizar la búsqueda en el ámbito cuyos objetos se van a exportar. En la ventana lateral que se abre, seleccione cada uno de los objetos que quiera exportar. Filtre la selección mediante el cuadro de búsqueda o el tipo. Una vez que haya seleccionado todos los objetos que se van a exportar, use el botón **Agregar** de la parte inferior de la página.

1. Para cada objeto seleccionado, seleccione las opciones de exportación que prefiera, como _Solo definición_ o _Definición y asignaciones_, para una definición de directiva. Luego, seleccione la pestaña **Revisar y exportar** o el botón **Siguiente: Revisar y exportar** situado en la parte inferior de la página.

   > [!NOTE]
   > Si se elige la opción _Definición y asignaciones_, solo se exportarán las asignaciones de directiva comprendidas en el ámbito establecido por el filtro al agregar la definición de directiva.

1. En la pestaña **Revisar y exportar**, compruebe que los detalles coincidan y, luego, use el botón **Exportar** situado en la parte inferior de la página.

1. Compruebe el repositorio de GitHub, la rama y la _carpeta de nivel raíz_ para ver que los recursos seleccionados se hayan exportado en el control de código fuente.

Los recursos de Azure Policy se exportan en la siguiente estructura dentro del repositorio y la _carpeta de nivel raíz_ de GitHub que haya seleccionado:

```text
|
|- <root level folder>/  ________________ # Root level folder set by Directory property
|  |- policies/  ________________________ # Subfolder for policy objects
|     |- <displayName>_<name>____________ # Subfolder based on policy displayName and name properties
|        |- policy.json _________________ # Policy definition
|        |- assign.<displayName>_<name>__ # Each assignment (if selected) based on displayName and name properties
|
```

## <a name="export-with-azure-cli"></a>Exportación con la CLI de Azure

Cada una de las definiciones, iniciativas y asignaciones de Azure Policy se pueden exportar como JSON con la [CLI de Azure](/cli/azure/install-azure-cli). Cada uno de estos comandos usa un parámetro **name** para especificar el objeto para el que se va a obtener el JSON. La propiedad **name** suele ser un _GUID_, no el valor **displayName** del objeto.

- Definición: [az policy definition show](/cli/azure/policy/definition#az_policy_definition_show)
- Iniciativa: [az policy set-definition show](/cli/azure/policy/set-definition#az_policy_set_definition_show)
- Asignación: [az policy assignment show](/cli/azure/policy/assignment#az_policy_assignment_show)

Este es un ejemplo de cómo obtener el JSON de una definición de directiva con el valor **name** de _VirtualMachineStorage_:

```azurecli-interactive
az policy definition show --name 'VirtualMachineStorage'
```

## <a name="export-with-azure-powershell"></a>Exportación con Azure PowerShell

Cada una de las definiciones, iniciativas y asignaciones de Azure Policy se pueden exportar como JSON con [Azure PowerShell](/powershell/azure/). Cada uno de estos cmdlets usa un parámetro **name** para especificar el objeto para el que se va a obtener el JSON. La propiedad **Name** suele ser un _GUID_, no el valor **displayName** del objeto.

- Definición: [Get-AzPolicyDefinition](/powershell/module/az.resources/get-azpolicydefinition)
- Iniciativa: [Get-AzPolicySetDefinition](/powershell/module/az.resources/get-azpolicysetdefinition)
- Asignación: [Get-AzPolicyAssignment](/powershell/module/az.resources/get-azpolicyassignment)

Este es un ejemplo de cómo obtener el JSON de una definición de directiva con el valor **Name** de _VirtualMachineStorage_:

```azurepowershell-interactive
Get-AzPolicyDefinition -Name 'VirtualMachineStorage' | ConvertTo-Json -Depth 10
```

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
