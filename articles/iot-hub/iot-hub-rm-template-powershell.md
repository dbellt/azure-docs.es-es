---
title: Creación de un centro de IoT de Azure mediante una plantilla (PowerShell) | Microsoft Docs
description: Cómo usar una plantilla de Azure Resource Manager para crear un IoT Hub con Azure PowerShell.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/02/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ac3a75b2e00e0e769f758cf82eec7cfc6e5ec4c2
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031482"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-powershell"></a>Creación de un centro de IoT con una plantilla de Azure Resource Manager (PowerShell)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Obtenga información sobre cómo usar una plantilla de Azure Resource Manager para crear un IoT Hub y un grupo de consumidores. Las plantillas de Resource Manager son archivos JSON que definen los recursos que necesita para implementar la solución. Para obtener más información sobre el desarrollo de plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../azure-resource-manager/index.yml).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

La plantilla de Resource Manager usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/iothub-with-consumergroup-create/). Esta es una copia de la plantilla:

[!code-json[iothub-creation](~/quickstart-templates/quickstarts/microsoft.devices/iothub-with-consumergroup-create/azuredeploy.json)]

La plantilla crea un centro de IoT de Azure con tres puntos de conexión (centro de eventos, de la nube a dispositivo y mensajería) y un grupo de consumidores. Para ver más ejemplos de plantillas, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Devices&pageNumber=1&sort=Popular). El esquema de la plantilla de IoT Hub puede encontrarse [aquí](/azure/templates/microsoft.devices/iothub-allversions).

Existen varios métodos para la implementación de una plantilla.  En este tutorial se va a usar Azure PowerShell.

Para ejecutar el script de PowerShell, seleccione **Pruébelo** para abrir el shell de Azure Cloud. Para pegar el script, haga clic con el botón derecho en el shell y luego seleccione Pegar:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$iotHubName = Read-Host -Prompt "Enter the IoT Hub name"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.devices/iothub-with-consumergroup-create/azuredeploy.json" `
    -iotHubName $iotHubName
```

Como puede ver en el script de PowerShell, la plantilla utilizada es de plantillas de inicio rápido de Azure. Para usar una propia, primero debe cargar el archivo de plantilla en el shell de Cloud y luego usar el modificador `-TemplateFile` para especificar el nombre de archivo.  Para ver un ejemplo, consulte [Implementar la plantilla](../azure-resource-manager/templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell#deploy-the-template).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado un centro de IoT mediante una plantilla de Azure Resource Manager, quizá quiera seguir explorando:

* Consulte las funcionalidades de la [API de REST del proveedor de recursos de IoT Hub][lnk-rest-api].
* Para más información sobre las funcionalidades de Azure Resource Manager, consulte [Información general de Azure Resource Manager][lnk-azure-rm-overview].
* Para conocer la sintaxis y las propiedades JSON que se usan en las plantillas, consulte [Microsoft.Devices resource types](/azure/templates/microsoft.devices/iothub-allversions) (Tipos de recursos de Microsoft.Devices).

Para obtener más información sobre cómo desarrollar para IoT Hub, consulte los siguientes artículos:

* [Introducción a SDK para C][lnk-c-sdk]
* [SDK de IoT de Azure][lnk-sdks]

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azure/install-Az-ps
[lnk-rest-api]: /rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/management/overview.md
[lnk-powershell-arm]: ../azure-resource-manager/management/manage-resources-powershell.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/quickstart-linux.md
