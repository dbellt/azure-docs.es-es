---
title: Visualización de los datos de telemetría de inferencia del modelo de Azure Percept DK
description: Aprenda a ver los datos de telemetría de inferencia del modelo de visión de Azure Percept DK en Azure IoT Explorer
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/17/2021
ms.custom: template-how-to
ms.openlocfilehash: 3e09646614d37a97400a7e90b380048d97c0369c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754224"
---
# <a name="view-your-azure-percept-dks-model-inference-telemetry"></a>Visualización de los datos de telemetría de inferencia del modelo de Azure Percept DK

Siga esta guía para ver los datos de telemetría de inferencia del modelo de visión de Azure Percept DK en [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases).

## <a name="prerequisites"></a>Requisitos previos

- Kit de desarrollo Azure Percept DK
- [Suscripción de Azure](https://azure.microsoft.com/free/)
- [Experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md): ya ha conectado el kit de desarrollo a una red Wi-Fi, creado una instancia de IoT Hub y conectado el kit a esta instancia.
- [El modelo de IA de visión se ha implementado en Azure Percept DK](./how-to-deploy-model.md)

## <a name="view-telemetry"></a>Ver datos de telemetría

1. Conecte el kit de desarrollo.

1. Descargue e instale [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases). Si es un usuario de Windows, seleccione el archivo .msi.

    :::image type="content" source="./media/how-to-view-telemetry/azure-iot-explorer-download.png" alt-text="Pantalla de descarga de Azure IoT Explorer.":::

1. Conecte IoT Hub a Azure IoT Explorer:

    1. Vaya a [Azure Portal](https://portal.azure.com).

    1. Seleccione **Todos los recursos**.

        :::image type="content" source="./media/how-to-view-telemetry/azure-portal.png" alt-text="Página principal de Azure Portal.":::

    1. Seleccione la instancia de IoT Hub al que está conectado Azure Percept DK.

        :::image type="content" source="./media/how-to-view-telemetry/iot-hub.png" alt-text="Lista de IoT Hub en Azure Portal.":::

    1. En la parte izquierda de la página de IoT Hub, seleccione **Directivas de acceso compartido**.

        :::image type="content" source="./media/how-to-view-telemetry/shared-access-policies.png" alt-text="Página de IoT Hub que muestra directivas de acceso compartido.":::

    1. Haga clic en **iothubowner**.

        :::image type="content" source="./media/how-to-view-telemetry/iothubowner.png" alt-text="Pantalla de directivas de acceso compartido con iothubowner resaltado.":::

    1. Haga clic en el icono de copia azul situado junto a **Cadena de conexión (clave principal)** .

        :::image type="content" source="./media/how-to-view-telemetry/connection-string.png" alt-text="Ventana de iothubowner con el botón de copia de cadena de conexión resaltado.":::

    1. Abra Azure IoT Explorer y haga clic en **+ Agregar conexión**.

    1. Pegue la cadena de conexión en el cuadro **Cadena de conexión** de la ventana **Agregar cadena de conexión** y haga clic en **Guardar**.

        :::image type="content" source="./media/how-to-view-telemetry/add-connection-string.png" alt-text="Ventana de Azure IoT Explorer con cuadro para pegar la cadena de conexión.":::

    1. Señale el módulo de sistema de visión en un objeto para la inferencia del modelo.

    1. Seleccione **Telemetría**.

    1. Haga clic en **Iniciar** para ver los eventos de telemetría desde el dispositivo.

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo ver la [secuencia de vídeo de Azure Percept DK](./how-to-view-video-stream.md).