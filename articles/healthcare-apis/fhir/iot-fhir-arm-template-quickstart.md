---
title: 'Inicio rápido: Implementación de Azure IoT Connector for FHIR (versión preliminar) mediante una plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a implementar Azure IoT Connector for FHIR (versión preliminar) mediante una plantilla de Resource Manager.
author: rbhaiya
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: quickstart
ms.author: rabhaiya
ms.date: 01/21/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ec802cb193d8b6f6cc8eff0dedff066e684d6596
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704910"
---
# <a name="quickstart-use-an-azure-resource-manager-arm-template-to-deploy-azure-iot-connector-for-fhir-preview"></a>Inicio rápido: Uso de una plantilla de Azure Resource Manager para implementar Azure IoT Connector for FHIR (versión preliminar)

En este inicio rápido aprenderá a usar una plantilla de Azure Resource Manager para implementar el conector de Azure IoT para Recursos Rápidos de Interoperabilidad en Salud (FHIR&#174;), una característica de Azure API for FHIR. Para implementar una instancia activa de Azure IoT Connector for FHIR, esta plantilla también implementa un servicio Azure API for FHIR y una aplicación de Azure IoT Central que exporta los datos de telemetría de un simulador de dispositivos a un conector de Azure IoT para FHIR. Puede ejecutar la plantilla de Resource Manager para implementar Azure IoT Connector for FHIR a través de Azure Portal, PowerShell o la CLI.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal tras el inicio de sesión.

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implemente en Azure una instancia de Azure IoT Connector for FHIR mediante una plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fiomt-fhir%2Fmaster%2Fdeploy%2Ftemplates%2Fmanaged%2Fazuredeploy.json)

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

La [plantilla](https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json) define los siguientes recursos de Azure:

* **Microsoft.HealthcareApis/services**
* **Microsoft.HealthcareApis/services/iomtconnectors**
* **Microsoft.IoTCentral/IoTApps**

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="portal"></a>[Portal](#tab/azure-portal)

Seleccione el siguiente vínculo para implementar Azure IoT Connector for FHIR mediante la plantilla de Resource Manager en Azure Portal:

[:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implemente en Azure una instancia del servicio Azure IoT Connector for FHIR mediante la plantilla de Resource Manager en Azure Portal.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fiomt-fhir%2Fmaster%2Fdeploy%2Ftemplates%2Fmanaged%2Fazuredeploy.json)

En la página **Implementación de Azure API for FHIR**:

1. Si lo desea, cambie el valor predeterminado de **Suscripción** a una otra suscripción.

2. En **Grupo de recursos**, seleccione **Crear nuevo** y, después, especifique un nombre válido para el nuevo grupo de recursos y seleccione **Aceptar**.

3. Si ha creado un grupo de recursos, seleccione un valor de **Región** para el grupo de recursos.

4. Escriba un nombre para la nueva instancia de Azure API for FHIR en **FHIR Service Name** (Nombre de servicio de FHIR).

5. Elija la **ubicación** de la instancia de Azure API for FHIR. La ubicación puede ser la misma región de los recursos u otra.

6. Especifique un nombre para la instancia de Azure IoT Connector for FHIR en **Iot Connector Name** (Nombre de IoT Connector).

7. Especifique un nombre para una conexión creada en Azure IoT Connector for FHIR en **Connection Name** (Nombre de la conexión). La aplicación de Azure IoT Central usa esta conexión para insertar datos de telemetría del dispositivo simulado en Azure IoT Connector for FHIR.

8. Escriba un nombre para la nueva aplicación de Azure IoT Central en **Iot Central Name** (Nombre de IoT Central). Esta aplicación usará la plantilla *Continuous patient monitoring* (Supervisión continua de pacientes) para simular un dispositivo.

9. Elija la ubicación de la aplicación de IoT Central en la lista desplegable **IoT Central Location** (Ubicación de IoT Central). 

10. Seleccione **Revisar + crear**.

11. Después de leer los términos y condiciones, seleccione **Crear**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Si desea ejecutar los scripts de PowerShell en la máquina local, primero escriba `Connect-AzAccount` para configurar las credenciales de Azure.

Si el proveedor de recursos `Microsoft.HealthcareApis` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente. Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

Si el proveedor de recursos `Microsoft.IoTCentral` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente. Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.IoTCentral
```

Use el código siguiente para implementar el servicio Azure IoT Connector for FHIR mediante la plantilla de Resource Manager.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$fhirServiceName = Read-Host -Prompt "Enter a name for the new Azure API for FHIR service"
$location = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$iotConnectorName = Read-Host -Prompt "Enter a name for the new Azure IoT Connector for FHIR"
$connectionName = Read-Host -Prompt "Enter a name for the connection with Azure IoT Connector for FHIR"
$iotCentralName = Read-Host -Prompt "Enter a name for the new Azure IoT Central Application"
$iotCentralLocation = Read-Host -Prompt "Enter a location for the new Azure IoT Central Application"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion
Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure IoT Connector for FHIR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json `
    -fhirServiceName $fhirServiceName `
    -location $location
    -iotConnectorName $iotConnectorName
    -connectionName $connectionName
    -iotCentralName $iotCentralName
    -iotCentralLocation $iotCentralLocation
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[CLI](#tab/CLI)

Si el proveedor de recursos `Microsoft.HealthcareApis` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente. Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurecli-interactive
az extension add --name healthcareapis
```

Si el proveedor de recursos `Microsoft.IoTCentral` no se ha registrado en la suscripción, puede registrarlo con el código interactivo siguiente.

```azurecli-interactive
az extension add --name azure-iot
```

Use el código siguiente para implementar el servicio Azure IoT Connector for FHIR mediante la plantilla de Resource Manager.

```azurecli-interactive
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter a name for the new Azure API for FHIR service: " fhirServiceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " location &&
read -p "Enter a name for the new Azure IoT Connector for FHIR: " iotConnectorName &&
read -p "Enter a name for the connection with Azure IoT Connector for FHIR: " connectionName &&
read -p "Enter a name for the new Azure IoT Central Application: " iotCentralName &&
read -p "Enter a location for the new Azure IoT Central Application: " iotCentralLocation &&

params='fhirServiceName='$fhirServiceName' location='$location' iotConnectorName='$iotConnectorName' connectionName='$connectionName' iotCentralName='$iotCentralName' iotCentralLocation='$iotCentralLocation &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure IoT Connector for FHIR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/microsoft/iomt-fhir/master/deploy/templates/managed/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> La implementación tarda unos minutos en completarse. Anote los nombres del servicio Azure API for FHIR, de la aplicación de Azure IoT Central y del grupo de recursos, ya que los usará para revisar los recursos implementados más adelante.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para ver información general del nuevo servicio Azure API for FHIR, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Azure API for FHIR**.

2. En la lista de FHIR, seleccione el nuevo servicio. La página **Información general** aparecerá para el nuevo servicio Azure API for FHIR.

3. Para validar que se ha aprovisionado la nueva cuenta de Azure API for FHIR, seleccione el vínculo situado junto a **Extremo de metadatos de FHIR** para capturar la instrucción de funcionalidad del servicio. El vínculo tiene el formato `https://<service-name>.azurehealthcareapis.com/metadata`. Si la cuenta se ha aprovisionado, se muestra un archivo JSON grande.

4. Para validar que la nueva instancia de Azure IoT Connector for FHIR está aprovisionada, seleccione **IoT Connector (versión preliminar)** en el menú de navegación izquierdo para abrir la página **IoT Connectors**. La página debe mostrar la instancia aprovisionada de Azure IoT Connector for FHIR con el valor *En línea* en *Estado*, el valor *1* en *Conexiones* y tanto *Asignación de dispositivos* como *Asignación de FHIR* muestran el icono *Correcto*.

5. En [Azure Portal](https://portal.azure.com), busque y seleccione **Aplicaciones de IoT Central**.

6. En la lista de Aplicaciones de IoT Central, seleccione el nuevo servicio. Aparecerá la página **Información general** de la nueva aplicación de IoT Central.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Ejecute el siguiente código interactivo para ver los detalles del servicio Azure API for FHIR. Tendrá que escribir el nombre del nuevo servicio de FHIR y el grupo de recursos.

```azurepowershell-interactive
$fhirServiceName = Read-Host -Prompt "Enter the name of your Azure API for FHIR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new FHIR service has been provisioned"

$requestUri="https://" + $fhirServiceName + ".azurehealthcareapis.com/metadata"
$metadata = Invoke-WebRequest -Uri $requestUri
$metadata.RawContent
Read-Host "Press [ENTER] to continue"
```

> [!NOTE]
> En este momento, Azure IoT Connector for FHIR no proporciona comandos de PowerShell. Para validar que Azure IoT Connector se ha aprovisionado correctamente, use el proceso de validación que se proporciona en la pestaña **Portal**.

Ejecute el siguiente código interactivo para ver los detalles de la instancia de Azure IoT Central. Tendrá que escribir el nombre de la nueva aplicación de IoT Central y del grupo de recursos.

```azurepowershell-interactive
$iotCentralName = Read-Host -Prompt "Enter the name of your Azure IoT Central application"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzIotCentralApp -ResourceGroupName $resourceGroupName -Name $iotCentralName" -Verbose
Get-AzIotCentralApp -ResourceGroupName $resourceGroupName -Name $iotCentralName
```

# <a name="cli"></a>[CLI](#tab/CLI)

Ejecute el siguiente código interactivo para ver los detalles del servicio Azure API for FHIR. Tendrá que escribir el nombre del nuevo servicio de FHIR y el grupo de recursos.

```azurecli-interactive
read -p "Enter the name of your Azure API for FHIR service: " fhirServiceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az healthcareapis service show --resource-group $resourceGroupName --resource-name $fhirServiceName" &&
az healthcareapis service show --resource-group $resourceGroupName --resource-name $fhirServiceName &&
read -p "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned: " &&
requestUrl='https://'$fhirServiceName'.azurehealthcareapis.com/metadata' &&
curl --url $requestUrl &&
read -p "Press [ENTER] to continue: "
```

> [!NOTE]
> En este momento, Azure IoT Connector for FHIR no proporciona comandos de la CLI. Para validar que Azure IoT Connector se ha aprovisionado correctamente, use el proceso de validación que se proporciona en la pestaña **Portal**.

Ejecute el siguiente código interactivo para ver los detalles de la instancia de Azure IoT Central. Tendrá que escribir el nombre de la nueva aplicación de IoT Central y del grupo de recursos.

```azurecli-interactive
read -p "Enter the name of your IoT Central application: " iotCentralName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az iot central app show -g $resourceGroupName -n $iotCentralName" &&
az iot central app show -g $resourceGroupName -n $iotCentralName &&
```

---

## <a name="connect-your-iot-data-with-the-azure-iot-connector-for-fhir-preview"></a>Conexión de los datos de IoT con el conector de Azure IoT para FHIR (versión preliminar)
> [!IMPORTANT]
> La plantilla de asignación de dispositivos que se proporciona en esta guía está diseñada para funcionar con la exportación de datos (heredada) dentro de IoT Central.

Actualmente, la aplicación de IoT Central no proporciona una plantilla de Resource Manager o comandos de PowerShell y de la CLI para establecer la exportación de datos. Por tanto, siga las instrucciones que se describen a continuación mediante Azure Portal.  

Una vez que haya implementado la aplicación de IoT Central, los dos dispositivos simulados listos para usar comenzarán a generar telemetría. En este tutorial, se ingerirá la telemetría del simulador *Smart Vitals Patch* en FHIR a través del conector de Azure IoT para FHIR. Para exportar los datos de IoT a Azure IoT Connector for FHIR, es preciso [configurar una exportación de datos (heredada) en IoT Central](../../iot-central/core/howto-export-data-legacy.md). En la página Exportación de datos (heredado):
- Elija *Azure Event Hubs* como destino de exportación.
- Seleccione el valor *Usar una cadena de conexión* para el campo **Espacio de nombres de Event Hubs**.
- Proporcione la cadena de conexión del conector de Azure IoT para FHIR que obtuvo en un paso anterior en el campo **Cadena de conexión**.
- Mantenga la opción **Telemetría** *activada* para el campo **Datos para exportar**.

---

## <a name="view-device-data-in-azure-api-for-fhir"></a>Visualización de los datos del dispositivo en Azure API for FHIR

Para ver los recursos de tipo Observation basados en FHIR que creó Azure IoT Connector for FHIR en su servidor de FHIR, utilice Postman. Configure [Postman para acceder a Azure API for FHIR](access-fhir-postman-tutorial.md) y realice una solicitud `GET` a `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4` para ver los recursos de tipo Observation de FHIR con el valor de la frecuencia cardíaca.

> [!TIP]
> Asegúrese de que el usuario tiene acceso adecuado al plano de datos de Azure API for FHIR. Use el [control de acceso basado en roles de Azure (Azure RBAC)](configure-azure-rbac.md) para asignar los bloques del plano de datos requeridos.

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

En esta guía de inicio rápido, ha implementado la característica Conector de Azure IoT para FHIR en el recurso de Azure API for FHIR. Seleccione en los pasos siguientes para más información sobre el conector de Azure IoT para FHIR:

Conozca las distintas fases del flujo de datos en el conector de Azure IoT para FHIR.

>[!div class="nextstepaction"]
>[Flujo de datos del conector de Azure IoT para FHIR](iot-data-flow.md)

Aprenda a configurar el conector de IoT mediante plantillas de asignación de dispositivos y FHIR.

>[!div class="nextstepaction"]
>[Plantillas de asignación del conector de Azure IoT para FHIR (versión preliminar)](iot-mapping-templates.md)

*En Azure Portal, el conector de Azure IoT para FHIR se conoce como conector de IoT (versión preliminar). FHIR es una marca registrada de HL7 y se usa con el permiso de HL7.
