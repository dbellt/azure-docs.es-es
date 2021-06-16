---
title: 'Inicio rápido: Creación de una instancia de Azure SignalR Service: plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a crear una instancia de Azure SignalR Service mediante una plantilla de Azure Resource Manager.
author: sffamily
ms.author: zhshang
ms.date: 10/02/2020
ms.topic: quickstart
ms.service: signalr
ms.custom: devx-track-azurepowershell - subject-armqs - mode-arm
ms.openlocfilehash: 624240a6bb1846606fbdbf0b3a635935df9bdd5b
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891831"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-signalr-service"></a>Inicio rápido: Uso de una plantilla de Resource Manager para implementar Azure SignalR Service

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear una instancia de Azure SignalR Service. Azure SignalR Service se puede implementar mediante Azure Portal, PowerShell o la CLI.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal tras el inicio de sesión.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Botón para implementar Azure SignalR Service en Azure mediante una plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.signalrservice%2fsignalr%2fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

# <a name="portal"></a>[Portal](#tab/azure-portal)

Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).
* Si desea ejecutar el código localmente, [Azure PowerShell](/powershell/azure/install-az-ps).

# <a name="cli"></a>[CLI](#tab/CLI)

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).
* Si desea ejecutar el código en la máquina local:
    * Un shell de Bash (como Git Bash, que se incluye en [Git para Windows](https://gitforwindows.org)).
    * [Azure CLI](/cli/azure/install-azure-cli).

---

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/signalr/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.signalrservice/signalr/azuredeploy.json":::

La plantilla define un recurso de Azure:

* [**Microsoft.SignalRService/SignalR**](/azure/templates/microsoft.signalrservice/signalr)

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="portal"></a>[Portal](#tab/azure-portal)

Seleccione el siguiente vínculo para implementar Azure SignalR Service mediante la plantilla de Resource Manager en Azure Portal:

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Botón para implementar Azure SignalR Service en Azure mediante la plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.signalrservice%2fsignalr%2fazuredeploy.json)

En la página **Implementación de Azure SignalR Service**:

1. Si lo desea, cambie el valor predeterminado de **Suscripción**.

2. En **Grupo de recursos**, seleccione **Crear nuevo** y, después, especifique un nombre válido para el nuevo grupo de recursos y seleccione **Aceptar**.

3. Si ha creado un grupo de recursos, seleccione un valor de **Región** para el grupo de recursos.

4. Si lo desea, escriba un nuevo **nombre** y la **ubicación** (por ejemplo, **eastus2**) de Azure SignalR Service. Si no especifica un nombre, se genera automáticamente. La ubicación de Azure SignalR Service puede ser la misma que la de la región del grupo de recursos, o bien otra diferente. Si no especifica una ubicación, se establece en la misma región que la del grupo de recursos.

5. En **Pricing Tier** (Plan de tarifa), elija (**Free_F1** o **Standard_S1**), escriba el valor de **Capacity** (Capacidad) (número de unidades de SignalR) y, en **Service Mode** (Modo de servicio), elija **Default** (Predeterminado) (se requiere un servidor concentrador), **Serverless** (Sin servidor) (no permite ninguna conexión de servidor) o **Classic** (Clásico) (se enruta al servidor concentrador solo si el concentrador tiene conexión de servidor). Luego, elija entre **Enable Connectivity Logs** (Habilitar registros de conectividad) o **Enable Messaging Logs** (Habilitar registros de mensajería).

    > [!NOTE]
    > En el plan de tarifa **Free_F1**, la capacidad está limitada a una unidad.

    :::image type="content" source="./media/signalr-quickstart-azure-signalr-service-arm-template/deploy-azure-signalr-service-arm-template-portal.png" alt-text="Captura de pantalla de la plantilla de Resource Manager para crear una instancia de Azure SignalR Service en Azure Portal.":::

6. Seleccione **Revisar + crear**.

7. Después de leer los términos y condiciones, seleccione **Crear**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Si desea ejecutar los scripts de PowerShell en la máquina local, primero escriba `Connect-AzAccount` para configurar las credenciales de Azure.

Use el siguiente código para implementar Azure SignalR Service mediante la plantilla de Resource Manager. El código le pedirá los siguientes elementos:

* El nombre y la región de la nueva instancia de Azure SignalR Service
* El nombre y la región de un nuevo grupo de recursos
* El plan de tarifa de Azure (**Free_F1** o **Standard_S1**)
* La capacidad de la unidad de SignalR (1, 2, 5, 10, 20, 50 o 100)
  > [!NOTE]
  > En el plan de tarifa **Free_F1**, la capacidad está limitada a una unidad.
* El modo de servicio: **Default** (Predeterminado) para exigir un servidor concentrador, **Serverless** (Sin servidor) para no permitir ninguna conexión de servidor o **Classic** (Clásico) para el enrutamiento a un servidor concentrador solo si el concentrador tiene una conexión de servidor.
* Si se habilitan los registros de conectividad o de mensajería (**true** o **false**).

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure SignalR Service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

$priceTier = Read-Host -Prompt "Enter the pricing tier (Free_F1 or Standard_S1)"
$unitCapacity = Read-Host -Prompt "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100)"
$servicingMode = Read-Host -Prompt "Enter the service mode (Default, Serverless, or Classic)"
$enableConnectionLogs = Read-Host -Prompt "Specify whether to enable connectivity logs (true or false)"
$enableMessageLogs = Read-Host -Prompt "Specify whether to enable messaging logs (true or false)"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion

$paramObjHashTable = @{
    name = $serviceName
    location = $serviceLocation
    pricingTier = $priceTier
    capacity = [int]$unitCapacity
    serviceMode = $servicingMode
    enableConnectivityLogs = $enableConnectionLogs
    enableMessagingLogs = $enableMessageLogs
}

Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure SignalR Service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateParameterObject $paramObjHashTable `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.signalrservice/signalr/azuredeploy.json
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Use el siguiente código para implementar Azure SignalR Service mediante la plantilla de Resource Manager. El código le pedirá los siguientes elementos:

* El nombre y la región de la nueva instancia de Azure SignalR Service
* El nombre y la región de un nuevo grupo de recursos
* El plan de tarifa de Azure (**Free_F1** o **Standard_S1**)
* La capacidad de la unidad de SignalR (1, 2, 5, 10, 20, 50 o 100)
    > [!NOTE]
    > En el plan de tarifa **Free_F1**, la capacidad está limitada a una unidad.
* El modo de servicio: **Default** (Predeterminado) para exigir un servidor concentrador, **Serverless** (Sin servidor) para no permitir ninguna conexión de servidor o **Classic** (Clásico) para el enrutamiento a un servidor concentrador solo si el concentrador tiene una conexión de servidor.
* Si se habilitan los registros de conectividad o de mensajería (**true** o **false**).

```azurecli-interactive
read -p "Enter a name for the new Azure SignalR Service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter the pricing tier (Free_F1 or Standard_S1): " priceTier &&
read -p "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100): " unitCapacity &&
read -p "Enter the service mode (Default, Serverless, or Classic): " servicingMode &&
read -p "Specify whether to enable connectivity logs (true or false): " enableConnectionLogs &&
read -p "Specify whether to enable messaging logs (true or false): " enableMessageLogs &&
params='name='$serviceName' location='$serviceLocation' pricingTier='$priceTier' capacity='$unitCapacity' serviceMode='$servicingMode' enableConnectivityLogs='$enableConnectionLogs' enableMessagingLogs='$enableMessageLogs &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure SignalR Service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.signalrservice/signalr/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> La implementación puede tardar unos minutos en completarse. Anote los nombres de la instancia de Azure SignalR Service y del grupo de recursos, que se usa para revisar más adelante los recursos implementados.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para ver información general de la nueva instancia de Azure SignalR Service, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **SignalR**.

2. En la lista de SignalR, seleccione el nuevo servicio. Aparece la página **Información general** de la nueva instancia de Azure SignalR Service.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Ejecute el siguiente código interactivo para ver los detalles de la instancia de Azure SignalR Service. Tendrá que escribir el nombre y el grupo de recursos del nuevo servicio.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure SignalR Service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Ejecute el siguiente código interactivo para ver los detalles de la instancia de Azure SignalR Service. Tendrá que escribir el nombre y el grupo de recursos del nuevo servicio.

```azurecli-interactive
read -p "Enter the name of your Azure SignalR Service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az signalr show --resource-group $resourceGroupName --name $serviceName" &&
az signalr show --resource-group $resourceGroupName --name $serviceName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, con lo que se eliminan los recursos que contiene.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), busque la opción **Grupos de recursos** y selecciónela.

2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.

3. En la página de **información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.

4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to delete"
Write-Verbose "Remove-AzResourceGroup -Name $resourceGroupName" -Verbose
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
read -p "Enter the name of the resource group to delete: " resourceGroupName &&
echo "DELETE A RESOURCE GROUP (AND ITS RESOURCES):  az group delete --name $resourceGroupName" &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="next-steps"></a>Pasos siguientes

Para ver un tutorial paso a paso que le guíe en el proceso de creación de una plantilla de Resource Manager, consulte:

> [!div class="nextstepaction"]
> [ Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
