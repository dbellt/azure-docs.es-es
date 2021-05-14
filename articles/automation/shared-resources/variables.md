---
title: Administración de variables en Azure Automation
description: En este artículo se explica cómo trabajar con variables en runbooks y configuraciones DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 03/28/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9f1ace00356583dbb6102317e3d157fb58682710
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832586"
---
# <a name="manage-variables-in-azure-automation"></a>Administración de variables en Azure Automation

Los recursos de variables son valores que están disponibles para todos los runbooks y configuraciones de DSC en su cuenta de Automation. Puede administrarlos desde Azure Portal, PowerShell, un runbook o en una configuración de DSC.

Las variables de Automation son útiles para los siguientes escenarios:

- Uso compartido de un valor entre varios runbooks o configuraciones de DSC.

- Uso compartido de un valor entre varios trabajos del mismo runbook o configuración de DSC.

- Administración de un valor que usan los runbooks o las configuraciones de DSC desde el portal o la línea de comandos de PowerShell. Un ejemplo es un conjunto de elementos de configuración comunes, como una lista de nombres de máquinas virtuales específica, un grupo de recursos específico, un nombre de dominio de AD y más.  

Azure Automation conserva las variables y hace que estén disponibles aunque se produzca un error en un runbook o en la configuración de DSC. Este comportamiento permite que un runbook o configuración de DSC establezca un valor para que otro runbook lo use, o bien que lo use el mismo runbook o la misma configuración de DSC la siguiente vez que se ejecute.

Azure Automation almacena cada variable cifrada de forma segura. Al crear una variable, puede especificar su cifrado y almacenamiento por Azure Automation como recurso seguro. Después de crear la variable, no se puede cambiar su estado de cifrado sin volver a crearla. Si tiene variables en la cuenta de Automation que almacenan datos confidenciales que todavía no están cifrados, debe eliminarlos y volver a crearlos como variables cifradas. Una recomendación de Azure Security Center es cifrar todas las variables de Azure Automation como se describe en [Las variables de la cuenta de Automation deben estar cifradas](../../security-center/recommendations-reference.md#recs-compute). Si tiene variables sin cifrar que quiere excluir de esta recomendación de seguridad, consulte [Exclusión de un recurso de las recomendaciones y la puntuación segura](../../security-center/exempt-resource.md) para crear una regla de exención.

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Azure Automation con una clave única que se genera para cada cuenta de Automation. Azure Automation almacena la clave en la instancia de Key Vault administrada por el sistema. Antes de almacenar un recurso seguro, Azure Automation carga la clave desde Key Vault y después la usa para cifrar el recurso.

## <a name="variable-types"></a>Tipos de variables

Cuando se crea una variable con Azure Portal, debe especificar un tipo de datos en la lista desplegable para que el portal pueda mostrar el control adecuado para escribir el valor de la variable. Los siguientes tipos de variable están disponibles en Azure Automation:

* String
* Entero
* DateTime
* Boolean
* Null

La variable no se limita al tipo de datos especificado. Debe establecer la variable mediante Windows PowerShell si se desea especificar un valor de un tipo diferente. Si indica `Not defined`, el valor de la variable se establece en NULL. Debe establecer el valor con el cmdlet [Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable) o el cmdlet interno `Set-AutomationVariable`. Use `Set-AutomationVariable` en los runbooks que están diseñados para ejecutarse en el entorno de espacio aislado de Azure o en un Hybrid Runbook Worker de Windows.

Azure Portal no se puede usar para crear o cambiar el valor de un tipo de variable complejo. Sin embargo, puede proporcionar un valor de cualquier tipo mediante Windows PowerShell. Los tipos complejos se recuperan como [Newtonsoft.Json.Linq.JProperty](https://www.newtonsoft.com/json/help/html/N_Newtonsoft_Json_Linq.htm) para un tipo de objeto complejo, en lugar de un [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject) de tipo PSObject.

Puede almacenar varios valores en una única variable mediante la creación de una matriz o tabla hash y guardarlos en la variable.

>[!NOTE]
>Las variables de nombre de máquina virtual pueden tener un máximo de 80 caracteres. Las variables de grupo de recursos pueden tener un máximo de 90 caracteres. Consulte [Reglas y restricciones de nomenclatura para los recursos de Azure](../../azure-resource-manager/management/resource-name-rules.md).

## <a name="powershell-cmdlets-to-access-variables"></a>Cmdlets de PowerShell para acceder a las variables

Los cmdlets de la tabla siguiente permiten crear y administrar variables de Automation con PowerShell. Se suministran como un componente de los [módulos Az](modules.md#az-modules).

| Cmdlet | Descripción |
|:---|:---|
|[Get-AzAutomationVariable](/powershell/module/az.automation/get-azautomationvariable) | Recupera el valor de una variable existente. Si el valor es de un tipo simple, se recupera ese mismo tipo. Si es un tipo complejo, se recupera un tipo `PSCustomObject`. <sup>1</sup>|
|[New-AzAutomationVariable](/powershell/module/az.automation/new-azautomationvariable) | Crea una nueva variable y establece su valor.|
|[Remove-AzAutomationVariable](/powershell/module/az.automation/remove-azautomationvariable)| Quita una variable existente.|
|[Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable)| Establece el valor de una variable existente. |

<sup>1</sup> Este cmdlet no se puede usar para recuperar el valor de una variable cifrada. La única forma de hacerlo es usando el cmdlet `Get-AutomationVariable` interno en un runbook o una configuración de DSC. Por ejemplo, para ver el valor de una variable cifrada, puede crear un runbook para obtener la variable y escribirla en el flujo de salida:

```powershell
$encryptvar = Get-AutomationVariable -Name TestVariable
Write-output "The encrypted value of the variable is: $encryptvar"
```

## <a name="internal-cmdlets-to-access-variables"></a>Cmdlets internos para acceder a las variables

Los cmdlets internos de la tabla siguiente se usan para acceder a las variables en los runbooks y las configuraciones de DSC. Estos cmdlets se presentan con el módulo global `Orchestrator.AssetManagement.Cmdlets`. Para más información, consulte [Cmdlets internos](modules.md#internal-cmdlets).

| Cmdlet interno | Descripción |
|:---|:---|
|`Get-AutomationVariable`|Recupera el valor de una variable existente.|
|`Set-AutomationVariable`|Establece el valor de una variable existente.|

> [!NOTE]
> Evite usar variables en el parámetro `Name` de cmdlet `Get-AutomationVariable` en un runbook o configuración de DSC. El uso de una variable puede complicar la detección de dependencias entre runbooks y variables de Automation durante el tiempo de diseño.

## <a name="python-functions-to-access-variables"></a>Funciones de Python para acceder a las variables

Las funciones de la siguiente tabla sirven para acceder a las variables de un runbook de Python 2 y Python 3. Actualmente, los runbooks de Python 3 están en versión preliminar.

|Funciones de Python|Descripción|
|:---|:---|
|`automationassets.get_automation_variable`|Recupera el valor de una variable existente. |
|`automationassets.set_automation_variable`|Establece el valor de una variable existente. |

> [!NOTE]
> Para acceder a las funciones del activo, debe importar el módulo `automationassets` en la parte superior del runbook de Python.

## <a name="create-and-get-a-variable"></a>Creación y obtención de una variable

>[!NOTE]
>Si va a quitar el cifrado de una variable, debe eliminarla y volver a crearla sin cifrado.

### <a name="create-and-get-a-variable-using-the-azure-portal"></a>Creación y obtención de una variable con Azure Portal

1. En la cuenta de Automation, en el panel izquierdo, seleccione **Variables** en **Recursos compartidos**.
2. En la página **Variables**, seleccione **Agregar una variable**.
3. Complete las opciones de la página **Nueva variable** y, luego, haga clic en **Crear** para guardar la nueva variable.

> [!NOTE]
> Una vez que una variable cifrada se guarde, no podrá verse en el portal. Solo se podrá actualizar.

### <a name="create-and-get-a-variable-in-windows-powershell"></a>Creación y obtención de una variable en Windows PowerShell

El runbook o la configuración de DSC usa el cmdlet `New-AzAutomationVariable` para crear una nueva variable y establecer su valor inicial. Si la variable está cifrada, la llamada debe usar el parámetro `Encrypted`. El script puede recuperar el valor de la variable mediante `Get-AzAutomationVariable`.

>[!NOTE]
>Un script de PowerShell no puede recuperar un valor cifrado. La única forma de hacerlo es usar el cmdlet `Get-AutomationVariable` interno.

En el siguiente ejemplo se muestra cómo crear una variable de tipo cadena y después se devuelve su valor.

```powershell
$rgName = "ResourceGroup01"
$accountName = "MyAutomationAccount"
$variableValue = "My String"

New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" 
-AutomationAccountName "MyAutomationAccount" -Name 'MyStringVariable' `
-Encrypted $false -Value 'My String'
$string = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
-AutomationAccountName "MyAutomationAccount" -Name 'MyStringVariable').Value
```

En el siguiente ejemplo se muestra cómo crear una variable con un tipo complejo y después se recuperan sus propiedades. En este caso, se usa un objeto de máquina virtual de [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) que especifica un subconjunto de sus propiedades.

```powershell
$rgName = "ResourceGroup01"
$accountName = "MyAutomationAccount"

$vm = Get-AzVM -ResourceGroupName "ResourceGroup01" -Name "VM01" | Select Name, Location, Extensions
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" -AutomationAccountName "MyAutomationAccount" -Name "MyComplexVariable" -Encrypted $false -Value $vm

$vmValue = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
-AutomationAccountName "MyAutomationAccount" -Name "MyComplexVariable"

$vmName = $vmValue.Value.Name
$vmTags = $vmValue.Value.Tags
```

## <a name="textual-runbook-examples"></a>Ejemplo de un runbook textual

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los comandos de ejemplo siguientes muestran cómo establecer y recuperar una variable en un runbook textual. En este ejemplo se da por hecho que ha creado variables de entero con el nombre **numberOfIterations** y **numberOfRunnings**, así como una variable de cadena con el nombre **sampleMessage**.

```powershell
$rgName = "ResourceGroup01"
$accountName = "MyAutomationAccount"

$numberOfIterations = Get-AutomationVariable -Name "numberOfIterations"
$numberOfRunnings = Get-AutomationVariable -Name "numberOfRunnings"
$sampleMessage = Get-AutomationVariable -Name "sampleMessage"

Write-Output "Runbook has been run $numberOfRunnings times."

for ($i = 1; $i -le $numberOfIterations; $i++) {
    Write-Output "$i`: $sampleMessage"
}
Set-AutomationVariable -Name numberOfRunnings -Value ($numberOfRunnings += 1)
```

# <a name="python-2"></a>[Python 2](#tab/python2)

En el siguiente ejemplo se muestra cómo obtener una variable, establecerla y administrar una excepción de una variable inexistente en un runbook de Python 2.

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print "variable not found"
```

# <a name="python-3"></a>[Python 3](#tab/python3)

En el siguiente ejemplo se muestra cómo obtener una variable, establecerla y administrar una excepción de una variable inexistente en un runbook de Python 3 (versión preliminar).

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print ("variable not found")
```

---

## <a name="graphical-runbook-examples"></a>Ejemplos de runbook gráfico

En un runbook gráfico, puede agregar actividades para **Get-AutomationVariable** o **Set-AutomationVariable** de los cmdlet internos. Basta con hacer clic con el botón derecho en cada variable en el panel Biblioteca del editor gráfico y seleccionar la actividad que se va a agregar.

![Adición de una variable al lienzo](../media/variables/runbook-variable-add-canvas.png)

La imagen siguiente muestra las actividades de ejemplo para actualizar una variable con un valor simple en un runbook gráfico. En este ejemplo, la actividad para `Get-AzVM` recupera una única máquina virtual de Azure y guarda el nombre del equipo en una variable de cadena de Automation existente. No importa si el [vínculo es una canalización o una secuencia](../automation-graphical-authoring-intro.md#use-links-for-workflow), ya que el código solo espera un único objeto en la salida.

![Establecimiento de una variable simple](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los cmdlets que se usan para acceder a las variables, consulte [Administración de módulos en Azure Automation](modules.md).

- Para información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).

- Para obtener más información sobre las configuraciones DSC, consulte [Introducción a State Configuration](../automation-dsc-overview.md).
