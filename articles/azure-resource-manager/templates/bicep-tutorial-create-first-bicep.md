---
title: 'Tutorial: Creación e implementación de archivos Bicep de Azure Resource Manager'
description: Cree su primer archivo Bicep para implementar recursos de Azure. En este tutorial, conocerá la sintaxis de los archivos Bicep y aprenderá a implementar una cuenta de almacenamiento.
author: mumian
ms.date: 04/12/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 953ca06c3f59f1b39b7b5a5d99ef38814b97c565
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319330"
---
# <a name="tutorial-create-and-deploy-first-azure-resource-manager-bicep-file"></a>Tutorial: Creación e implementación del primer archivo Bicep de Azure Resource Manager

Este tutorial es una introducción a [Bicep](./bicep-overview.md). En él, se explica cómo puede crear un archivo Bicep inicial e implementarlo en Azure. También conocerá la estructura del archivo Bicep y sabrá qué herramientas va a necesitar para trabajar con este tipo de archivos. Se tarda unos **12 minutos** en realizar este tutorial, pero el tiempo real variará en función del número de herramientas que necesite instalar.

Este tutorial es el primero de una serie. A medida que avance por la serie, irá modificando paso a paso el archivo Bicep inicial hasta que haya explorado todos los componentes esenciales de un archivo de este tipo. Estos componentes son la base sobre la que podrá crear archivos Bicep más complejos. Esperamos que, al finalizar esta serie, haya conseguido la confianza suficiente para crear sus propios archivos Bicep y pueda automatizar las implementaciones con estos archivos.

Si desea más detalles sobre las ventajas de usar Bicep y por qué debería automatizar la implementación con estos archivos, consulte este artículo sobre [Bicep](./bicep-overview.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="get-tools"></a>Obtención de las herramientas

Para empezar, vamos a asegurarnos de que tiene las herramientas necesarias para crear e implementar archivos Bicep. Instale estas herramientas en el equipo local.

### <a name="editor"></a>Editor

Para poder crear archivos Bicep, necesita un buen editor. Le recomendamos Visual Studio Code con la extensión Bicep. Si necesita instalar estas herramientas, consulte [Configuración del entorno de desarrollo de Bicep](./bicep-install.md#development-environment).

### <a name="command-line-deployment"></a>Implementación desde la línea de comandos

Puede usar la CLI de Azure o Azure PowerShell para implementar archivos de Bicep. Para la CLI de Azure, necesita la versión 2.20.0 o posterior. En el caso de Azure PowerShell, necesita la versión 5.6.0 o posterior. Para obtener instrucciones de instalación, consulte:

- [Azure PowerShell](/powershell/azure/install-az-ps)
- [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows)
- [Instalación de la CLI de Azure en Linux](/cli/azure/install-azure-cli-linux)
- [Instalación de la CLI de Azure en macOS](/cli/azure/install-azure-cli-macos)

Después de instalar Azure PowerShell o la CLI de Azure, asegúrese de iniciar sesión por primera vez. Para obtener ayuda, consulte [Inicio de sesión: PowerShell](/powershell/azure/install-az-ps#sign-in) o [ CLI de Azure](/cli/azure/get-started-with-azure-cli#sign-in).

Perfecto. Ya tenemos todo para empezar a conocer Bicep.

## <a name="create-your-first-bicep-file"></a>Creación del primer archivo Bicep

1. Abra Visual Studio Code con la extensión Bicep instalada.
1. En el menú **Archivo**, seleccione **Nuevo archivo** para crear un archivo nuevo.
1. En el menú **Archivo**, seleccione **Guardar como**.
1. Llame _azuredeploy_ al archivo y seleccione la extensión _bicep_. El nombre completo del archivo será _azuredeploy.json_.
1. Guarde el archivo en la estación de trabajo. Seleccione una ruta de acceso que sea fácil de recordar, ya que tendrá que utilizarla más adelante para implementar el archivo Bicep.
1. Copie el siguiente contenido y péguelo en el archivo:

    ```bicep
    resource stg 'Microsoft.Storage/storageAccounts@2019-06-01' = {
      name: '{provide-unique-name}'  // must be globally unique
      location: 'eastus'
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
      properties: {
        supportsHttpsTrafficOnly: true
      }
    }
    ```

    Este es el aspecto que presenta el entorno de Visual Studio Code:

    ![Primer archivo Bicep de Visual Studio Code](./media/Bicep-tutorial-create-first-bicep/resource-manager-visual-studio-code-first-bicep.png)

    La declaración del recurso tiene cuatro componentes:

    - **resource** (recurso): palabra clave.
    - **symbolic name** (stg) [nombre simbólico]: el nombre simbólico es un identificador que hace referencia al recurso en el archivo Bicep. No es cuál será el nombre del recurso cuando se implemente. El nombre del recurso se define mediante la propiedad **name**.  Vea el cuarto componente de esta lista. Para facilitar el seguimiento de los tutoriales, el nombre simbólico que se utiliza en esta serie para el recurso de la cuenta de almacenamiento es **stg**. Para ver cómo usar el nombre simbólico para obtener una lista completa de las propiedades del objeto, consulte [Agregar salidas](./bicep-tutorial-add-outputs.md).
    - **tipo de recurso** (Microsoft.Storage/storageAccounts@2019-06-01): se compone del proveedor de recursos (Microsoft.Storage), el tipo de recurso (storageAccounts) y apiVersion (2019-06-01). Cada proveedor de recursos publica sus propias versiones de API, por lo que este valor es específico del tipo. Puede encontrar más tipos y valores apiVersion para varios recursos de Azure en la [referencia de plantillas de ARM](/azure/templates/).
    - **properties** [propiedades] (todo lo que está dentro = {...}): son las propiedades específicas que le gustaría especificar para este tipo de recurso. Son exactamente las mismas propiedades que están disponibles en una plantilla de ARM. Cada recurso tiene una propiedad `name`. La mayoría de los recursos también tienen una propiedad `location`, que establece la región donde se implementa el recurso. Las demás propiedades varían según el tipo de recurso y la versión de API. Es importante comprender la relación entre la versión de API y las propiedades disponibles, así que lo analizaremos con más detalle.

        En esta cuenta de almacenamiento, puede ver la versión de la API: [storageAccounts 2019-06-01](/azure/templates/microsoft.storage/2019-06-01/storageaccounts). Tenga en cuenta que no ha agregado todas las propiedades al archivo Bicep. Muchas de las propiedades son opcionales. El proveedor de recursos `Microsoft.Storage` podría publicar una nueva versión de API, pero la versión que está implementando no tiene que cambiar. Puede seguir usando esa versión con la seguridad de que los resultados de la implementación serán coherentes.

        Si la versión de la API es anterior (por ejemplo, [storageAccounts 2016-05-01](/azure/templates/microsoft.storage/2016-05-01/storageaccounts)), el conjunto de propiedades disponible será menor.

        Si decide cambiar la versión de la API de un recurso, no olvide comprobar las propiedades de esa versión y ajustar el archivo Bicep como corresponda.

    Para más información, consulte [Nociones sobre la estructura y la sintaxis de los archivos Bicep](./bicep-file.md).

    Hay un comentario para la propiedad name.  Use `//` para los comentarios de una sola línea o `/* ... */` para los comentarios de varias líneas.

1. Reemplace `{provide-unique-name}`, incluidas las llaves `{}`, por un nombre de cuenta de almacenamiento que sea único.

    > [!IMPORTANT]
    > El nombre de la cuenta de almacenamiento debe ser único en Azure. El nombre debe tener solo letras minúsculas o números. No debe superar los 24 caracteres. Para el nombre, puede probar a utilizar **store1** como prefijo y, a continuación, agregar sus iniciales y la fecha del día. Por ejemplo, podría usar el nombre **store1abc09092019**.

    No es fácil adivinar un nombre único para una cuenta de almacenamiento y no resulta eficaz para automatizar implementaciones de gran tamaño. Más adelante en esta serie de tutoriales, utilizaremos características de Bicep con las que le resultará más fácil crear un nombre único.

1. Guarde el archivo.

Enhorabuena, ha creado su primer archivo Bicep.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para empezar a trabajar con Azure PowerShell o la CLI de Azure, inicie sesión con sus credenciales de Azure.

Seleccione las pestañas en las siguientes secciones de código para elegir entre Azure PowerShell y la CLI de Azure. Los ejemplos de la CLI de este artículo están escritos para el shell de Bash.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az login
```

---

Si tiene varias suscripciones de Azure, seleccione la suscripción que desee usar. Reemplace `[SubscriptionID/SubscriptionName]` y los corchetes `[]` por la información de la suscripción:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cuando implementa un archivo Bicep, debe especificar un grupo que contenga los recursos. Antes de ejecutar el comando de implementación, cree el grupo de recursos con la CLI de Azure o Azure PowerShell.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name myResourceGroup `
  -Location "Central US"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group create \
  --name myResourceGroup \
  --location "Central US"
```

---

## <a name="deploy-bicep-file"></a>Implementación del archivo Bicep

Bicep es una abstracción transparente en plantillas de Azure Resource Manager (plantillas de ARM). Cada archivo Bicep se compila en una plantilla de ARM estándar antes de su implementación. Puede compilar el archivo Bicep en una plantilla de ARM antes de implementarlo o puede implementarlo directamente. Para implementar el archivo Bicep, utilice la CLI de Azure o Azure PowerShell. Use el grupo de recursos que creó. Asigne un nombre a la implementación para que pueda identificarla fácilmente en el historial de implementaciones. Para facilitar el proceso, cree también una variable que almacene la ruta de acceso del archivo Bicep. Esta variable facilita la ejecución de los comandos de implementación, ya que no es necesario volver a escribir la ruta de acceso cada vez que se implementa. Reemplace `{provide-the-path-to-the-bicep-file}`, incluidas las llaves `{}`, por la ruta del archivo Bicep y el nombre de extensión del archivo, _.bicep_.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
$bicepFile = "{provide-the-path-to-the-bicep-file}"
New-AzResourceGroupDeployment `
  -Name firstbicep `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
bicepFile="{provide-the-path-to-the-bicep-file}"
az deployment group create \
  --name firstbicep \
  --resource-group myResourceGroup \
  --template-file $bicepFile
```

---

El comando de implementación devuelve los resultados. Busque `ProvisioningState` para ver si la implementación se realizó correctamente.

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú izquierdo, seleccione **Grupos de recursos**.

1. Seleccione el grupo de recursos que implementó en el último procedimiento. El nombre predeterminado es **myResourceGroup**. No debería ver ningún recurso implementado en el grupo de recursos.

1. Observe que en la esquina superior derecha de la información general se muestra el estado de la implementación. Seleccione **1 Correcta**.

   ![Ver el estado de implementación](./media/bicep-tutorial-create-first-bicep/deployment-status.png)

1. Verá un historial de implementación del grupo de recursos. Seleccione **firstbicep**.

   ![Selección de la implementación](./media/bicep-tutorial-create-first-bicep/select-from-deployment-history.png)

1. Verá un resumen de la implementación. Hay una cuenta de almacenamiento implementada. Observe que a la izquierda puede ver las entradas, las salidas y la plantilla que se usó durante la implementación.

   ![Ver el resumen de la implementación](./media/bicep-tutorial-create-first-bicep/view-deployment-summary.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha creado un archivo Bicep sencillo para implementarlo en Azure.  En tutoriales posteriores, aprenderá a agregar parámetros, variables, salidas y módulos a este tipo de archivos. Estas características sientan las bases para crear archivos Bicep mucho más complejos.

> [!div class="nextstepaction"]
> [Adición de parámetros](bicep-tutorial-add-parameters.md)
