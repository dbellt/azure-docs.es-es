---
title: Creación de una instancia de Azure IoT Hub mediante Azure IoT Tools para VS Code | Microsoft Docs
description: En este artículo se muestra cómo usar Azure IoT Tools para Visual Studio Code para crear una instancia de Azure IoT Hub en un grupo de recursos.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/04/2019
ms.author: junhan
ms.openlocfilehash: e8d6bebffc46b0b158aeb975ae1e32a863b956d2
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655883"
---
# <a name="create-an-iot-hub-using-the-azure-iot-tools-for-visual-studio-code"></a>Creación de una instancia de IoT Hub mediante Azure IoT Tools para Visual Studio Code

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Este artículo muestra cómo usar [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) para crear una instancia de Azure IoT Hub. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este artículo, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- [Visual Studio Code](https://code.visualstudio.com/)

- [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) instalado para Visual Studio Code.


## <a name="create-an-iot-hub-and-device-in-an-iot-project"></a>Creación de un centro de IoT y un dispositivo en un proyecto de IoT

En los pasos siguientes se muestra cómo crear una instancia de IoT Hub y registrar un dispositivo en el centro de un proyecto de IoT en Visual Studio Code.

En lugar de aprovisionar una instancia de Azure IoT Hub y el dispositivo desde Azure Portal, puede hacerlo en VS Code sin tener que salir del entorno de desarrollo. Los pasos de esta sección le muestran cómo hacerlo.

1. En la nueva ventana de proyecto que se abre, haga clic en `F1` para abrir la paleta de comandos y escriba y seleccione **Azure IoT Device Workbench: Provision Azure Services...** (aprovisionar servicios de Azure). Siga la guía paso a paso para finalizar el aprovisionamiento de Azure IoT Hub y crear el dispositivo de IoT Hub.

    ![Comando de aprovisionamiento](media/iot-hub-create-use-iot-toolkit/provision.png)

    > [!NOTE]
    > Si no ha iniciado sesión en Azure, siga la notificación emergente para iniciar sesión.

1. Seleccione la suscripción que quiere usar.

    ![Seleccionar suscripción](media/iot-hub-create-use-iot-toolkit/select-subscription.png)

1. A continuación, seleccione un grupo de recursos existente o cree [uno nuevo](../azure-resource-manager/management/overview.md#terminology).

    ![Selección del grupo de recursos](media/iot-hub-create-use-iot-toolkit/select-resource-group.png)

1. En el grupo de recursos que ha especificado, siga las instrucciones para seleccionar una instancia de IoT Hub existente o crear una nueva.

    ![Pasos de selección de IoT Hub](media/iot-hub-create-use-iot-toolkit/iot-hub-provision.png)

    ![Seleccionar IoT Hub](media/iot-hub-create-use-iot-toolkit/select-iot-hub.png)

    ![IoT Hub seleccionado](media/iot-hub-create-use-iot-toolkit/iot-hub-selected.png)

1. En la ventana de salida, verá el centro de Azure IoT Hub que ha aprovisionado.

    ![IoT Hub aprovisionado](media/iot-hub-create-use-iot-toolkit/iot-hub-provisioned.png)

1. Seleccione o cree un dispositivo de IoT Hub en la instancia de Azure IoT Hub que ha aprovisionado.

    ![Pasos para seleccionar el dispositivo de IoT](media/iot-hub-create-use-iot-toolkit/iot-device-provision.png)

    ![Seleccionar el dispositivo de IoT aprovisionado](media/iot-hub-create-use-iot-toolkit/select-iot-device.png)

1. Ya ha aprovisionado el centro de Azure IoT Hub y ha creado un dispositivo en él. Además, la cadena de conexión del dispositivo se guardará en VS Code.

    ![Aprovisionamiento realizado](media/iot-hub-create-use-iot-toolkit/provision-done.png)



## <a name="create-an-iot-hub-without-an-iot-project"></a>Creación de un centro de IoT sin un proyecto de IoT

En los pasos siguientes se muestra cómo puede crear una instancia de IoT Hub sin un proyecto de IoT en Visual Studio Code.

1. En Visual Studio Code, abra la vista **Explorer** (Explorador).

2. En la parte inferior de Explorer, expanda la sección **Azure IoT Hub**. 

   ![Expandir los dispositivos de Azure IoT Hub](./media/iot-hub-create-use-iot-toolkit/azure-iot-hub-devices.png)

3. Haga clic en los puntos suspensivos **...** en el encabezado de la sección **Azure IoT Hub**. Si no ve el botón de los puntos suspensivos, mantenga el mouse sobre el encabezado. 

4. Seleccione **Crear IoT Hub**.

5. Se mostrará una ventana emergente en la esquina inferior derecha que le permite iniciar sesión en Azure por primera vez.

6. Seleccione una suscripción de Azure. 

7. Seleccione un grupo de recursos.

8. Seleccione una ubicación.

9. Seleccione un plan de tarifa.

10. Escriba un nombre único global para la instancia de IoT Hub.

11. Espere unos minutos hasta que se cree la instancia de IoT Hub.

## <a name="next-steps"></a>Pasos siguientes

Ya ha implementado una instancia de IoT Hub mediante Azure IoT Tools para Visual Studio Code. Para más información, consulte los artículos siguientes:

* [Uso de Azure IoT Tools para Visual Studio Code a fin de enviar y recibir mensajes entre el dispositivo e IoT Hub](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

* [Uso de Azure IoT Tools para Visual Studio Code para la administración de dispositivos de Azure IoT Hub](iot-hub-device-management-iot-toolkit.md)

* Consulte la [página wiki de Azure IoT Hub para VS Code](https://github.com/microsoft/vscode-azure-iot-toolkit/wiki).
