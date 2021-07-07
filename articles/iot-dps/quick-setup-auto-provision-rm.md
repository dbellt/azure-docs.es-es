---
title: 'Inicio rápido: Creación de una instancia de Azure IoT Hub Device Provisioning Service (DPS) mediante una plantilla de Azure Resource Manager'
description: En este inicio rápido de Azure, aprenderá a crear una instancia de Azure IoT Hub Device Provisioning Service (DPS) mediante una plantilla de Azure Resource Manager.
author: wesmc7777
ms.author: wesmc
ms.date: 01/27/2021
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, subject-armqs, devx-track-azurecli
ms.openlocfilehash: 8ec795e7ad54d2101573d857fa52b5aaad0ed71d
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112032400"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-dps-with-an-arm-template"></a>Inicio rápido: Configuración de IoT Hub Device Provisioning Service (DPS) con una plantilla de Resource Manager

Puede usar una plantilla de [Azure Resource Manager](../azure-resource-manager/management/overview.md) para configurar mediante programación los recursos de nube de Azure necesarios para aprovisionar los dispositivos. En estos pasos se muestra cómo crear un centro de IoT y una nueva instancia de IoT Hub Device Provisioning Service con una plantilla de Resource Manager. El centro de IoT también está vinculado al recurso de DPS mediante la plantilla, lo que le permite asignar dispositivos al centro en función de las directivas de asignación configuradas.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

En este inicio rápido se usa [Azure Portal](../azure-resource-manager/templates/deploy-portal.md) y la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md) para realizar los pasos de programación necesarios para crear un grupo de recursos e implementar la plantilla, pero se puede usar también [PowerShell](../azure-resource-manager/templates/deploy-powershell.md), .NET, Ruby u otros lenguajes de programación. 

Si su entorno cumple los requisitos previos y ya está familiarizado con el uso de plantillas de Resource Manager, al seleccionar el botón **Implementar en Azure** que aparece a continuación se abrirá la plantilla para la implementación en Azure Portal.

[![Información general de implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2Fquickstarts%2Fmicrosoft.devices%2Fiothub-device-provisioning%2fazuredeploy.json)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]


## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/iothub-device-provisioning/).

> [!NOTE]
> Actualmente no se pueden crear inscripciones con nuevos recursos de DPS mediante plantillas de ARM. Se trata de una petición común y comprensible que se está considerando implementar.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.devices/iothub-device-provisioning/azuredeploy.json":::

En la plantilla anterior se definen dos recursos de Azure:

* [**Microsoft.Devices/iothubs**](/azure/templates/microsoft.devices/iothubs): crea un centro de IoT de Azure.
* [**Microsoft.Devices/provisioningservices**](/azure/templates/microsoft.devices/provisioningservices): crea una instancia de Azure IoT Hub Device Provisioning Service con el nuevo centro de IoT ya vinculado.


## <a name="deploy-the-template"></a>Implementación de la plantilla

#### <a name="deploy-with-the-portal"></a>Implementación con el portal

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla para la implementación. La plantilla crea un centro de IoT y un recurso de DPS. El centro se vinculará en el recurso de DPS.

    [![Pasos de implementación en Azure en el portal](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2Fquickstarts%2Fmicrosoft.devices%2Fiothub-device-provisioning%2fazuredeploy.json)

2. Seleccione o escriba los siguientes valores y, a continuación, elija **Revisar y crear**.

    ![Parámetros de implementación de la plantilla de Resource Manager en el portal](./media/quick-setup-auto-provision-rm/arm-template-deployment-parameters-portal.png)    

    A menos que se especifique, use el valor predeterminado para crear el centro de IoT y el recurso de DPS.

    | Campo | Descripción |
    | :---- | :---------- |
    | **Suscripción** | Seleccione la suscripción a Azure. |
    | **Grupo de recursos** | Seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y haga clic en **Aceptar**. |
    | **Región** | Seleccione una región para los recursos. Por ejemplo **Este de EE. UU.** |
    | **Nombre de IoT Hub** | Escriba un nombre para el centro de IoT que sea globalmente único en el espacio de nombres *azure-devices.net*. Lo necesitará en la sección siguiente cuando valide la implementación. |
    | **Nombre del servicio Device Provisioning** | Escriba un nombre para el nuevo recurso de Device Provisioning Service (DPS). El nombre debe ser globalmente único dentro del espacio de nombres *.azure-devices-provisioning.net*. Este nombre lo necesitará en la sección siguiente cuando valide la implementación. |
    
3. En la siguiente pantalla, lea los términos. Si está de acuerdo con todos los términos, haga clic en **Crear**. 

    La implementación tardará unos minutos en completarse. 

    Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).


#### <a name="deploy-with-the-azure-cli"></a>Implementación con la CLI de Azure

Es necesario usar la CLI de Azure 2.6 o una versión posterior. Si está usando la CLI de Azure localmente, ejecute `az --version` para comprobar su versión.

Inicie sesión en la cuenta de Azure y seleccione su suscripción.

1. Si usa la CLI de Azure localmente en lugar de hacerlo en el portal, tendrá que iniciar sesión. Para iniciar sesión en el símbolo del sistema, ejecute el [comando de inicio de sesión](/cli/azure/get-started-with-az-cli2):
    
    ```azurecli
    az login
    ```

    Siga las instrucciones para realizar la autenticación mediante el código e inicie sesión en la cuenta de Azure a través de un explorador web.

2. Si tiene varias suscripciones de Azure, iniciar sesión en Azure le concede acceso a todas las cuentas de Azure asociadas con las credenciales. Use el siguiente [comando para mostrar las cuentas de Azure](/cli/azure/account) que tiene disponibles para su uso:
    
    ```azurecli
    az account list -o table
    ```

    Use el siguiente comando para seleccionar la suscripción que quiere usar para ejecutar los comandos con los que se crearán el centro de IoT y los recursos de DPS. Puede usar el nombre de la suscripción o el identificador de la salida del comando anterior:

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

3. Copie y pegue los siguientes comandos en el símbolo del sistema de la CLI. A continuación, presione **Entrar** para ejecutarlos.
   
    > [!TIP]
    > Los comandos solicitarán la ubicación del grupo de recursos. Para ver una lista de las ubicaciones disponibles, ejecute primero el comando:
    >
    > `az account list-locations -o table`
    >
    >
    
    ```azurecli-interactive
    read -p "Enter a project name that is used for generating resource names:" projectName &&
    read -p "Enter the location (i.e. centralus):" location &&
    templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.devices/iothub-device-provisioning/azuredeploy.json" &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
    echo "Press [ENTER] to continue ..." &&
    read
    ```

4. Los comandos le pedirán la siguiente información. Proporcione cada valor y presione **Entrar**.

    | Parámetro | Descripción |
    | :-------- | :---------- |
    | **Nombre del proyecto** | El valor de este parámetro se usará para crear un grupo de recursos que contenga todos los recursos. La cadena `rg` se agregará al final del valor del nombre del grupo de recursos. |
    | **ubicación** | Este valor es la región donde residirán todos los recursos. |
    | **iotHubName** | Escriba un nombre para el centro de IoT que sea globalmente único en el espacio de nombres *azure-devices.net*. Lo necesitará en la sección siguiente cuando valide la implementación. |
    | **provisioningServiceName** | Escriba un nombre para el nuevo recurso de Device Provisioning Service (DPS). El nombre debe ser globalmente único dentro del espacio de nombres *.azure-devices-provisioning.net*. Este nombre lo necesitará en la sección siguiente cuando valide la implementación. |

    La CLI de Azure se usa para implementar la plantilla. Además de la CLI de Azure, también puede usar Azure PowerShell, Azure Portal y la API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).


## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Para comprobar la implementación, ejecute el siguiente [comando para enumerar los recursos](/cli/azure/resource#az_resource_list) y busque el nuevo servicio de aprovisionamiento y el centro de IoT en la salida:

    ```azurecli
     az resource list -g "${projectName}rg"
    ```

2. Para comprobar que el centro ya está vinculado al recurso de DPS, ejecute el siguiente [comando para mostrar la extensión de DPS](/cli/azure/iot/dps#az_iot_dps_show).

    ```azurecli
     az iot dps show --name <Your provisioningServiceName>
    ```

    Observe los centros que están vinculados en el miembro `iotHubs`.


## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido de esta colección se basan en los valores de esta. Si tiene previsto seguir trabajando con las siguientes guías de inicio rápido o tutoriales, no elimine los recursos creados en esta guía de inicio rápido. Si no tiene previsto continuar, puede usar Azure Portal o la CLI de Azure para eliminar el grupo de recursos y todos sus recursos.

Para eliminar un grupo de recursos y todos sus recursos desde Azure Portal, simplemente abra el grupo de recursos y haga clic en **Eliminar grupo de recursos** en la parte superior.

Para eliminar el grupo de recursos implementado mediante la CLI de Azure:

```azurecli
az group delete --name "${projectName}rg"
```

También puede eliminar grupos de recursos y recursos individuales desde Azure Portal, PowerShell o las API REST, así como con los SDK de las plataformas compatibles publicados para Azure Resource Manager o IoT Hub Device Provisioning Service.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un centro de IoT y una instancia de Device Provisioning Service, y ha vinculado los dos recursos. Para saber cómo usar esta configuración para aprovisionar un dispositivo, vaya al inicio rápido en el que se explica cómo crear un dispositivo.

> [!div class="nextstepaction"]
> [Inicio rápido para aprovisionar un dispositivo](./quick-create-simulated-device-symm-key.md)
