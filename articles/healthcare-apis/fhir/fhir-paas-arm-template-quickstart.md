---
title: 'Inicio rápido: Implementación de Azure API for FHIR mediante una plantilla de ARM'
description: En este inicio rápido aprenderá a implementar Azure API for Fast Healthcare Interoperability Resources (FHIR®) mediante una plantilla de Azure Resource Manager.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.author: cavoeg
ms.date: 05/10/2021
ms.openlocfilehash: 97f57484c47bfc58f3a4cd30ed979ffb5b5d1c27
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704930"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-api-for-fhir"></a>Inicio rápido: Uso de una plantilla de Resource Manager para implementar Azure API for FHIR

En este inicio rápido aprenderá a usar una plantilla de Azure Resource Manager (plantilla de ARM) para implementar Azure API for Fast Healthcare Interoperability Resources (FHIR®). Puede implementar Azure API for FHIR mediante Azure Portal, PowerShell o la CLI.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal tras el inicio de sesión.

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implemente una instancia del servicio Azure API for FHIR mediante una plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.healthcareapis%2fazure-api-for-fhir%2fazuredeploy.json)

## <a name="prerequisites"></a>Prerrequisitos

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

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azure-api-for-fhir/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.healthcareapis/azure-api-for-fhir/azuredeploy.json":::

La plantilla define un recurso de Azure:

* **Microsoft.HealthcareApis/services**

<!--

Replace the line above with the following line once https://docs.microsoft.com/azure/templates/microsoft.healthcareapis/services goes live:

* [**Microsoft.HealthcareApis/services**](/azure/templates/microsoft.healthcareapis/services)

-->

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="portal"></a>[Portal](#tab/azure-portal)

Seleccione el siguiente vínculo para implementar Azure API for FHIR con la plantilla de Resource Manager en Azure Portal:

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implemente una instancia del servicio Azure API for FHIR con la plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2fquickstarts%2fmicrosoft.healthcareapis%2fazure-api-for-fhir%2fazuredeploy.json)

En la página **Implementación de Azure API for FHIR**:

1. Si lo desea, cambie el valor predeterminado de **Suscripción** a una otra suscripción.

2. En **Grupo de recursos**, seleccione **Crear nuevo** y, después, especifique un nombre válido para el nuevo grupo de recursos y seleccione **Aceptar**.

3. Si ha creado un grupo de recursos, seleccione un valor de **Región** para el grupo de recursos.

4. Escriba un nuevo valor en **Nombre de servicio** y elija el valor de **Ubicación** para Azure API for FHIR. La ubicación puede ser la misma región de los recursos u otra.

    :::image type="content" source="./media/fhir-paas-arm-template-quickstart/deploy-azure-api-fhir.png" alt-text="Implemente Azure API for FHIR con la plantilla de Resource Manager en Azure Portal.":::

5. Seleccione **Revisar + crear**.

6. Después de leer los términos y condiciones, seleccione **Crear**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Si desea ejecutar los scripts de PowerShell en la máquina local, primero escriba `Connect-AzAccount` para configurar las credenciales de Azure.

Si el proveedor de recursos `Microsoft.HealthcareApis` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente. Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

Use el código siguiente para implementar el servicio Azure API for FHIR mediante la plantilla de Resource Manager. El código le pedirá el nuevo nombre del servicio FHIR, el nombre de un nuevo grupo de recursos y sus ubicaciones.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure API for FHIR service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion
Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure API for FHIR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.healthcareapis/azure-api-for-fhir/azuredeploy.json `
    -serviceName $serviceName `
    -location $serviceLocation
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Si el proveedor de recursos `Microsoft.HealthcareApis` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente. Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurecli-interactive
az extension add --name healthcareapis
```

Use el código siguiente para implementar el servicio Azure API for FHIR mediante la plantilla de Resource Manager. El código le pedirá el nuevo nombre del servicio FHIR, el nombre de un nuevo grupo de recursos y sus ubicaciones.

```azurecli-interactive
read -p "Enter a name for the new Azure API for FHIR service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
params='serviceName='$serviceName' location='$serviceLocation &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure API for FHIR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.healthcareapis/azure-api-for-fhir/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> La implementación tarda unos minutos en completarse. Anote los nombres del servicio Azure API for FHIR y del grupo de recursos, los usará para revisar los recursos implementados más adelante.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para ver información general del nuevo servicio Azure API for FHIR, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Azure API for FHIR**.

2. En la lista de FHIR, seleccione el nuevo servicio. La página **Información general** aparecerá para el nuevo servicio Azure API for FHIR.

3. Para validar que se ha aprovisionado la nueva cuenta de Azure API for FHIR, seleccione el vínculo situado junto a **Extremo de metadatos de FHIR** para capturar la instrucción de funcionalidad del servicio. El vínculo tiene el formato `https://<service-name>.azurehealthcareapis.com/metadata`. Si la cuenta se ha aprovisionado, se muestra un archivo JSON grande.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Ejecute el siguiente código interactivo para ver los detalles del servicio Azure API for FHIR. Tendrá que escribir el nombre y el grupo de recursos del nuevo servicio.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure API for FHIR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned"

$requestUri="https://" + $serviceName + ".azurehealthcareapis.com/metadata"
$metadata = Invoke-WebRequest -Uri $requestUri
$metadata.RawContent
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Ejecute el siguiente código interactivo para ver los detalles del servicio Azure API for FHIR. Tendrá que escribir el nombre y el grupo de recursos del nuevo servicio.

```azurecli-interactive
read -p "Enter the name of your Azure API for FHIR service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName" &&
az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName &&
read -p "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned: " &&
requestUrl='https://'$serviceName'.azurehealthcareapis.com/metadata' &&
curl --url $requestUrl &&
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

Para ver un tutorial paso a paso que le guíe en el proceso de creación de una plantilla de Resource Manager, consulte [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../../azure-resource-manager/templates/template-tutorial-create-first-template.md)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado Azure API for FHIR en su suscripción. Para establecer opciones de configuración adicionales en Azure API for FHIR, continúe con la guía paso a paso de configuración adicional. Si está listo para empezar a usar Azure API for FHIR, lea más información sobre cómo registrar aplicaciones.

>[!div class="nextstepaction"]
>[Configuración adicional en Azure API for FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Introducción al registro de aplicaciones](fhir-app-registration.md)
