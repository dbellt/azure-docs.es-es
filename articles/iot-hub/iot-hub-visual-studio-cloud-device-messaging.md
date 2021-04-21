---
title: Uso de VS Cloud Explorer para administrar la mensajería de dispositivos Azure IoT Hub
description: Aprenda a usar Cloud Explorer para Visual Studio para supervisar los mensajes del dispositivo a la nube y enviar mensajes de la nube al dispositivo en Azure IoT Hub.
author: shizn
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: xshi
ms.openlocfilehash: 8461a77d06a63c2ac319323a91b5577ca4dce1cf
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567037"
---
# <a name="use-cloud-explorer-for-visual-studio-to-send-and-receive-messages-between-your-device-and-iot-hub"></a>Uso de Cloud Explorer para Visual Studio para enviar y recibir mensajes entre el dispositivo e IoT Hub

![Diagrama integral](./media/iot-hub-visual-studio-cloud-device-messaging/e-to-e-diagram.png)

En este artículo, aprenderá a usar Cloud Explorer para Visual Studio para supervisar los mensajes del dispositivo a la nube y para enviar mensajes de la nube al dispositivo. Los mensajes del dispositivo a la nube podrían ser datos de sensor que el dispositivo recopila y luego envía a IoT Hub. Los mensajes de la nube al dispositivo podrían ser comandos que IoT Hub envía al dispositivo. Por ejemplo, hacer parpadear un LED que está conectado al dispositivo.

[Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS) es una extensión útil de Visual Studio que le permite ver los recursos de Azure, inspeccionar sus propiedades y realizar acciones de desarrollador clave desde dentro de Visual Studio. Este artículo se centra en cómo usar Cloud Explorer para enviar y recibir mensajes entre el dispositivo y su concentrador.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure activa.

- Una instancia de Azure IoT Hub en su suscripción.

- Microsoft Visual Studio 2017 Update 9 o posterior. En este artículo se utiliza [Visual Studio 2019](https://www.visualstudio.com/vs/).

- El componente Cloud Explorer desde el Instalador de Visual Studio, que se selecciona de forma predeterminada con la carga de trabajo de Azure.

## <a name="update-cloud-explorer-to-latest-version"></a>Actualización de Cloud Explorer a la versión más reciente

El componente Cloud Explorer desde el Instalador de Visual Studio 2017 solo admite la supervisión de mensajes del dispositivo a la nube y de la nube al dispositivo. Para usar Visual Studio 2017, descargue e instale la versión más reciente de [Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS).

## <a name="sign-in-to-access-your-hub"></a>Inicio de sesión para acceder al centro

Para acceder al concentrador, siga estos pasos:

1. En Visual Studio, seleccione **Ver** > **Cloud Explorer** para abrir Cloud Explorer.

1. Seleccione el icono de administración de cuentas para mostrar las suscripciones.

    ![Icono de administración de cuentas](media/iot-hub-visual-studio-cloud-device-messaging/account-management-icon.png)

1. Si ha iniciado sesión en Azure, aparecerán las cuentas. Para iniciar sesión en Azure por primera vez, elija **Agregar una cuenta**.

1. Seleccione las suscripciones de Azure que quiere usar y seleccione **Aplicar**.

1. Expanda la suscripción y luego **IoT Hubs**.  En cada centro, puede ver sus dispositivos.

    ![Lista de dispositivos](media/iot-hub-visual-studio-cloud-device-messaging/hub-device-list.png)

## <a name="monitor-device-to-cloud-messages"></a>Supervisión de mensajes de dispositivo a nube

Para supervisar los mensajes que se envían desde el dispositivo a IoT Hub, siga estos pasos:

1. Haga clic con el botón derecho en su instancia de IoT Hub o en su dispositivo y seleccione **Start Monitoring D2C Message** (Iniciar la supervisión del mensaje de D2C).

    ![Iniciar la supervisión del mensaje de D2C](media/iot-hub-visual-studio-cloud-device-messaging/start-monitoring-d2c-message-vs2019.png)

1. Los mensajes supervisados aparecen en **Salida**.

    ![Resultado de supervisión del mensaje de D2C](media/iot-hub-visual-studio-cloud-device-messaging/monitor-d2c-message-result-vs2019.png)

1. Para detener la supervisión, haga clic con el botón derecho en cualquier instancia de IoT Hub o en cualquier dispositivo y seleccione **Stop Monitoring D2C Message** (Detener la supervisión del mensaje de D2C).

## <a name="send-cloud-to-device-messages"></a>Envío de mensajes de nube a dispositivo

Para enviar un mensaje desde IoT Hub al dispositivo, siga estos pasos:

1. Haga clic con el botón derecho en el dispositivo y seleccione **Send C2D Message** (Enviar mensaje de C2D).

1. Escriba el mensaje en el cuadro de entrada.

    ![Enviar mensaje de C2D](media/iot-hub-visual-studio-cloud-device-messaging/send-c2d-message-test.png)

    Los resultados aparecen en **Salida**.

    ![Resultado de enviar mensaje de C2D](media/iot-hub-visual-studio-cloud-device-messaging/send-c2d-message-result-vs2019.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora sabe cómo supervisar los mensajes del dispositivo a la nube y enviar mensajes de la nube al dispositivo entre el dispositivo de IoT y la instancia de Azure IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]