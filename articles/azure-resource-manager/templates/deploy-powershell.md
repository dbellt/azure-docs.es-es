---
title: Implementación de recursos con una plantilla y PowerShell
description: Use Azure Resource Manager y Azure PowerShell para implementar recursos para Azure. Los recursos se definen en una plantilla de Resource Manager o un archivo Bicep.
ms.topic: conceptual
ms.date: 03/25/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: db43613b1b359e6d6378e081f3f61ee1449bd641
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108314650"
---
# <a name="deploy-resources-with-arm-templates-and-azure-powershell"></a>Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell

En este artículo, se explica cómo se utiliza Azure PowerShell con plantillas de Azure Resource Manager (plantillas de ARM) o archivos Bicep para implementar recursos en Azure. Si no está familiarizado con los conceptos de implementación y administración de las soluciones de Azure, vea la [información general sobre plantillas](overview.md) o la [visión general sobre Bicep](bicep-overview.md).

Para implementar archivos Bicep, necesita [Azure PowerShell versión 5.6.0 o posterior](/powershell/azure/install-az-ps).

## <a name="prerequisites"></a>Requisitos previos

Necesita una plantilla para implementar. Si aún no tiene ninguna, descargue una [plantilla de ejemplo](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json) desde el repositorio Azure Quickstart Templates y guárdela. El nombre de archivo local que se utiliza en este artículo es _C:\MyTemplates\azuredeploy.json_.

Debe instalar Azure PowerShell y conectarse a Azure:

- **Instale los cmdlets de Azure PowerShell en el equipo local.** Para más información, consulte el artículo de [introducción a Azure PowerShell](/powershell/azure/get-started-azureps).
- **Conéctese a Azure utilizando [Connect-AZAccount](/powershell/module/az.accounts/connect-azaccount)** . Si tiene varias suscripciones de Azure, es posible que también tenga que ejecutar [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext). Para más información, consulte [Use multiple Azure subscriptions](/powershell/azure/manage-subscriptions-azureps) (Uso de varias suscripciones de Azure).

Si no tiene instalado PowerShell, puede usar Azure Cloud Shell. Para obtener más información, vea [Implementación de plantillas de Resource Manager desde Cloud Shell](deploy-cloud-shell.md).

## <a name="deployment-scope"></a>Ámbito de la implementación

La implementación puede tener como destino un grupo de recursos, una suscripción, un grupo de administración o un inquilino. Según el ámbito de la implementación, usará comandos diferentes.

- Para implementar en un **grupo de recursos**, utilice [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

  ```azurepowershell
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile <path-to-template-or-bicep>
  ```

- Para hacer la implementación en una **suscripción**, use [New-AzSubscriptionDeployment](/powershell/module/az.resources/new-azdeployment), que es un alias del cmdlet `New-AzDeployment`:

  ```azurepowershell
  New-AzSubscriptionDeployment -Location <location> -TemplateFile <path-to-template-or-bicep>
  ```

  Para más información sobre las implementaciones en el nivel de suscripción, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](deploy-to-subscription.md).

- Para hacer la implementación en un **grupo de administración**, use [New-AzManagementGroupDeployment](/powershell/module/az.resources/New-AzManagementGroupDeployment).

  ```azurepowershell
  New-AzManagementGroupDeployment -Location <location> -TemplateFile <path-to-template-or-bicep>
  ```

  Para obtener más información sobre las implementaciones de nivel de grupo de administración, consulte [Creación de recursos en el nivel de grupo de administración](deploy-to-management-group.md).

- Para hacer la implementación en un **inquilino**, use [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

  ```azurepowershell
  New-AzTenantDeployment -Location <location> -TemplateFile <path-to-template-or-bicep>
  ```

  Para más información sobre las implementaciones a nivel de inquilino, consulte [Creación de recursos en el nivel de inquilino](deploy-to-tenant.md).

Para cada ámbito, el usuario que implementa la plantilla debe tener permisos para crear recursos.

## <a name="deployment-name"></a>Nombre de implementación

Al implementar una plantilla de Resource Manager, puede asignarle un nombre a la implementación. Este nombre puede ayudarle a recuperar la implementación del historial de implementaciones. Si no especifica un nombre para la implementación, se utilizará el nombre del archivo de la plantilla. Por ejemplo, si implementa una plantilla llamada `azuredeploy.json` y no especifica ningún nombre para la implementación, el nombre que se asignará será `azuredeploy`.

Cada vez que se ejecuta una implementación, se agrega una entrada al historial de implementación del grupo de recursos con el nombre de la implementación. Si ejecuta otra implementación y le asigna el mismo nombre, la entrada anterior se reemplazará por la implementación actual. Si desea que todas las entradas del historial de implementaciones sean diferentes, asigne un nombre único a cada implementación.

Para crear un nombre único, puede asignar un número aleatorio.

```azurepowershell-interactive
$suffix = Get-Random -Maximum 1000
$deploymentName = "ExampleDeployment" + $suffix
```

También puede agregar un valor de fecha.

```azurepowershell-interactive
$today=Get-Date -Format "MM-dd-yyyy"
$deploymentName="ExampleDeployment"+"$today"
```

Si ejecuta implementaciones simultáneas en el mismo grupo de recursos utilizando el mismo nombre de implementación, solo se completará la última implementación. Aquellas implementaciones que tengan el mismo nombre y no hayan finalizado se sustituirán por la última implementación. Por ejemplo, si ejecuta una implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage1` y, al mismo tiempo, ejecuta otra implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage2`, solo se implementará una única cuenta de almacenamiento. La cuenta de almacenamiento resultante será `storage2`.

Sin embargo, si ejecuta una implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage1` e inmediatamente después ejecuta otra implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage2`, tendrá dos cuentas de almacenamiento. Una se llamará `storage1` y la otra, `storage2`. Sin embargo, solo tendrá una entrada en el historial de implementaciones.

Si especifica un nombre único para cada implementación, podrá ejecutarlas simultáneamente sin conflictos. Si ejecuta una implementación llamada `newStorage1` que implementa la cuenta de almacenamiento `storage1` y, al mismo tiempo, ejecuta otra implementación llamada `newStorage2` que implementa la cuenta de almacenamiento `storage2`, tendrá dos cuentas de almacenamiento y dos entradas en el historial de implementación.

Para evitar conflictos con las implementaciones simultáneas y garantizar que las entradas del historial de implementaciones son únicas, asigne un nombre diferente a cada implementación.

## <a name="deploy-local-template-or-bicep-file"></a>Implementación de una plantilla local o un archivo Bicep

Puede implementar una plantilla desde la máquina local o una que esté almacenada externamente. En esta sección se describe la implementación de una plantilla local.

Si va a realizar la implementación en un grupo de recursos que no existe, cree el grupo de recursos. El nombre del grupo de recursos solo puede incluir caracteres alfanuméricos, puntos, guiones bajos, guiones y paréntesis. Puede tener hasta 90 caracteres. El nombre no puede terminar con un punto.

```azurepowershell
New-AzResourceGroup -Name ExampleGroup -Location "Central US"
```

Para implementar una plantilla local o archivo Bicep, use el parámetro `-TemplateFile` en el comando de implementación. En el ejemplo siguiente también se muestra cómo establecer un valor de parámetro que procede de la plantilla.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile <path-to-template-or-bicep>
```

La implementación puede tardar unos minutos en finalizar.

## <a name="deploy-remote-template"></a>Implementación de una plantilla remota

> [!NOTE]
> Actualmente, Azure PowerShell no admite la implementación de archivos Bicep remotos. Use la [CLI de Bicep](./bicep-install.md#development-environment) para compilar el archivo de Bicep en una plantilla JSON y, luego, cargue el archivo JSON en la ubicación remota.

En lugar de almacenar las plantillas de ARM en el equipo local, quizás prefiera almacenarlas en una ubicación externa. Puede almacenar plantillas en un repositorio de control de código fuente (por ejemplo, GitHub). O bien, puede almacenarlas en una cuenta de Azure Storage para el acceso compartido en su organización.

[!INCLUDE [Deploy templates in private GitHub repo](../../../includes/resource-manager-private-github-repo-templates.md)]

Si va a realizar la implementación en un grupo de recursos que no existe, cree el grupo de recursos. El nombre del grupo de recursos solo puede incluir caracteres alfanuméricos, puntos, guiones bajos, guiones y paréntesis. Puede tener hasta 90 caracteres. El nombre no puede terminar con un punto.

```azurepowershell
New-AzResourceGroup -Name ExampleGroup -Location "Central US"
```

Para implementar una plantilla externa, use el parámetro `-TemplateUri`.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name remoteTemplateDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

En el ejemplo anterior, se requiere un identificador URI accesible públicamente para la plantilla, que funciona con la mayoría de los escenarios porque la plantilla no debe incluir datos confidenciales. Si tiene que especificar datos confidenciales (por ejemplo, una contraseña de administrador), pase ese valor como un parámetro seguro. Sin embargo, si quiere administrar el acceso a la plantilla, considere la posibilidad de usar [especificaciones de plantilla](#deploy-template-spec).

Para implementar plantillas vinculadas remotas con una ruta de acceso relativa que estén almacenadas en una cuenta de almacenamiento, use `QueryString` para especificar el token de SAS:

```azurepowershell
New-AzResourceGroupDeployment `
  -Name linkedTemplateWithRelativePath `
  -ResourceGroupName "myResourceGroup" `
  -TemplateUri "https://stage20210126.blob.core.windows.net/template-staging/mainTemplate.json" `
  -QueryString $sasToken
```

Para obtener más información, vea [Uso de rutas de acceso relativas para plantillas vinculadas](./linked-templates.md#linked-template).

## <a name="deploy-template-spec"></a>Implementación de la especificación de plantilla

> [!NOTE]
> Actualmente, Azure PowerShell no admite la creación de especificaciones de plantilla mediante archivos Bicep. Sin embargo, puede crear un archivo Bicep con el recurso [Microsoft.Resources/templateSpecs](/azure/templates/microsoft.resources/templatespecs) para implementar una especificación de plantilla. A continuación, se muestra un [ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/create-template-spec-using-template/azuredeploy.bicep).
En lugar de implementar una plantilla local o remota, puede crear una [especificación de plantilla](template-specs.md). La especificación de plantilla es un recurso de su suscripción de Azure que contiene una plantilla de ARM. Facilita el uso compartido de la plantilla de forma segura con los usuarios de la organización. Use el control de acceso basado en rol de Azure (RBAC de Azure) para conceder acceso a la especificación de la plantilla. Esta funcionalidad actualmente está en su versión preliminar.

En los ejemplos siguientes se muestra cómo se crea e implementa una especificación de plantilla.

En primer lugar, proporcione la plantilla de Resource Manager para crear la especificación de plantilla.

```azurepowershell
New-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0 `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateJsonFile ./mainTemplate.json
```

A continuación, obtenga el identificador de la especificación de plantilla e impleméntelo.

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0).Version.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

Para obtener más información, consulte [Especificaciones de plantilla de Azure Resource Manager (versión preliminar)](template-specs.md).

## <a name="preview-changes"></a>Vista previa de los cambios

Antes de implementar la plantilla, puede obtener una vista previa de los cambios que la plantilla realizará en su entorno. Use la [operación Y si](template-deploy-what-if.md) para comprobar que la plantilla realiza los cambios esperados. La operación y si también valida que la plantilla no tenga errores.

## <a name="pass-parameter-values"></a>Pase de valores de parámetros

Para pasar valores de parámetros, puede usar parámetros en línea o un archivo de parámetros.

### <a name="inline-parameters"></a>Parámetros en línea

Para pasar parámetros en línea, proporcione los nombres de parámetro con el comando `New-AzResourceGroupDeployment`. Por ejemplo, para pasar una cadena y una matriz a una plantilla, use:

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile <path-to-template-or-bicep> `
  -exampleString "inline string" `
  -exampleArray $arrayParam
```

También puede obtener el contenido del archivo y proporcionar ese contenido como un parámetro en línea.

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile <path-to-template-or-bicep> `
  -exampleString $(Get-Content -Path c:\MyTemplates\stringcontent.txt -Raw) `
  -exampleArray $arrayParam
```

Obtener un valor de parámetro de un archivo es útil cuando se necesita proporcionar valores de configuración. Por ejemplo, puede proporcionar [valores de cloud-init para una máquina virtual Linux](../../virtual-machines/linux/using-cloud-init.md).

Si necesita pasar una matriz de objetos, cree las tablas hash en PowerShell y agréguelas a una matriz. Pase esa matriz como un parámetro durante la implementación.

```powershell
$hash1 = @{ Name = "firstSubnet"; AddressPrefix = "10.0.0.0/24"}
$hash2 = @{ Name = "secondSubnet"; AddressPrefix = "10.0.1.0/24"}
$subnetArray = $hash1, $hash2
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile <path-to-template-or-bicep> `
  -exampleArray $subnetArray
```

### <a name="parameter-files"></a>Archivos de parámetros

En lugar de pasar parámetros como valores en línea en el script, quizá le resulte más fácil usar un archivo JSON que contiene los valores de parámetro. El archivo de parámetros puede ser un archivo local o un archivo externo con un identificador URI accesible. Tanto la plantilla de ARM como el archivo Bicep usan archivos de parámetros JSON.

Para más información sobre el archivo de parámetro, consulte [Creación de un archivo de parámetros de Resource Manager](parameter-files.md).

Para pasar un archivo de parámetros local, use el parámetro `TemplateParameterFile`:

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile <path-to-template-or-bicep> `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Para pasar un archivo de parámetros externo, use el parámetro `TemplateParameterUri`:

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -TemplateParameterUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.parameters.json
```

## <a name="next-steps"></a>Pasos siguientes

- Para revertir a una implementación correcta cuando se produce un error, consulte [Revertir en caso de error a una implementación correcta](rollback-on-error.md).
- Para especificar cómo controlar los recursos que existen en el grupo de recursos, pero que no están definidos en la plantilla, consulte [Modos de implementación de Azure Resource Manager](deployment-modes.md).
- Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
- Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla de Resource Manager privada con el token de SAS](secure-template-with-sas-token.md).
